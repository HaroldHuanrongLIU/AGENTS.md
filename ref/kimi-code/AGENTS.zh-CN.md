# 仓库级智能体指南

使用与用户相同的语言回复。

这是一个面向智能体辅助开发的 TypeScript 单体仓库。根目录 `AGENTS.md` 只保留高频必需规则：项目地图、硬性约束和工作流要求，即每个任务都需要了解的内容。

## 工作原则

- 从第一性原理出发。以真实需求、代码事实和验证结果为依据；如果目标不清楚，先与用户讨论。
- 以代码而非文档为准。除非用户明确要求，否则不要仅为了理解实现而阅读普通 Markdown 文档。
- 修改代码前，阅读相关代码和最新约束，并遵循目录树中最近的 `AGENTS.md`。
- 保持改动聚焦，不要顺便夹带无关重构。
- 提交时不添加任何共同作者署名，也不要在提交消息、PR 描述或任何解释性文本中透露智能体身份。

## 项目地图

- `apps/kimi-code`：CLI / TUI 应用。通过 `@moonshot-ai/kimi-code-sdk` 使用核心能力，不得直接依赖引擎包。编写或修改其终端 UI 时，使用 `write-tui` 技能（`.agents/skills/write-tui/SKILL.md`）。
- 浏览器 Web UI：**其源码已不在本仓库中。**它在 code-app 仓库（`apps/web`）中开发，以已提交的预构建包 `apps/kimi-code/dist-web` 形式分发（被 gitignore 忽略，通过强制添加纳入版本控制）。使用 `KIMI_CODE_REPO=<this checkout> pnpm run sync:web` 从 code-app 同步；凡需发布不同的 Web UI，都必须在同一次变更中同步并提交该包。`apps/kimi-code/scripts/check-web-assets.mjs` 防止打包时缺少资源包。要使用本仓库服务器开发 Web UI，在此运行 `pnpm dev:server`，并通过 `KIMI_SERVER_URL` 将 code-app 的 `pnpm dev:web` 指向它。
- `apps/vis`、`apps/vis/server`、`apps/vis/web`：会话和回放的可视化调试工具。
- `apps/kimi-inspect`：面向 kap-server `/api/v1/debug` RPC 接口的 Web 检查器，提供工作区 / 会话浏览器、按会话展示的对话记录、按作用域展示的 Service 面板和依赖注入单元检查视图。参见 `apps/kimi-inspect/AGENTS.md`。
- `packages/agent-core-v2`：DI × Scope 智能体引擎（kap-server 背后的 v2 移植版本）。包含四级 `LifecycleScope`：`App` / `Workspace` / `Session` / `Agent`（`app/scopes.ts`），以及 L3 单元层（`Service`/`Fiber` 单元、集合扩展点、`src/features/` 中的 Feature 接缝）；没有 App 级会话生命周期门面，调用方按 `ISessionIndex` → `IWorkspaceLifecycleService.handlerFor` → handler 的顺序组合调用。
- `packages/node-sdk`：公共 TypeScript SDK 和测试 / 运行支撑框架。
- `packages/kosong`：LLM / 提供商抽象层。
- `packages/oauth`：Kimi OAuth 和托管认证工具。
- `packages/telemetry`：共享的客户端遥测基础设施。
- `packages/transcript`：同构的对话记录渲染数据层，包含 L1 智能体粒度存储、L2 幂等操作、L3 `off/turn/block/delta` 订阅粒度、L4 不依赖框架的视图注册表，以及基于轮次游标的分页。使用纯 TypeScript（浏览器安全，不导入引擎）；是对话记录契约类型（`src/contract/`）和操作批次排序契约的唯一归属。
- `packages/kap-server`：Kimi Code 服务器，由 `@moonshot-ai/agent-core-v2` 支撑；通过 REST + WebSocket（`/api/v1` + `/api/v1/ws`）暴露会话，同时提供 `/api/v1/debug/*` 反射 RPC 接口（`--debug-endpoints`，绑定回环地址 + bearer 认证）。
- `packages/remote-control`：Kimi Remote Control 隧道客户端，将本机注册到中继，并把 HTTP/WebSocket 流量转发到本地服务器，使用整机级单实例锁；由 kap-server（`/api/v1/remote-control` 开关）和 CLI（`kimi web --remote-control`）使用。
- `packages/klient`：客户端 SDK，以契约驱动的门面包装 agent-core-v2（`global.*` / `session(id).*` / `agent(id).*`，经 zod 校验）；通过子路径入口提供传输（`@moonshot-ai/klient/ipc|memory`，两者返回相同的 `Klient`）；也存放端到端测试套件。参见 `packages/klient/AGENTS.md`。
- `packages/tree-sitter-bash`：纯 TypeScript bash 解析器（无运行时依赖，无 wasm）；`parse(source, { timeoutMs, maxNodes })` 在确定性预算内运行，并返回可辨识的 `ParseResult`。调用方必须把已中止或 hasError 的语法树视为“无法分析”，并执行降级处理。它只负责解析，不作安全判断；参见该包 README 的“Known differences”一节。
- `packages/minidb`：kap-server 搜索索引背后的嵌入式 JSON 文档存储（`MiniDb`），采用快照 + WAL 持久化和独占写锁，支持超出内存容量的全文检索层，以及持久化的索引代次。参见 `packages/minidb/AGENTS.md`。

