- 修改公共 Protocol 或 Server `HttpApi` 后，在 `packages/client` 中运行 `bun run generate`。不要直接编辑生成的客户端文件。
- 运行时依赖方向保持为 Schema → Core 和 Protocol，再由 Core 和 Protocol → Server。Client 运行时代码可以依赖 Schema 和 Protocol，但绝不能依赖 Core 或 Server；`sdk` 负责组合 Client、Core 和 Server。
- 本仓库不使用 Changesets。不要添加 `.changeset` 文件，应遵循现有发布流程。
- 默认分支为 `dev`。除非另有说明，否则分支和工作树都基于 `origin/dev`，PR 目标为 `dev`，diff 也与 `origin/dev` 比较。

## 连接实时服务的 TUI 测试

- 从开发工作树运行 `bun run dev:live`，针对当前选定的 `opencode` 后台服务器和实时会话测试 TUI。
- 必要时在脚本后传入目录，例如 `bun run dev:live /path/to/project`。
- 脚本通过 `opencode service status` 发现服务器，从 `opencode service get password` 注入其私有本地凭据，并使用 `dev` TUI 存储通道，使标签页和其他客户端本地状态与已安装的客户端一致。
- 此工作流优先使用 `dev:live`，而不是普通的 `bun run dev`。工作树客户端版本不同时，隐式的托管服务连接可能替换正在使用的服务器；显式 `--server` 会发出警告并继续运行，不替换服务器。

## TUI Story 示例

- 用户要求 TUI story 时，在 `packages/tui/src/feature-plugins/system/storybook` 下添加由 fixture 驱动的 story，并在 `index.tsx` 中注册。
- 渲染真实的生产组件，而不是外观仿制品。提交和其他副作用应限制在 story 内部，以便安全地反复探索。
- 通过 story 快捷键暴露有意义的状态维度，并在 `StoryFooter` 中列出；如果状态组合可能让 fixture 难以理解，应提供重置命令。
- 从开发工作树使用 `OPENCODE_STORY=<story-id> bun run dev:live` 运行指定 story；涉及布局时，测试窄屏和宽屏终端尺寸。

## TUI 主题 Token

- 按语义角色选择主题 token，而不是按当前颜色选择。不要使用原始 `theme.hue` 值，也不要借用语义无关的 token 来实现偏好的外观。
- `text.feedback` 和 `background.feedback` 仅用于结果或状态反馈，例如错误、警告、成功消息和信息提示。表单控件文本、序号和选择标记使用 `formfield` 状态，操作使用 `action` 状态。
- 如果主题没有提供所需语义角色的 token，先为该角色扩展主题 schema、默认值、解析逻辑和类型，再在组件中使用。不要挪用外观最接近的现有 token。
- 修改公开的主题 token 接口时，除受影响的 TUI 组件外，还要验证内置明亮 / 深色默认主题和自定义主题的回退路径。

## 分支名称

使用简短的分支名称，最多三个单词，以连字符分隔。不要使用斜杠，也不要使用 `feat/` 或 `fix/` 等类型前缀。

例如：`session-recovery`、`fix-scroll-state`、`regenerate-sdk`。

## 提交消息和 PR 标题

采用 conventional commit 风格的提交消息和 PR 标题：`type(scope): summary`。

有效类型为 `feat`、`fix`、`docs`、`chore`、`refactor` 和 `test`。scope 可选；必要时使用受影响的包或区域，例如 `core`、`opencode`、`tui`、`app`、`desktop`、`sdk` 或 `plugin`。

例如：`fix(tui): simplify thinking toggle styling`、`docs: update contributing guide`、`chore(sdk): regenerate types`。

## 风格指南

### 通用原则

- 除非可组合或可复用，否则将逻辑保留在一个函数内。
- 在负责未知值的边界处只校验一次。向内部传递有类型的值，而不是重复执行 `typeof value === "object"` 和属性存在性检查。不要对 schema、构造函数或内部类型已经保证的值进行防御性重复校验。
- 不要预先提取仅使用一次的辅助函数。应在调用位置内联逻辑，除非该函数会被复用、隐藏了确实复杂的边界，或具有清晰独立的名称，能改善调用方的可读性。
- 为推测性或概率极低的竞态、安全边界情况增加复杂度前，先向用户解释具体失败方式、发生概率和复杂度成本，并取得认可。不要为理论上的健壮性悄悄扩大范围。
- 尽可能避免 `try`/`catch`。
- 避免使用 `any` 类型。
- 尽可能使用 Bun API，例如 `Bun.file()`。
- 尽可能依赖类型推断；除非导出或清晰度需要，否则避免显式类型标注或接口。
- 优先使用函数式数组方法（flatMap、filter、map），而不是 for 循环；在 filter 中使用类型守卫，以保持后续类型推断。
- 在 `src/config` 中新增配置模块时，遵循现有的文件顶部自导出模式，例如 `export * as ConfigAgent from "./agent.js"`。
- 在 Effect 生成器中，先将服务绑定到具名变量，再调用方法。不要使用 `yield* (yield* Foo.Service).bar()` 这类嵌套服务 yield。

