这是 **Cline** 单体仓库（monorepo）。工具链为 **Bun 1.4.2**（包管理器和任务运行器），运行时为 **Node >=22**。不要使用 npm/yarn/pnpm。

不要添加 `.changeset/` 文件或其他按 PR 单独记录的变更日志条目；本仓库不使用 changesets。发布说明在发布时写入相应的 `CHANGELOG.md`。

## 云端智能体说明

### Cline CLI
- 从源码运行：`bun run cli`（交互模式：`bun run cli -i`；单次执行：在命令后追加提示词）。该命令会定位到 `apps/cli`，并**自动启动 `@cline/cline-hub` 守护进程**，无需单独启动 hub。
- 使用 `bun run cli doctor` 检查本地健康状态；`bun run cli version` 输出版本号。
- 实际执行一轮智能体交互需要 **LLM 提供商凭据**。没有凭据时，默认的 `cline` 提供商会立即报 `Unauthorized` 错误，交互式 TUI 则显示提供商登录界面。通过 `cline auth` 或提供商环境变量（例如 `ANTHROPIC_API_KEY`、`CLINE_API_KEY`、`OPENROUTER_API_KEY`）配置；参见 `apps/cli/README.md`。

### 构建 / Lint / 测试
- SDK 包（`@cline/shared|llms|agents|core|sdk`）通过编译后的 `dist/` 相互解析（其 `exports` 仅指向 `dist/`，没有 `development` 源码条件）。修改 SDK 依赖或源码后，在运行 CLI 或 SDK 测试前，**必须**执行 `bun run build:sdk`，否则导入会因缺少 `@cline/*` 或 `dist/` 而失败。正在运行的进程**不会**热重载 SDK 源码变更，必须重新构建并重启。\
- 已知的云环境测试异常：`@cline/core` 中的测试 `src/services/workspace/workspace-manifest.test.ts > readGitWorkspaceState > prefers origin and returns the current branch` 会失败，因为云虚拟机配置了 git `insteadOf` 规则，将 GitHub 远程地址改写为 `https://x-access-token:...@github.com/...`。这是环境引起的异常，并非代码缺陷。
- 某些 `@cline/cli` 端到端断言（`bun -F @cline/cli test:e2e`）可能因工具列表字符串的精确格式而失败；应视为已有的测试与实现不一致，而不是环境问题。

### GUI 显示
- 虚拟 X 显示已在 **`DISPLAY=:1`** 上运行（与截图使用同一个桌面）。使用 `DISPLAY=:1` 启动的 GUI 应用（VS Code、Tauri 桌面窗口）会显示在那里，并可截图；无需自行启动 `xvfb`。优先在 `tmux` 会话中启动长时间运行的 GUI 或开发进程（参见 tmux 指南），使其持续运行。

### VS Code 扩展（`apps/vscode`，包名 `claude-dev`）
工具链已预装并持久保存在虚拟机中：生成的 gRPC/proto 代码、随附的 `ripgrep` 二进制文件（`apps/vscode/bin/`）、构建后的 webview（`webview-ui/build`）、esbuild 打包产物（`dist/extension.js`）、VS Code 本体（`/usr/bin/code`），以及测试所需的 GUI 系统库。
- **代码生成前置步骤：**在 `apps/vscode` 中执行 `bun run protos`，重新生成 `src/generated/*` 和 webview grpc 客户端。`dev`、`build:webview` 和 `check-types` 脚本已经会执行它，因此这些命令会处理 proto 变更；仅在修改 `.proto` 文件且不进行完整构建时，才需要手动运行。
- **构建：**先执行 `bun run build:webview`（webview UI，约 15 秒），再执行 `bun esbuild.mjs`（扩展打包）。`bun run package` 执行完整的生产构建。
- **运行（开发宿主）：**`DISPLAY=:1 code --no-sandbox --user-data-dir=/tmp/vscode-userdata --extensionDevelopmentPath=/workspace/apps/vscode <some-folder>`，然后点击活动栏中的 Cline 图标打开 webview。（此容器中必须使用 `--no-sandbox`。）
- **测试：**`bun run test:unit`（基于 bun，约 984 个测试，无需 VS Code 宿主）。`bun run test:integration`（使用 `@vscode/test-electron`，下载一个 VS Code 构建版本，并在 GUI 库环境下运行）和 `bun run test:e2e`（Playwright）会测试真实的扩展宿主，开销更大；所需 GUI 库已安装。
- 一次性依赖（已安装，此处列出以便必要时重建）：通过 `bun run download-ripgrep` 获取 ripgrep；VS Code 测试的 GUI 库见 `CONTRIBUTING.md`（`libnss3`、`libatk*`、`libgbm1`、`xvfb` 等）。

### 桌面应用（`apps/examples/desktop-app`，包名 `@cline/code`）
采用 Tauri v2（Rust）外壳、Next.js webview 和 Bun “sidecar” 后端。Rust 和 Tauri Linux 系统库已预装并持久保存。
- **无头模式（无 Rust / 窗口）：**分别运行后端和 UI：`bun run dev:sidecar`（Bun 后端位于 `127.0.0.1:3126`，提供 `ws://.../transport`）以及 `bun run dev:web`（Next.js UI 位于 `http://localhost:3125`）。
- **原生窗口：**`bun run dev`（`tauri dev`）；其 `beforeDevCommand` 会构建 sidecar 二进制文件并启动 `dev:web`（`:3125`），随后 Rust `main.rs` 启动 sidecar，因此应先释放端口 `3125`/`3126`。使用 `DISPLAY=:1` 启动以查看窗口。`libEGL: DRI3 error` 警告无害（使用软件渲染），WebKitGTK 窗口仍会正常显示。
- **Rust 版本注意事项：**crate 依赖图需要 Cargo 的 `edition2024` 特性，因此要求 **Rust ≥1.85**（虚拟机基础版本 1.83 会报“feature `edition2024` is required”）。此处工具链已通过 `rustup default stable` 更新（当前为 1.97）。首次 `cargo` 构建会下载并编译完整的 Tauri crate 依赖图（耗时几分钟）；后续构建会使用缓存。
- **系统库（已安装）：**`libwebkit2gtk-4.1-dev`、`libgtk-3-dev`、`libayatana-appindicator3-dev`、`librsvg2-dev`、`libxdo-dev`、`libssl-dev`、`build-essential`。
- **测试 / 类型检查：**`bun run typecheck`、`bun run test:chat-ui`（Vitest）。两者都会先触发 `build:ui`。