## 环境要求

- **Node.js**：`>=24.15.0`（来自根目录 `package.json` 的 `engines`；`.nvmrc` 为 `24.15.0`，供 nvm / fnm / mise 选择最低推荐版本）。
- **pnpm**：`10.33.0`（来自根目录 `package.json` 的 `packageManager`）。
- Node 版本不满足要求时，`pnpm install` 会失败，因为 `.npmrc` 设置了 `engine-strict=true`。

## 单体仓库工作区维护

- `pnpm-workspace.yaml` 是工作区成员的权威来源，但 `flake.nix` 中也包含**硬编码**的 `workspacePaths` 和 `workspaceNames` 列表。
- **每次添加或删除工作区包，都必须同时更新 `pnpm-workspace.yaml` 和 `flake.nix`。所有包都适用，包括没有任何其他包依赖的叶子包、测试包和端到端测试包。**
  - `pnpm-workspace.yaml` 使用通配模式（`packages/*`、`apps/*`），因此大多数包会自动纳入；`flake.nix` 完全手动维护，遗漏通常发生在这里。
  - `flake.nix` 的 `workspacePaths` 缺少路径，会导致相应文件被悄悄排除在 Nix 构建的 `src` 文件集之外。
  - `flake.nix` 的 `workspaceNames` 缺少名称，会使 `pnpmConfigHook` 失败，因为该工作区的依赖不会被获取。
- 自动化的“Check flake.nix workspace sync”检查（`scripts/check-nix-workspace.mjs`）只验证 **`@moonshot-ai/kimi-code` 的传递依赖闭包**。闭包之外的叶子包（例如无人导入的端到端测试包）即使未出现在 `flake.nix` 中，也会漏过检查。因此，检查通过**不能**证明 `flake.nix` 已完全同步；每次添加 / 删除包时都要手动更新，不要依赖检查发现遗漏。

## 通用编码规则

- `packages/agent-core-v2`、`packages/kap-server` 和 `packages/transcript` 是无注释区域：不得添加任何注释，包括行 / 块注释和 JSDoc（导出符号也不例外）；唯一例外是确有必要的 lint 抑制指令（`oxlint-disable` / `eslint-disable`），其他工具指令（`@ts-expect-error` 等）仍被禁止。`scripts/check-no-comments.mjs` 会检查 `src/`/`test/`/`scripts/` 下的 `.ts`/`.tsx`/`.mts`/`.mjs` 文件，作为 `pnpm lint` 的一部分执行。
- 可选对象属性直接传入 `undefined`，不要使用条件展开。
  - 正确：`{ user }`
  - 错误：`{ ...(user ? { user } : undefined) }`
- 可选对象属性的类型无需额外允许 `undefined`。
  - 正确：`interface Options { user?: User }`
  - 错误：`interface Options { user?: User | undefined }`
