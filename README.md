# General AGENTS.md

English | [简体中文](README.zh-CN.md)

A concise, reusable set of engineering and research guidelines for coding agents, synthesized from public agent instructions and practical workflows. Sized (~12–13 KB) to stay comfortably within per-file rule injection caps (such as 24 KB limits) while maintaining full coverage across daily engineering and AI/ML research.

Use [AGENTS.md](AGENTS.md) as the English guide or read the equivalent [Chinese version](AGENTS.zh-CN.md). The guide covers implementation, debugging, refactoring, reviews, ML experiments, and documentation—helping agents clarify consequential ambiguity, protect existing work and experimental provenance, stay within authorized scope, and back completion claims with evidence.

## What the guide covers

The guide contains 13 sections:

| Sections | Coverage |
| --- | --- |
| 1–2 | Task scope, authorization, context investigation, and planning |
| 3–4 | Design, implementation, proportionate defensive programming, change scope, contracts, and migrations |
| 5–7 | Workspace protection, version control, dependencies, tools, security, and privacy |
| 8–9 | Verification, failure diagnosis, self-review, and code review |
| 10–11 | Parallel collaboration, recovery, communication, and completion |
| 12 | Maintaining project instructions and preventing duplicate rule injection |
| 13 | Research & ML guardrails (prototyping vs. core code, silent tensor/leakage bugs, smoke tests, read-only provenance, and zero-fabrication metrics/citations) |

Simple tasks proceed directly with focused checks. Multi-step or high-impact work uses explicit plans, stronger behavioral or experimental evidence, and recovery state where needed. Specialized rules apply only when their scenarios arise.

## Design principles

1. **Make focused, complete changes.** Avoid unrelated edits while covering necessary callers, contracts, tests, and documentation.
2. **Support completion with meaningful verification.** Checks must demonstrate that intended behavior works—and in ML workflows, run end-to-end smoke tests before costly runs and verify against silent tensor, numerical, or leakage bugs.
3. **Respect authorization and protect existing assets.** Keep actions within approved scope, preserve collaborator changes, and treat raw datasets and finished experiment outputs as read-only.
4. **Reduce unjustified defensive code.** Base protective mechanisms on current contracts or credible, reachable failure paths; never swallow errors or fake success with silent fallbacks.
5. **Keep implementation direct and scientifically honest.** Reuse suitable APIs, preserve exact types and semantics, isolate reproducible baselines via configs, and never fabricate metrics or citations.

## How to use it

### 1. Choose a language and loading scope (avoid duplicate loading)

- **Global setup (recommended for personal daily use):** Point your agent's global instruction file (e.g., `~/.gemini/GEMINI.md`, `~/.codex/AGENTS.md`, or `~/.claude/CLAUDE.md`) to [AGENTS.md](AGENTS.md) (or [AGENTS.zh-CN.md](AGENTS.zh-CN.md))—preferably via a symlink (`ln -s /path/to/AGENTS.md/AGENTS.md ~/.gemini/GEMINI.md`) so updates propagate automatically and path-based deduplication works.
- **Avoid double injection:** When this guide is already loaded globally, **do not copy the full guide into target repositories' `AGENTS.md` files**. Keep repository-level `AGENTS.md` files strictly for project-specific build/test commands, architecture constraints, and local overrides.
- **Standalone repository setup:** Copy the full [AGENTS.md](AGENTS.md) or [AGENTS.zh-CN.md](AGENTS.zh-CN.md) into a project root only when sharing a self-contained repo with collaborators or running agents without global rules configured. Use one language as the active rule file rather than loading both.

### 2. Merge with existing project rules

Read any existing instructions before adding rules. Preserve the target project's actual build and test commands, architectural constraints, and authorization policies, resolving overlaps or conflicts explicitly. Place directory- or workflow-specific rules in local guides or on-demand skills.

### 3. Configure the agent to read the guide

Use the instruction-loading mechanism supported by your agent. Do not assume every agent automatically reads `AGENTS.md`, `AGENTS.zh-CN.md`, or nested guides. The guide operates within the platform's instruction hierarchy, permissions, and safety boundaries; adding the file does not grant permissions.

### 4. Check it against real tasks

Verify with a small engineering or experimental task first—checking goal comprehension, edit scope, and verification rigor—then refine local overrides based on observed failure modes.

## Maintaining the documents

When changing the general guide, update both language versions with 1-to-1 section and bullet parity across scope, strength, conditions, exceptions, and cross-references, and keep file sizes well below 24 KB. Treat instruction changes as behavior changes and keep the READMEs aligned.

## Acknowledgements

This guide draws on agent instructions and skills in the following repositories:

- [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)
- [anomalyco/opencode — General Principles](https://github.com/anomalyco/opencode/blob/dev/AGENTS.md#general-principles)
- [earendil-works/pi](https://github.com/earendil-works/pi)
- [MoonshotAI/kimi-code](https://github.com/MoonshotAI/kimi-code)
- [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
- [openclaw/openclaw](https://github.com/openclaw/openclaw)
- [wanshuiyin/HERO-Anti-OverDefense — scope rules](https://github.com/wanshuiyin/HERO-Anti-OverDefense/blob/main/RULES.md)
- [6Kmfi6HP/avoid-overkill — engineering rules](https://github.com/6Kmfi6HP/avoid-overkill/blob/main/skills/avoid-overkill/SKILL.md)
- [klarkxy/no-stop-gpt — proportional complexity and protective guarantees](https://github.com/klarkxy/no-stop-gpt/blob/main/skills/no-stop-gpt/SKILL.md)
- [lmy375/oh-no-slop — Python rules](https://github.com/lmy375/oh-no-slop/blob/main/rules/cn/py.md)
