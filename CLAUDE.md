# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a growing collection of Claude Agent Skills spanning multiple languages and disciplines (currently Node.js and Go; Python and DevOps are planned) — reusable, discoverable knowledge packages that get loaded into an agent's context when their trigger conditions match. There is no application code, build step, or test suite here; the "product" is the Markdown/skill content itself, plus small example assets referenced from that content.

## Distribution: this repo is a Claude Code plugin + marketplace

The repo root doubles as both a Claude Code **plugin** and a single-plugin **marketplace**, following the same pattern as other shared skill collections (e.g. addyosmani/agent-skills):

- `.claude-plugin/plugin.json` — plugin manifest (`name: agent-skills`). Its presence at the repo root means every top-level `skills/<name>/` directory is auto-discovered as a plugin skill once the plugin is installed.
- `.claude-plugin/marketplace.json` — marketplace catalog (`name: saldyy-skills`) listing this repo as its one plugin (`source: "."`). Note: the marketplace name intentionally differs from the plugin name — `agent-skills` is on Claude Code's reserved marketplace-name list, so only the *marketplace* needed a different name.
- Install path for others: `/plugin marketplace add saldyy/agent-skills` then `/plugin install agent-skills@saldyy-skills`. Local dev: `claude --plugin-dir /path/to/agent-skills`.

`.claude/skills/nodejs` is a symlink to `../../skills/nodejs`, so the same skill also auto-loads as a plain project skill for anyone who just clones the repo and opens it in Claude Code directly (no plugin install needed). **When adding a new skill directory under `skills/<name>/`, add a matching symlink under `.claude/skills/<name>` pointing to `../../skills/<name>` to keep both paths working.**

## Structure

Each skill lives under `skills/<skill-name>/` and follows this layout:

- `SKILL.md` — the entry point. YAML frontmatter (`name`, `description`, `metadata.tags`) followed by the skill body. The `description` field is what a model uses to decide *when* to load the skill, so it must enumerate concrete trigger phrases/keywords, not just a category name.
- `rules/*.md` — individual topic files, each with its own frontmatter (`name`, `description`, `metadata.tags`) and focused content (one topic per file, ~100–450 lines). `SKILL.md` links to every rule file and should stay in sync with what's on disk — if you add, remove, or rename a rule file, update the corresponding link/list in `SKILL.md`.
- `rules/assets/` — runnable example code referenced by rule files (e.g. `graceful-server.ts` + its `graceful-server.test.ts`). These are illustrative reference implementations, not a tested/built package — there's no `package.json`/CI wired to them in this repo.
- `tile.json` — a per-skill manifest (`name`, `version`, `summary`, `skills.<key>.path`) predating the plugin/marketplace setup above; not read by Claude Code itself, and not required for new skills (the `golang` skill omits it; `nodejs` keeps its existing one for back-compat). The `summary` field mirrors `SKILL.md`'s frontmatter `description` and should be kept in sync with it if both are kept.

## Working on a skill

- Keep `SKILL.md` as an index/router: short "when to use" guidance, common high-level workflows with pointers into `rules/*.md`, and a full links list at the bottom. Put the deep, worked-example content in the individual `rules/*.md` files, not in `SKILL.md` itself.
- When editing a skill's `description` (in `SKILL.md` frontmatter and `tile.json`'s `summary`), keep both copies consistent — they're meant to describe the same trigger surface.
- Code samples inside a skill's `rules/*.md` and `rules/assets/` should stay consistent with whatever conventions that skill documents (e.g. the `nodejs` skill's own guidance on ESM, native type stripping, `node:test`, etc. — see `skills/nodejs/rules/typescript.md` and `skills/nodejs/rules/testing.md`).
- There is no lint/build/test command for this repo itself; validate changes by reading the rendered Markdown and checking that internal links (`SKILL.md` → `rules/*.md`) still resolve.
