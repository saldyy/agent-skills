# Agent Skills

A collection of [Agent Skills](https://agentskills.io/) — domain-specific best practices that Claude Code and OpenCode load automatically while you're writing code in a given language.

## Skills Included

| Skill | Description |
|-------|-------------|
| **nodejs** | Type stripping (Node 24+), async patterns, error handling, streams, modules, testing, performance, caching, logging, graceful shutdown |
| **golang** | Error handling (`%w` vs `%v`, sentinel vs typed errors), concurrency (goroutine lifetimes, channels vs mutexes), data structures, table-driven testing, naming, core style |
| **terraform** | Module layout, variables/outputs, conditional resources, cross-variable validation, provider quirks, `validate` + `tflint` workflow |

Each skill is a `SKILL.md` router plus a `rules/` directory of focused topic files, loaded on demand rather than all at once.

## Install

### Claude Code

This repo doubles as a Claude Code plugin and a single-plugin marketplace.

**From anywhere, via the marketplace:**

```
/plugin marketplace add saldyy/agent-skills
/plugin install agent-skills@saldyy-agent-skills
```

**Local development**, from a clone of this repo:

```
claude --plugin-dir /path/to/agent-skills
```

Skill discovery is driven entirely by `.claude-plugin/plugin.json`'s `"skills": "./skills"` field — every top-level `skills/<name>/` directory is picked up automatically, with no per-skill registration or symlink needed.

### OpenCode

The repo ships an `opencode.json` that exposes the shared `skills/` directory to OpenCode. Open it from the repo root and OpenCode discovers the skills, or reference one directly in a prompt as `@golang`, `@nodejs`, or `@terraform`.

To use the skills from **another project**, point an OpenCode config at this repo's `skills/` directory. In your global config (`~/.config/opencode/opencode.json`):

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "skills": ["~/code/agent-skills/skills"]
}
```

Or in a project's `opencode.json`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "skills": ["/absolute/path/to/agent-skills/skills"]
}
```

Relative `skills` paths resolve from the active OpenCode working directory, so the `~/` or absolute form is safer outside this repo. `./skills` works when OpenCode starts at the repo root. OpenCode does not read `.claude-plugin/`, so the `opencode.json` entry is what makes the skills available there.

## Commands

- Claude Code: `/review-claude` — audits the skills, commands, and plugin manifests in this repo for broken links, missing frontmatter, dangling symlinks, stale version claims, and invalid plugin/marketplace JSON.
- OpenCode: `/review-skills` — points at the same audit, since OpenCode does not read `.claude/commands/`.

## Structure

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

See [CLAUDE.md](CLAUDE.md) for the full authoring conventions.

## License

MIT
