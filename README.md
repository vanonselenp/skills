# skills

Personal collection of [Agent Skills](https://www.anthropic.com/news/agent-skills), used interchangeably by **Claude Code** and **OpenCode**.

This repo is the single source of truth. Each harness consumes skills via symlinks pointing back here, so edits are live in both.

## Layout

```
skills/
├── _template/          # Copy this when adding a new skill
│   └── SKILL.md
└── <skill-name>/
    ├── SKILL.md        # Required: frontmatter + body
    └── ...             # Optional: scripts, references, assets
```

Each skill is a directory containing a `SKILL.md` with this frontmatter:

```markdown
---
name: skill-name
description: "What it does. When to use. Triggers on: phrase one, phrase two."
---
```

The `description` is what the harness uses to decide whether to invoke the skill, so be specific about *what it does* and *when to use it*.

## Adding a skill

1. `cp -r _template my-skill`
2. Edit `my-skill/SKILL.md` — update `name`, `description`, and body.
3. Symlink into each harness (see below).
4. Commit.

## Symlinking into each harness

From the repo root, for a skill named `my-skill`:

```sh
# Claude Code (user-level)
ln -s "$PWD/my-skill" ~/.claude/skills/my-skill

# OpenCode (verify path on your version — recent OpenCode supports SKILL.md
# under ~/.config/opencode/skill/ or similar; check `opencode --help`)
ln -s "$PWD/my-skill" ~/.config/opencode/skill/my-skill
```

To verify Claude Code picked it up: start a session in any directory and the skill name should appear in the available-skills list.

## Conventions

- One concern per skill. If a skill description has two unrelated trigger phrases, it's probably two skills.
- Keep `SKILL.md` short; put long references, prompts, or scripts in sibling files and link to them from the body.
- Skill names use kebab-case and match the directory name.