只使用一次的值应内联，以减少变量总数。

```ts
// Good
const journal = await Bun.file(path.join(dir, "journal.json")).json()

// Bad
const journalPath = path.join(dir, "journal.json")
const journal = await Bun.file(journalPath).json()
```

### 解构

避免不必要的解构。使用点记法保留上下文。

```ts
// Good
obj.a
obj.b

// Bad
const { a, b } = obj
```

### 导入

- 绝不为导入设置别名。不要使用 `import { foo as bar } from "..."`，也不要使用 `resolve as pathResolve` 之类的重命名导入。
- 绝不在类型位置使用 `import("...")` 引用，例如 `Schema.declare<import("@opencode/plugin/effect/plugin").Plugin["effect"]>`。只有两个导入确实发生名称冲突，且没有其他办法时，才允许将带别名的类型导入（`import type { Plugin as PluginDefinition } from "..."`）作为最后手段；仍应尽量避免。
- 绝不使用星号导入。不要使用 `import * as Foo from "..."` 或 `import type * as Foo from "..."`。
- 如果需要命名空间式的值，按名称导入模块自身导出的命名空间，例如 `import { Project } from "@opencode/core/project"`，然后引用 `Project.ID`。
- 仅在部分代码路径中需要的重型模块，应优先动态导入，尤其是在对启动速度敏感的入口中。在需要它的最小作用域顶部附近解构动态导入绑定，使其读起来像普通导入。避免内联链式调用，例如 `await import("./module").then((mod) => mod.value())` 或 `(await import("./module")).value()`。分支专用导入应保留在对应分支中，以维持延迟加载。

### 变量

优先使用 `const` 而不是 `let`。使用三元表达式或提前返回，避免重新赋值。

```ts
// Good
const foo = condition ? 1 : 2

// Bad
let foo
if (condition) foo = 1
else foo = 2
```

### 控制流

避免 `else` 语句，优先提前返回。

```ts
// Good
function foo() {
  if (condition) return 1
  return 2
}

// Bad
function foo() {
  if (condition) return 1
  else return 2
}
```

### 复杂逻辑

函数有多个校验分支或辅助细节时，让主函数呈现正常成功路径，将辅助细节放入其下方的小型辅助函数。

```ts
// Good
export function loadThing(input: unknown) {
  const config = requireConfig(input)
  const metadata = readMetadata(input)
  return createThing({ config, metadata })
}

function requireConfig(input: unknown) {
  ...
}
```

- 辅助函数放在其所支持代码附近；有助于可读性时，放在主导出函数下方。
- 不要把简单表达式过度抽象成大量一次性辅助函数；只有能命名真实概念（例如 `requireConfig` 或 `readMetadata`）时才提取。
- 除非辅助函数确实执行带副作用的工作，否则不要返回 `Effect`。同步解析、校验和选项构建应保持同步。
- 解析不可信 JSON 字符串时，优先使用 Effect schema 辅助方法，例如 `Schema.UnknownFromJsonString` 和 `Schema.decodeUnknownOption`，而不是在 `Effect.try` 中手动包装 `JSON.parse`。
- 为不明显的约束和出人意料的行为添加注释，不要为显而易见的赋值或控制流添加注释。

### Schema 定义（Drizzle）

字段名使用 snake_case，这样无需再用字符串重新定义列名。

```ts
// Good
const table = sqliteTable("session", {
  id: text().primaryKey(),
  project_id: text().notNull(),
  created_at: integer().notNull(),
})

// Bad
const table = sqliteTable("session", {
  id: text("id").primaryKey(),
  projectID: text("project_id").notNull(),
  createdAt: integer("created_at").notNull(),
})
```

## 测试

- 尽可能避免 mock；除非别无选择，否则完全不应使用 globalThis.\*。
- 测试实际实现，不要在测试中重复实现逻辑。
- 测试不能从仓库根目录运行（由 `do-not-run-tests-from-root` 防护）；应从 `packages/core` 等包目录运行。
- 编写或修改测试前，阅读 `.opencode/skills` 中对应的技能：`unit-tests`（bun test）、`tui-tests`（OpenTUI 渲染器测试）、`app-e2e-tests`（`packages/app/e2e` 中的 Playwright）或 `component-tests`（Storybook Playwright）。它们记录了导致 CI 测试缓慢或不稳定的原因。

