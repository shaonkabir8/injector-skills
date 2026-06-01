# injector-skills-help

Quick-reference card. One shot, no mode change.

## What it does

Prints a cheat sheet of all injector-skills modes, sibling skills, deactivation triggers, and how to set the default mode via env var or config file. One-shot display — does not flip the active mode, write flag files, or persist anything. Use when you forget the slash commands.

## How to invoke

```
/injector-skills-help
```

Also triggers on "injector-skills help", "what injector-skills commands", "how do I use injector-skills".

## Example output

```
Modes:
  /injector-skills              full (default)
  /injector-skills lite         lighter
  /injector-skills ultra        extreme
  /injector-skills wenyan       classical Chinese

Skills:
  /injector-skills-commit       terse Conventional Commits
  /injector-skills-review       one-line PR comments
  /injector-skills-stats        session token savings

Deactivate:
  "stop injector-skills" or "normal mode"
```

## See also

- [`SKILL.md`](./SKILL.md) — full reference card
- [Caveman README](../../README.md) — repo overview
