# Agent Skills

A collection of [Claude Agent Skills](https://agentskills.io/) — domain-specific best practices that Claude Code loads automatically while you're writing code in a given language.

## Skills Included

| Skill | Description |
|-------|-------------|
| **nodejs** | Type stripping (Node 24+), async patterns, error handling, streams, modules, testing, performance, caching, logging, graceful shutdown |
| **golang** | Error handling (`%w` vs `%v`, sentinel vs typed errors), concurrency (goroutine lifetimes, channels vs mutexes), data structures, table-driven testing, naming, core style |

Each skill is a `SKILL.md` router plus a `rules/` directory of focused topic files, loaded on demand rather than all at once.

## Install

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

## Commands

- `/review-claude` — audits the skills, commands, and plugin manifests in this repo itself for broken links, missing frontmatter, dangling symlinks, stale version claims, and invalid plugin/marketplace JSON.

## Structure

```
.
├── skills/
│   └── <name>/
│       ├── SKILL.md      # entry point: when to use, common workflows, links to rules/
│       └── rules/*.md    # one topic per file, loaded on demand
├── .claude/
│   └── commands/*.md     # project slash commands
└── .claude-plugin/
    ├── plugin.json       # plugin manifest; "skills": "./skills" auto-discovers every skill above
    └── marketplace.json  # marketplace catalog listing this repo as its one plugin
```

See [CLAUDE.md](CLAUDE.md) for the full authoring conventions.

## License

MIT