- 只有一个参数的内部方法，不应仅为风格统一而改成选项对象。
- 只沿抽象层级拆分函数：每个函数应处于同一个叙述层次（逐层下降规则，Step-down Rule）。不增加新抽象层的包装函数，尤其是只有一个调用位置的，应内联而非提取。
- 除包本身的 `index.ts` 外，其他 `index.ts` 文件应优先使用 `export * from './module';`。
- 不要新增过多测试文件。优先向对应组件或模块的现有测试文件添加测试。
- 测试因用户修改而失败时，默认先修复测试；除非实现确有缺陷，否则不要为满足旧测试而改变实现。
- 除非用户明确要求，否则不要为外部兼容性牺牲代码质量。破坏性变更应通过 changeset 和 `major` 版本升级处理，并遵守下方规则。

## 实验性功能

- 尚未公开的功能应放在实验性标志之后。标志由环境变量驱动，默认关闭：`KIMI_CODE_EXPERIMENTAL_<NAME>` 控制单个标志，`KIMI_CODE_EXPERIMENTAL_FLAG` 启用全部标志。优先级为：单个标志环境变量 > `[experimental]` 配置 > 总开关环境变量 > 标志自身的 `default`。发布时将条目的 `default` 改为 `true`。
  - 对于 `packages/agent-core-v2` 和 kap-server 模块：没有中央目录；在所属领域中于导入时通过 `registerFlagDefinition` 声明标志，然后通过 `IFlagService.enabled(id)` 检查。

## 在哪里更新指南

- 影响几乎所有任务的硬性规则：更新根目录 `AGENTS.md`。
- 仅影响特定目录的规则：更新最近的子目录 `AGENTS.md`。
- 项目地图条目保持 1–2 句话；包的详细文档放在其自身的 `AGENTS.md` 中。
- 指南更新应聚焦，并有代码事实支撑。

## 发布流程

- `main` 积累 changeset；`release/*` 维护 `ci: release packages` PR。合并对应的版本 PR，即授权在其合并提交的 CI 成功后发布。普通推送不会发布。
- npm 发布和原生构建产物保留现有 Changesets 流程。CDN 上传和路由仍需单独批准。迁移和 main 同步说明见 `.changeset/README.md`。

## 工作流要求

- 阅读代码时优先使用 `rg` / `rg --files`。
- 设计变更时，优先遵循现有边界和局部模式。
- 在公开文本和测试数据中，将真实内部标识替换为中性占位符，例如 `example.com`、`example.test` 和 `YOUR_API_KEY`。创建 PR 前，请一个只读智能体审查 diff，查找与具体上下文有关的内部标识。
- 创建 PR 时，使用 `write-pr` 技能（`.agents/skills/write-pr/SKILL.md`）撰写 PR 描述。PR 标题必须遵循 Conventional Commit 风格，例如 `chore: remove legacy format commands`。
- AI 智能体创建或更新 PR 时，填写 `.github/pull_request_template.md`：链接相关 issue 或解释问题，然后描述改动。不要保留占位文本，也不要只提交泛泛的 diff 总结。
- 不要提交含糊的 AI 生成 PR 文本。人类作者必须充分理解变更，能够解释代码、边界情况，以及该方案为何适合本仓库。
- 完成任务后、提交 PR 前，必须运行 `gen-changesets` 技能（见 `.agents/skills/gen-changesets/SKILL.md`），按其规则在 `.changeset/` 下生成 changeset。
- changeset 必须严格遵循 `.agents/skills/gen-changesets/SKILL.md`：用一句简短、面向用户的话，仅说明改变了什么；用户无法感知的变更不写。
- 生成 changeset 时，**绝不**自行决定 `major` 升级；先停下来解释，并获得用户明确确认。默认选择 `minor`，其次考虑 `patch`。参见 `.agents/skills/gen-changesets/SKILL.md`。
- 优先通过 `import ... from '#/...'` 导入，其作用与 `import ... from '@/...'` 相同。
- 不要提交一次性的草稿或探索文件。绝不暂存以下内容：
  - 智能体工作笔记或交接 / 总结文档（例如 `HANDOVER-*.md`、`HANDOFF-*.md`、`handoff.md`）。
  - 仓库根目录或 `design/` 目录下一次性的 UI/UX 原型或设计稿（例如 `*-designs.html`、`*-mockup.html`、`*-demo(s).html`）。唯一应被跟踪的 `.html` 文件是 Vite 的 `index.html` 入口文件。
  提交或创建 PR 前，运行 `git status` 和 `git diff --staged --stat`，移除所有符合上述模式的内容。临时工作放在被 gitignore 忽略的 `.tmp/` 中，而不是仓库根目录或源码树。

