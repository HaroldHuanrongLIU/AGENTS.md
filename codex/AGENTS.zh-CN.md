# Rust/codex-rs

在存放 Rust 代码的 codex-rs 目录中：

- crate 名称以 `codex-` 为前缀。例如，`core` 目录中的 crate 名为 `codex-core`。
- 使用 format! 时，只要可以将变量直接内嵌到 {} 中，就始终这样做。
- 执行本文说明前，如果仓库依赖的命令尚未安装（例如 `just`、`rg` 或 `cargo-insta`），先安装它们。
- 绝不添加或修改与 `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR` 或 `CODEX_SANDBOX_ENV_VAR` 有关的代码。
  - 你在沙箱中运行，每次使用 `shell` 工具时，都会设置 `CODEX_SANDBOX_NETWORK_DISABLED=1`。现有使用 `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR` 的代码已经考虑到这一点。它通常用于提前退出那些作者知道受沙箱限制而无法运行的测试。
  - 同样，使用 Seatbelt（`/usr/bin/sandbox-exec`）启动进程时，会为子进程设置 `CODEX_SANDBOX=seatbelt`。需要自行运行 Seatbelt 的集成测试无法在 Seatbelt 内运行，因此也常通过检查 `CODEX_SANDBOX=seatbelt`，在适当情况下提前退出测试。
- 始终按照 https://rust-lang.github.io/rust-clippy/master/index.html#collapsible_if 合并可折叠的 if 语句。
- 只要可行，始终按照 https://rust-lang.github.io/rust-clippy/master/index.html#uninlined_format_args 内嵌 format! 参数。
- 尽可能按照 https://rust-lang.github.io/rust-clippy/master/index.html#redundant_closure_for_method_calls 使用方法引用代替闭包。
- 避免使用 bool 或含义模糊的 `Option` 参数，迫使调用方写出 `foo(false)` 或 `bar(None)` 这类难读的代码。优先使用枚举、具名方法、newtype 或其他符合 Rust 惯例的 API 形式，使调用位置能够自行说明含义。
- 如果无法调整 API，且仍需在 Rust 中保留简短的位置字面量调用，遵循 `argument_comment_lint` 约定：
  - 通过位置传递 `None`、布尔值、数字字面量等含义不直观的参数时，在前面添加严格采用 `/*param_name*/` 格式的注释。
  - 当方法只有一个非 self 参数，且方法名与参数名一致时，可豁免，例如 `fn enabled(&self, enabled: bool)` 对应的 `.enabled(false)`。
  - 不要为字符串或字符字面量添加这类注释，除非确实能提升清晰度；lint 有意豁免了这些字面量。
  - 注释中的参数名必须与被调用函数的签名完全一致。
  - 可运行 `just argument-comment-lint` 在本地执行检查。该检查由 Bazel 驱动，若 Bazel 尚未预热，首次运行可能较慢，但增量运行应少于 15 秒。多数情况下，最好更新 PR 后由 CI 负责检查（或在提交 PR 后于后台异步运行）。注意，CI 会检查全部三个平台，本地运行不会。
- 尽可能让 `match` 语句穷尽所有情况，避免通配分支。
- 新增 trait 应包含文档注释，说明其职责，以及预期如何实现和使用它们。
- 不建议在 Rust trait 中使用 `#[async_trait]` 或 `#[allow(async_fn_in_trait)]`。
  - 优先采用原生 RPITIT trait 方法，并对返回的 future 显式声明 `Send` 约束，参见 `3c7f013f9735` / `#16630`。
  - 推荐的 trait 形式：
    `fn foo(&self, ...) -> impl std::future::Future<Output = T> + Send;`
  - 实现满足该契约时，仍可使用 `async fn foo(&self, ...) -> T`。
  - 不要用 `#[allow(async_fn_in_trait)]` 代替显式声明 future 契约。
