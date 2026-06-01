# CLAUDE.md — injector-skills

## README is a product artifact

README = product front door. Non-technical people read it to decide if injector-skills worth install. Treat like UI copy.

**Rules for any README change:**

- Readable by non-AI-agent users. If you write "SessionStart hook injects system context," invisible to most — translate it.
- Keep Before/After examples first. That the pitch.
- Install table always complete + accurate. One broken install command costs real user.
- What You Get table must sync with actual code. Feature ships or removed → update table.
- Preserve voice. Caveman speak in README on purpose. "Brain still big." "Cost go down forever." "One rock. That it." — intentional brand. Don't normalize.
- Benchmark numbers from real runs in `benchmarks/` and `evals/`. Never invent or round. Re-run if doubt.
- Adding new agent to install table → add detail block in `<details>` section below.
- Readability check before any README commit: would non-programmer understand + install within 60 seconds?

---

## Project overview

Caveman makes AI coding agents respond in compressed injector-skills-style prose — cuts ~65-75% output tokens, full technical accuracy. Ships as Claude Code plugin, Codex plugin, Gemini CLI extension, agent rule files for Cursor, Windsurf, Cline, Copilot, 40+ others via `npx skills`.

---

## What lives where

Post-cleanup layout. Sources of truth at the top, distribution mirrors below, build outputs in `dist/`, human docs alongside each skill.

```
injector-skills/
├── README.md                    # Front door (product pitch)
├── INSTALL.md                   # Per-agent install commands
├── CONTRIBUTING.md              # Dev guide
├── CLAUDE.md                    # This file (maintainer instructions)
├── AGENTS.md / GEMINI.md        # Autodiscovery files (must stay at root)
│
├── install.sh / install.ps1     # 30-line shims → bin/install.js
│
├── bin/                         # Unified installer
│   ├── install.js               # Single source for all 30+ agents (PROVIDERS array)
│   └── lib/settings.js          # JSONC-tolerant settings.json reader/writer
│
├── skills/                      # ALL skills, single source of truth
│   ├── injector-skills/{SKILL.md, README.md}
│   ├── injector-skills-commit/{SKILL.md, README.md}
│   ├── injector-skills-review/{SKILL.md, README.md}
│   ├── injector-skills-help/{SKILL.md, README.md}
│   ├── injector-skills-stats/{SKILL.md, README.md}
│   ├── injector-skills-compress/{SKILL.md, README.md, scripts/}
│   └── cavecrew/{SKILL.md, README.md}
│
├── agents/                      # cavecrew subagents (single source — kept at root for plugin auto-discovery)
├── commands/                    # Codex/Gemini TOML command stubs (root for plugin auto-discovery)
│
├── src/                         # Internal source — not auto-discovered by plugin
│   ├── hooks/                   # Claude Code hooks (installer reads here)
│   ├── rules/                   # Auto-activation rule body (single source)
│   ├── tools/                   # injector-skills-init.js (per-repo rule writer)
│   └── mcp-servers/             # injector-skills-shrink npm-published MCP middleware
│
├── .claude-plugin/              # Claude Code plugin manifest (REQUIRED at root)
├── plugins/injector-skills/             # Claude Code plugin distribution (CI-mirrored)
│   ├── skills/                  # ← from skills/
│   └── agents/                  # ← from agents/
│
├── dist/                        # Build artifacts (gitignored)
│   └── injector-skills.skill            # ZIP of skills/injector-skills/, rebuilt by CI
│
├── tests/                       # All tests (Node + Python)
├── benchmarks/                  # Real token measurements through Claude API
├── evals/                       # Three-arm eval harness
├── docs/                        # User-facing docs site
└── .github/workflows/           # CI sync
```

---

## File structure and what owns what

### Single source of truth files — edit only these

