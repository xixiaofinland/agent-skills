# Claude Code Installation

Run the following steps in order:

## 1. Install the skill

Create the plugin skills directory if it doesn't exist:

```bash
mkdir -p ~/.claude/plugins/sf/skills
```

Clone this repo into it:

```bash
git clone https://github.com/xixiaofinland/sf-workflow \
  ~/.claude/plugins/sf/skills/salesforce-development
```

Verify the skill file is in place:

```bash
ls ~/.claude/plugins/sf/skills/salesforce-development/SKILL.md
```

## Done

Inform the user the skill is installed and they should restart Claude Code for it to take effect.

## Updating

To update to the latest version:

```bash
cd ~/.claude/plugins/sf/skills/salesforce-development && git pull
```

Inform the user the skill has been updated.
