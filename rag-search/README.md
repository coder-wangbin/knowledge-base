# SearchRAG

# Search 知识库优先 RAG 调用链详解

## 1. 概述

Search API 的默认策略是“知识库优先”的 RAG（Retrieval-Augmented Generation）流程：

1. **构建检索范围**：依据会话、会话 Scope、临时上传文件/URL，确定本轮可访问的文档与知识库集合；
2. **生成检索请求**：封装 `doc_chat_model.ChatReq` 交给知识库检索服务（KS），内含用户、公司、范围、历史对话、FAQ/Resample 等上下文；
3. **流式检索+生成**：KS 返回包含命中的段落、引用结构、相关问题的流式 `ChatV1Resp`，服务端一边消费、一边做敏感词校验与 SSE 推送；
4. **持久化与索引**：对最终答案、引用和上下文元数据进行数据库、ES 双写，供会话回放与检索历史查询。

下文按“请求准备 → 检索执行 → 数据结构 → 错误与兜底 → 落库”逐段拆解。

## 2. 请求准备阶段

### 2.1 会话校验与上下文恢复

- `getReq`

  - 调用 `docModel.SqaGetConversation`、`SqaGetScope` 确保会话仍存在；
  - 拉取历史消息 `SqaGetMessages`，剔除未完成/当前 messageID，再解析 JSON 文本成为 `doc_chat_model.ChatTurn` 列表，供 KS 做多轮对话检索；
  - 若历史回答包含“**为您请求大模型结果如下：**”等提示，先截断，避免提示串入查询。

### 2.2 Scope 计算与展示

- `getScope`

  - 根据本次 `req.DocIDs` 查询 `docModel.SqaGetDocs`，构建前端所需 `[]entity.SearchScope`（文档标题、类型、URL 等）；
  - 结果写入 `g.scope`，既用于 SSE 返回，也用于后续 answer 里描述引用范围。

### 2.3 Chat 请求拼装

- `getChatReq`

  - 汇总 docIDs/libraryIDs：优先会话绑定文档，其次用户指定 docIDs/目录/隐藏库；若范围为空则置 `[-1]`，兼容 KS API；
  - 过滤已下架文档（`doc.Status!=available`）或空库，必要时直接返回“未找到文档”兜底；
  - 构造 `doc_chat_model.ChatReq`：

    - `Query`、`Biz`、`LlmType` 由会话模式与请求参数综合得出；
    - `DialogState` 携带历史轮次；
    - `Search` 字段含 `DocIds`、`LibraryIds`、`DirectoryIds`、`User/Company` 身份、`IsInSingleDoc` 标记；
    - `DoSample`（是否重采样）、`UseFaq`、`ShowRationale` 等控制生成行为；
    - 在 Dev/Test 环境尊重 `x-skb-chat-debug` 头以开启调试。

### 2.4 执行步骤排序

- `assignChatStep`

  - 默认 sequence：`[知识库]`；
  - 若 `req.Model` 要求 Model/Web 模式，则在末尾拼接 `BizModeChat`、`BizModeWebQA`；
  - SSE 输出需据此决定是否拼接“为您请求大模型结果”“为您请求互联网结果”提示。

## 3. 知识库检索执行（callSKBChat）

### 3.1 调度与兜底

1. `tryDefaultAnswer`：命中 `defaultanswer` 中的关键词对，则直接 SSE 返回，跳过 KS；
2. 若未命中，记录 `g.curBizMode=BizModeStandard`，序列化 `chatReq` 供日志排查后调用 `ksModel.ChatStream`；
3. `stream.Recv` 循环处理：

   - EOF 且 `msgInfo.Answer` 为空时回落到 `defaultAnswer`；
   - `resp.Result.IsAnswerable` 为 `false` 时，返回 `isNext=true` 以便后续 LLM/Web 继续补答；
   - 每帧调用 `handleStreamResp` 解析数据并 SSE 下发。

### 3.2 流响应解析

