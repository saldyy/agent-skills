---
description: Review the skills, commands, and plugin manifests under .claude/ and .claude-plugin/ in this repo for structural and content problems — broken SKILL.md links, missing/incomplete frontmatter, dangling symlinks, stale or unverifiable version claims, duplicated guidance across rule files, and invalid plugin/marketplace JSON. Use when the user asks to review, audit, lint, or check the skills/commands in this repository, or after adding/editing a skill.
allowed-tools: Read, Grep, Glob
---

# Review .claude/

Review the content that Claude Code actually loads from this repo: skills, commands, and the plugin/marketplace manifests that expose them. This is a content review, not a general code review — there's no application code here, so "bugs" mean structural breaks, stale claims, and drift between related files.

## Scope

- `.claude/commands/*.md` — project commands (including this file)
- `.claude/skills/*` — this directory should not exist (see Structure checks below); if it does, treat any entry as a finding
- `skills/<name>/SKILL.md` and `skills/<name>/rules/*.md`
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`

## Checks

**Frontmatter**
- Every `SKILL.md` and command file has `name` and `description`.
- `description` names concrete trigger phrases/keywords a user would actually type — flag generic descriptions ("helpful utilities for X") that won't match real requests.
- `metadata` (if present) is a map, not a scalar or list.

**Structure**
- Every file linked from a `SKILL.md`'s rules list actually exists at that path.
- Every file under `rules/*.md` is linked from its `SKILL.md` — flag orphaned rule files nothing points to.
- `.claude-plugin/plugin.json` declares `"skills": "./skills"`, which auto-discovers every `skills/<name>/` directory — no `.claude/skills/<name>` symlink is needed for a skill to load, and `.claude/skills/` should not exist at all per `CLAUDE.md`. Flag any symlink or file found under `.claude/skills/` as a finding to remove.

**Content accuracy**
- Version-specific claims (language/runtime versions, flags, API availability) are internally consistent within a file and across sibling rule files covering the same topic.
- Code samples use syntax/APIs consistent with the versions the skill claims to target.
- Flag claims that read as unverified or suspiciously specific (exact version numbers, benchmark figures) with no apparent basis in the repo.

**Duplication**
- Flag near-duplicate guidance repeated across multiple rule files instead of stated once and cross-linked — costs context on every load.

**Plugin/marketplace manifests**
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` are valid JSON.
- `plugin.json`'s `name` and the plugin entry `name` in `marketplace.json` match.
- Marketplace `name` isn't on Claude Code's reserved list (`agent-skills`, `claude-code-marketplace`, etc.) — it must differ from any reserved name even when the plugin name itself matches one.
- `source` paths in `marketplace.json` resolve to real directories relative to the marketplace root.

## Output

Report findings with the `ReportFindings` tool, most severe first. Each finding needs the file, a one-sentence summary of the defect, and a concrete failure scenario (what a future agent or user hits because of it) — not just "this could be cleaner." If nothing survives scrutiny, report an empty list rather than padding it with style nitpicks.