## 检查

- 从仓库根目录运行 `bun run check`，作为标准的完整 lint 和类型检查验证。
- 聚焦迭代时，从受影响的包目录（例如 `packages/core`）运行 `bun typecheck`。绝不直接运行 `tsc`。
- 本地 `typecheck` 脚本使用 `bun check`。CI 和包发布运行 `typecheck:tsc`，它使用官方 TypeScript 编译器（`@typescript/native`）。两者不一致时，以 `tsc` 为准；从包目录运行 `bun run typecheck:tsc` 复现。
- `bun check` 旨在匹配 TypeScript 7.1，而不是 `process.versions.typescript` 报告的 `7.0.2`（oven-sh/bun#46536）。当 `bun check` 与固定版本 `@typescript/native` 中的 `tsc` 报出不同错误时，在 https://github.com/oven-sh/bun/issues 提交无依赖的最小复现，附上两者输出。`bun check` 不生成声明文件，因此 TS5088 或 TS2883 等声明生成错误只出现在 `tsc` 中属于预期行为，不要报告这类差异。

## 会话核心

- 持久化事件保持最小：记录无法进一步约简的新事实，不重复可通过按序归并聚合历史推导出的状态。如果使用方需要自包含的视图，则在投影和读取模型中补充先前状态或派生状态。
- 将提示词的持久化接纳与模型执行分离。`Session.prompt(...)` 发布 `session.inbox.enqueued`，其投影插入一行持久化 `session_inbox` 记录，然后调度作为唤醒提示的 `SessionExecution.wake(sessionID)`；如果 `resume: false` 请求仅接纳，则不唤醒。投递时发布 `session.inbox.delivered`；其投影在同一事务中消费收件箱行并插入可见消息。`session_inbox` 只存储尚未消费的工作。
- 复用 Session ID 即采用已有 Session。当 Session 和类型一致时，复用用户或合成收件箱条目的 ID 是幂等的：首次接纳生效，重试的载荷、元数据和投递模式都会被忽略，无论条目仍待处理还是已投递（已投递时从投影消息中核对，无需保留入队历史）。跨 Session 或跨类型复用会失败。控制条目保留其操作特有的冲突行为。
- `SessionExecution` 保持进程全局，并以 Session ID 为基础。其本地实现负责进程内的 Session 协调器，仅在开始处理队列时，通过 `SessionStore` 和 `LocationServiceMap.get(session.location)` 确定运行位置；任何层都不应接收 Session ID。中断针对该 Session 当前活跃的进程内所有权链；对已知但空闲、或本地不拥有的 Session 执行中断是空操作，而公共 API 会拒绝未知 Session。
- `SessionRunner`、模型解析、工具注册表、权限和文件系统都应限定在 Location 作用域内。省略 `Location.workspaceID` 表示隐式本地运行位置；显式工作区身份保留给未来的运行位置语义。
- 每个 step 都是一次明确的 `llm.stream` 调用，从重新加载的历史继续。不要添加内存中的工具循环。
- 实现集群前，本地 Session 队列处理保持在进程内。`SessionRunCoordinator` 合并同一 Session 的显式恢复请求，合并提示词唤醒，并允许不同 Session 并发运行。预写的执行认领记录标记一个进程内忙碌时段，用于重启恢复：最终完成、失败或用户中断会释放它；关机中断和进程死亡则保留它。启动恢复会恢复已认领的顶层 Session，并对每次执行尝试进行持久化计数。该认领记录只是恢复标记，不代表集群所有权、隔离令牌机制或恰好执行一次的保证。
- 保持投递术语明确。提示词默认采用 steer（引导当前执行）。在安全的 step 边界，steer 模式的压缩请求优先，但不越过第一个 steer 模式的移动控制项；其他 steer 请求保持入队顺序。在空闲边界，steer 优先；其他情况下，每次恰好投递一个排队条目，然后运行器重新评估是否继续。收件箱条目在投递前可取消，或在 queue 与 steer 模式间切换。将新用户输入提升为当前输入时，会重置选中智能体的 step 配额；一批 steer 只重置一次。
- 一个 step 对应一次逻辑 LLM 调用；其持久化记录仅覆盖模型可见的范围。不要写“provider turn”，也不要单独用“turn”表示一次调用：“turn”保留给未来的 assistant-turn 单位，包含从提示词被提升为当前输入，到会话即将进入空闲状态之间的所有 step。
- 将事件重放的所有权与集群化 Session 执行的所有权分开。
