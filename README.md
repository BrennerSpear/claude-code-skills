# Agent Skills

Universal skills for AI coding agents. Platform-agnostic — works with Claude Code, OpenClaw, and any agent that follows the [AgentSkills](https://agentskills.io) spec.

## Skills

| Skill | Description |
|-------|-------------|
| `agent-browser` | Headless browser automation for scraping and testing |
| `check-research` | Poll and save async research results (Parallel AI) |
| `create-mcp` | Create MCP servers from API docs |
| `diff-review` | Review diffs with context |
| `execute-brain-dump-tasks` | Execute tasks from processed brain dumps |
| `merge-upstream` | Merge upstream changes safely |
| `modal` | Deploy to Modal.com |
| `new-project` | Scaffold new projects |
| `process-brain-dump` | Extract tasks from voice/text brain dumps |
| `supabase` | Supabase helpers |
| `ui-scaffold` | Initialize shadcn/ui with base-nova theme |

## Usage

Each agent platform symlinks to this folder:
- OpenClaw: `~/.openclaw/skills` → here
- Claude Code: `~/.claude/skills` → here

## Adding a Skill

```bash
mkdir ~/.agents/skills/my-skill
cat > ~/.agents/skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: What this skill does
---

# My Skill

Instructions for the agent...
EOF
```

## Publishing

```bash
# To ClawHub
clawhub publish ~/.agents/skills/my-skill

# Commit to GitHub
cd ~/.agents/skills
git add my-skill
git commit -m "Add my-skill"
git push
```
