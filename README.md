# Contentsquare skills

Use Agent skills to let your AI coding assistant (GitHub Copilot, Cursor, Claude Code, or any compatible tool) handle Contentsquare configuration like Web/Mobile setup, version migration, feature implementation, and much more.

> [!WARNING]
> This skill guides your AI coding agent through the Contentsquare configuration. The agent you use (Cursor, Claude Code, Copilot) is your own tool and operates under your settings and permissions. Contentsquare is not responsible for how your agent interprets or applies the instructions contained in these files.

## What You Can Do

**Install and verify the Contentsquare tag**: Let your agent handle the installation of the Contentsquare tag on your website, ensuring it's set up correctly to start collecting data.

```
Add contentsquare to my website using tag ID as `81c677ba742d8`
```

## Installation

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
