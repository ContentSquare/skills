# Contentsquare skills

Use Agent skills to let your AI coding assistant (GitHub Copilot, Cursor, Claude Code, or any compatible tool) handle Contentsquare configuration like Web/Mobile setup, version migration, feature implementation, and much more.

> [!WARNING]
> This skill guides your AI coding agent through the Contentsquare configuration. The agent you use (Cursor, Claude Code, Copilot) is your own tool and operates under your settings and permissions. Contentsquare is not responsible for how your agent interprets or applies the instructions contained in these files.

## What You Can Do

**Install and verify the Contentsquare tag**: Let your agent handle the installation of the Contentsquare tag on your website, ensuring it's set up correctly to start collecting data.

```
Add contentsquare to my website using tag ID as `81c677ba742d7`
```

## Installation

### As a plugin (Claude Code & GitHub Copilot CLI)

This repository is also an [Agent Skills](https://agentskills.io) plugin marketplace, so you can install the Contentsquare tools as a first-class plugin.

**Claude Code:**

```shell
/plugin marketplace add ContentSquare/skills
/plugin install contentsquare-web@contentsquare
```

**GitHub Copilot CLI:**

```shell
copilot plugin marketplace add ContentSquare/skills
copilot plugin install contentsquare-web@contentsquare
```

Then, in natural language: "Install Contentsquare, tag ID `81c677ba742d7`" — the agent loads the skill automatically.

### Using skills package manager (skills.sh)

For agents supporting the open agent [skills.sh](https://skills.sh) ecosystem:

```bash
npx skills add contentsquare/skills
```

Works with Claude Code, Cursor, GitHub Copilot, and other compatible agents.

### Manual installation

1. Copy the skills inside this repository `./skills` folder and place them in the location your AI assistant uses for context discovery:

| Assistant | Recommended location |
|-----------|---------------------|
| GitHub Copilot | `.github/skills/` or `.agents/skills/` |
| Cursor | `.cursor/rules/` or project root |
| Claude Code | `.claude/skills/` or project root |
| Any (generic) | `.agents/skills/` |

You can verify if the skill is discoverable by prompting your AI agent "Which skills are available for you to use?".

## Available Skills

| Skill | Description |
|-------|-------------|
| `contentsquare/contentsquare-web-tag-install` | Install and verify the Contentsquare tracking tag in any web project |

## Repository layout (for maintainers)

The plugin marketplace reuses single authored sources; the rest are generated copies kept in sync by CI.

| Path | Role |
|------|------|
| `skills/contentsquare-web-tag-install/SKILL.md` | **Synced** from the wizard repo — do not hand-edit (see below) |
| `.claude-plugin/marketplace.json` | **Authored** marketplace catalog (read by Claude Code and Copilot) |
| `plugins/contentsquare-web/.claude-plugin/plugin.json` | **Authored** plugin manifest |
| `plugins/contentsquare-web/skills/contentsquare-web-tag-install/SKILL.md` | Generated copy of the skill — do not edit |

After editing an authored source, run `./scripts/sync-plugin.sh` and commit the result. CI fails if the generated copies drift.

### Where the tag-install skill comes from

The body of `contentsquare-web-tag-install` is owned by the private wizard repo at
`ContentSquare/wizard` → `skills/standalone-web-tag-install.md`. A workflow there opens a PR to this
repo whenever that source changes, updating both the skill and its plugin copy. **Do not hand-edit the
skill here** — edit the source in the wizard repo, or your change will be overwritten by the next sync.