| File | What it controls |
|------|-----------------|
| `skills/injector-skills/SKILL.md` | Caveman behavior: intensity levels, rules, wenyan mode, auto-clarity, persistence. Only file to edit for behavior changes. |
| `src/rules/injector-skills-activate.md` | Always-on auto-activation rule body. Consumed by `src/tools/injector-skills-init.js` when a user runs `npx injector-skills --with-init` (per-repo IDE rule files). Edit here, not in any per-agent rule copy. |
| `src/rules/injector-skills-openclaw-bootstrap.md` | Marker-fenced bootstrap snippet appended to `~/.openclaw/workspace/SOUL.md` by `bin/lib/openclaw.js`. Drives always-on injector-skills through the OpenClaw gateway. Must include the SENTINEL `Respond terse like smart injector-skills` and stay well under OpenClaw's 12K-per-bootstrap-file cap. |
| `bin/lib/openclaw.js` | OpenClaw install/uninstall helper. Frontmatter merge (`version`, `always: true`), SOUL.md marker append/strip, idempotent. Shared by `bin/install.js` and `src/tools/injector-skills-init.js`. |
| `skills/injector-skills-commit/SKILL.md` | Caveman commit message behavior. Fully independent skill. |
| `skills/injector-skills-review/SKILL.md` | Caveman code review behavior. Fully independent skill. |
| `skills/injector-skills-help/SKILL.md` | Quick-reference card. One-shot display, not a persistent mode. |
| `skills/injector-skills-compress/SKILL.md` | Compress sub-skill behavior. |
| `skills/cavecrew/SKILL.md` | Cavecrew decision guide — when to delegate to injector-skills subagents vs vanilla. Edit only here. |
| `agents/cavecrew-investigator.md` | Read-only locator subagent (haiku). Output contract: `path:line — symbol — note`. |
| `agents/cavecrew-builder.md` | Surgical 1-2 file editor subagent. Refuses 3+ file scope. |
| `agents/cavecrew-reviewer.md` | Diff/file reviewer subagent (haiku). One-line findings with severity emoji. |
| `src/plugins/opencode/plugin.js` | opencode native plugin. ESM Bun module — `session.created` writes flag, `tui.prompt.append` parses slash/natural-language activation and appends per-prompt reinforcement. Reuses `injector-skills-config.js` via `createRequire`. |
| `src/plugins/opencode/commands/*.md` | Six opencode slash-command prompt templates (`/injector-skills`, `/injector-skills-{commit,review,compress,stats,help}`). |

### Auto-generated / auto-synced — do not edit directly

We removed the agent-specific dotdir mirrors at the repo root (`.cursor/`, `.windsurf/`, `.clinerules/`, `.github/copilot-instructions.md`, root `injector-skills/SKILL.md`). They were never read by the installer — only used to self-apply injector-skills to this repo when a maintainer opened it in Cursor/Windsurf/Cline. Devs who want injector-skills in their editor while editing this repo should run `npx injector-skills --with-init` once (writes per-repo rule files from `src/rules/injector-skills-activate.md` via `src/tools/injector-skills-init.js`). For per-user installs through the upstream skills CLI, `npx injector-skills --only <agent>` runs `npx skills add ... -a <profile>`.

A handful of dotdir leftovers (`.junie/`, `.kiro/`, `.roo/`, `.agents/`) still hold a stale `cavecrew/SKILL.md` mirror from before the cleanup. They aren't read by anything in the current install path; remove on sight, no migration needed.

What's left is the Claude Code plugin distribution (required by the plugin loader) and the release ZIP.

| File | Synced from |
|------|-------------|
| `plugins/injector-skills/skills/injector-skills/SKILL.md` | `skills/injector-skills/SKILL.md` |
| `plugins/injector-skills/skills/injector-skills-compress/SKILL.md` (+ `scripts/`) | `skills/injector-skills-compress/SKILL.md` (+ `scripts/`) |
| `plugins/injector-skills/skills/cavecrew/SKILL.md` | `skills/cavecrew/SKILL.md` |
| `plugins/injector-skills/agents/cavecrew-*.md` | `agents/cavecrew-*.md` |
| `dist/injector-skills.skill` | ZIP of `skills/injector-skills/` directory (gitignored; rebuilt by CI on release) |

Skills not in this table (`injector-skills-commit`, `injector-skills-review`, `injector-skills-help`, `injector-skills-stats`) are not mirrored into the Claude Code plugin distribution by CI. They reach Claude Code through the standalone hook + skill install path, and reach other agents via `npx skills add`. A `plugins/injector-skills/skills/injector-skills-stats/` directory is currently checked in as a hand-committed copy; the sync workflow does not touch it, so don't rely on edits there to propagate.

