# General AGENTS.md

English | [简体中文](README.zh-CN.md)

A reusable set of engineering guidelines for coding agents, drawing on public agent instructions and engineering practices. It prioritizes portability and comprehensive coverage, with explicit conditions for applying each workflow.

Use [AGENTS.md](AGENTS.md) as the English guide or read the equivalent [Chinese version](AGENTS.zh-CN.md). The guide covers implementation, debugging, refactoring, reviews, documentation, and related engineering work. It helps agents clarify consequential ambiguity, preserve existing work and behavior, stay within authorized scope, and support completion claims with evidence.

## What the guide covers

The guide contains 12 sections:

| Sections | Coverage |
| --- | --- |
| 1–2 | Task scope, authorization, context investigation, and planning |
| 3–4 | Design, implementation, proportionate defensive programming, change scope, contracts, and migrations |
| 5–7 | Workspace protection, version control, dependencies, tools, security, and privacy |
| 8–9 | Verification, failure diagnosis, self-review, and code review |
| 10–11 | Parallel collaboration, recovery, communication, and completion |
| 12 | Maintaining project instructions and their authoritative sources |

Simple tasks can proceed with focused checks. More complex or consequential work calls for explicit plans, stronger evidence, and recovery arrangements where needed. Specialized rules apply only when their scenarios arise.

## Design principles

1. **Make focused, complete changes.** Avoid unrelated edits while covering necessary callers, contracts, tests, and documentation.
2. **Support completion with meaningful verification.** Checks must demonstrate that the intended behavior works.
3. **Respect authorization and protect existing work.** Keep actions within the approved scope and preserve user and collaborator changes.
4. **Reduce unjustified defensive code.** Base protective mechanisms on current contracts or credible, reachable failure paths; preserve required security, data integrity, recovery, compatibility, and verification.

## How to use it

### 1. Choose a language

- For English, copy the root [AGENTS.md](AGENTS.md) into your project's instruction file.
- For Chinese, use the contents of [AGENTS.zh-CN.md](AGENTS.zh-CN.md). If your agent reads a file named AGENTS.md, save the Chinese content under that name in the target project.

Use one language as the active instruction document. The other version can remain a reading reference; it does not need to be loaded a second time. These are Markdown documents with no package to install.

### 2. Merge with existing project rules

Read any existing instructions before copying. If the target project already has an AGENTS.md, merge the relevant guidance rather than overwriting it. Preserve its actual build and test requirements, architectural constraints, and authorization policies; resolve overlapping or conflicting rules explicitly.

Project-specific requirements follow the project's own rules. Add project-specific constraints and pointers to authoritative configuration or workflow documentation. Place rules for a particular directory or subsystem in the appropriate local guide.

### 3. Configure the agent to read the guide

Use the instruction-loading mechanism supported by your agent. If it recognizes a project-root AGENTS.md, place the active guide there. Otherwise, provide the guide through its supported instruction file, configuration, or task context. Do not assume that every agent automatically reads AGENTS.md, AGENTS.zh-CN.md, or nested guides.

The guide operates within the platform's instruction hierarchy, permissions, and safety boundaries; adding the file does not grant permissions.

### 4. Check it against real tasks

Start with a small task to check the agent's understanding of the goal, scope of changes, and verification results, then adjust the guide based on actual problems.

## Maintaining the documents

When changing the general guide, update both language versions and compare each rule's scope, strength, conditions, exceptions, and cross-references. Treat instruction changes as behavior changes. Keep the READMEs aligned with the guide.

## Acknowledgements

This guide draws on agent instructions and skills in the following repositories:

- [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)
- [anomalyco/opencode](https://github.com/anomalyco/opencode)
- [earendil-works/pi](https://github.com/earendil-works/pi)
- [MoonshotAI/kimi-code](https://github.com/MoonshotAI/kimi-code)
- [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
- [openclaw/openclaw](https://github.com/openclaw/openclaw)
- [wanshuiyin/HERO-Anti-OverDefense — scope rules](https://github.com/wanshuiyin/HERO-Anti-OverDefense/blob/main/RULES.md)
- [6Kmfi6HP/avoid-overkill — engineering rules](https://github.com/6Kmfi6HP/avoid-overkill/blob/main/skills/avoid-overkill/SKILL.md)
- [klarkxy/no-stop-gpt — proportional complexity and protective guarantees](https://github.com/klarkxy/no-stop-gpt/blob/main/skills/no-stop-gpt/SKILL.md)
- [lmy375/oh-no-slop — Python rules](https://github.com/lmy375/oh-no-slop/blob/main/rules/cn/py.md)
