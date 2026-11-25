# 通用 AGENTS.md

[English](README.md) | 简体中文

一套精炼、可复用的 coding 与科研 agent 工作指南，结合公开 agent 指令与工程/科研实践整理而成。单文件体积控制在 ~10–13 KB，远低于主流平台 ~24 KB 的单文件规则截断线，同时完整覆盖日常软件工程与 AI/ML 科研场景。

英文指南见 [AGENTS.md](AGENTS.md)，对应中文版见 [AGENTS.zh-CN.md](AGENTS.zh-CN.md)。指南覆盖实现、调试、重构、审查、机器学习实验与文档工作，帮助 agent 澄清影响结果的关键歧义、保护既有代码与实验资产、严守授权边界，并以有效证据支持完成结论。

## 指南覆盖什么

指南包含 13 个部分：

| 章节 | 内容 |
| --- | --- |
| 1–2 | 任务范围、授权、上下文调查与计划 |
| 3–4 | 设计、实现、适度防御性编程、改动范围、契约与迁移 |
| 5–7 | 工作区保护、版本控制、依赖、工具、安全与隐私 |
| 8–9 | 验证、失败诊断、自查与代码审查 |
| 10–11 | 并行协作、恢复、沟通与完成条件 |
| 12 | 项目指令维护与防止全局/局部重复注入 |
| 13 | 科研与机器学习守则（快速原型 vs. 严谨工程、张量与防静默泄露检查、全链路 smoke test、数据与实验只读溯源、指标与引用零编造） |

简单任务可直接执行并做针对性检查；多步骤或高影响任务需给出明确计划、行为或实验验证证据，以及必要的恢复安排。专项规则仅在对应场景出现时生效。

## 设计原则

1. **改动聚焦且完整。** 避免无关修改，同时覆盖必要的调用方、契约、测试和文档。
2. **以有效验证支持完成结论。** 检查须证明目标行为成立；在机器学习任务中，长训或高成本评测前先做全链路 smoke test，并重点排查张量维度、数值异常与数据泄露。
3. **尊重授权并保护已有资产。** 操作限于获批范围，保留用户与协作者改动，将原始数据集与已完成实验结果视为只读资产。
4. **减少无依据的防御代码。** 防护机制须基于当前契约或可信可达的失败路径，严禁吞异常或静默回退返回伪成功。
5. **保持实现直接与科研诚实。** 复用适用 API、保持精确类型与求值语义，通过配置隔离可复现实验分支，绝不编造实验指标或文献引用。

## 使用方法

### 1. 选择语言与加载层级（避免重复加载）

- **全局配置（个人日常推荐）：** 将所选 agent 的全局指令文件（如 `~/.gemini/GEMINI.md`、`~/.codex/AGENTS.md` 或 `~/.claude/CLAUDE.md`）指向本仓库的 [AGENTS.md](AGENTS.md) 或 [AGENTS.zh-CN.md](AGENTS.zh-CN.md)——优先使用软链接（如 `ln -s /path/to/AGENTS.md/AGENTS.md ~/.gemini/GEMINI.md`），既能自动同步更新，也能让按真实路径去重的平台避免重复注入。
- **避免全局与项目根目录重复加载：** 当本指南已作为全局规则生效时，**不要再把完整指南复制粘贴到具体项目的 `AGENTS.md` 中**；各代码仓或论文仓根目录的 `AGENTS.md` 仅写项目专属的构建/测试命令、目录规范和局部覆盖规则。
- **独立仓库分发：** 仅在与未配置全局规则的协作者共享独立仓库时，将一种语言版本完整放入目标项目根目录，无需同时加载中英文两份。

### 2. 与已有项目规则合并

合并前先阅读目标项目既有指令，保留项目真实的构建与测试要求、架构约束和授权策略，显式消解重叠或冲突规则。目录或专项流程的细则应放在局部指南或按需触发的 Skill 中。

### 3. 配置 agent 读取指南

使用所选 agent 支持的指令加载机制，不要假定所有工具都会自动读取 `AGENTS.md`、`AGENTS.zh-CN.md` 或嵌套目录指南。指南在平台的指令优先级、权限与安全边界内生效；添加文件本身不会授予额外权限。

### 4. 用实际任务检查效果

先用一个小型工程或实验任务检验 agent 的目标理解、改动范围和验证质量，再针对实际暴露的问题微调局部规则。

## 文档维护

修改通用指南时，同步更新中英文两个版本，逐条对齐各章节与条目的范围、强度、条件、例外和交叉引用，并将单文件体积严格控制在 24 KB 上限以内。将指令修改视为行为变更，保持 README 与指南一致。

## 致谢（Acknowledgements）

本指南参考了以下仓库的 agent 指令与技能：

- [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)
- [anomalyco/opencode — 通用原则](https://github.com/anomalyco/opencode/blob/dev/AGENTS.md#general-principles)
- [earendil-works/pi](https://github.com/earendil-works/pi)
- [MoonshotAI/kimi-code](https://github.com/MoonshotAI/kimi-code)
- [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
- [openclaw/openclaw](https://github.com/openclaw/openclaw)
- [wanshuiyin/HERO-Anti-OverDefense — 范围约束](https://github.com/wanshuiyin/HERO-Anti-OverDefense/blob/main/RULES.md)
- [6Kmfi6HP/avoid-overkill — 工程规则](https://github.com/6Kmfi6HP/avoid-overkill/blob/main/skills/avoid-overkill/SKILL.md)
- [klarkxy/no-stop-gpt — 适度复杂度与必要保障](https://github.com/klarkxy/no-stop-gpt/blob/main/skills/no-stop-gpt/SKILL.md)
- [lmy375/oh-no-slop — Python 规则](https://github.com/lmy375/oh-no-slop/blob/main/rules/cn/py.md)