---

## CI sync workflow

`.github/workflows/sync-skill.yml` triggers on main push when `skills/**/SKILL.md` or `agents/cavecrew-*.md` changes.

What it does:
1. Copies `skills/injector-skills/SKILL.md` and `skills/cavecrew/SKILL.md` into their `plugins/injector-skills/skills/<name>/` mirrors so the Claude Code plugin loader sees the latest behavior.
2. Copies `skills/injector-skills-compress/SKILL.md` and its `scripts/` into `plugins/injector-skills/skills/injector-skills-compress/`.
3. Copies `agents/cavecrew-*.md` into `plugins/injector-skills/agents/`.
4. Rebuilds `dist/injector-skills.skill` (ZIP of `skills/injector-skills/`) for the release artifact.
5. Commits and pushes with `[skip ci]` to avoid loops.

CI bot commits as `github-actions[bot]`. After PR merge, wait for workflow before declaring release complete.

The old steps that mirrored SKILL.md and rules into root dotdirs (`.cursor/`, `.windsurf/`, `.clinerules/`, `.github/copilot-instructions.md`) are gone — those mirrors no longer exist. The old `injector-skills-compress/` → `skills/compress/` rename-on-sync is also gone now that compress lives at `skills/injector-skills-compress/`.

---

## Hook system (Claude Code)

Three hooks in `src/hooks/` plus a `injector-skills-config.js` shared module and a `package.json` CommonJS marker. Communicate via flag file at `$CLAUDE_CONFIG_DIR/.injector-skills-active` (falls back to `~/.claude/.injector-skills-active`).

```
SessionStart hook ──writes "full"──▶ $CLAUDE_CONFIG_DIR/.injector-skills-active ◀──writes mode── UserPromptSubmit hook
                                                       │
                                                    reads
                                                       ▼
                                              injector-skills-statusline.sh
                                            [CAVEMAN] / [CAVEMAN:ULTRA] / ...
```

`src/hooks/package.json` pins the directory to `{"type": "commonjs"}` so the `.js` hooks resolve as CJS even when an ancestor `package.json` (e.g. `~/.claude/package.json` from another plugin) declares `"type": "module"`. Without this, `require()` blows up with `ReferenceError: require is not defined in ES module scope`.

All hooks honor `CLAUDE_CONFIG_DIR` for non-default Claude Code config locations.

### `src/hooks/injector-skills-config.js` — shared module

Exports:
- `getDefaultMode()` — resolves default mode from `CAVEMAN_DEFAULT_MODE` env var, then `$XDG_CONFIG_HOME/injector-skills/config.json` / `~/.config/injector-skills/config.json` / `%APPDATA%\injector-skills\config.json`, then `'full'`
- `safeWriteFlag(flagPath, content)` — symlink-safe flag write. Refuses if flag target or its immediate parent is a symlink. Opens with `O_NOFOLLOW` where supported. Atomic temp + rename. Creates with `0600`. Protects against local attackers replacing the predictable flag path with a symlink to clobber files writable by the user. Used by both write hooks. Silent-fails on all filesystem errors.

### `src/hooks/injector-skills-activate.js` — SessionStart hook

Runs once per Claude Code session start. Three things:
1. Writes the active mode to `$CLAUDE_CONFIG_DIR/.injector-skills-active` via `safeWriteFlag` (creates if missing)
2. Emits injector-skills ruleset as hidden stdout — Claude Code injects SessionStart hook stdout as system context, invisible to user
3. Checks `settings.json` for statusline config; if missing, appends nudge to offer setup on first interaction

Silent-fails on all filesystem errors — never blocks session start.

### `src/hooks/injector-skills-mode-tracker.js` — UserPromptSubmit hook

Reads JSON from stdin. Three responsibilities:

