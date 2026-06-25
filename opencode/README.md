# Opencode学习


opencode：https://github.com/anomalyco/opencode

oh-my-openagent：https://github.com/code-yeongyu/oh-my-openagent

opencode  是基础的命令行 Agent 工具环境，而 oh-my-opencode

 是基于它的一套**超级配置包/插件包**（类似于 zsh  和 oh-my-zsh  的关系）。

**区别在于：**

1. **开箱即用**:

opencode  需要你自己配置 Agent、Tool 和 Prompt；oh-my-opencode

 内置了一整套经过调优的专家 Agent（如 Sisyphus, Oracle, Librarian）。

1. **更强的能力**: 它引入了

Ultrawork  (自动工作) 和 Prometheus  (规划师) 模式，大大增强了复杂任务的处理能力。

1. **生态集成**: 比如

opencode-antigravity-auth

 这种插件，能让你直接使用 Google 内部的 Gemini 模型，或者通过 Copilot 使用 GPT-5/Opus 模型，无需自己折腾 API Key。

## **怎么用？（两种模式）**

1. **懒人模式 (Ultrawork)**

如果你只想快速搞定一个功能，不想废话，就在 Prompt 里加上 ultrawork  或 ulw ：

"ulw 给我的 Next.js 项目增加登录功能"

Agent 会自动分析代码、调研最佳实践、写代码、测试，全自动执行直到完成。

1. **专家模式 (Prometheus Planner)** 如果你要做的任务很复杂（涉及几十个文件、重构、关键改动），按 **Tab** 键进入 Prometheus 模式：
2. 它会像顾问一样**面试**你，确认需求细节。
3. 生成一份详细的**执行计划书**。
4. 输入 `/start-work` ，然后 Sisyphus (工头 Agent) 会指挥一群小弟 Agent 并行工作，分步骤完成计划。

# \~/.config/opencode

`~/.config/opencode/`配置目录下放以下系统提示词，系统提示词参考如下：

*(配图见飞书原文档)*

### 关于系统提示词位置OpenCode官网说明如下：


>
AGENTS.md 是自动加载的
文档原文：
> 您还可以在 \~/.config/opencode/AGENTS.md 文件中设置全局规则。这些规则会应用于所有 opencode 会话。
>
> 加载优先级：
1. 本地 AGENTS.md（项目根目录）     ← 自动发现
2. 全局 \~/.config/opencode/AGENTS.md ← 自动发现
3. Claude Code \~/.claude/CLAUDE.md   ← 自动发现（兼容回退）
不需要在任何配置文件中声明，OpenCode 启动时自动扫描并加载。
instructions 字段的作用
> **2026-06-01 更新**：已添加 CodeGraph MCP 至 ，并在全局  中配置了 CodeGraph 自动索引管理规则（新项目自动 init、老项目自动 sync）。详见文末 # CodeGraph — 代码知识图谱 MCP 章节。
> 文档原文：
> 您可以在 opencode.json 中指定自定义指令文件。这允许您和团队复用现有规则，而无需将它们复制到 AGENTS.md 中。
>
> instructions 是用来加载额外的、非标准位置的规则文件的，比如：
- CONTRIBUTING.md
- docs/guidelines.md
- .cursor/rules/\*.md
- 远程 URL


# \~/.config/opencode/oh-my-openagent.json

这是[oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent)的配置文件 可以给每个子代理配置不同的模型去使用。

> 配置之前先使用 /connect 链接到对应的provider
>
> 如 Alibaba(China) 对应的provider就是 `alibaba-cn` 不同的provider对应的模型前缀也不一样。

下边是 oh-my-openagent.json 的配置 示例 （当前有Claude Opus效果更好）：

*(配图见飞书原文档)*

# 最佳实践 OpenCode➕Ghostty➕yazi