- 编写测试时，优先比较整个对象是否相等，而不是逐字段比较。
- 不要为静态定义的值添加测试。
- 不要为已删除的逻辑添加负向测试。
- 不要向 `docs/` 目录添加通用产品文档或面向用户的文档。Codex 官方文档存放在其他位置。app-server API 文档是例外，见下方 app-server 指南。
- 优先使用私有模块，并显式导出 crate 的公共 API。
- 如果修改了 `ConfigToml` 或嵌套配置类型，运行 `just write-config-schema` 更新 `codex-rs/core/config.schema.json`。
- 处理 MCP 工具调用时，优先通过 `codex-rs/codex-mcp/src/mcp_connection_manager.rs` 处理工具及工具调用的修改。尽量缩小改动范围，利用现有抽象，避免让代码穿过多层函数调用。
- 不要无必要地调用 `reset_client_session`；让增量检查逻辑决定是否复用上一请求。
- 如果修改 Rust 依赖（`Cargo.toml` 或 `Cargo.lock`），从仓库根目录运行 `just bazel-lock-update`，刷新 `MODULE.bazel.lock`，并在同一次变更中包含该锁文件更新。CI 会检查锁文件是否存在偏差。
- Bazel 不会自动让 Rust 在编译期访问源码树中的文件。如果添加 `include_str!`、`include_bytes!`、`sqlx::migrate!` 或类似的构建期文件或目录读取操作，请更新该 crate 的 `BUILD.bazel`（`compile_data`、`build_script_data` 或测试数据）；否则即使 Cargo 通过，Bazel 仍可能失败。
- 不要创建仅被引用一次的小型辅助方法。
- 跟踪异步任务时，在函数或方法定义上添加 `#[tracing::instrument(...)]`，而不是在调用位置通过 `.instrument(...)` 为 future 附加 span。添加插桩前，检查被调用函数，或它直接委托的实现方法，是否已经插桩。
- 避免大型模块：
  - 优先新增模块，而不是继续扩充现有模块。
  - Rust 模块的目标规模为不超过 500 行代码，不含测试。
  - 如果文件超过约 800 行，除非有充分且已记录的理由，否则应在新模块中添加功能，而不是继续扩充该文件。
  - 这条规则尤其适用于经常修改、已经容易聚集无关改动的文件，例如 `codex-rs/tui/src/app.rs`、`codex-rs/tui/src/bottom_pane/chat_composer.rs`、`codex-rs/tui/src/bottom_pane/footer.rs`、`codex-rs/tui/src/chatwidget.rs`、`codex-rs/tui/src/bottom_pane/mod.rs`，以及类似的核心编排模块。
  - 从大模块中提取代码时，将相关测试及模块 / 类型文档一并移到新实现附近，使不变量与负责维护它们的代码保持接近。
  - 除非改动很小，否则避免向 `codex-rs/tui/src/chatwidget.rs` 添加新的独立方法；优先新增模块 / 文件，让 `chatwidget.rs` 专注于编排。
- 运行 Rust 命令（例如 `just fix` 或 `just test`）时要耐心等待，绝不要尝试按 PID 杀死它们。Rust 锁可能导致执行缓慢，这是正常现象。

完成本仓库任意位置的代码修改后，自动在 `codex-rs` 目录运行 `just fmt`，无需请求批准。另外，执行测试：

1. 不要直接运行 `cargo test`。使用 `just test`，使测试遵循仓库默认设置。
2. 运行被修改项目的测试。例如，修改了 `codex-rs/tui`，就运行 `just test -p codex-tui`。
3. 上述测试通过后，如果修改了 common、core 或 protocol，则使用 `just test` 运行完整测试套件。日常本地运行避免使用 `--all-features`，因为它会扩大构建矩阵，并显著增加 `target/` 的磁盘占用；仅在确实需要完整特性覆盖时使用。项目专属测试或单个测试可直接运行，但运行完整测试套件前必须询问用户。

完成较大的 `codex-rs` 变更前，在 `codex-rs` 目录运行 `just fix -p <project>`，修复代码中的 lint 问题。优先通过 `-p` 限定范围，避免缓慢的全工作区 Clippy 构建；只有修改共享 crate 时才运行不带 `-p` 的 `just fix`。运行 `fix` 或 `fmt` 后，不要重复运行测试。

## `codex-core` crate

随着时间推移，`codex-core` crate（位于 `codex-rs/core/`）变得越来越臃肿：它是最大的 crate，因此向 `codex-core` 添加新内容，往往比提取并重构所需库代码更容易。后者本可让新代码既不依赖 `codex-core`，也不继续扩大 `codex-core` 的规模。

因此：**克制向 codex-core 添加代码的冲动！**

尤其在引入新概念、功能或 API 时，向 `codex-core` 添加代码前，应考虑：

- 是否有其他现存 crate 更适合容纳这些新代码。
- 是否应在 Cargo 工作区新增 crate 来承载新功能。必要时重构现有代码以实现这一点。

同样，在代码审查中，应明确反对那些无必要地向 `codex-core` 添加代码的 PR。

## 代码审查规则

### Crate API 暴露范围

尽量缩小 crate 的 API 暴露范围。避免不断增加仅供测试使用的辅助函数。

### 模型可见上下文

Codex 维护一份上下文（消息历史），在推理请求中发送给模型。