- `handleStreamResp`

  - 解析 `resp.Code`，非零默认 `defaultAnswer`；
  - `toStreamMessage` 负责拆分 `result` 字段：

    - `getReasoning` 分段提取模型思考，累计 50 字检查一次敏感词，最终输出 `Reasoning`；
    - `getAnswer` 拼接段落/句子，同时为引用加上 `[sourceIndex-subIndex]` 标记；
    - 引用处理：

      - 根据 `resp.Result.Ref` 构造 `sourcesMap`，聚合同一 docID 的命中片段、坐标、highlight；
      - `addSourceExt` 补充对应文档的库信息、URL、类型；
    - 相关问题 & FAQ：读取 `resp.Result.RelatedQuestions` 与 `Extra.faq`，分别做敏感词过滤、来源库名拼接，并在答案里插入 `[qa-x]` 索引；
  - 依据 `g.curBizMode` 决定是否拼接 `modelAnswer`/`webAnswer` 前缀；
  - 生成 `entity.Response{Data: entity.Message}`，调用 `handler.SendSSEResponse` 发送事件 ID（字符串 `eventID`）。

### 3.3 SSE 与 Stop 控制

- 每个流片段使用累加 ID，前端可据此顺序渲染；
- 若 `resp.Code!=0` 或 `shouldStop=true`（如敏感词触发）则立即跳出循环；
- `StopSearch` API 将 messageID 写入 Redis，`generateHandler.checkStop` 在 `defer` 中读取并决定是否跳过 `saveMessage2DB`/`saveEs`，同时清除键。

## 4. 关键数据结构关系

| 数据结构 | 主要字段 | 用途 |
|-|-|-|
| `entity.SearchReq` | `ConversationID`, `MessageID`, `Query`, `DocIDs`, `DirIDs`, `WebQA`, `UseFaq`, `ShowRationale` 等 | HTTP 请求体，决定查询范围与模式 |
| `doc_chat_model.ChatReq` | `Query`, `Biz`, `LlmType`, `DialogState`, `Search{DocIds, LibraryIds, DirectoryIds, User, Company}`, `DoSample`, `UseFaq` | 发送给 KS 的统一检索/生成参数 |
| `doc_chat_model.ChatV1Resp` | `Code`, `Message`, `Result{AnsParas, Ref, RelatedQuestions, Extra, IsAnswerable, RewrittenQuery, Reasoning, Rationale}` | KS 的流式返回，包含答案、引用、推荐问题等 |
| `entity.Message` | `ConversationID`, `MessageID`, `Query`, `Answer`, `Sources`, `RelatedQueries`, `QaSources`, `Scope`, `Reasoning`, `Rationale` | SSE 下行与 DB/ES 持久化的统一结构 |
| `entity.SearchSource` | `DocID`, `Title`, `Positions`, `HighlightBlocks`, `LibraryID/Name`, `DocumentType`, `Url`, `PubType` | 前端引用材料展示 |

## 5. 错误处理与兜底

- **敏感词**：输入 Query、答案、Reasoning、相关问题分别通过 `CheckAndSendModeration`/`CheckModeration` 审核，命中即返回默认提示并设置 `shouldStop=true`；
- **空范围**：`getChatReq` 在所有文档被删除时直接返回 `ErrCodeNotFound`，提示“搜索范围中未找到该问题的答案”；
- **KS 失败**：`stream.Recv` 报错或 `resp.Code!=0` 时 SSE 返回 `defaultAnswer`，并终止后续步骤；
- **不可回答**：`result.IsAnswerable=false` 会将 `isNext` 置 `true`，允许 LLM/Web 继续补答；若最终仍无内容，则走 `defaultAnswer`。

## 6. 落库与索引

- `saveMessage2DB`

  - 若未命中 stop 标记，将 `entity.Message` 序列化为 JSON 存入 `model.SqaMessage`；
  - 补写 `RewrittenQuery`、`ScopeType`、`SingleDoc`、`UseFaq`、`ShowRationale` 等字段；
  - 若当前会话尚无标题，基于 `entity.GetConversationTitle(query)` 自动生成后 `SqaSaveConversation`。
- `updateConv`

  - 统计搜索次数、更新时间，并保留答案前 255 字作为列表摘要；
  - stop 情况仅删除 Redis 标记，避免覆盖历史数据。
- `saveEs`

  - 构造 `entity.MessageInfo`，通过 `es.SqaAddHistory` 写入 ES，支持 `/sqa/history-search` 与审计；
- `handler.SendSSEResp(...SseIDClose...)`

  - 所有步骤完成后发出关闭事件，通知前端收流。