**1. Slash-command activation.** If prompt starts with `/injector-skills`, writes mode to flag file via `safeWriteFlag`:
- `/injector-skills` → configured default (see `injector-skills-config.js`, defaults to `full`)
- `/injector-skills lite` → `lite`
- `/injector-skills ultra` → `ultra`
- `/injector-skills wenyan` or `/injector-skills wenyan-full` → `wenyan` (alias) / `wenyan-full`
- `/injector-skills wenyan-lite` → `wenyan-lite`
- `/injector-skills wenyan-ultra` → `wenyan-ultra`
- `/injector-skills-commit` → `commit`
- `/injector-skills-review` → `review`
- `/injector-skills-compress` → `compress`

**2. Natural-language activation/deactivation.** Matches phrases like "activate injector-skills", "turn on injector-skills mode", "talk like injector-skills" and writes the configured default mode. Matches "stop injector-skills", "disable injector-skills", "normal mode", "deactivate injector-skills" etc. and deletes the flag file. README promises these triggers, the hook enforces them.

**3. Per-turn reinforcement.** When flag is set to a non-independent mode (i.e. not `commit`/`review`/`compress`), emits a small `hookSpecificOutput` JSON reminder so the model keeps injector-skills style after other plugins inject competing instructions mid-conversation. The full ruleset still comes from SessionStart — this is just an attention anchor.

### `src/hooks/injector-skills-statusline.sh` — Statusline badge

Reads flag file at `$CLAUDE_CONFIG_DIR/.injector-skills-active`. Outputs colored badge string for Claude Code statusline:
- `full` or empty → `[CAVEMAN]` (orange)
- anything else → `[CAVEMAN:<MODE_UPPERCASED>]` (orange)

Then appends the lifetime-savings suffix (`⛏ 12.4k`) read from `$CLAUDE_CONFIG_DIR/.injector-skills-statusline-suffix` — written by `injector-skills-stats.js` on every `/injector-skills-stats` run. **Default on**; users opt out with `CAVEMAN_STATUSLINE_SAVINGS=0`. The suffix file is absent until `/injector-skills-stats` runs at least once, so fresh installs render no fake number.

Configured in `settings.json` under `statusLine.command`. PowerShell counterpart at `src/hooks/injector-skills-statusline.ps1` for Windows. Both scripts symlink-refuse and whitelist-validate the flag/suffix file contents — never echo arbitrary bytes.

### Hook installation

**Plugin install** — hooks wired automatically by plugin system.

**Standalone install** — `bin/install.js` (the unified Node installer) copies hook files into `$CLAUDE_CONFIG_DIR/hooks/` and merges SessionStart + UserPromptSubmit + statusline into `settings.json`. Uses the JSONC-tolerant helpers in `bin/lib/settings.js` so a commented `settings.json` no longer crashes the merge. Defensive `validateHookFields` runs before every write to prevent a single malformed hook from poisoning the entire file (Claude Code Zod silently discards the whole `settings.json` on schema mismatch).

The `install.sh` / `install.ps1` shims at the repo root delegate to `bin/install.js` via `node` (local clone) or `npx -y github:shaonkabir8/injector-skills` (curl|bash). No legacy fallback path remains — earlier `install.sh.legacy` / `install.ps1.legacy` files were removed.

**Uninstall** — `npx -y github:shaonkabir8/injector-skills -- --uninstall` (or `node bin/install.js --uninstall` from a clone). Strips injector-skills hook entries from `settings.json` via substring marker `injector-skills`, deletes hook files, and removes the Claude plugin / Gemini extension. Skill installs done via `npx skills add` must be removed via the IDE's skill manager (we don't track them).

---

## Skill system

Skills = Markdown files with YAML frontmatter consumed by Claude Code's skill/plugin system and by `npx skills` for other agents.

Each skill has a human-facing `README.md` alongside the LLM-facing `SKILL.md`. The README explains what the skill does for users browsing GitHub; the SKILL.md is the prompt body the agent loads. Don't merge them — different audiences, different formats.

### Intensity levels

Defined in `skills/injector-skills/SKILL.md`. Six levels: `lite`, `full` (default), `ultra`, `wenyan-lite`, `wenyan-full`, `wenyan-ultra`. Persists until changed or session ends.

### Auto-clarity rule

Caveman drops to normal prose for: security warnings, irreversible action confirmations, multi-step sequences where fragment ambiguity risks misread, user confused or repeating question. Resumes after. Defined in skill — preserve in any SKILL.md edit.