1. 不重写历史，上下文必须以增量方式构建。
2. 避免频繁修改上下文，导致缓存未命中。
3. 不允许无界条目，注入模型上下文的所有内容都必须有明确的大小边界和硬性上限。
4. 任何条目不得超过 10K token。
5. 将新增的、单条可能超过 1k token 的内容标记为 P0。这些内容需要额外的人工审查。
6. 所有注入片段必须在 `core/context` 中定义为结构体，并实现 ContextualUserFragment trait。

### 破坏性变更

检查外部集成接口中的破坏性变更：

- app-server API
- 原始响应项事件（`rawResponseItem/*`），即使它们仍处于实验阶段
- CLI 参数
- 配置加载
- 从现有 rollout 记录恢复会话

### 测试编写指南

对于智能体变更，优先编写集成测试，而非单元测试。集成测试位于 `core/suite`，通过 `test_codex` 设置 codex 测试实例。

改变智能体逻辑的功能**必须**添加集成测试：

- 列出需要测试的主要逻辑变化和用户可见行为。

如果需要单元测试，将其放在专用测试文件（\*\_tests.rs）中。
避免在主实现中加入仅供测试使用的函数。

检查是否已有辅助工具能使测试更精简、更易读。

### 变更规模指南（800 行）

除非是机械性修改，否则总改动行数不应超过 800 行。
复杂逻辑变更应控制在 500 行以内。

如果变更更大，应考虑是否可以拆成便于审查的阶段，并找出最小、完整且可先行合入的阶段。
阶段划分建议应基于实际 diff、依赖关系和受影响的调用位置。

## TUI 样式约定

参见 `codex-rs/tui/styles.md`。

## TUI 代码约定

- 使用 ratatui 的 Stylize trait 提供的简洁样式辅助方法。
  - 普通 span：使用 "text".into()。
  - 带样式的 span：使用 "text".red()、"text".green()、"text".magenta()、"text".dim() 等。
  - 优先使用这些方式，而不是直接通过 `Span::styled` 和 `Style` 构造样式。
  - 示例：补丁摘要中的文件行。
    - 推荐：vec!["  └ ".into(), "M".red(), " ".dim(), "tui/src/app.rs".dim()]

### TUI 样式（ratatui）

- 优先使用 Stylize 辅助方法：尽可能用 "text".dim()、.bold()、.cyan()、.italic()、.underlined()，而不是手动构造 Style。
- 优先使用简单转换：span 使用 "text".into()，行使用 vec![…].into()；类型推断存在歧义时（例如 Paragraph::new/Cell::from），使用 Line::from(spans) 或 Span::from(text)。
- 计算得到的样式：如果 Style 在运行时计算，可以使用 `Span::styled`（也可使用 `Span::from(text).set_style(style)`）。
- 避免硬编码白色：不要使用 `.white()`；优先使用默认前景色（不设置颜色）。
- 链式调用：通过链式调用组合辅助方法以提升可读性（例如 url.cyan().underlined()）。
- 单个元素：优先使用 "text".into()；仅当上下文无法明确目标类型，或 .into() 会要求额外类型标注时，才使用 Line::from(text) 或 Span::from(text)。
- 构建行：目标类型明确且无需额外标注时，用 vec![…].into() 构建 Line；否则使用 Line::from(vec![…])。
- 避免无意义改写：没有明确的可读性或功能收益时，不要在等价形式之间重构（Span::styled ↔ set_style、Line::from ↔ .into()）；遵循当前文件的约定，不要仅为满足 .into() 而引入类型标注。
- 紧凑性：优先选择经 rustfmt 格式化后仍能保持单行的形式；如果 Line::from(vec![…]) 和 vec![…].into() 中只有一种不换行，就选它；如果都换行，就选择换行更少的形式。

### 文本换行

- 普通字符串始终使用 textwrap::wrap 换行。
- 对 ratatui Line 换行时，使用 tui/src/wrapping.rs 中的辅助函数，例如 word_wrap_lines / word_wrap_line。
- 需要缩进换行后的内容时，尽可能使用 RtOptions 的 initial_indent / subsequent_indent 选项，而不是自行编写逻辑。
- 如果有一组行需要统一添加前缀（首行与后续行的前缀可不同），使用 line_utils 中的 `prefix_lines` 辅助函数。

## 测试

### 测试模块组织

- 新增测试模块时，将内容定义在同级独立文件中，不要内联到实现文件。
- 使用显式的 `#[path = "..._tests.rs"]` 属性，让测试文件名清楚描述用途且易于查找：

  ```rust
  #[cfg(test)]
  #[path = "parser_tests.rs"]
  mod tests;
  ```