## 7. 时序摘要（文本示意）

```Plaintext
客户端 → /sqa/search
  svcImpl.Search
    ↳ generateHandler.run
        ↳ getReq → getScope → getChatReq → assignChatStep
        ↳ callSKBChat
            ↳ ks.ChatStream (循环 Recv)
                ↳ handleStreamResp → SSE event (含 Answer/Sources)
        ↳ defer: checkStop → saveMessage2DB → updateConv → saveEs → SSE CLOSE
```

## 8. 扩展建议

1. **检索命中埋点**：记录 doc/library 命中率、FAQ 命中率，便于调参与优化索引；
2. **引用可视化**：将 `HighlightBlocks` 与 PDF 页码在前端做热区联动，提升 RAG 解释性；
3. **流式重试**：对单帧 `resp.Code!=0` 的情况尝试容错或自动切换深度模式，减少纯兜底场景。

---

# SearchApi

# Search API 设计说明

## 1. 背景与目标

Search API（`pkg/handler/sqa/search.go` 中的 `Search` 方法）为知识检索问答场景提供流式 SSE 接口 `/sqa/search`，核心目标是：

- 在用户指定的知识库/文档范围内召回答案，必要时联动大模型或 Web 检索补充回答；
- 通过敏感词、并发、停止等控制保证服务安全可控；
- 将对话上下文、答案与引用材料可靠落盘，支撑后续的会话列表、历史搜索与审计。

## 2. 入口流程一览

1. **参数校验**：`checkSearchParam` 绑定 `entity.SearchReq` 并执行 `Valid`；失败时立即通过 SSE 返回错误事件。
2. **输入安全**：调用 `handler.CheckAndSendModeration` 对 query 做敏感词审核，命中直接返回提示。
3. **并发限制**：原子计数 `_chatTaskNum` 与上限 `chatTaskMaxNum=15` 控制整体 Chat 并发，总线级互斥锁 `mu` 确保单请求串行化关键段。
4. **生成器封装**：构造 `generateHandler`，包含请求上下文、仓库实例、消息缓存等，随后执行 `run()` 构建并触发后续调用链。

## 3. 依赖组件

| 组件 | 作用 | 主要调用点 |
|-|-|-|
| `doc.Service` | 会话/消息/文档元信息查询与持久化 | `getReq`, `getScope`, `saveMessage2DB`, `updateConv` |
| `library.Service` | 知识库元信息补充 | `getChatReq`, `addSourceExt` |
| `ks.Service` | Chat/GPT 召回流式接口 | `callSKBChat`, `callLLMChat`, `callWebChat` |
| `es.Service` | 历史搜索索引 | `saveEs` |
| `qa.Service` | 问答库命中与来源补充 | `toStreamMessage` |
| `user.Service` | QA 来源编辑人信息 | `toStreamMessage` |
| `redis.Service` | Stop 控制、会话并发锁 | `StopSearch`, `checkStop` |
| `defaultanswer` / `profile` | 兜底答案、企业 FAQ | `tryDefaultAnswer`, `tryProfileAnswer` |
| `handler` 中 SSE/审核工具 | SSE 推送、敏感词检查 | 全流程 |

## 4. 主调用链

### 4.1 对话准备

- **`getReq`**：

  - 校验会话与 Scope 存在性（`docModel.SqaGetConversation`/`SqaGetScope`）。
  - 加载历史消息转为 `ChatDialogState`，剔除未完成记录与模型提示段，支撑多轮对话。
- **`getScope`**：根据本次 docIDs 查询文档，生成前端展示所需的 `entity.SearchScope` 列表。

### 4.2 搜索范围与 Chat 请求

- **`getChatReq`**：

  - 汇总 doc/library/目录范围，过滤已删除文档；若范围为空则注入 `-1` 防止空 slice。
  - 构造 `doc_chat_model.ChatReq`，带入对话上下文、用户/企业信息、范围、模式（`Biz`/`LlmType`）、FAQ/Resample 配置；在 Dev/Test 环境追加调试头。
- **`assignChatStep`**：

  - 根据 `req.Model`、`req.WebQA` 和会话模式确定执行顺序（如 `skbqa -> llmchat -> web_qa`），用于串行控制不同回答形态。

