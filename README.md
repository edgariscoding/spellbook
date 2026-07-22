# spellbook

A personal collection of [Claude Code](https://claude.com/claude-code) plugins — a
spellbook of useful charms for solo development on GitHub.

## Plugins

| Plugin | Charm | What it does |
| - | - | - |
| [`erecto`](./plugins/erecto) | The Structure-Erecting Charm | Stands up a new repo's working setup once — interviews you, then writes its CLAUDE.md, issue template, and gitignore. |
| [`legilimens`](./plugins/legilimens) | The Mind-Reading Charm | Draws a design out of your head one question at a time, then writes the spec. Never writes code. |
| [`lumos`](./plugins/lumos) | The Wand-Lighting Charm | The workflow spine — lights the way through a change from scope to ship, naming legilimens and revelio at their steps. |
| [`revelio`](./plugins/revelio) | The Revealing Charm | Multi-lens, scope-disciplined code review that reveals hidden issues before you open a GitHub PR. |

Together they span a change: **erecto** sets the repo up, **legilimens** designs the
change, **lumos** guides the build, and **revelio** reviews it before you ship.
The universal working defaults that apply in every session live in
[`home/CLAUDE.md`](./home/CLAUDE.md) (see [Personal defaults](#personal-defaults-global-baseline)).

## Install

Add this marketplace, then install the plugins you want:

```text
/plugin marketplace add edgariscoding/spellbook
/plugin install erecto@spellbook
/plugin install legilimens@spellbook
/plugin install lumos@spellbook
/plugin install revelio@spellbook
```

Restart Claude Code after installing. To update a plugin later, run:

```text
/plugin update <name>@spellbook
```

### Local development

To work on the plugins from this checkout, register the marketplace by path:

```text
/plugin marketplace add /Users/edgar/dev/spellbook
/plugin install erecto@spellbook
/plugin install legilimens@spellbook
/plugin install lumos@spellbook
/plugin install revelio@spellbook
```

## Personal defaults (global baseline)

`home/CLAUDE.md` holds the universal working defaults — working style, the
no-placeholders rule, editing and output norms — the ambient behavior that should
apply in *every* session, not only inside a scaffolded repo. A plugin can't
contribute always-on context, so this ships as a `CLAUDE.md` import rather than a
skill.

Install it once per machine, **non-destructively** (append, never overwrite):

```text
# back up any existing global file first (-n never clobbers an existing backup)
cp -n ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.bak 2>/dev/null || true
# then add this import line to ~/.claude/CLAUDE.md (adjust the path to your checkout):
@~/dev/spellbook/home/CLAUDE.md
```

The `@import` pulls the baseline in; keep any machine-specific lines alongside it.
`erecto` reminds you if this import is missing when it scaffolds a repo.

## Repository layout

```text
spellbook/
├── .claude-plugin/
│   └── marketplace.json        # marketplace manifest (lists the plugins)
├── home/
│   └── CLAUDE.md               # universal global baseline (imported into ~/.claude/CLAUDE.md)
└── plugins/                   # each plugin dir also has a README.md
    ├── erecto/                 # one-time repo scaffolder
    │   ├── .claude-plugin/plugin.json
    │   ├── README.md
    │   └── skills/erecto/
    │       ├── SKILL.md
    │       └── assets/         # invariants-block.md + task.md (materialized per repo)
    ├── legilimens/
    │   ├── .claude-plugin/plugin.json
    │   ├── README.md
    │   └── skills/legilimens/SKILL.md
    ├── lumos/                  # the workflow spine
    │   ├── .claude-plugin/plugin.json
    │   ├── README.md
    │   └── skills/lumos/SKILL.md
    └── revelio/
        ├── .claude-plugin/plugin.json
        ├── README.md
        └── skills/revelio/
            ├── SKILL.md
            └── assets/review-report-template.md
```

## Versioning

Each plugin sets its own semver in `plugins/<name>/.claude-plugin/plugin.json`.
Claude Code picks a plugin's cache version in priority order — the `version` in its
`plugin.json` if set, else its marketplace entry version, else the commit SHA — and
users get an update only when that effective version changes. So **bump the plugin's
`version` when it changes**. Bumping the marketplace's top-level `version` in the
same commit is harmless and makes the release obvious, so the habit here is to do both.

## License

MIT — free to use and adapt.