- 这仅适用于新增测试模块。不要仅为了遵循此约定，就移动或重写现有内联的 `#[cfg(test)] mod tests { ... }` 模块。

### 快照测试

本仓库通过 `insta` 使用快照测试，尤其在 `codex-rs/tui` 中，用于验证渲染输出。

**要求：**凡是影响用户可见 UI 的变更（包括新增 UI），都必须有相应的 `insta` 快照覆盖：尚无测试时新增快照测试，已有测试时更新快照。将快照更新的审查和接受纳入 PR 流程，使 UI 影响便于审查，后续 diff 也能保持直观。

有意改变 UI 或文本输出时，按以下步骤更新快照：

- 运行测试，生成更新后的快照：
  - `just test -p codex-tui`
- 检查待处理项：
  - `cargo insta pending-snapshots -p codex-tui`
- 直接阅读仓库中生成的 `*.snap.new` 文件以审查变更，或预览特定文件：
  - `cargo insta show -p codex-tui path/to/file.snap.new`
- 只有确定要接受该 crate 中所有新快照时，才运行：
  - `cargo insta accept -p codex-tui`

如果尚未安装该工具：

- `cargo install --locked cargo-insta`

### 基准测试

可通过 `just bench` 运行 cargo 基准测试；编写新的基准测试时使用 divan crate。

使用 `just bench-smoke` 以单次迭代试运行基准测试，确认它能正常工作。

### 测试断言

- 测试应使用 pretty_assertions::assert_eq，以获得更清晰的差异。如果尚未导入，在测试模块顶部导入它。
- 尽可能使用深度相等比较。对整个对象执行 `assert_eq!()`，而不是逐字段断言。
- 避免在测试中修改进程环境；优先从上层传入由环境派生的标志或依赖。

### 在测试中启动工作区二进制文件（Cargo 与 Bazel）

- 测试需要启动第一方二进制文件时，优先使用 `codex_utils_cargo_bin::cargo_bin("...")`，而不是 `assert_cmd::Command::cargo_bin(...)` 或 `escargot`。
  - 在 Bazel 下，二进制文件和资源可能位于 runfiles 中；使用 `codex_utils_cargo_bin::cargo_bin` 解析绝对路径，确保 `chdir` 后路径仍然有效。
- 在 Bazel 下定位 fixture 文件或测试资源时，避免使用 `env!("CARGO_MANIFEST_DIR")`。优先使用 `codex_utils_cargo_bin::find_resource!`，使路径在 Cargo 和 Bazel runfiles 环境中都能正确解析。

### 集成测试

#### codex_core 集成测试

- 编写端到端 Codex 测试时，优先使用 `core_test_support::responses` 中的工具。
- 默认使用 `TestCodexBuilder::build_with_auto_env()`，确保新测试能适用于 app 与 exec 使用不同操作系统的情况。详情见 $remote-tests。
- 所有 `mount_sse*` 辅助函数都返回 `ResponseMock`；保留它，以便对发出的 `/responses` POST 请求体进行断言。
- 当测试只应发出一个 POST 时，使用 `ResponseMock::single_request()`；要检查捕获的每个 `ResponsesRequest`，使用 `ResponseMock::requests()`。
- `ResponsesRequest` 提供辅助方法（`body_json`、`input`、`function_call_output`、`custom_tool_call_output`、`call_output`、`header`、`path`、`query_param`），使断言可以针对结构化载荷，而无需手动遍历 JSON。
- 使用提供的 `ev_*` 构造函数和 `sse(...)` 构建 SSE 载荷。
- 优先使用 `wait_for_event`，而不是 `wait_for_event_with_timeout`。
- 优先使用 `mount_sse_once`，而不是 `mount_sse_once_match` 或 `mount_sse_sequence`。

- 典型模式：

  ```rust
  let mock = responses::mount_sse_once(&server, responses::sse(vec![
      responses::ev_response_created("resp-1"),
      responses::ev_function_call(call_id, "shell", &serde_json::to_string(&args)?),
      responses::ev_completed("resp-1"),
  ])).await;

  codex.submit(Op::UserTurn { ... }).await?;

  // Assert request body if needed.
  let request = mock.single_request();
  // assert using request.function_call_output(call_id) or request.json_body() or other helpers.
  ```

#### app-server 集成测试

- 测试应覆盖 app-server 的公共 JSON-RPC API。
- 使用与 core 集成测试相似的服务器模拟方式。
- 默认使用 `TestAppServer::builder().build()` 和 `TestAppServer::send_thread_start_request_with_auto_env()`，确保新测试能适用于 app 与 exec 使用不同操作系统的情况。详情见 `$remote-tests`。

