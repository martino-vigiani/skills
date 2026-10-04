# Skills

Four Claude Code plugins for code maintenance, performance work, Swift testing and writing. Each plugin can be installed separately. These are instruction files and agent definitions, rather than standalone applications.

## Choose a plugin

| Plugin | Contents | Use it for |
|---|---|---|
| [core-agents](plugins/core-agents/agents/) | `fixer`, `fixer-quick`, `explore`, `verifier` | Scoped edits, read-only code lookup and checking claims against source. |
| [optimize-app](plugins/optimize-app/skills/optimize-app/SKILL.md) | Performance workflow and iOS, web and backend references | Record a baseline, change one thing, then compare measured results. |
| [swift-quality](plugins/swift-quality/) | `swift-test-expert`, `e2e-tester` and SwiftUI reference notes | Swift unit and UI tests, regression checks and SDK-specific guidance. |
| [writing](plugins/writing/skills/humanize/SKILL.md) | `humanize` and its style rules | Rewrite prose while preserving its facts, language and register. |

Most agent and workflow instructions are written in Italian. The SwiftUI reference notes are in English. The marketplace currently lists version `0.1.0` for all four plugins.

## Install

In Claude Code, add the marketplace, then install the plugin you need:

```text
/plugin marketplace add martino-vigiani/skills
/plugin install core-agents@martino-vigiani
```

Replace `core-agents` with `optimize-app`, `swift-quality` or `writing` as needed. `/plugin marketplace` lets you inspect the collection before installing more plugins.

The plugins have no dependencies on each other. Their workflows still need the tools of the project being worked on, such as Xcode for Swift UI tests. Agent definitions include model settings, which your Claude Code installation must support.

## How the workflows behave

- The fixers require an explicit list of files they may edit. They leave changes uncommitted and report what was verified.
- The verifier tries to disprove a claim using the actual source and returns `CONFIRMED`, `REFUTED` or `PARTIAL` with file references.
- The performance workflow records its baseline before editing. Results must contain comparable measurements or say that they were not measured.
- The Swift test agents distinguish a code bug from a broken test. They check that tests executed and use controlled mutations to check whether new tests can detect a regression.
- The writing workflow delegates a style pass and checks the result for lost facts. Its `humanize` procedure is the version shipped in this repository.

Read the linked definitions for the full rules before using them. Instruction quality alone does not prove that a change, benchmark or test is correct.

## Repository layout

[marketplace.json](.claude-plugin/marketplace.json) registers the four plugins. Each directory under [plugins/](plugins/) contains its own `.claude-plugin/plugin.json`, agent definitions, skills or reference files.

To change a plugin, edit its definition and keep its marketplace entry and plugin manifest consistent. Check that relative reference paths still resolve. This repository has no application build or automated test command.

## Credits and license

The SwiftUI reference package identifies its guidance as Apple material; its attribution remains in the source files. These separate projects are useful references and are not bundled here:

- [impeccable](https://github.com/pbakaus/impeccable), by Paul Bakaus.
- [app-store-review](https://github.com/JustinPerea/app-store-review-skill), by Justin Perea.
- [make-interfaces-feel-better](https://github.com/jakubkrehel/make-interfaces-feel-better), by Jakub Krehel.
- [taste-skill](https://github.com/Leonxlnx/taste-skill), by Leonxlnx.

[MIT](LICENSE). Maintained by Martino Vigiani. Contact: martino@subralabs.com.
