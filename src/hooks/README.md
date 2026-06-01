# Caveman Hooks

These hooks are **bundled with the injector-skills plugin** and activate automatically when the plugin is installed. No manual setup required.

If you installed injector-skills standalone (without the plugin), the unified Node installer at `bin/install.js` wires them into your `settings.json` for you — run `node bin/install.js --only claude` from a clone, or `npx -y github:shaonkabir8/injector-skills -- --only claude` for the curl-pipe path.

## What's Included

### `injector-skills-activate.js` — SessionStart hook

- Runs once when Claude Code starts
- Writes `full` to `$CLAUDE_CONFIG_DIR/.injector-skills-active` (default `~/.claude/.injector-skills-active`) via the symlink-safe `safeWriteFlag` helper
- Emits injector-skills rules as hidden SessionStart context
- Detects missing statusline config and emits setup nudge (Claude will offer to help)

### `injector-skills-mode-tracker.js` — UserPromptSubmit hook

- Fires on every user prompt, checks for `/injector-skills` commands and natural-language activation/deactivation phrases ("talk like injector-skills", "stop injector-skills", "normal mode")
- Writes the active mode to the flag file when a injector-skills command is detected; deletes it on deactivation
- Emits a small per-turn reinforcement reminder when the flag is set to a non-independent mode (`lite`/`full`/`ultra`/`wenyan*`)
- Supports: `lite`, `full`, `ultra`, `wenyan`, `wenyan-lite`, `wenyan-full`, `wenyan-ultra`, `commit`, `review`, `compress`

### `injector-skills-statusline.sh` / `injector-skills-statusline.ps1` — Statusline badge script

- Reads `$CLAUDE_CONFIG_DIR/.injector-skills-active` (default `~/.claude/.injector-skills-active`) and outputs a colored badge
- Shows `[CAVEMAN]`, `[CAVEMAN:ULTRA]`, `[CAVEMAN:WENYAN]`, etc.
- Appends the lifetime savings suffix `⛏ 12.4k` from `$CLAUDE_CONFIG_DIR/.injector-skills-statusline-suffix` (written by `injector-skills-stats.js` on each `/injector-skills-stats` run; absent until the first run, so fresh installs render no fake number). Opt out with `CAVEMAN_STATUSLINE_SAVINGS=0`.

## Statusline Badge

The statusline badge shows which injector-skills mode is active directly in your Claude Code status bar.

**Plugin users:** If you do not already have a `statusLine` configured, Claude will detect that on your first session after install and offer to set it up for you. Accept and you're done.

If you already have a custom statusline, injector-skills does not overwrite it and Claude stays quiet. Add the badge snippet to your existing script instead.

**Standalone users:** the unified installer (`bin/install.js`, invoked by the `install.sh` / `install.ps1` shims at the repo root) wires the statusline automatically if you do not already have a custom statusline. If you do, the installer leaves it alone and prints the merge note.

**Manual setup:** If you need to configure it yourself, add one of these to `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash /path/to/injector-skills-statusline.sh"
  }
}
```

```json
{
  "statusLine": {
    "type": "command",
    "command": "powershell -ExecutionPolicy Bypass -File C:\\path\\to\\injector-skills-statusline.ps1"
  }
}
```

Replace the path with the actual script location (e.g. `~/.claude/hooks/` for standalone installs, or the plugin install directory for plugin installs).

**Custom statusline:** If you already have a statusline script, add this snippet to it:

```bash
injector-skills_text=""
injector-skills_flag="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/.injector-skills-active"
if [ -f "$injector-skills_flag" ]; then
  injector-skills_mode=$(cat "$injector-skills_flag" 2>/dev/null)
  if [ "$injector-skills_mode" = "full" ] || [ -z "$injector-skills_mode" ]; then
    injector-skills_text=$'\033[38;5;172m[CAVEMAN]\033[0m'
  else
    injector-skills_suffix=$(echo "$injector-skills_mode" | tr '[:lower:]' '[:upper:]')
    injector-skills_text=$'\033[38;5;172m[CAVEMAN:'"${injector-skills_suffix}"$']\033[0m'
  fi
fi
```

Badge examples:
- `/injector-skills` → `[CAVEMAN]`
- `/injector-skills ultra` → `[CAVEMAN:ULTRA]`
- `/injector-skills wenyan` → `[CAVEMAN:WENYAN]`
- `/injector-skills-commit` → `[CAVEMAN:COMMIT]`
- `/injector-skills-review` → `[CAVEMAN:REVIEW]`

## How It Works

```
SessionStart hook ──writes "full"──▶ $CLAUDE_CONFIG_DIR/.injector-skills-active ◀──writes mode── UserPromptSubmit hook
                                              │
                                           reads
                                              ▼
                                     Statusline script
                                    [CAVEMAN:ULTRA] │ ...
```

SessionStart stdout is injected as hidden system context — Claude sees it, users don't. The statusline runs as a separate process. The flag file is the bridge.

## Uninstall

If installed via plugin: disable the plugin — hooks deactivate automatically.

If installed via the standalone Node installer:
```bash
npx -y github:shaonkabir8/injector-skills -- --uninstall
# or, from a clone:
node bin/install.js --uninstall
```

Or manually:
1. Remove the injector-skills hook files from `$CLAUDE_CONFIG_DIR/hooks/` (default `~/.claude/hooks/`): `injector-skills-activate.js`, `injector-skills-mode-tracker.js`, `injector-skills-stats.js`, `injector-skills-config.js`, and `injector-skills-statusline.{sh,ps1}`.
2. Remove the SessionStart, UserPromptSubmit, and statusLine entries from `$CLAUDE_CONFIG_DIR/settings.json`.
3. Delete `$CLAUDE_CONFIG_DIR/.injector-skills-active` (and `$CLAUDE_CONFIG_DIR/.injector-skills-statusline-suffix` if you ran `/injector-skills-stats`).