## 代码审查规则

这些规则适用于所有拉取请求审查，无论自动还是人工。规则所指的用户群体和契约文件列于 `.agents/skills/review-pr/surfaces.md`。

### 审查范围：PR 的 diff，而不是整个仓库

- 只审查 PR 相对于基线的 diff。PR 未触及、在基线中已存在的问题，应作为非阻断备注报告，不能作为阻断性发现；唯一例外是 PR 的行为直接触发或放大了该既有问题。
- 一次性报告同一类问题：如果某项发现适用于 diff 中多个位置，应一次列出全部位置，而不是分多轮陆续提出。
- 不报告纯风格或个人偏好问题。每项发现都必须有严重等级：P0 阻断 / P1 应修复 / P2 建议。
- 复审采用增量方式：只审查上次审查以来的变化；不重复未改动代码上的发现，不重新提出已解决的讨论。将已解决讨论视为真正解决前，验证当前代码：只有问题确已修复，或该发现已不再成立时，才不再报告。
- 各领域规则保留各自的权威来源：changeset（`.agents/skills/gen-changesets/SKILL.md`）、无注释区域（通用编码规则）、PR 文本（`write-pr` 技能）；按这些规则审查，不按临时偏好判断。

### 枚举行为变化，而不仅是缺陷

变更前可用的任何输入，包括配置键、环境变量、CLI 标志、提供商响应结构、旧版本写入的会话、客户端请求或钩子载荷，在变更后都必须保持相同行为，除非 PR 明确声明了变化。对于每个被删除或收窄的分支、条件、默认值或提示词句子，都要追问：以前谁会走到这里，现在会走向哪里？对于每个新条件，都要追问：哪些现有输入现在会优先匹配它？如果 PR 将某条路径称为“未改变”，但其分支条件已经移动，应指出这一问题。

### 更改默认值或删除行为，必须说明损失并提供退出途径

所有使用旧默认值的人都会受到影响。要求 changeset 写明用户会失去什么行为，不能只说明新默认值；提供配置、环境变量或标志作为恢复旧行为的途径，或由维护者在 PR 中明确认可；并添加测试，固定仍保留旧行为的用户群体所依赖的行为。

### 提示词文本就是行为

编辑或删除 `packages/agent-core-v2/src/**/*.md` 下的句子（系统提示词、工具描述、提醒、覆盖层、内置技能），都会改变收到这些提示词的所有用户的智能体行为。“没有测试引用这句话”不能证明没有影响。要求 PR 说明：哪些群体会收到这段文本（每个会话、计划模式、Tower 等由标志控制的功能），该句子约束了什么，谁依赖它，以及现在由什么来维持该约束。

### 契约文件是变更警戒线

`packages/agent-core-v2/docs/` 下的清单（`config-manifest.toml`、`wire-manifest.d.ts`、`state-manifest.d.ts`）、`packages/kap-server/test/__snapshots__/apiSurface.snapshot.test.ts.snap`、`packages/node-sdk/src/index.ts`、`packages/agent-core-v2/src/features/externalHooks/`、`packages/acp-server/` 和 `apps/kimi-code/src/cli/` 会被仓库外部使用：包括 code-app 中的桌面和 Web 应用、VS Code 扩展、Zed 等 ACP 客户端、SDK 用户、钩子脚本以及无头输出解析器。它们变化时，要求 PR 写明使用方，并说明如何让上一版本的数据和客户端继续工作。

### 移植和重构必须带上旧路径的功能清单

变更替换或绕过现有路径时，要求列出旧路径完成的事项，包括遵循的环境变量、回退机制和接受的输入，并说明每项在新路径中的对应位置。悄悄丢失任何一项都属于回归，而不是清理。

### 静默失败比崩溃更严重

如果变更让产品悄悄忽略配置、悄悄批准或跳过操作，或悄悄丢弃数据，应列为最严重的问题，因为用户得不到可供报告的信号。