> 由于iterm2过于笨重，且占用内存多，多开过于卡顿，然后发现了[Ghostty](https://ghostty.org/) 使用Rust编写 ，多开bash要比`iterm2➕zsh`这老一套速度快十倍不止，然后在VibeCoding的过程中偶尔还需要看AI输出的代码结果，此时可以使用`yazi`这个工具 如图右边的效果 可以快速预览目录下的文本内容，没有IDE这些的卡顿延迟体验很好。

### Ghostty 的背景故事

 [Mitchell Hashimoto](https://zhida.zhihu.com/search?content_id=271509655&content_type=Article&match_order=1&q=Mitchell+Hashimoto&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NzgzODEyNTYsInEiOiJNaXRjaGVsbCBIYXNoaW1vdG8iLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNzE1MDk2NTUsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.MFjynaLl6CEUG7lxl0yWf9NSkmy8EWreDvv5lFDenA8&zhida_source=entity)在开发Terraform、[Consul](https://zhida.zhihu.com/search?content_id=271509655&content_type=Article&match_order=1&q=Consul&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NzgzODEyNTYsInEiOiJDb25zdWwiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNzE1MDk2NTUsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.rwucXLl-t8b2ZI8n5YX7NLrFZpyvK0qK9p9AJctaMW4&zhida_source=entity)等工具时，发现现有终端工具性能瓶颈严重。2024年底，他用[Zig语言](https://zhida.zhihu.com/search?content_id=271509655&content_type=Article&match_order=1&q=Zig%E8%AF%AD%E8%A8%80&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NzgzODEyNTYsInEiOiJaaWfor63oqIAiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNzE1MDk2NTUsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.IdArl1Qn22EVOB1MyUVEuTD_cWoHgL5ftMpmmKSb6dU&zhida_source=entity)从零开始编写Ghostty，2025年正式开源，2026年已成为开发者圈最火的终端工具。

Anthropic官方推荐Ghostty作为Claude Code的首选终端，官方推荐理由是：

- ✅ 性能极致：超级快，GPU加速，渲染速度比iTerm2快3-5倍，Claude输出再长也不卡，滚动丝滑像丝绸。
- ✅ AI工具友好：完美支持Claude Code、Cursor等AI编程工具，而且持Kitty图形协议（Claude让你画图？图片直接在终端显示！）、Quick Terminal下拉幽灵、一键分屏、永久保存布局。
- ✅ 资源占用低：内存占用仅为iTerm2的1/3
- ✅ 开发者体验：Mitchell Hashimoto（HashiCorp创始人）亲自操刀

如果你对比Ghostty和其他终端，Ghostty在性能、易用性、AI工具支持三方面全面领先。

### yazi安装指南

> 可以直接打开opencode  告诉opencode让它帮你安装并且配置好 yazi 这个工具 然后等着执行完毕就可以了。

### Ghostty配置参考

[Ghostty：Claude Code的最佳搭档，终端生产力核武器](https://zhuanlan.zhihu.com/p/2016625071427428917)

#### \~/.config/ghostty/config 配置参考如下

```SQL
font-family = JetBrainsMonoNerdFont
font-size = 14
font-thicken = true
adjust-cell-height = 2

background-opacity = 0.9
background-blur-radius = 20
macos-titlebar-style = transparent
window-padding-x = 10
window-padding-y = 8
window-save-state = always
window-theme = auto

cursor-style = bar
cursor-style-blink = true
cursor-opacity = 0.8

mouse-hide-while-typing = true
copy-on-select = clipboard

quick-terminal-position = top
quick-terminal-screen = mouse
quick-terminal-autohide = true
quick-terminal-animation-duration = 0.15

clipboard-paste-protection = true
clipboard-paste-bracketed-safe = true

shell-integration = detect

keybind = cmd+t=new_tab
keybind = cmd+shift+left=previous_tab
keybind = cmd+shift+right=next_tab
keybind = cmd+w=close_surface

keybind = cmd+d=new_split:right
keybind = cmd+shift+d=new_split:down
keybind = cmd+alt+left=goto_split:left
keybind = cmd+alt+right=goto_split:right
keybind = cmd+alt+up=goto_split:top
keybind = cmd+alt+down=goto_split:bottom

keybind = cmd+plus=increase_font_size:1
keybind = cmd+minus=decrease_font_size:1
keybind = cmd+zero=reset_font_size

keybind = global:ctrl+grave_accent=toggle_quick_terminal

keybind = cmd+shift+e=equalize_splits
keybind = cmd+shift+f=toggle_split_zoom

keybind = cmd+shift+comma=reload_config

scrollback-limit = 25000000

```

### Ghostty快捷键

- Cmd + D → 左右添加屏幕（最常用！左边Claude写代码，右边debug）
- Cmd + Shift + Enter → 一键放大当前屏幕（看Claude长输出超爽，再按恢复）
- Cmd + W → 关闭当前屏幕
- Cmd + Shift + , → 重载配置（改完主题或分屏后一定要按！）
- Cmd + Q → 完全退出Ghostty（重启推荐用这个）

其他的详细功能如下：

*1️⃣分屏功能（无需tmux）*
水平分屏：

- 快捷键：`Cmd + D`
- 效果：右侧新建终端

垂直分屏：

- 快捷键：`Cmd + Shift + D`
- 效果：下方新建终端

切换分屏：

- 快捷键：`Cmd + [` / `Cmd + ]`
- 或鼠标点击

关闭当前分屏：

- 快捷键：`Cmd + W`

自定义分屏比例：

```Plain Text
# 配置文件中添加 split-divider-size = 2px split-divider-color = #3b4252
```


*2️⃣标签页管理*
新建标签页：`Cmd + T`
切换标签页：

- `Cmd + 1` \~ `Cmd + 9`：切换到第1-9个标签
- `Cmd + Shift + [` / `Cmd + Shift + ]`：前后切换

关闭标签页：`Cmd + Shift + W`


*3️⃣搜索功能*
开启搜索：`Cmd + F`
搜索技巧：

- 支持正则表达式
- 大小写敏感切换：`Cmd + Shift + F`
- 高亮所有匹配项

*4️⃣历史记录管理*
查看历史输出：

- 向上滚动查看
- `Cmd + ↑`：跳到历史开头
- `Cmd + ↓`：跳到历史结尾

历史记录大小：

scrollback-limit = 10000


*5️⃣主题切换*
内置主题列表（部分）：

- `catppuccin-mocha`（深色，推荐）
- `dracula`
- `nord`
- `gruvbox`
- `tokyo-night`
- `one-dark`

实时切换主题：

```Plain Text
打开配置文件
修改 theme = dracula
保存（Ghostty自动重载）
```

查看所有主题：

```Bash
ghostty +list-themes
```

### OpenCode多轮对话上下文管理

### OpenCode其他插件推荐

##### Plugin推荐

`oc-tweaks` 桌面通知 + 语言压缩 + 自动记忆。

`opencode-fmt` 格式化输出，有代码格式化的功能。

`oh-my-openagent@latest` 多Agent工具，支持UltraWorker模式，提升使用体验。

`@franlol/opencode-md-table-formatter@latest` 表格数据输出格式化。

`opencode-mermaid-renderer@latest` mermaid流程图浅显易懂 使用这个能直接在命令行渲染出来。

`@different-ai/opencode-browser` 可以直接控制你的浏览器，继承Cookie等，需配合Chrome扩展，可完成自动化测试等任务。

```JSON
  "plugin": [
    "oh-my-openagent@latest",
    "opencode-mermaid-renderer@latest",
    "@franlol/opencode-md-table-formatter@latest",
    "opencode-fmt",
    "oc-tweaks",
    "@different-ai/opencode-browser"
  ],
```

##### MCP推荐

`dbhub`的使用可以直接把数据库配置贴给opencode让它帮你配置 也可以完全自动化如果是Go项目 可以配置GitHub切分支钩子函数，然后读取到配置自动把对应配置文件切换过去，这样opencode在执行任务的时候能配合代码一起看数据库执行SQL。`rexmedia-image`则是悟空的文生图插件。

`codegraph` 代码知识图谱 MCP，提前索引代码的符号关系和调用链，大幅减少 Agent 探索代码库的 token 消耗（实测 \~57% 更少 token）。详见文末 # CodeGraph 章节。

```JSON
  "mcp": {
    "dbhub": {
      "type": "local",
      "command": [
        "node",
        "/Users/mac/.config/opencode/dbhub/dist/index.js",
        "--config=/Users/mac/.config/opencode/dbhub/dbhub.toml"
      ],
      "enabled": true
    },
    "rexmedia-image": {
      "type": "remote",
      "url": "https://mcp-gw.dingtalk.com/server/a3bb4c0991d53ce46db8c20cb78f76eb6ddd364b3c1d29b32d151fe35440dc92?key=3da1e0af9d895be1eda6272784e0b445",
      "enabled": true
    },
    "codegraph": {
      "type": "local",
      "command": ["codegraph", "serve", "--mcp"],
      "enabled": true
    }
  },
```

### OpenCode 相关插件自动更新脚本

*(配图见飞书原文档)*

# CodeGraph — 代码知识图谱 MCP

## 是什么

> 📖 **完整版独立指南**：→ [CodeGraph 完全指南](../codegraph/)（工作原理 · 安装配置 · 十大工具详解 · 实测数据 · 多分支最佳实践）

[CodeGraph](https://github.com/colbymchenry/codegraph) 是一个 **100% 本地** 的代码知识图谱工具，为 OpenCode (以及 Claude Code、Cursor、Codex、Gemini 等) 提供预索引的代码语义理解能力。**35K+ GitHub Stars**，MIT 协议。

**核心原理**：用 tree-sitter 提前把你的代码解析成 "谁调用了谁、谁实现了谁、谁导入了谁" 的知识图谱，存到本地 SQLite。Agent 不再需要 grep/glob/Read 反复扫文件来理解代码，而是直接查图——**一次 MCP 调用取代几十次工具调用**。

## 快速上手

```bash
# 1. 一键安装（自带 Node 运行时，无需单独装 Node）
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh

# 2. 为每个项目建索引（一次性，Go 项目约 2-5 秒）
cd /path/to/your-project
codegraph init -i

# 3. 重启 OpenCode 即可，Agent 自动使用
```

安装器会自动检测已安装的 Agent（OpenCode、Claude Code、Cursor 等）并配置 MCP server，**零手动配置**。

## 实测效果

| 指标 | 平均节省 | Go/Gin 项目实测（198 文件） |
|-|-|-|
| **Token 消耗** | \~57% 更少 | \~39% |
| **工具调用次数** | \~62% 更少 | \~58% |
| **API 成本** | \~25% 更便宜 | \~35-40% |
| **响应时间** | \~23% 更快 | \~25% |

> 数据来源：CodeGraph 官方 benchmark（7 个真实开源项目，4 runs/arm，Claude Opus 4.8）

**多分支项目（如 SKB，15 个客户交付分支）收益更大**：每次切换分支后 Agent 不需要重新探索代码结构，节省 \~65% token。

## 提供的 MCP 工具

| 工具 | 用途 | 示例 |
|-|-|-|
| `codegraph_search` | 按名称搜索符号 | "找所有叫 handleFragment 的函数" |
| `codegraph_context` | 构建任务上下文 | "理解视频片段流水线怎么走的" |
| `codegraph_trace` | 追踪调用路径 | "Pre handler 怎么调用到 ASR 的?" |
| `codegraph_callers` | 查找谁调用了某函数 | "谁在调用 saveParseResult?" |
| `codegraph_callees` | 查找某函数调用了谁 | "handleFragment 调用了哪些函数?" |
| `codegraph_impact` | 分析改动影响范围 | "改了 status 字段会影响哪些模块?" |
| `codegraph_explore` | 批量获取符号源码 | "给我看 Pre/ASR/NLP handler 的实现" |
| `codegraph_status` | 查看索引健康状态 | "索引是最新的吗?" |

## 我当前的 OpenCode 配置

已在 `~/.config/opencode/opencode.json` 中配置：

```json
"codegraph": {
  "type": "local",
  "command": ["codegraph", "serve", "--mcp"],
  "enabled": true
}
```

## 自动化管理

已在 `~/.config/opencode/AGENTS.md` 中添加了 CodeGraph 自动化规则，内容如下：

> ### CodeGraph 自动管理
>
> 会话启动时（处理任何用户请求之前），检查项目根目录 `.codegraph/` 是否存在：
>
> - **不存在** → 自动执行 `codegraph init -i`（首次建索引，约 2-5 秒，一次性操作）
> - **已存在** → 无需操作（CodeGraph MCP 连接时自动执行 connect-time catch-up 增量同步）
>
> 切 Git 分支后无需手动 `codegraph sync`，文件监听器自动检测变更并增量同步（2 秒去抖）。如急需同步可手动执行 `codegraph sync`。

这意味着：

- **新项目**：Agent 会自动 init，不需要你手动操作
- **老项目有变更**：MCP 启动时自动增量同步
- **切换分支**：文件监听器自动感知，2 秒后索引更新
- **完全无需人工干预**

## 当前已索引的项目

| 项目 | 文件数 | 节点数 | 边数 | DB 大小 |
|-|-|-|-|-|
|  (`~/Projects/project-a`) | 205 | 3,758 | 9,484 | 7.3 MB |
| SKB (`~/Projects/project-b`) | 291 | 7,046 | 24,203 | 14.8 MB |

```bash
# 查看索引状态
cd ~/Projects/project-a && codegraph status
cd ~/Projects/project-b && codegraph status
```

## 关键特性一览

- **20+ 语言支持**：Go、TypeScript、Python、Rust、Java、C/C++、Swift、Kotlin、Dart 等
- **框架路由识别**：自动识别 Gin/Express/Django/Spring 等 14 个框架的路由定义
- **影响分析**：改一个函数前，一键看所有调用方和被调用方
- **全文搜索**：SQLite FTS5 全文索引，比 grep 快得多
- **自动同步**：原生 OS 文件监听（FSEvents/inotify），写完代码 2 秒后索引自动更新
- **100% 本地**：代码不出机器，无外部 API 调用，无网络依赖
- **零配置**：无配置文件，自动遵守 `.gitignore`

## 注意事项

- Go 隐式接口满足关系（`interface` → `impl`）目前有已知 bug，可能不完整
- Go 泛型方法跨文件声明时调用边可能丢失（known issue）
- 索引数据库存储在项目根目录的 `.codegraph/` 中，建议加入 `.gitignore`

> 注：部分配图仅存于飞书原文档中，此处为纯文本版本。
