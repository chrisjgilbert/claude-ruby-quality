# claude-ruby-quality

A **self-hosted Claude Code plugin marketplace** distributing Ruby/Rails
code-quality skills. It currently ships one plugin, `ruby-code-smells`, which
reviews Ruby/Rails code for code smells using thoughtbot's
[Ruby Science](https://github.com/thoughtbot/ruby-science) catalog and names the
matching refactoring for each finding.

> Credit: the smell/refactoring catalog is based on thoughtbot's *Ruby Science*
> (https://github.com/thoughtbot/ruby-science).

This marketplace is **self-hosted** — there's no community/official submission.
Users add the marketplace by repo and install the plugin from it.

## Install

The marketplace name (`ruby-quality-marketplace`) comes from the `name` field in
`.claude-plugin/marketplace.json` — keep these commands in sync if you rename it.

```
/plugin marketplace add chrisjgilbert/claude-ruby-quality
/plugin install ruby-code-smells@ruby-quality-marketplace
```

Then run the skill — Claude invokes it automatically when you ask it to check
for code smells, or you can call it directly as `/ruby-code-smells:ruby-code-smells`.

> Note: the `displayName` field on the marketplace plugin entry requires Claude
> Code v2.1.143 or later (it falls back to `name` on older versions).

## Updating

Third-party plugins do **not** auto-update by default. To ship an update:

1. Bump `version` in `ruby-code-smells/.claude-plugin/plugin.json` (semver) and
   commit/push. An unchanged version is skipped by clients.
2. Users refresh and reinstall:

   ```
   /plugin marketplace update ruby-quality-marketplace
   /plugin install ruby-code-smells@ruby-quality-marketplace
   ```

## Adding another skill later

The plugin auto-discovers everything under its `skills/` directory, so adding a
second skill needs **no marketplace.json change**. Just drop in:

```
ruby-code-smells/skills/<new-skill-name>/SKILL.md
```

with `name` and `description` frontmatter, then bump the plugin `version` so
users pick it up.

## Layout

```
.
├── README.md
├── .claude-plugin/
│   └── marketplace.json          # marketplace catalog (name, owner, plugins)
└── ruby-code-smells/             # the plugin
    ├── .claude-plugin/
    │   └── plugin.json           # plugin manifest (name, description, version)
    └── skills/
        └── ruby-code-smells/
            └── SKILL.md          # the skill
```
