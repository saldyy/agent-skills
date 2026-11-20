# Agent Skills — Repository Guide

This file provides guidance to AI coding agents — Claude Code and OpenCode — when working with code in this repository.

## Repository purpose

This is a growing collection of Agent Skills spanning multiple languages and disciplines (currently Node.js, Go, and Terraform; Python and DevOps are planned) — reusable, discoverable knowledge packages that get loaded into an agent's context when their trigger conditions match. There is no application code, build step, or test suite here; the "product" is the Markdown/skill content itself, plus small example assets referenced from that content.

## Distribution: one skills directory, two agent runtimes

The skills are plain `skills/<name>/SKILL.md` + `rules/*.md` trees and are shared by both runtimes. Each runtime gets its own manifest:

### Claude Code plugin + marketplace

The repo root doubles as both a Claude Code **plugin** and a single-plugin **marketplace**, following the same pattern as other shared skill collections (e.g. addyosmani/agent-skills):

- `.claude-plugin/plugin.json` — plugin manifest (`name: agent-skills`). Its presence at the repo root means every top-level `skills/<name>/` directory is auto-discovered as a plugin skill once the plugin is installed.
- `.claude-plugin/marketplace.json` — marketplace catalog (`name: saldyy-agent-skills`) listing this repo as its one plugin (`source: "."`). Note: the marketplace name intentionally differs from the plugin name — `agent-skills` is on Claude Code's reserved marketplace-name list, so only the *marketplace* needed a different name.
- Install path for others: `/plugin marketplace add saldyy/agent-skills` then `/plugin install agent-skills@saldyy-agent-skills`. Local dev: `claude --plugin-dir /path/to/agent-skills`.

### OpenCode

- `opencode.json` at the repo root adds `./skills` to OpenCode's `skills` sources. OpenCode does **not** read `.claude-plugin/` and does not discover a top-level `skills/` directory by default, so this config entry is what makes the skills available when the repo is opened in OpenCode.
- Relative `skills` paths resolve from the active OpenCode working directory (normally the repo root). If you invoke OpenCode from a subdirectory, use an absolute path or `~/...` in the config instead.
- OpenCode compatibility also covers `.claude/skills`, but this repo intentionally does not create one (see below).
- `AGENTS.md` is a symlink to `CLAUDE.md`, so OpenCode loads the same repository guidance. OpenCode falls back to `CLAUDE.md` when `AGENTS.md` is absent, but the explicit link keeps the intent obvious and survives a global `AGENTS.md` being present.

The plugin manifest's `"skills": "./skills"` field already tells Claude Code to auto-discover every top-level `skills/<name>/` directory as a plugin skill. **When adding a new skill directory under `skills/<name>/`, no additional registration is needed in either runtime** — `opencode.json` already points at the whole `skills/` tree. Nothing should be created under `.claude/` — do not add a `.claude/skills/<name>` symlink. There is no `.claude/skills/` directory in this repo; don't recreate one.

## Commands

- `.claude/commands/*.md` holds Claude Code project slash commands (e.g. `review-claude.md` → `/review-claude`) — plain Markdown files with the same frontmatter fields as a skill, but no supporting directory. This is the canonical review command.
- `.opencode/commands/*.md` holds the equivalent OpenCode commands (e.g. `review-skills.md` → `/review-skills`). OpenCode does not read `.claude/commands/`, so the OpenCode command is a thin pointer to the canonical checklist rather than a copy that can drift.

Use a command instead of a full skill when the content is a one-shot procedure invoked by name, not background knowledge the agent should auto-load.

## Structure

Each skill lives under `skills/<skill-name>/` and follows this layout:

- `SKILL.md` — the entry point. YAML frontmatter (`name`, `description`, `metadata.tags`) followed by the skill body. The `description` field is what a model uses to decide *when* to load the skill, so it must enumerate concrete trigger phrases/keywords, not just a category name. Both Claude Code and OpenCode read this format; frontmatter fields specific to one runtime (e.g. the `terraform` skill's Claude Code `hooks`) are ignored by the other.
- `rules/*.md` — individual topic files, each with its own frontmatter (`name`, `description`, `metadata.tags`) and focused content (one topic per file, ~100–450 lines). `SKILL.md` links to every rule file and should stay in sync with what's on disk — if you add, remove, or rename a rule file, update the corresponding link/list in `SKILL.md`.
- `rules/assets/` — runnable example code referenced by rule files (e.g. `graceful-server.ts` + its `graceful-server.test.ts`). These are illustrative reference implementations, not a tested/built package — there's no `package.json`/CI wired to them in this repo.
- `tile.json` — a per-skill manifest (`name`, `version`, `summary`, `skills.<key>.path`) predating the plugin/marketplace setup above; not read by Claude Code or OpenCode, and not required for new skills (the `golang` skill omits it; `nodejs` keeps its existing one for back-compat). The `summary` field mirrors `SKILL.md`'s frontmatter `description` and should be kept in sync with it if both are kept.

Repository-level files:

```
.
├── skills/
│   └── <name>/
│       ├── SKILL.md      # entry point: when to use, common workflows, links to rules/
│       └── rules/*.md    # one topic per file, loaded on demand
├── .claude/
│   └── commands/*.md     # Claude Code slash commands
├── .opencode/
│   └── commands/*.md     # OpenCode slash commands
├── .claude-plugin/
│   ├── plugin.json       # Claude Code plugin manifest; "skills": "./skills" auto-discovers every skill above
│   └── marketplace.json  # Claude Code marketplace catalog listing this repo as its one plugin
├── opencode.json         # OpenCode config; "skills": ["./skills"] exposes every skill above
├── AGENTS.md -> CLAUDE.md
└── CLAUDE.md             # repository guidance shared by both runtimes
```

## Working on a skill

- Keep `SKILL.md` as an index/router: short "when to use" guidance, common high-level workflows with pointers into `rules/*.md`, and a full links list at the bottom. Put the deep, worked-example content in the individual `rules/*.md` files, not in `SKILL.md` itself.
- When editing a skill's `description` (in `SKILL.md` frontmatter and `tile.json`'s `summary`), keep both copies consistent — they're meant to describe the same trigger surface.
- Code samples inside a skill's `rules/*.md` and `rules/assets/` should stay consistent with whatever conventions that skill documents (e.g. the `nodejs` skill's own guidance on ESM, native type stripping, `node:test`, etc. — see `skills/nodejs/rules/typescript.md` and `skills/nodejs/rules/testing.md`).
- Write skill bodies so they work in any agent runtime. Don't say "ask Claude" or rely on Claude-only mechanics in the prose: skill frontmatter `hooks` (as used by `terraform`) run only in Claude Code, so state the equivalent manual step for OpenCode and other runtimes.
- There is no lint/build/test command for this repo itself; validate changes by reading the rendered Markdown and checking that internal links (`SKILL.md` → `rules/*.md`) still resolve. When OpenCode is available, `opencode api get '/api/skill?location[directory]=<repo>'` lists the skills it actually discovered.
