---
name: injector-skills-help
description: >
  Quick-reference card for all injector-skills modes, skills, and commands.
  One-shot display, not a persistent mode. Trigger: /injector-skills-help,
  "injector-skills help", "what injector-skills commands", "how do I use injector-skills".
---

# Caveman Help

Display this reference card when invoked. One-shot — do NOT change mode, write flag files, or persist anything. Output in injector-skills style.

## Modes

| Mode | Trigger | What change |
|------|---------|-------------|
| **Lite** | `/injector-skills lite` | Drop filler. Keep sentence structure. |
| **Full** | `/injector-skills` | Drop articles, filler, pleasantries, hedging. Fragments OK. Default. |
| **Ultra** | `/injector-skills ultra` | Extreme compression. Bare fragments. Tables over prose. |
| **Wenyan-Lite** | `/injector-skills wenyan-lite` | Classical Chinese style, light compression. |
| **Wenyan-Full** | `/injector-skills wenyan` | Full 文言文. Maximum classical terseness. |
| **Wenyan-Ultra** | `/injector-skills wenyan-ultra` | Extreme. Ancient scholar on a budget. |

Mode stick until changed or session end.

## Skills

| Skill | Trigger | What it do |
|-------|---------|-----------|
| **injector-skills-commit** | `/injector-skills-commit` | Terse commit messages. Conventional Commits. ≤50 char subject. |
| **injector-skills-review** | `/injector-skills-review` | One-line PR comments: `L42: bug: user null. Add guard.` |
| **injector-skills-compress** | `/injector-skills-compress <file>` | Compress .md files to injector-skills prose. Saves ~46% input tokens. |
| **injector-skills-help** | `/injector-skills-help` | This card. |

## Deactivate

Say "stop injector-skills" or "normal mode". Resume anytime with `/injector-skills`.

## Configure Default Mode

Default mode = `full`. Change it:

**Environment variable** (highest priority):
```bash
export CAVEMAN_DEFAULT_MODE=ultra
```

**Config file** (`~/.config/injector-skills/config.json`):
```json
{ "defaultMode": "lite" }
```

Set `"off"` to disable auto-activation on session start. User can still activate manually with `/injector-skills`.

Resolution: env var > config file > `full`.

## More

Full docs: https://github.com/shaonkabir8/injector-skills
