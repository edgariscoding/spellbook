# spellbook

A personal collection of [Claude Code](https://claude.com/claude-code) plugins — a
spellbook of useful charms for solo development on GitHub.

## Plugins

| Plugin | Charm | What it does |
| - | - | - |
| [`legilimens`](./plugins/legilimens) | The Mind-Reading Charm | Draws a design out of your head one question at a time, then writes the spec. Never writes code. |
| [`revelio`](./plugins/revelio) | The Revealing Charm | Multi-lens, scope-disciplined code review that reveals hidden issues before you open a GitHub PR. |

The two bookend a change: `legilimens` before you build it, `revelio` before you
ship it.

## Install

Add this marketplace, then install the plugins you want:

```text
/plugin marketplace add edgariscoding/spellbook
/plugin install revelio@spellbook
/plugin install legilimens@spellbook
```

Restart Claude Code after installing. To update later, bump happens automatically
when the plugin version changes:

```text
/plugin update revelio@spellbook
```

### Local development

To work on the plugins from this checkout, register the marketplace by path:

```text
/plugin marketplace add /Users/edgar/Development/spellbook
/plugin install revelio@spellbook
/plugin install legilimens@spellbook
```

## Repository layout

```text
spellbook/
├── .claude-plugin/
│   └── marketplace.json        # marketplace manifest (lists the plugins)
└── plugins/
    ├── legilimens/
    │   ├── .claude-plugin/plugin.json
    │   └── skills/legilimens/SKILL.md
    └── revelio/
        ├── .claude-plugin/plugin.json
        └── skills/revelio/
            ├── SKILL.md
            └── assets/review-report-template.md
```

## Versioning

Each plugin sets its own semver in `plugins/<name>/.claude-plugin/plugin.json`.
Claude Code caches by the marketplace's top-level `version`, so when a plugin
changes, bump **both** the plugin's `version` and the top-level `version` in
`.claude-plugin/marketplace.json` in the same commit — otherwise installs keep
running the cached older version.

## License

MIT — see [`LICENSE`](./LICENSE) if present, otherwise free to use and adapt.
