# General AGENTS.md

English | [简体中文](README.zh-CN.md)

A reusable set of engineering guidelines for coding agents, synthesized from six reference documents. It prioritizes portability and comprehensive coverage, with explicit conditions for applying each workflow.

Use [AGENTS.md](AGENTS.md) as the English guide or read the equivalent [Chinese version](AGENTS.zh-CN.md). The guide covers implementation, debugging, refactoring, reviews, documentation, and related engineering work. It helps agents clarify consequential ambiguity, preserve existing work and behavior, stay within authorized scope, and support completion claims with evidence.

## What the guide covers

The guide contains 12 sections:

| Sections | Coverage |
| --- | --- |
| 1–2 | Task scope, authorization, context investigation, and planning |
| 3–4 | Design, implementation, change scope, contracts, and migrations |
| 5–7 | Workspace protection, version control, dependencies, tools, security, and privacy |
| 8–9 | Verification, failure diagnosis, self-review, and code review |
| 10–11 | Parallel collaboration, recovery, communication, and completion |
| 12 | Maintaining project instructions and their authoritative sources |

Simple tasks can proceed with focused checks. More complex or consequential work calls for explicit plans, stronger evidence, and recovery arrangements where needed. Specialized rules apply only when their scenarios arise.

## Six reference sources

The synthesis is based on the local reference copies linked below. These are snapshots for comparison; upstream documents may have changed since collection. Each reference directory includes an English document and a Chinese translation.

| Reference | Local documents | Upstream | Main contributions |
| --- | --- | --- | --- |
| Karpathy-style guidelines | [English](karpathy/AGENTS.md) · [中文](karpathy/AGENTS.zh-CN.md) | Original upstream URL not recorded | Explicit assumptions, focused changes, avoiding unnecessary complexity, and verifiable goals |
| OpenCode | [English](opencode/AGENTS.md) · [中文](opencode/AGENTS.zh-CN.md) | [anomalyco/opencode](https://github.com/anomalyco/opencode) | Validation at the owning boundary, meaningful abstractions, generated outputs, and tests of real implementations |
| Pi | [English](pi/AGENTS.md) · [中文](pi/AGENTS.zh-CN.md) | [earendil-works/pi](https://github.com/earendil-works/pi) | Shared-workspace protection, precise staging, dependency and installation safety, and concrete communication |
| Kimi Code | [English](kimi-code/AGENTS.md) · [中文](kimi-code/AGENTS.zh-CN.md) | [MoonshotAI/kimi-code](https://github.com/MoonshotAI/kimi-code) | Behavior inventories, consumer and compatibility checks, prompts as behavior, and scoped reviews |
| Hermes | [English](hermes/AGENTS.md) · [中文](hermes/AGENTS.zh-CN.md) | [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) | Checking original design intent, behavior-contract tests, and routing to local instructions |
| OpenClaw | [English](openclaw/AGENTS.md) · [中文](openclaw/AGENTS.zh-CN.md) | [openclaw/openclaw](https://github.com/openclaw/openclaw) | Authorized execution and recovery, clear ownership, complete migrations, effective verification, and stopping conditions |

“Karpathy-style” identifies the local reference in this collection; its original upstream URL is not recorded here. Earlier references under [useless/](useless/) are excluded from the current six-source synthesis.

## Design principles

1. **Portability and coverage first.** Cover the engineering lifecycle without requiring a particular language, framework, operating system, agent, or release system. Preserve necessary conditions and exceptions instead of optimizing for an arbitrary length limit.
2. **Scale the process to the task.** Apply planning, verification, review, and recovery requirements according to complexity, impact, and risk. Routine reversible choices can proceed independently; consequential unresolved ambiguity requires clarification.
3. **Make the smallest complete change.** Include affected callers, contracts, tests, and documentation while leaving unrelated work alone. Judge abstractions by responsibility and usefulness, and compatibility by actual commitments.
4. **Verify behavior with relevant evidence.** Ask whether a check could pass while the original problem remains. Distinguish implementation defects, incorrect test expectations, environment failures, and intentional contract changes. Reuse valid evidence and stop when the task and required checks are complete.
5. **Respect authorization and ownership.** Preserve user and collaborator changes, recognize valid existing authorization, and keep new side effects within their approved scope. Clarify uncertain external outcomes before retrying.
6. **Resolve differences through applicability.** The references disagree on imports, comments, dependency versions, rebasing, compatibility, and testing. The general guide expresses decision criteria and defers project-specific choices to the relevant project rules.
7. **Keep rules maintainable.** Give each rule an authoritative home, place local constraints near their scope, and use clear triggers for specialized workflows. Keep both language versions aligned in meaning.

Project-specific paths, versions, commands, style preferences, architectural mechanisms, and release policies remain in the reference material rather than becoming universal requirements.

## How to use it

### 1. Choose a language

- For English, copy the root [AGENTS.md](AGENTS.md) into your project's instruction file.
- For Chinese, use the contents of [AGENTS.zh-CN.md](AGENTS.zh-CN.md). If your agent reads a file named AGENTS.md, save the Chinese content under that name in the target project.

Use one language as the active instruction document. The other version can remain a reading reference; it does not need to be loaded a second time. These are Markdown documents with no package to install.

### 2. Merge with existing project rules

Read any existing instructions before copying. If the target project already has an AGENTS.md, merge the relevant guidance rather than overwriting it. Preserve its actual build and test requirements, architectural constraints, and authorization policies; resolve overlapping or conflicting rules explicitly.

Add project-specific constraints and pointers to authoritative configuration or workflow documentation. Place rules for a particular directory or subsystem in the appropriate local guide. The six reference directories are study material, not instruction sets to activate together in a target project.

### 3. Configure the agent to read the guide

Use the instruction-loading mechanism supported by your agent. If it recognizes a project-root AGENTS.md, place the active guide there. Otherwise, provide the guide through its supported instruction file, configuration, or task context. Do not assume that every agent automatically reads AGENTS.md, AGENTS.zh-CN.md, or nested guides.

The guide operates within the platform's instruction hierarchy, permissions, and safety boundaries; adding the file does not grant permissions.

### 4. Check it against real tasks

Start with a small task and check whether the agent identifies the intended result, respects existing work, makes a complete focused change, and reports meaningful verification. For larger work, also check contract preservation, coordination, recovery, and completion criteria. Refine rules in response to concrete failures or recurring ambiguity.

## Repository layout

| Path | Purpose |
| --- | --- |
| [AGENTS.md](AGENTS.md) | General guide in English |
| [AGENTS.zh-CN.md](AGENTS.zh-CN.md) | Equivalent Chinese guide |
| [README.md](README.md), [README.zh-CN.md](README.zh-CN.md) | Project introduction and usage in both languages |
| Six reference directories listed above | Source snapshots and their Chinese translations |
| [useless/](useless/) | Earlier references excluded from the current synthesis |

## Maintaining the documents

When changing the general guide, update both language versions and compare each rule's scope, strength, conditions, exceptions, and cross-references. Treat instruction changes as behavior changes. Keep the READMEs aligned with the guide and the actual source collection, and keep reference snapshots separate from edits to the synthesized guide.
