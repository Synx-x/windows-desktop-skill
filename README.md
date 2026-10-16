<!-- plain: off -->
<div align="center">
  <h1>windows-desktop</h1>
  <p><strong>Drive native Windows apps from any MCP host, reliably, with you watching.</strong></p>
  <p>
    <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT"></a>
  </p>
</div>

`windows-desktop` is an agent skill for the `windows` MCP server. It tells the agent how to find a window, read its UI tree, click, type and paste. It works in SSMS, Notepad, Explorer, desktop Outlook or any Win32, WPF or UWP app.

```bash
npx skills add https://github.com/Synx-x/windows-desktop-skill -g -y
```

Or copy `skills/windows-desktop` into `~/.claude/skills/`. Then ask your agent to drive an app ("run this in SSMS", "click through that dialog"). The skill loads before the first `mcp__windows__*` call.

## Why this skill

Desktop automation fails in small, repeatable ways. Examples are a hidden terminal, a stale window handle, a Ctrl key that never registers, a pasted query run against the wrong database. This skill records the fixes, so the agent stops rediscovering them each session.

## What the agent does

| Step | What happens |
|------|--------------|
| Layout | Terminal and target window split side by side first, so you can watch. The target widens to 40/60 or 30/70 when its UI needs room |
| Target | Every call passes `windowHandle`. The handle is re-found after an app restarts |
| Input | Shortcuts through `keyboard_control` with modifiers, bulk text through the clipboard, waits through `ui_wait` |
| Verify | A click counts as dispatched, not done. The agent checks the result before the next step |
| Quirks | SSMS, locked workstations and Store-stub launches are handled by name |

## Guarantees

- The terminal is never hidden or minimized, and never narrower than 30% of the screen.
- Production servers get read-only `SELECT` statements, unless you ask for a write in the current turn.
- Every pasted SQL batch starts with `USE <db>;`.
- A locked workstation is named as the cause, instead of the tool taking the blame.

## Requires

- Windows 10 or 11, with the target app in your desktop session.
- A `windows` MCP server that exposes `window_management`, `keyboard_control`, `clipboard`, `screenshot_control` and the `ui_*` tools.

## Repository

- `skills/windows-desktop/SKILL.md`: the instructions the agent follows
- `skills/windows-desktop/references/ssms.md`: SSMS quirks
- `LICENSE`: MIT
