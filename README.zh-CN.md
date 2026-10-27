# 通用 AGENTS.md

[English](README.md) | 简体中文

一套综合六份参考文档整理的、可复用的 coding agent 工程指南。以通用性和全面性优先，并明确各项流程的适用条件。

英文指南见 [AGENTS.md](AGENTS.md)，对应中文版见 [AGENTS.zh-CN.md](AGENTS.zh-CN.md)。指南覆盖实现、调试、重构、审查、文档及相关工程工作，帮助 agent 澄清会影响结果的重要歧义、保护已有工作和行为、遵守授权范围，并以证据支持完成结论。

## 指南覆盖什么

指南包含 12 个部分：

| 章节 | 内容 |
| --- | --- |
| 1–2 | 任务范围、授权、上下文调查与计划 |
| 3–4 | 设计、实现、改动范围、契约与迁移 |
| 5–7 | 工作区保护、版本控制、依赖、工具、安全与隐私 |
| 8–9 | 验证、失败诊断、自查与代码审查 |
| 10–11 | 并行协作、恢复、沟通与完成条件 |
| 12 | 项目指令及其权威来源的维护 |

简单任务可以直接执行并做针对性检查。更复杂或影响更大的工作需要明确计划、更充分的证据，以及必要的恢复安排。专项规则仅在对应场景出现时生效。

## 设计原则

1. **改动聚焦且完整。** 避免无关修改，同时覆盖必要的调用方、契约、测试和文档。
2. **以有效验证支持完成结论。** 检查必须能证明目标行为成立。
3. **尊重授权并保护已有工作。** 操作应在获准范围内，保留用户和协作者的已有改动。

## 使用方法

### 1. 选择语言

- 使用英文时，将根目录的 [AGENTS.md](AGENTS.md) 复制到目标项目的指令文件中。
- 使用中文时，采用 [AGENTS.zh-CN.md](AGENTS.zh-CN.md) 的内容。如果 agent 读取名为 AGENTS.md 的文件，就在目标项目中将中文内容保存为该文件名。

选择一种语言作为实际生效的指令文档，另一种可以作为阅读参考，无需重复加载。这些文件都是 Markdown 文档，无需安装软件包。

### 2. 与已有项目规则合并

复制前先阅读已有指令。如果目标项目已经有 AGENTS.md，应合并相关规则，而不是直接覆盖文件。保留项目实际的构建与测试要求、架构约束和授权政策，并明确处理重叠或冲突的规则。

项目具体要求由对应项目规则确定。补充项目专属约束，以及指向权威配置或流程文档的入口。某个目录或子系统的规则应放在对应局部指南中。

### 3. 配置 agent 读取指南

使用所选 agent 支持的指令加载方式。如果它识别项目根目录的 AGENTS.md，就将实际使用的指南放在那里；否则通过其支持的指令文件、配置或任务上下文提供。不要假定所有 agent 都会自动读取 AGENTS.md、AGENTS.zh-CN.md 或嵌套目录中的指南。

指南在平台的指令优先级、权限和安全边界内生效；添加这个文件本身不会授予权限。

### 4. 用实际任务检查效果

先用一个小任务检查 agent 的目标理解、改动范围和验证结果，再根据实际问题调整指南。

## 文档维护

修改通用指南时，同步更新两种语言版本，逐条比较规则的范围、强度、条件、例外和交叉引用。将指令修改视为行为变更。README 应与指南保持一致。

## 致谢（Acknowledgements）

本指南参考了以下仓库的 agent 指令：

- [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)
- [anomalyco/opencode](https://github.com/anomalyco/opencode)
- [earendil-works/pi](https://github.com/earendil-works/pi)
- [MoonshotAI/kimi-code](https://github.com/MoonshotAI/kimi-code)
- [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
- [openclaw/openclaw](https://github.com/openclaw/openclaw)
