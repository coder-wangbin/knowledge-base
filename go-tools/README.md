# Go热编译 - Air

# 安装Air

```Plain Text
go install github.com/air-verse/air@latest
```

# 配置文件

> 放在项目根目录下 新建 .air.toml 文件

```Bash
root = "."
testdata_dir = "testdata"
tmp_dir = "tmp"

[build]
  args_bin = []
  bin = "./tmp/main"  # 二进制文件的输出路径
  cmd = "go build -o ./tmp/main ./cmd/server"  # 编译命令，注意这里要指向源代码所在目录
  delay = 1000
  exclude_dir = ["assets", "tmp", "vendor", "testdata"]
  exclude_file = []
  exclude_regex = ["_test.go"]
  exclude_unchanged = false
  follow_symlink = false
  full_bin = ""
  include_dir = []
  include_ext = ["go", "tpl", "tmpl", "html"]
  include_file = []
  kill_delay = "0s"
  log = "build-errors.log"
  poll = false
  poll_interval = 0
  post_cmd = []
  pre_cmd = []
  rerun = false
  rerun_delay = 500
  send_interrupt = false
  stop_on_error = false

[color]
  app = ""
  build = "yellow"
  main = "magenta"
  runner = "green"
  watcher = "cyan"

[log]
  main_only = false
  time = false

[misc]
  clean_on_exit = false

[screen]
  clear_on_rebuild = false
  keep_scroll = true
```

# 启动命令

```Plain Text
air server
```

# 其他工具

**Fresh**:

- **简介**: 另一个自动重载工具，能够监控文件更改并自动重启应用。
- **特点**: 内置预构建和秒级重启，支持自定义配置。
- **安装**:

```Bash
go install github.com/gravityblast/fresh@latest
```

**CompileDaemon**:

- **简介**: 一个简单的监视工具，能够监控代码变化并重新编译和重启 Go 应用。
- **特点**: 支持多种事件触发操作，可以自定义行为。
- **安装**:

```Bash
go install github.com/codegangsta/compiledaemon@latest
```

**Gorud**:

- **简介**: 另一款热重载工具，可以在代码更改时重建和重启服务。
- **特点**: 支持多种编译模式，适合大型项目。
- **安装**:

```Bash
go install github.com/oklog/oklog@latest
```

---

#  General Global Rules

---

# Global Rules

> 适用对象：软件开发工程师
> **Always respond in 简体中文**

---

## 一、通用工程礼节（General Engineering Etiquette）

- 优先保证代码简洁、清晰、可读，可读性高于技巧性。
- 禁止过度设计，在满足当前需求的前提下选择最简单可行方案。
- 严格控制圈复杂度，函数保持单一职责，逻辑可拆则拆。
- 避免重复代码（DRY），但不要为了复用而强行抽象。
- 模块职责必须清晰，依赖方向单一。
- 在合适场景下使用设计模式，必须说明使用原因，禁止为了模式而模式。

---

## 二、代码解释与实现要求

- 解释代码时必须“说人话”，避免堆砌专业术语，必要时进行解释。
- 复杂逻辑必须配图，使用 `mermaid` 风格（流程图 / 时序图 / 状态图）。
- 实现功能时必须同时给出：

  - 实现原理说明
  - 清晰的执行步骤（Step by Step）
  - 关键流程的 `mermaid` 图示

---

## 三、代码修改强约束

- 修改或解释前，必须完整阅读所有相关代码，不得偷懒或基于猜测修改。
- 遵循最小化修改原则（Minimal Change），尽量不影响其他模块。
- 禁止顺手重构、顺便优化与当前需求无关的代码。
- 修改完成后，必须假定至少 10 条输入用例，并给出对应的预期结果。

---

## 四、Bug 修复实验性规则（Experimental Bug Fix Rule）

当被要求修复 Bug 时，必须严格按照以下流程执行，不得跳步：

1. **理解问题（Understand）**

   - 复述 Bug 现象与影响范围
   - 明确是否可复现
2. **分析原因（Analyze）**

   - 至少提出两种不同的根本原因假设
3. **制定计划（Plan）**

   - 针对每个假设说明验证方法
   - 给出最终修复方案
4. **请求确认（Confirm）**

   - 在修改代码前，必须向我确认修复计划
5. **执行修复（Execute）**

   - 严格按照确认方案修改代码
6. **自我审查（Review）**

   - 检查是否引入新问题或破坏原有逻辑
7. **修改说明（Explain）**

   - 说明修改内容、修改原因及不修改的风险

---

## 五、中文注释与编码兼容规则（强制）

- 绝对不能删除、修改或替换任何已有中文注释（无论是否相关）。
- 中文注释必须完整原样保留，包括内容、标点、空格和格式。
- 可以在中文注释周围新增英文注释或说明，但不得改动中文注释。
- 即使进行重构、补全或优化，所有中文注释也必须保持不变。
- 不重新格式化任何现有代码和中文注释。
- 只允许在文件末尾追加内容。
- 不引入 BOM。
- 中文注释保持 UTF-8 原样编码。
- Walkthrougt 以简体中文输出

---

## 六、默认行为约定

- 在未明确要求时，优先选择：**可维护性 > 可读性 > 性能 > 技巧**。
- 当需求存在不确定性时：

  - 明确指出不确定点
  - 给出合理假设
  - 说明假设可能带来的影响

---

- Always respond in 简体中文