## App-server API 开发最佳实践

以下指南适用于 `codex-rs` 中的 app-server 协议开发，尤其是：

- `app-server-protocol/src/protocol/common.rs`
- `app-server-protocol/src/protocol/v2.rs`

### 核心规则

- 所有当前 API 开发都应在 app-server v2 中进行。不要向 v1 添加新的 API。
- 统一遵循载荷命名方式：请求载荷使用 `*Params`，响应使用 `*Response`，通知使用 `*Notification`。
- RPC 方法以 `<resource>/<method>` 形式暴露，`<resource>` 保持单数（例如 `thread/read`、`app/list`）。
- 始终通过 `#[serde(rename_all = "camelCase")]`，在传输格式中将字段暴露为 camelCase，除非带标签的联合类型或明确的兼容性要求需要针对性重命名。
- 始终通过一致的 serde 和 TS `rename_all = "camelCase"` 注解，在传输格式中将字符串枚举值暴露为 camelCase，除非明确的兼容性要求需要针对性重命名。
- 例外：配置 RPC 载荷应使用 snake_case，以对应 config.toml 键名（见 `app-server-protocol/src/protocol/v2.rs` 中的配置读取 / 写入 / 列表 API）。
- 始终在 v2 请求 / 响应 / 通知类型上设置 `#[ts(export_to = "v2/")]`，使生成的 TypeScript 位于正确的命名空间。
- 绝不对 v2 API 载荷字段使用 `#[serde(skip_serializing_if = "Option::is_none")]`。
  例外：有意不带参数的客户端到服务器请求可以使用：
  `params: #[ts(type = "undefined")] #[serde(skip_serializing_if = "Option::is_none")] Option<()>`。
- 保持 Rust 与 TS 的传输字段重命名一致。如果字段或变体使用 `#[serde(rename = "...")]`，则添加对应的 `#[ts(rename = "...")]`。
- 对可辨识联合类型，在两个序列化器中都使用显式标签：`#[serde(tag = "type", ...)]` 和 `#[ts(tag = "type", ...)]`。
- API 边界优先使用普通 `String` ID（必要时在内部进行 UUID 解析 / 转换）。
- 时间戳应为整数 Unix 秒（`i64`），命名为 `*_at`（例如 `created_at`、`updated_at`、`resets_at`）。
- 对于实验性 API：使用 `#[experimental("method/or/field")]`；需要字段级控制时派生 `ExperimentalApi`；当某方法仅有部分字段属于实验性功能时，在 `common.rs` 中使用 `inspect_params: true`。

### 客户端到服务器的请求载荷（`*Params`）

- 每个可选字段都必须标注 `#[ts(optional = nullable)]`。不要在客户端到服务器请求载荷（`*Params`）之外使用 `#[ts(optional = nullable)]`。
- 可选集合字段（例如 `Vec`、`HashMap`）必须使用 `Option<...>` 加 `#[ts(optional = nullable)]`。不要使用 `#[serde(default)]` 表示可选集合，也不要在 v2 载荷字段上使用 `skip_serializing_if`。
- 如果希望布尔字段被省略时表示 `false`，使用 `#[serde(default, skip_serializing_if = "std::ops::Not::not")] pub field: bool`，而不是 `Option<bool>`。
- 新增列表方法时，默认实现基于游标的分页：请求字段为 `pub cursor: Option<String>` 和 `pub limit: Option<u32>`；响应字段为 `pub data: Vec<...>` 和 `pub next_cursor: Option<String>`。

### 开发流程

- API 结构变化时，重新生成 schema fixture：
  `just write-app-server-schema`
  （如果涉及实验性 API fixture，还需运行 `just write-app-server-schema --experimental`）。
- 使用 `just test -p codex-app-server-protocol` 验证。
- 避免编写仅对 `common.rs` 中单个请求字段的实验性标记进行断言的样板测试；应依靠 schema 生成 / 测试与行为覆盖。

## Python 开发最佳实践

### 无需兼容 Python 2

本项目使用 Python 3+。不要使用 `__future__` 模块。

如果需要考虑不同 3.xx 小版本之间的特性兼容性，检查最近的 `pyproject.toml` 中的 `requires-python` 字段，了解支持的最低运行时版本。

## 平台支持

除非功能明确针对特定操作系统，否则测试和功能必须支持 Linux、macOS 和 Windows。

Codex 支持相互连接的 app-server 与 exec-server 运行在不同操作系统上。有关这些配置的集成测试，参见 `$remote-tests` 技能。
