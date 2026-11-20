---
description: Review the skills, commands, and manifests in this repo for OpenCode and Claude Code — broken SKILL.md links, missing/incomplete frontmatter, dangling symlinks, stale version claims, duplicated guidance, and invalid opencode.json or .claude-plugin JSON. Use when the user asks to review, audit, lint, or check the skills/commands in this repository, or after adding/editing a skill.
---

# Review the skills repo

Review this repository using the checklist in `.claude/commands/review-claude.md` — it is the single source of truth for both runtimes and already covers `opencode.json`, the `AGENTS.md` link, OpenCode skill discovery, and the Claude Code plugin/marketplace manifests.

Read that file first, then apply its **Scope**, **Checks**, and **Output** sections. OpenCode does not read `.claude/commands/`, which is why this pointer exists; keep it in sync if the canonical command moves.

Report findings in chat, most severe first, with the file, a one-sentence defect summary, and a concrete failure scenario.