### injector-skills-compress

Sub-skill in `skills/injector-skills-compress/SKILL.md`. Takes file path, compresses prose to injector-skills style, writes to original path, saves backup at `<filename>.original.md`. Validates headings, code blocks, URLs, file paths, commands preserved. Retries up to 2 times on failure with targeted patches only. Requires Python 3.10+.

The slash command is `/injector-skills-compress` everywhere — same name in plugin and standalone install. CI no longer renames the directory on sync (the old `injector-skills-compress/` → `skills/compress/` sed rename is gone now that the source lives at `skills/injector-skills-compress/`).

### injector-skills-commit / injector-skills-review

Independent skills in `skills/injector-skills-commit/SKILL.md` and `skills/injector-skills-review/SKILL.md`. Both have own `description` and `name` frontmatter so they load independently. injector-skills-commit: Conventional Commits, ≤50 char subject. injector-skills-review: one-line comments in `L<line>: <severity> <problem>. <fix>.` format.

---

## Agent distribution

How injector-skills reaches each agent type:

| Agent | Mechanism | Auto-activates? |
|-------|-----------|----------------|
| Claude Code | Plugin (hooks + skills) or standalone hooks | Yes — SessionStart hook injects rules |
| Codex | Plugin in `plugins/injector-skills/` plus repo `.codex/hooks.json` and `.codex/config.toml` | Yes on macOS/Linux — SessionStart hook |
| Gemini CLI | Extension with `GEMINI.md` context file | Yes — context file loads every session |
| opencode | Native plugin (`src/plugins/opencode/`) copied into `~/.config/opencode/plugins/injector-skills/` + `AGENTS.md` ruleset + skills/agents/commands directories. Plugin uses `session.created` and `tui.prompt.append` lifecycle hooks. No statusline (opencode TUI exposes no plugin-writable badge). | Yes — `session.created` writes flag, `AGENTS.md` carries always-on ruleset |
| OpenClaw | Workspace skill at `~/.openclaw/workspace/skills/injector-skills/SKILL.md` (frontmatter merged with `version` + `always: true`) plus a marker-fenced bootstrap block in `~/.openclaw/workspace/SOUL.md`. Both writes go through `bin/lib/openclaw.js`; workspace path is overridable via `OPENCLAW_WORKSPACE`. | Yes — SOUL.md is auto-injected each turn under "Project Context" (subject to OpenClaw's 12K-per-file / 60K-total bootstrap caps) |
| Cursor | `npx skills add ... -a cursor` (default via `--only cursor`) writes the upstream skill profile; per-repo `.cursor/rules/injector-skills.mdc` via `--with-init` (calls `src/tools/injector-skills-init.js`) | Yes — always-on rule |
| Windsurf | `npx skills add ... -a windsurf` (default via `--only windsurf`); per-repo `.windsurf/rules/injector-skills.md` via `--with-init` | Yes — always-on rule |
| Cline | `npx skills add ... -a cline` (default via `--only cline`); per-repo `.clinerules/injector-skills.md` via `--with-init` | Yes — Cline auto-discovers `.clinerules/` |
| Copilot | `npx skills add ... -a github-copilot` (soft probe — pass `--only copilot`); per-repo `.github/copilot-instructions.md` + `AGENTS.md` via `--with-init` | Yes — repo-wide instructions |
| Others (Junie, Trae, Warp, Tabnine, Mistral, Qwen, Devin, Droid, ForgeCode, Bob, Crush, iFlow, OpenHands, Qoder, Rovo Dev, Replit, Antigravity, …) | `npx skills add shaonkabir8/injector-skills -a <profile>` | No — user must say `/injector-skills` each session |

opencode reaches Tier 1 minus the statusline (opencode's TUI has no plugin-writable badge). Mode flag lives at `~/.config/opencode/.injector-skills-active` for any external tooling that wants to surface it.

For agents without hook systems, the always-on snippet lives in `INSTALL.md`'s "Want it always on?" section — keep current with `src/rules/injector-skills-activate.md`.

**Adding a new agent.** Edit the `PROVIDERS` array in `bin/install.js` — single source of truth, no more bash/PS1 dual-source drift. Each entry has `id`, `label`, `mech`, `detect` (clause spec like `command:foo||dir:$HOME/x`), optional `profile` (vercel-labs/skills slug), optional `soft: true` (config-dir-only detection).

1. The profile slug must exist in upstream [vercel-labs/skills](https://github.com/vercel-labs/skills). Verify against the README before merging — wrong slugs cause `npx skills add` to fail at runtime, not at install-script load.
2. Run `node bin/install.js --list` to confirm the new row renders correctly.
3. Soft probes (config-dir-only) are fine but tag them with `soft: true`. They render with `(soft)` in `--list` so users know detection is best-effort.

---

## Evals

`evals/` has three-arm harness:
- `__baseline__` — no system prompt
- `__terse__` — `Answer concisely.`
- `<skill>` — `Answer concisely.\n\n{SKILL.md}`

Honest delta = **skill vs terse**, not skill vs baseline. Baseline comparison conflates skill with generic terseness — that cheating. Harness designed to prevent this.

`llm_run.py` calls `claude -p --system-prompt ...` per (prompt, arm), saves to `evals/snapshots/results.json`. `measure.py` reads snapshot offline with tiktoken (OpenAI BPE — approximates Claude tokenizer, ratios meaningful, absolute numbers approximate).

Add skill: drop `skills/<name>/SKILL.md`. Harness auto-discovers. Add prompt: append line to `evals/prompts/en.txt`.

Snapshots committed to git. CI reads without API calls. Only regenerate when SKILL.md or prompts change.

---

## Benchmarks

`benchmarks/` runs real prompts through Claude API (not Claude Code CLI), records raw token counts. Results committed as JSON in `benchmarks/results/`. Benchmark table in README generated from results — update when regenerating.

To reproduce: `uv run python benchmarks/run.py` (needs `ANTHROPIC_API_KEY` in `.env.local`).

---

## Key rules for agents working here

- Edit `skills/<name>/SKILL.md` for behavior changes. Never edit synced copies under `plugins/injector-skills/skills/`.
- Edit `src/rules/injector-skills-activate.md` for auto-activation rule changes. Never edit any per-agent rule copy a user has on their machine.
- Edit `src/rules/injector-skills-openclaw-bootstrap.md` for the OpenClaw SOUL.md bootstrap snippet. Keep the `<!-- injector-skills-begin -->` / `<!-- injector-skills-end -->` markers and the `Respond terse like smart injector-skills` sentinel — `bin/lib/openclaw.js` keys idempotency off both. If you change the embedded fallback in `bin/lib/openclaw.js`, keep it byte-equivalent to the file.
- Per-skill human docs live in `skills/<name>/README.md`. The LLM-facing body is in `SKILL.md`. Don't merge them — different audiences.
- Build artifacts go in `dist/`. Never check files into `dist/` manually — CI rebuilds them on push, and `dist/` is gitignored.
- README most important file for user-facing impact. Optimize for non-technical readers. Preserve injector-skills voice.
- `INSTALL.md` is the per-agent install reference. Keep the install table in `README.md` short and link out to `INSTALL.md` for the full matrix.
- Benchmark and eval numbers must be real. Never fabricate or estimate.
- CI workflow commits back to main after merge. Account for when checking branch state.
- Hook files must silent-fail on all filesystem errors. Never let hook crash block session start.
- Any new flag file write must go through `safeWriteFlag()` in `injector-skills-config.js`. Direct `fs.writeFileSync` on predictable user-owned paths reopens the symlink-clobber attack surface.
- Hooks must respect `CLAUDE_CONFIG_DIR` env var, not hardcode `~/.claude`. Same for `bin/install.js` / statusline scripts.
- `bin/install.js` is the only installer source. `install.sh` / `install.ps1` at repo root are 30-line shims that delegate to it. Never re-add per-OS install logic to the shims — that's how we got the Windows quoting bug (#249).
- Any settings.json read in installer or hooks must go through `bin/lib/settings.js` `readSettings()` so JSONC comments don't crash the merge. Any settings.json write must run through `validateHookFields()` first.