### 4.3 执行阶段

- **`run`** 主流程：依次调用步骤，遇到错误或 `isNext=false` 时提前结束；

  1. `callSKBChat`：知识库问答（默认），若 `tryDefaultAnswer` 命中则直接返回；否则调用 `ks.ChatStream` 并处理流。
  2. `callLLMChat`：模型问答；在执行前尝试 `tryProfileAnswer`，避免重复生成。
  3. `callWebChat`：Web 检索；先向前端推送“互联网结果”提示，然后串流返回。
- **流处理 `handleStreamResp`**：统一将模型响应转换为 `entity.Message`，包含：

  - `toStreamMessage` 解包答案段落、引用材料、相关问题、Reasoning/Rationale；对答复与推荐问题再次做敏感词审核；
  - 追加 `modelAnswer`/`webAnswer` 前缀，拼接首段回答与思考过程；
  - 通过 `handler.SendSSEResponse` 发送事件编号（整型字符串）和数据。
- **SSE 结束**：流程完成或发生错误后发送 `SseIDClose` 事件，前端关闭流。

## 5. SSE 协议约定

- **事件 ID**：从 0 开始递增的字符串，便于前端区分片段；错误使用 `handler.SseIDError`，结束使用 `handler.SseIDClose`。
- **数据封装**：`entity.Response`，`Data` 字段承载 `entity.Message`；`RequestID` 由 `requestid.Get` 注入。
- **多阶段拼接**：

  - 知识库回答输出首段；若后续触发大模型/互联网模式，会在答案前自动拼接提示段落，保持上下文一致。
  - Reasoning 模块以 50 字增量复核敏感词，确保长思考过程可控。

## 6. 风控与限流

- **敏感词**：输入、答案、Reasoning、推荐问题分别调用 `CheckAndSendModeration`/`CheckModeration`；命中时立即返回默认提示并设置 `shouldStop=true`。
- **并发限制**：

  - `_chatTaskNum` + `chatTaskMaxNum` 控制全局任务数量；
  - `sync.Mutex` `mu` 确保同一时间只有一个生成任务在临界区内执行，避免共享状态竞态。
- **停止机制**：`StopSearch` 将 messageID 写入 Redis（15s TTL）；`generateHandler.checkStop` 在 `defer` 中读取并决定是否跳过持久化，同时删除键。

## 7. 数据持久化与索引

- **消息记录 `saveMessage2DB`**：

  - 序列化 `entity.Message` 为 JSON 存储在 `model.SqaMessage`；补齐 `RewrittenQuery`、`ScopeType`、`SingleDoc` 等字段；
  - 若会话标题为空，用 `entity.GetConversationTitle` 自动生成。
- **会话状态 `updateConv`**：

  - 更新摘要内容、搜索次数、更新时间；在 `stopSave=true` 时仅清理 Redis stop 标记。
- **ES 历史 `saveEs`**：

  - 通过 `es.SqaAddHistory` 写入 `entity.MessageInfo`，支撑搜索历史查询。

## 8. 兜底策略

- **默认问答库**：`defaultanswer` 根据关键词命中企业级静态问答。
- **Profile 搜索**：调用 `profile.Search` 返回企业 Profile 配置答案。
- **FAQ 来源**：当模型 Extra 标记命中 FAQ 时，补充问答库来源列表并在答案中追加 `[qa-x]` 标记。
- **无答案场景**：若模型在流末仍未输出文本，返回通用提示 `defaultAnswer`。

## 9. 典型调用链（知识库优先）

1. `/sqa/search` → `svcImpl.Search`
2. `generateHandler.run`
3. `getReq` → `getScope` → `getChatReq` → `assignChatStep`
4. `callSKBChat`（内部多次 `handleStreamResp`）
5. `saveMessage2DB` → `updateConv` → `saveEs`
6. SSE `SseIDClose` 结束

## 10. 扩展与演进建议

- **监控**：补充 `_chatTaskNum`、stop 命中率、模块命中率指标，便于容量规划。
- **链路可视化**：将 `chatStep` 与每段时延上报到 tracing，帮助分析多模式串行的耗时。
- **配置化风控**：将敏感词策略、兜底答案映射迁移至配置中心，减少代码变更频率。
