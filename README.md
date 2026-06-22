# claude-ruby-quality

A **self-hosted Claude Code plugin marketplace** distributing Ruby/Rails
code-quality skills. It ships one plugin, **`ruby-quality`**, with two skills:

- **`ruby-code-smells`** — reviews Ruby/Rails production code for code smells
  using thoughtbot's [Ruby Science](https://github.com/thoughtbot/ruby-science)
  catalog and names the matching refactoring for each finding.
- **`ruby-test-smells`** — reviews RSpec/Minitest test suites for test smells
  using Gerard Meszaros' [xUnit Test Patterns](http://xunitpatterns.com),
  thoughtbot's [Let's Not](https://thoughtbot.com/blog/lets-not), and Sandi
  Metz's *Magic Tricks of Testing*.

> Credit: the production-code catalog is based on thoughtbot's *Ruby Science*
> (https://github.com/thoughtbot/ruby-science); the test catalog on Gerard
> Meszaros' *xUnit Test Patterns*, thoughtbot's *Let's Not*, and Sandi Metz's
> *The Magic Tricks of Testing*.

This marketplace is **self-hosted** — there's no community/official submission.
Users add the marketplace by repo and install the plugin from it.

## Install

The marketplace name (`ruby-quality-marketplace`) comes from the `name` field in
`.claude-plugin/marketplace.json` — keep these commands in sync if you rename it.

```
/plugin marketplace add chrisjgilbert/claude-ruby-quality
/plugin install ruby-quality@ruby-quality-marketplace
```

Installing the plugin enables both skills. Claude invokes a skill automatically
when you ask it to check for code smells or review tests, or you can call them
directly: `/ruby-quality:ruby-code-smells` and `/ruby-quality:ruby-test-smells`.

> Note: the `displayName` field on the marketplace plugin entry requires Claude
> Code v2.1.143 or later (it falls back to `name` on older versions).

## Updating

Third-party plugins do **not** auto-update by default. To ship an update:

1. Bump `version` in `ruby-quality/.claude-plugin/plugin.json` (semver) and
   commit/push. An unchanged version is skipped by clients.
2. Users refresh and reinstall:

   ```
   /plugin marketplace update ruby-quality-marketplace
   /plugin install ruby-quality@ruby-quality-marketplace
   ```

## Adding more later

**Another skill** — the plugin auto-discovers everything under its `skills/`
directory, so this needs **no marketplace.json change**. Drop in
`ruby-quality/skills/<new-skill-name>/SKILL.md` with `name` and `description`
frontmatter, then bump the plugin's `version`.

**A whole new plugin** — create `<plugin>/.claude-plugin/plugin.json` and
`<plugin>/skills/<name>/SKILL.md`, then add one entry to the `plugins` array in
`.claude-plugin/marketplace.json` pointing `source` at `./<plugin>`.

## Layout

```
.
├── README.md
├── .claude-plugin/
│   └── marketplace.json          # marketplace catalog (name, owner, plugins[])
└── ruby-quality/                 # the plugin
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        ├── ruby-code-smells/     # skill 1 — production code smells
        │   └── SKILL.md
        └── ruby-test-smells/     # skill 2 — test smells
            └── SKILL.md
```
