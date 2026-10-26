# Hermes Agent — 开发指南

面向参与 hermes-agent 开发的 AI 编程助手和开发者。本根文件是导航入口：先列出全局规则，再提供路由表。在某个目录中工作时，该区域的 `AGENTS.md` 会自动加载；编辑前先阅读。`python scripts/check` 将本文件限制为 12k 字符，将根目录到具体区域的每条文档链限制为 30k 字符，以便在上下文窗口为 128k 及以上的模型中完整加载；详细说明放入指南。

**绝不放弃正确的解决方案。**

## Hermes 是什么

Hermes 是个人 AI 智能体，在 CLI、消息网关（Telegram、Discord、Slack 等约 20 个平台）、TUI 和 Electron 桌面应用中运行同一个智能体核心。它能跨会话学习（记忆 + 技能）、向子智能体委派任务、运行定时任务，并操作真实的终端和浏览器。扩展能力主要通过**插件和技能**实现，而不是不断扩充核心。

两项不变量几乎决定了所有设计取舍，也应作为审查任何变更的依据：

- **每段对话的提示词缓存不可破坏。**修改过去的上下文、替换工具集、重新加载记忆，或在对话中途重建系统提示词，都会破坏已缓存的前缀，使用户成本成倍增加；唯一例外是上下文压缩。会改变系统提示词状态的斜杠命令应延后到下一会话生效，同时提供可主动选择的 `--now`（`/skills install --now`）。
- **核心保持精简，能力放在外围。**每次 API 调用都会发送所有模型工具，因此新增*核心*工具的门槛很高。新能力应通过 CLI 命令 + 技能、受服务条件控制的工具，或插件提供，而不是扩充核心接口。

## 贡献评判标准

这是项目的意图层，供贡献者和自动分流清理程序使用。后者仅可基于 `implemented_on_main`、`cannot_reproduce` 或 `incoherent` 关闭条目；基于偏好的关闭决定由维护者作出，存在疑问时应保持 PR 开放。包含示例的详细说明见 `website/docs/developer-guide/contributing.md` 的 Contribution rubric 一节。

**欢迎的贡献：**真实缺陷修复（能在 `main` 复现，指出具体代码行，覆盖整类问题及相邻路径）；外围能力扩展（适配器、提供商、模型、UI 功能），并接入现有设置流程；将巨型文件重构为模块；优先扩展而非重复实现（同类 PR 达到 3 个以上时，提取抽象基类 ABC + 编排器）；行为契约测试；对于解析、配置、安全和 I/O 变更，使用真实导入和临时 `HERMES_HOME` 进行端到端测试；通过 cherry-pick 保留有价值的改动和原作者署名。

**即使实现完善也不接受：**没有具体使用方的钩子；为非秘密配置新增 `HERMES_*` 环境变量（`.env` 只放秘密，行为配置放入 `config.yaml`）；终端 + 文件或技能已能完成工作时，仍新增核心工具；为指导类工具添加 `offset`/`limit` 分页；通过破坏功能来保证安全的“修复”；未经主动选择启用就向外发送遥测；仅检测变更的测试；触碰核心文件的插件；放在核心源码树中的第三方产品（应发布为独立插件仓库）。

**认定缺陷之前，**既要验证问题本身，也要验证设计意图（`git log -p -S "<symbol>"`）：隔离往往是有意设计的（各 profile 本就应相互独立）；某种缺失可能承担关键作用；如果所谓修复无法指出缺陷具体出现在哪一行，其前提就尚未得到验证。

**安全：**以 `SECURITY.md` 为范围界定依据。属于 §3.1 的发现必须通过私密渠道报告（GitHub Security Advisories 或 security@nousresearch.com），绝不放入公开 issue、PR、提交或评论；§3.2 的加固属于普通公开工作。指出跨越了 §2 中的哪条边界，并提供在 `main` 上的复现。

**改动范围优先级**（从前向后选择第一个能解决问题的层级）：扩展现有代码 → CLI 命令 + 技能 → 受服务条件控制的工具（`check_fn` 判断可达性 / 是否主动启用，绝不用来决定每个会话的工具暴露范围：`tools/AGENTS.md`）→ 插件 → 目录中的 MCP 服务器 → 新核心工具（基础、广泛适用，且其他方式无法实现）。

## 开发环境

