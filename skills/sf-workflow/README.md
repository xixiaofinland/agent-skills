# salesforce-development

A Claude Code agent skill for Salesforce development using the modern [sf CLI](https://developer.salesforce.com/tools/salesforcecli) (v2+).

Without this skill, code agents default to generic approaches — wrong CLI generation, wrong deploy strategy, missing safety guards. This skill encodes the correct Salesforce development workflow so agents behave correctly out of the box.

## What it covers

- **Deploy & retrieve** — single file, named component, tracked changes; correct handling of Apex and LWC companion files
- **Apex tests** — right flags, coverage workflow, safety guards against broad test runs
- **Create metadata** — Apex classes, triggers, Lightning Web Components, Aura components
- **Diff local vs org** — compare a local file against the org version before deploying
- **Retrieve org metadata** — browse and pull components that don't exist locally yet
- **Org management** — list orgs, set target org, open in browser
- **Safety rules** — no wildcards, no package.xml, no `RunAllTestsInOrg` without explicit instruction

## Installation

Each agent has a dedicated install guide it can follow autonomously. Clone this repo, then tell your agent:

> "Follow the instructions in the INSTALL.md file to install this skill."

The agent reads its own guide, runs the necessary commands, verifies the result, and reports back — no manual steps needed.

| Agent              | Install guide                                         |
| ------------------ | ----------------------------------------------------- |
| Claude Code        | [`.claude/INSTALL.md`](.claude/INSTALL.md)            |
| Codex              | [`.codex/INSTALL.md`](.codex/INSTALL.md)              |
| OpenCode           | [`.opencode/INSTALL.md`](.opencode/INSTALL.md)        |
| GitHub Copilot CLI | <!-- supported but not documented yet --> coming soon |

Restart your agent after installation. The skill activates automatically when you work in a Salesforce project (`sfdx-project.json` or `.forceignore` present).

## Requirements

- [Salesforce CLI v2+](https://developer.salesforce.com/tools/salesforcecli) (`sf` command)
- An authenticated org (`sf org login web`)

## Structure

```
├── SKILL.md              # Skill definition — loaded when the skill triggers
└── references/
    ├── deploy.md         # Deploy/retrieve patterns, source tracking, conflicts
    ├── testing.md        # Apex test execution, coverage, debug logs
    ├── metadata.md       # Create/delete/rename metadata, browsing org inventory
    └── org.md            # Org management, scratch orgs, sandboxes, auth
```

## Related

- [sf.nvim](https://github.com/xixiaofinland/sf.nvim) — Neovim plugin this skill's workflow is based on
