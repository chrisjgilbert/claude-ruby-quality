# claude-ruby-quality

A **self-hosted Claude Code plugin marketplace** distributing Ruby/Rails
code-quality skills. It ships two plugins:

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
/plugin install ruby-code-smells@ruby-quality-marketplace
/plugin install ruby-test-smells@ruby-quality-marketplace
```

Install either or both. Claude invokes a skill automatically when you ask it to
check for code smells or review tests, or you can call them directly:
`/ruby-code-smells:ruby-code-smells` and `/ruby-test-smells:ruby-test-smells`.

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

## Adding more later

**Another skill inside an existing plugin** — the plugin auto-discovers
everything under its `skills/` directory, so this needs **no marketplace.json
change**. Drop in `ruby-code-smells/skills/<new-skill-name>/SKILL.md` (or under
`ruby-test-smells/`) with `name` and `description` frontmatter, then bump that
plugin's `version`.

**A whole new plugin** (like `ruby-test-smells`) — create
`<plugin>/.claude-plugin/plugin.json` and `<plugin>/skills/<name>/SKILL.md`,
then add one entry to the `plugins` array in
`.claude-plugin/marketplace.json` pointing `source` at `./<plugin>`.

## Layout

```
.
├── README.md
├── .claude-plugin/
│   └── marketplace.json          # marketplace catalog (name, owner, plugins[])
├── ruby-code-smells/             # plugin 1 — production code smells
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── skills/
│       └── ruby-code-smells/
│           └── SKILL.md
└── ruby-test-smells/             # plugin 2 — test smells
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        └── ruby-test-smells/
            └── SKILL.md
```