`source ./activate`（fish：`activate.fish`，PowerShell：`activate.ps1`）会配置并激活 PM 环境；先选择隔离的 `HERMES_HOME`/`HERMES_RUNTIME_DIR`（`website/docs/reference/package-management.md#developer-workflow`）。测试需要 `CONTRIBUTING.md` 中单独的测试环境。**`python scripts/check`** 使用 CI 固定版本的工具，执行 CI 中全部阻断性 lint 检查；`--install-hook pre-push` 会在每次推送时运行它（拉取更新后重新运行，以刷新钩子）。`# noqa` 不能豁免棘轮式质量检查发现的问题；应使用 `# health: allow <RULE> -- <why>`。

## 项目结构

数量随时变化，应以文件系统为准。关键入口如下：

```
hermes-agent/
├── run_agent.py          # AIAgent facade; the turn loop lives in agent/turn_*.py
├── model_tools.py        # Tool orchestration, discover_builtin_tools(), handle_function_call()
├── toolsets.py           # TOOLSETS dict, _HERMES_CORE_TOOLS
├── cli.py                # HermesCLI (REPL, slash dispatch) + hermes_cli/cli_*_mixin.py
├── hermes_state.py       # SessionDB facade; hermes_state_*.py siblings
├── hermes_constants.py   # get_hermes_home(), display_hermes_home() — profile-aware paths
├── agent/                # turn loop phases, providers, memory, compression, prompt builder
├── hermes_cli/           # CLI subcommands, setup, config, plugins loader, updater, web_routers/
├── tools/                # Tool implementations (tools/registry.py) + environments/ backends
├── gateway/              # run.py facade + run_*.py phases + session*.py + platforms/
├── plugins/              # memory/, context_engine/, model-providers/, kanban/, image_gen/, ...
├── skills/               # Built-in skills (by category)   optional-skills/: shipped, not active
├── ui-tui/, tui_gateway/ # Ink terminal UI + its Python JSON-RPC backend (also serves Desktop)
├── apps/desktop/         # Electron desktop app (+ apps/shared)   web/: dashboard SPA
├── cron/                 # jobs.py + scheduler.py (+ scheduler_*.py)
├── pm/, hermes_platform/ # dependency/environment manager; machine facts + executable lookup
├── scripts/              # check, run_tests.sh, code_health/, ci/
├── website/              # Docusaurus docs (developer-guide/ holds the long-form area docs)
└── tests/                # Pytest suite, mirrors the source tree
```

**用户状态：**`~/.hermes/config.yaml`（设置）、`.env`（仅秘密）、`logs/`（`hermes logs`）；全部通过 `get_hermes_home()` 适配当前 profile。

### 门面 + 同级模块布局

原来的每个巨型文件都改为**门面**（公共入口 + 其他包导入的名称），加上按 `<stem>_<topic>.py` 命名的**同级模块**，每个模块负责一个主题（`hermes_state.py`、`gateway/run.py`、`tools/mcp_tool.py`、`hermes_cli/kanban.py`、`hermes_cli/web_server.py`、`cli.py` → `hermes_cli/cli_*_mixin.py`、`run_agent.py` → `agent/turn_*.py`）。

- **按主题查找代码：**使用 `grep -rn "def name" <dir>/<stem>_*.py`，而不是通读门面文件。
- **同级模块在函数内部延迟导入门面**，绝不形成模块级循环依赖。
- **在生产代码实际读取的位置打补丁：**如果同级模块在函数内执行 `from <facade> import name`，门面就是替换接缝；对定义该符号的模块打补丁则会悄无声息地落空。
- **按单元实施规模和复杂度只降不升的约束**（`scripts/code_health/config.py`）：新函数的圈复杂度 CC ≤ 20、行数 ≤ 300、嵌套深度 ≤ 6；文件 ≤ 2,000 行。已经超限的单元只能缩减（将函数移到同级模块以抵消增长，或将新测试放入新测试文件）。行为实现应放在同级模块中，绝不放在门面中；按名称逐项判断的分支链应改为字典到处理器的映射。
- **内部移动不添加重新导出的兼容层；**内部路径不是 API。移动符号时，必须在同一个 PR 中修正文档（搜索 `website/docs`、`skills/` 及每个 `AGENTS.md`）。

## 全局适用规则

