# AGENTS 指南

goose 是用 Rust 编写的 AI 智能体框架，提供 CLI 和 Electron 桌面界面。

## 贡献流程

对于计划向上游提交拉取请求的工作，应以 issue 为准。在 [Goose Issues 看板](https://github.com/orgs/aaif-goose/projects/1)上跟踪 issue 状态。

- 为拉取请求实现某个 issue 前，确认它已在看板上，且 Status 为 **Ready**。
- 不要实现处于 **Inbox**、**Needs info** 或 **Accepted / design** 状态的 issue。应先协助解决 issue 讨论中的问题。
- 修改代码前，阅读已达成一致的设计、约束、非目标和验证计划。
- 实现应保持在 issue 已约定的范围内。
- 如果实现过程中发现需要重大设计变更，应先回到 issue 讨论，再继续工作。
- 每个外部拉取请求都必须链接到它实现的 Ready issue，并说明如何执行了验证计划。
- 新建 issue 时，按 `.github/ISSUE_TEMPLATE/` 中对应的模板组织内容，并设置 issue 类型（例如 Bug、Feature）。`gh issue create` 不会自动应用模板。

维护者指派的工作、紧急安全修复、发布自动化，以及本地或探索性修改，不要求有 Ready issue。

## MCP 服务器目录

goose 正在弃用项目自有的 MCP 服务器目录，改用[官方 MCP Registry](https://github.com/modelcontextprotocol/registry) 及其 `server.json` 格式。

- 不要向 `documentation/static/servers.json` 添加新的第三方服务器；此类贡献已不再接受。
- 应引导服务器作者发布到官方 MCP Registry。
- 在实现基于 Registry 的发现和安装功能期间，将现有 goose 目录视为旧版数据。
- 允许维护、迁移或删除现有目录条目，前提是这些变更支持迁移，且在已获批准的 issue 范围内。

决策和迁移方向见[讨论 #10830](https://github.com/aaif-goose/goose/discussions/10830)。

## GitHub 沟通

issue 和拉取请求评论应面向人类读者，不要写成详尽的工作日志。

- 开头先给出结论或需要采取的行动。
- 评论保持简洁，不重复讨论串中已有的上下文。
- 使用短段落或项目符号；仅当实现细节会影响决策或审查时才加入。
- 优先写一条清晰的总结，避免多条零碎的进度评论。

## 环境设置
```bash
source bin/activate-hermit
cargo build
```

## 命令

### 构建
```bash
cargo build                   # debug
cargo build --release         # release
just release-binary           # release binary
```

### 测试
```bash
export GOOSE_DISABLE_KEYRING=1  # avoid system keyring prompts
cargo test                   # all tests
cargo test -p goose          # specific crate
cargo test --package goose --test mcp_integration_test
just record-mcp-tests        # record MCP
```

### Lint / 格式化
```bash
cargo fmt
cargo clippy --all-targets -- -D warnings
```

### UI
```bash
just run-ui                  # start desktop
cd ui/desktop && pnpm run typecheck
cd ui/desktop && pnpm test   # test UI
```

## 目录结构
```
crates/       # Rust workspace members — see root Cargo.toml (`members = ["crates/*"]`)
ui/desktop/   # Electron app
ui/text/      # deprecated ACP TUI (see ui/text/README.md)
```

部分工作区 crate（包括组成 GDK 的 crate）发布到 crates.io，并提供公共 API。GDK crate 的权威列表是 `release-plz.toml` 中设置了 `release = true`、`version_group = "gdk"` 的包集合，GDK 发布由此驱动；运行 `python3 crates/goose-sdk/scripts/gdk-release.py crates` 可输出该列表。其他 crate（例如 `goose` 和 `goose-cli`）不提供稳定的公共 API；它们的 `pub` 项属于内部实现细节，可能随时变更而不另行通知。

## 开发循环
```bash
# 1. source bin/activate-hermit
# 2. Make changes
# 3. cargo fmt
```

### 仅当用户要求构建 / 测试你的修改时，才运行以下命令：
```
# 1. cargo build
# 2. GOOSE_DISABLE_KEYRING=1 cargo test -p <crate>
# 3. cargo clippy --all-targets -- -D warnings
```

## 规则

- 测试：优先使用 tests/ 目录，例如 crates/goose/tests/。
- 测试：添加功能时，更新 goose-self-test.yaml，重新构建，再运行 `goose run --recipe goose-self-test.yaml` 进行验证。
- 错误：使用 anyhow::Result。
- 提供商：实现 Provider trait，参见 providers/base.rs。
- MCP：扩展位于 crates/goose-mcp/。
- 桌面 UI：使用 ACP SDK 类型或本地 `src/types/*` 类型。不要从 `ui/desktop/src/api` 导入生成的 OpenAPI 类型或客户端代码。

## 代码质量

- 注释：编写能够自解释的代码，优先使用清晰的命名，而不是注释。
- 注释：绝不添加只是重复代码行为的注释。
- 注释：仅为复杂算法、不明显的业务逻辑，或解释“为什么”而非“是什么”的内容添加注释。
- 简洁性：不需要可选的内容不要设为可选，编译器会提供约束。
- 简洁性：布尔值应默认为 false，而不是设为可选。
- 错误：不要添加没有提供有效信息的错误上下文（例如错误已经说明失败时，再加 `.context("Failed to X")`）。
- 简洁性：避免过度防御性代码，信任 Rust 的类型系统。
- 日志：清理已有日志；除非用于错误或安全事件，否则不要增加日志。

## 禁止事项

- 禁止：重新创建 `ui/desktop/src/api`，或向 `ui/desktop` 添加 `@hey-api/openapi-ts`。
- Cargo.toml：对于人工编写的依赖变更，使用 `cargo add`，不要手动编辑依赖条目，除非有明确理由。
- Cargo.toml：自动依赖升级 PR 不受此限制；必须手动编辑时，保持 `Cargo.lock` 一致。
- 禁止：跳过 cargo fmt。
- 禁止：未运行 clippy 就合并。
- 禁止：为显而易见的操作（`// Initialize`、`// Return result`）、getter/setter、构造函数或标准 Rust 惯用写法添加注释。
- 禁止：原地覆盖正在使用的二进制文件（例如用 `cp`/`fs.copyFileSync` 写入已有可执行文件）。应先解除目标链接或进行原子重命名，否则 macOS 会以“Code Signature Invalid”为由向正在运行的进程发送 SIGKILL。

## 入口
- CLI：crates/goose-cli/src/main.rs
- UI：ui/desktop/src/main.ts
- 智能体：crates/goose/src/agents/agent.rs
