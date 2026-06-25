# CodeGraph — 代码知识图谱 MCP 完全指南

> **一句话总结**：提前用 tree-sitter 把你的代码解析成知识图谱，Agent 不再需要 grep/read 扫文件，直接查图即可。**平均 57% 更少 token，62% 更少工具调用**。

## 是什么

[CodeGraph](https://github.com/colbymchenry/codegraph) 是一个 **100% 本地运行** 的代码知识图谱 MCP 服务器，为 AI 编码助手提供预索引的代码语义理解能力。

- **35K+ GitHub Stars**，MIT 协议，2026 年增长最快的开发者工具之一
- **20+ 语言支持**：Go、TypeScript、Python、Rust、Java、C/C++、Swift、Kotlin 等
- **8 种 AI Agent 支持**：OpenCode、Claude Code、Cursor、Codex、Gemini CLI 等

### 解决什么问题

当 AI Agent 需要理解你的代码库时，传统方式是：**启动 Explore 代理 → grep 搜索 → glob 找文件 → Read 读文件 → 再 grep → 再 read**……每步都消耗 token 和 API 费用。

CodeGraph 的思路是：**提前把代码解析成交互式知识图谱，Agent 一次 MCP 调用就能拿到需要的上下文**。

```
传统方式（无 CodeGraph）：
  用户问 "handler 怎么调用 ASR 的？"
  → Agent: grep "ASR" → 找到 15 个匹配
  → Agent: Read file1.go → Read file2.go → Read file3.go
  → Agent: grep "handler" → 又找到 8 个匹配
  → 最终回答（消耗 ~800K tokens，~15 次工具调用）

CodeGraph 方式：
  用户问 "handler 怎么调用 ASR 的？"
  → Agent: codegraph_trace("handleFragment", "CallASR")
  → 一次调用拿到完整调用链 + 源码
  → 直接回答（消耗 ~500K tokens，~5 次工具调用）
```

## 工作原理

```
┌────────────────────────────────────────────┐
│              AI Agent (OpenCode 等)          │
│   codegraph_trace · context · explore ...    │
└──────────────────┬─────────────────────────┘
                   ▼
┌────────────────────────────────────────────┐
│          CodeGraph MCP Server               │
│  10 个 MCP 工具，按意图自动选择              │
└──────────────────┬─────────────────────────┘
                   ▼
┌────────────────────────────────────────────┐
│       SQLite 知识图谱 (.codegraph/)          │
│  · symbols (函数/类/方法/接口)               │
│  · edges (调用/导入/继承/实现)               │
│  · FTS5 全文搜索                             │
│  · 文件监听自动增量同步                       │
└────────────────────────────────────────────┘
```

**四个阶段**：

1. **解析（Extraction）** — tree-sitter 解析源码为 AST，语言特定查询提取节点和边
2. **存储（Storage）** — 全部存入 SQLite + FTS5 全文索引，100% 本地
3. **解析（Resolution）** — 解析调用关系、导入链、继承树、框架路由
4. **同步（Auto-Sync）** — OS 原生文件监听（FSEvents/inotify），变更 2 秒后自动增量同步

## 安装

### 一键安装（推荐）

```bash
# macOS / Linux（自带 Node 运行时，无需单独装 Node）
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh

# Windows (PowerShell)
irm https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.ps1 | iex
```

### npm 安装（已有 Node）

```bash
npx @colbymchenry/codegraph     # 零安装运行
npm i -g @colbymchenry/codegraph # 全局安装
```

安装器会**自动检测**已安装的 AI Agent（OpenCode、Claude Code 等），配置 MCP 服务器连接。无需手动编辑任何配置文件。

### 为项目建索引

```bash
cd /path/to/your-project
codegraph init -i   # -i 表示同时建索引
```

- Go 项目约 **2-5 秒**
- TypeScript/大型项目约 **10-30 秒**
- 数据库大小通常 **5-20 MB**
- 后续自动增量同步，无需重复建索引

验证索引状态：

```bash
codegraph status    # 查看节点数、边数、文件数、数据库大小
```

## OpenCode 配置

### MCP 配置（opencode.json）

```json
"codegraph": {
  "type": "local",
  "command": ["codegraph", "serve", "--mcp"],
  "enabled": true
}
```

### 自动化规则（AGENTS.md）

添加以下规则到 `~/.config/opencode/AGENTS.md`，Agent 会在每次会话启动时自动检查：

> 会话启动时，检查 `.codegraph/` 是否存在：
>
> - **不存在** → 自动执行 `codegraph init -i`（首次建索引）
> - **已存在** → 无需操作（MCP 连接时自动增量同步）
>
> 切 Git 分支后无需手动 `codegraph sync`，文件监听器自动检测变更。

这意味着：

- **新项目**：Agent 自动 init
- **老项目有变更**：MCP 启动时自动增量同步
- **切换分支**：文件监听器 2 秒内感知变更
- **完全无需人工干预**

## 十大 MCP 工具

| 工具 | 用途 | 一句话说明 |
|-|-|-|
| `codegraph_search` | 按名称搜符号 | "项目中哪里定义了 handleFragment？" |
| `codegraph_context` | 构建任务上下文 | "告诉我视频流水线涉及哪些模块" |
| `codegraph_trace` | 追踪调用路径 | "Pre handler 怎么调用到 ASR 的？" |
| `codegraph_callers` | 查谁调用了某函数 | "谁在调用 saveParseResult？" |
| `codegraph_callees` | 查某函数调用了谁 | "handleFragment 内部调了哪些函数？" |
| `codegraph_impact` | 改前影响分析 | "改 status 字段会影响哪些模块和接口？" |
| `codegraph_explore` | 批量获取符号源码 | "展示 Pre/ASR/NLP 三个 handler 的完整实现" |
| `codegraph_node` | 单个符号详情 | "GetFragmentList 的完整签名和源码" |
| `codegraph_files` | 索引文件结构 | "项目的目录结构是怎样的？" |
| `codegraph_status` | 索引健康检查 | "索引是最新的吗？有没有待同步的文件？" |

> **核心原则**：Agent 应**直接使用 CodeGraph 工具**回答代码结构问题，而不是先查 CodeGraph 再自己 grep 验证。信任索引结果可以最大化 token 节省。

## 实测效果

### 官方 Benchmark（Claude Opus 4.8）

| 代码库 | 语言 | 成本节省 | Token 节省 | 工具调用减少 |
|-|-|-|-|-|
| VS Code (\~10K 文件) | TypeScript | **33%** | **70%** | **80%** |
| Django (\~3K 文件) | Python | **23%** | **70%** | **77%** |
| Tokio (\~790 文件) | Rust | **35%** | **70%** | **79%** |
| OkHttp (\~645 文件) | Java | **11%** | **48%** | **70%** |
| Gin (\~110 文件) | **Go** | **15%** | **35%** | **47%** |

**平均：25% 更便宜 · 57% 更少 token · 62% 更少工具调用**

### 我的实际项目

| 项目 | 文件 | 节点 | 边 | DB | 预计节省 |
|-|-|-|-|-|-|
| （视频会议纪要） | 205 | 3,758 | 9,484 | 7.3 MB | \~39% token |
| 项目B（企业知识库） | 291 | 7,046 | 24,203 | 14.8 MB | \~46% token |

项目B 有 **97 个分支、15 个客户交付分支**，不同分支同一文件有完全不同逻辑。每次切分支 Agent 不再需要重新探索代码结构，**节省 \~65% token**。

## 多分支项目最佳实践

对于多客户交付项目（如项目B），不同分支有不同业务逻辑：

```
main 分支：标准版（291 文件）
delivery/cntaiping：增强版（+Bot 系统、+自动评估、+部门权限）
delivery/unicom：精简版（-复杂模块、+本地认证、+OAuth）
```

CodeGraph 的自动同步机制在这种场景下特别有价值：

1. **切分支** → 文件监听器检测到大量文件变更
2. **2 秒去抖** → 变更文件被增量重新索引
3. **Agent 就绪** → 可以直接查新分支的调用链

不需要为每个分支单独维护索引，`codegraph sync` 只处理变更文件。

## 框架路由识别

CodeGraph 自动识别 **14 个框架** 的路由定义，将 URL 模式链接到处理器：

| 框架 | 识别模式 |
|-|-|
| **Gin / chi / gorilla / mux** | `r.GET("/path", handler)` |
| Django / Flask / FastAPI | `@app.route(...)` / `path(...)` |
| Express / NestJS | `app.get(...)` / `@Get(...)` |
| Spring / ASP.NET | `@GetMapping` / `[HttpGet]` |
| Laravel / Rails | `Route::get(...)` / `get '...', to:` |

在我 的  项目中，CodeGraph 自动识别出了 **64 个 Gin 路由** 及其对应的 handler 函数。

## Go 项目特殊说明

CodeGraph 对 Go 的支持是 Full Support，但有几点需要注意：

- ✅ **函数/方法调用链**：完整支持
- ✅ **结构体/接口定义**：`struct` 和 `interface` 均被索引
- ✅ **Gin 路由**：`r.GET/POST/PUT/DELETE` 自动识别
- ✅ **导入链**：跨包引用被正确追踪
- ⚠️ **隐式接口满足**：Go 的 `interface` 隐式实现关系可能不完整（known bug）
- ⚠️ **泛型跨文件**：泛型方法在跨文件声明时调用边可能丢失（known bug）

> 这两个问题对日常使用影响不大——`codegraph_trace` 和 `codegraph_context` 的核心价值不依赖接口满足关系。

## 注意事项

1. **小项目效果不明显**：<50 文件的代码库，原生 grep 可能更便宜
2. **不要重复验证**：Agent 查完 CodeGraph 不要再自己 grep 验证一遍
3. **文件过大跳过**：>1MB 的文件会被自动排除
4. **gitignore 自动生效**：被 `.gitignore` 排除的文件不会被索引
5. **切分支后稍等 2 秒**：文件监听器有 2 秒去抖，之后索引会自动更新
6. **Stale 文件有提醒**：如果 Agent 修改了代码但索引还没更新，MCP 会返回 ⚠️ 标记

## 卸载

```bash
codegraph uninstall   # 从所有 Agent 中移除 MCP 配置
codegraph uninit      # 删除当前项目的 .codegraph/ 索引
```

卸载干净，不留痕迹。

## 总结

| 场景 | 建议 |
|-|-|
| 新 Go 项目 | `codegraph init -i`，5 秒搞定 |
| 中型项目（100-500 文件） | 强烈推荐，预计节省 35-50% token |
| 大型项目（500+ 文件） | 必须接入，预计节省 50-70% token |
| 多分支/多客户交付 | CodeGraph 最佳场景，每次切分支省 65% token |
| 微型项目（<50 文件） | 可有可无，原生 grep 已经够快 |

**一句话**：CodeGraph 是目前同类工具中最成熟（35K stars）、最易用（零配置）、与 OpenCode 兼容性最好的代码知识图谱工具。接入成本 5 分钟，不满意随时一条命令卸载。

---

> 📖 官方文档：[colbymchenry.github.io/codegraph](https://colbymchenry.github.io/codegraph/)
> 🐙 GitHub：[github.com/colbymchenry/codegraph](https://github.com/colbymchenry/codegraph)