- **不堆砌防御性代码：**不要为不会失败的代码添加包装层或 `try/except: pass`；不要添加无人设置的标志；没有端到端验证，不要接入死代码。注释保留“为什么”，删除“做了什么”。
- **绝不根据 argv 子字符串推断进程身份；**使用 `gateway.status.looks_like_gateway_command_line` / `hermes_cli.update_cmd._hermes_holder_subcommand`（HX003；详情见 `hermes_cli/AGENTS.md`）。
- **绝不硬编码 `~/.hermes`：**路径使用 `get_hermes_home()`，文本使用 `display_hermes_home()`（HX001）。`_get_profiles_root()` 有意以 HOME 为锚点。
- **一个进程服务多个 profile。**在交互轮次之外运行的代码（启动探测、驱逐、定时器、延迟回调、RPC 方法、线程切换、子进程启动）必须显式绑定所属 profile 的作用域；`os.environ`、模块全局变量和导入时取值保存的是启动 profile 的内容，因此未绑定的读取会悄悄误用默认 profile。绑定位置和接缝见 `gateway/AGENTS.md` 的 Profile scope 一节（HX002/HX004/HX005/HX012、PS-P05/P06）。
- **机器信息和可执行文件查找统一通过 `hermes_platform`**（`hermes_platform/AGENTS.md`）。
- **依赖必须带版本上限**（`>=floor,<next_major`；git URL 和 Actions 固定到 SHA）；编辑 `pyproject.toml` 后运行 `hermes pm lock` 并提交 `uv.lock`；绝不直接使用 pip/uv 修改 Hermes 环境。完整政策见 `pm/AGENTS.md`。
- **TypeScript（桌面、TUI、网站）：**优先使用功能自身管理的 nanostores，而不是层层传递状态；路由根组件保持轻量，钩子职责窄小，操作模块就近放置；props 和共享结构使用 `interface`；优先表驱动分发而不是条件分支链；`src/app` 放路由，`src/store` 放原子状态，`src/lib` 放纯辅助函数。
- **提交和 PR：**合并前 rebase 到 `main`（从过时分支 squash 会悄悄撤销较新的修复）；每项修复提供 1–2 个不变量测试，并证明它们在基线版本上失败。
- **测试：**始终使用 `scripts/run_tests.sh`，绝不直接运行 `pytest`；测试行为契约，绝不编写仅检测变更或读取源码的测试；特定主机行为应通过 `@pytest.mark.platforms(...)` 在对应主机上测试，绝不伪造 `sys.platform`；测试绝不写入 `~/.hermes/`。其余要求见 `tests/AGENTS.md`。

## 路由表：在 X 中工作 → 阅读 X/AGENTS.md

| 区域 | 阅读 | 涵盖内容 |
|---|---|---|
| `run_agent.py`、`agent/` | `agent/AGENTS.md` | 轮次阶段、缓存与消息流不变量、压缩、模型 / 辅助模型解析 |
| `cli.py`、`hermes_cli/` | `hermes_cli/AGENTS.md` | CLI mixin、斜杠命令注册表、配置系统、皮肤、`hermes update`、profile / 多路复用 |
| `gateway/` | `gateway/AGENTS.md` | 适配器、消息防护、流式输出、通知、token 锁、Profile scope 一节 |
| `gateway/platforms/` 中的新适配器 | `gateway/platforms/ADDING_A_PLATFORM.md` | 逐步编写适配器的指南 |
| `tools/`、`toolsets.py`、`model_tools.py` | `tools/AGENTS.md` | 添加工具、注册表、工具集、委派、会话作用域的接口工具 |
| `plugins/`、`hermes_cli/plugins*.py` | `plugins/AGENTS.md` | 插件种类、原生兼容契约、仓库内插件政策 |
| `plugin-catalog/` | `plugin-catalog/README.md` | 目录准入规则（开发者指南中有镜像，必须保持一致） |
| `tui_gateway/`、`ui-tui/` | `tui_gateway/AGENTS.md` | 进程模型、JSON-RPC 传输、斜杠命令流程 |
| `web/`、`hermes_cli/web_routers/` | `web/AGENTS.md` | 仪表盘嵌入真实 TUI |
| `apps/desktop/` | `apps/desktop/AGENTS.md`、`apps/desktop/src/AGENTS.md` | `serve` 后端、斜杠命令面板、Bot Mode |
| `skills/`、`optional-skills/`、`agent/curator*.py` | `skills/AGENTS.md` | frontmatter、编写标准、curator |
| `cron/`、kanban | `cron/AGENTS.md` | 调度器不变量、任务字段、看板分发器 |
| `tests/` | `tests/AGENTS.md` | 运行器、放置位置、操作系统标记、`wine2e`、禁止的测试形式 |
| `pm/`、`pyproject.toml` | `pm/AGENTS.md` | 版本固定政策、PM 管理的环境、插件隔离 |
| `hermes_platform/` | `hermes_platform/AGENTS.md` | 主机信息、解析器 |

详细背景资料：`website/docs/developer-guide/`。工作流规则（PR / issue / 审查 / 有价值改动保留流程）位于 `hermes-agent-dev` 技能中，不在此处。
