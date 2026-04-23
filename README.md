# agent-skills

Personal agent skills for use with the [skills CLI](https://github.com/vercel-labs/skills).

## Skills

| Skill         | Description                                                                                                                                |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `plan-first`  | Structured planning workflow — analyzes project, asks clarifying questions, creates TODO.md, gets approval, then executes task by task     |
| `clean-apex`  | Clean-code guidance for Apex: naming, 3-tier architecture, error handling, unit vs integration testing                                     |
| `sf-workflow` | Salesforce development tasks using the modern `sf` CLI — deploy, retrieve, test, org management                                            |
| `book-tutor`  | Socratic tutor for any book or course — guided questioning, comprehension checks, chapter-by-chapter progress tracking, vault note capture |

## Usage

```bash
# Install all skills globally (Claude Code, Pi, OpenCode)
skills add -g xixiaofinland/agent-skills -a claude-code pi opencode -y

# Install a specific skill
skills add -g xixiaofinland/agent-skills --skill plan-first -a claude-code pi opencode -y

# List installed global skills
skills ls -g

# Update all global skills to latest
skills update -g
```

## Adding a New Skill

1. Create `skills/<name>/SKILL.md` following the [Agent Skills standard](https://agentskills.io)
2. Push to `main`
3. Run `skills update -g` to pull the update locally
