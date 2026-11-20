---
description: Review the skills, commands, and plugin/marketplace manifests in this repo for structural and content problems — broken SKILL.md links, missing/incomplete frontmatter, dangling symlinks, stale or unverifiable version claims, duplicated guidance across rule files, and invalid opencode.json or .claude-plugin JSON. Use when the user asks to review, audit, lint, or check the skills/commands in this repository, or after adding/editing a skill.
allowed-tools: Read, Grep, Glob
---

# Review the skills repo

Review the content that Claude Code actually loads from this repo: skills, commands, and the plugin/marketplace manifests that expose them. Also check the OpenCode config so the two runtimes don't drift apart. This is a content review, not a general code review — there's no application code here, so "bugs" mean structural breaks, stale claims, and drift between related files.

## Scope

- `.claude/commands/*.md` — project commands (including this file)
- `.opencode/commands/*.md` — OpenCode command equivalents
- `.claude/skills/*` — this directory should not exist (see Structure checks below); if it does, treat any entry as a finding
- `skills/<name>/SKILL.md` and `skills/<name>/rules/*.md`
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`
- `opencode.json`, `AGENTS.md`, `CLAUDE.md`, and `README.md`

## Checks

**Frontmatter**
- Every `SKILL.md` and command file has `name` and `description`.
- `description` names concrete trigger phrases/keywords a user would actually type — flag generic descriptions ("helpful utilities for X") that won't match real requests.
- `metadata` (if present) is a map, not a scalar or list.
- Frontmatter uses fields each runtime supports. OpenCode ignores Claude-only fields such as `hooks` and `allowed-tools`; flag prose that assumes they run in OpenCode.

**Structure**
- Every file linked from a `SKILL.md`'s rules list actually exists at that path.
- Every file under `rules/*.md` is linked from its `SKILL.md` — flag orphaned rule files nothing points to.
- `.claude-plugin/plugin.json` declares `"skills": "./skills"`, which auto-discovers every `skills/<name>/` directory — no `.claude/skills/<name>` symlink is needed for a skill to load, and `.claude/skills/` should not exist at all per `CLAUDE.md`. Flag any symlink or file found under `.claude/skills/` as a finding to remove.
- `AGENTS.md` is a valid symlink to `CLAUDE.md`; flag a dangling link or a divergent copy.

**OpenCode discovery**
- `opencode.json` is valid JSON(C) and includes `"$schema": "https://opencode.ai/config.json"`.
- Its `skills` array exposes `./skills` (or an equivalent path) so every top-level skill is discoverable.
- Every `skills/<name>/SKILL.md` is in the directory form OpenCode discovers (`<source>/<name>/SKILL.md`), not nested deeper than the search limit.
- Skill IDs are path-derived, lowercase kebab-case, and unique; flag a frontmatter `name` that hides a mismatched directory ID.
- No skill sets `metadata.opencode/autoinvoke: false` or `disable-model-invocation: true` unless intentionally hidden.

**Content accuracy**
- Version-specific claims (language/runtime versions, flags, API availability) are internally consistent within a file and across sibling rule files covering the same topic.
- Code samples use syntax/APIs consistent with the versions the skill claims to target.
- Flag claims that read as unverified or suspiciously specific (exact version numbers, benchmark figures) with no apparent basis in the repo.
- Any Claude-only automation (e.g. the `terraform` skill's `hooks`) has a documented manual equivalent for OpenCode in the skill body.

**Duplication**
- Flag near-duplicate guidance repeated across multiple rule files instead of stated once and cross-linked — costs context on every load.

**Plugin/marketplace manifests**
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` are valid JSON.
- `plugin.json`'s `name` and the plugin entry `name` in `marketplace.json` match.
- Marketplace `name` isn't on Claude Code's reserved list (`agent-skills`, `claude-code-marketplace`, etc.) — it must differ from any reserved name even when the plugin name itself matches one.
- `source` paths in `marketplace.json` resolve to real directories relative to the marketplace root.
- Install names in `README.md` and `CLAUDE.md` match the manifests exactly.

## Output

Report findings in chat, most severe first. Each finding needs the file, a one-sentence summary of the defect, and a concrete failure scenario (what a future agent or user hits because of it) — not just "this could be cleaner." If nothing survives scrutiny, report an empty list rather than padding it with style nitpicks.
