---
name: windows-desktop
description: "Windows desktop observation and control through a windows MCP server: window targeting by handle, UI Automation reads, screenshots, and reliable keyboard, mouse and clipboard input. Use before the first mcp__windows__* call, when driving SSMS, Notepad, Explorer, desktop Outlook or any Win32, WPF or UWP window. Triggers on: drive, computer use, use the desktop, open SSMS, run it in SSMS, click in, control the window."
author: Synx-x
license: MIT
platforms: [windows]
compatibility: "Windows 10 or 11 and a windows MCP server that exposes window_management, keyboard_control, clipboard, screenshot_control and the ui_* tools."
---

# windows-desktop

Use `windows-desktop` when an agent needs to observe or operate a local Windows app: find a
window, read its UI tree, take a screenshot, click, type, paste or press keys.

## When to Use

Use this skill when:

- The user wants the agent to control a native Windows app (SSMS, Notepad, Explorer, desktop Outlook).
- You need window state from UI Automation or a window screenshot.
- You are about to make the first `mcp__windows__*` call in a session.
- A desktop action needs a targeted window instead of a blind shell command.

Do not use this for websites or headless automation when a browser tool is available. Do not
treat an app as safe to change because the MCP connection works.

## Setup

1. Call `window_management(action='find', processName='WindowsTerminal')`.
2. If the tool is missing, the `windows` MCP server is not connected. Add it to the host's MCP
   config, restart the host, and repeat step 1.
3. If the call succeeds but `find` returns nothing, the app is not in this desktop session, or
   the workstation is locked (see [App quirks](#app-quirks)).

## Procedure

1. Lay out the terminal and the target side by side ([Layout](#layout-first)).
2. `find` the target and keep its handle. Pass `windowHandle` on every later call.
3. Activate the target before keyboard or mouse input.
4. Orient with `ui_snapshot` or a window screenshot, then act.
5. Check the result with `ui_wait` or a fresh screenshot. Never trust a dispatched click alone.
6. Re-`find` the handle after any app restart.

## Layout first

Put the terminal and the target window side by side before any click or keystroke. The user
watches the terminal while the app is driven, so neither window may hide the other.

1. Find both windows with `window_management(action='find', processName=...)`.
2. Read the work area with `screenshot_control(action='list_monitors')`. Use the primary
   monitor's `workArea`, which excludes the taskbar.
3. Split 50/50 with `window_management(action='set_bounds', ...)`:
   - terminal: `x = workArea.x`, `width = workArea.width / 2`, full work-area height.
   - target: `x = workArea.x + workArea.width / 2`, `width = workArea.width / 2`, full height.
   - A maximized window ignores `set_bounds`, so `restore` it first.
4. Give the target more width when its UI needs it: step to 40/60, then 30/70. Never go below
   30% for the terminal, and never minimize it.
5. Re-check the layout after launching or re-opening an app, because new windows open maximized.

## Target by handle

- Every input call passes `windowHandle`. A restart changes the PID and the handle.
- Activate the target (`window_management action='activate'`) before keyboard or mouse input.
- Mouse coordinates passed with `windowHandle` are window-relative. Take them from
  `screenshot_control(target='window', windowHandle=..., annotate=false, includeImage=true)`,
  whose pixels match window coordinates.

## Input that works

- **Shortcuts:** `keyboard_control(action='press', key='v', modifiers='ctrl')`. Never put the
  modifier in `key`. PowerShell `SendKeys` drops the Ctrl on the first key after activation and
  types a plain letter, so do not use it.
- **Bulk text** (SQL, code): `clipboard(action='set', text=...)` then Ctrl+V. Pass raw
  characters. Never HTML-escape `<`, `>` or `&` in clipboard text.
- **Waiting:** prefer `ui_wait` or a fresh window screenshot over blind sleeps.

## App quirks

- **SSMS (WPF):** see [SSMS](references/ssms.md).
- **Locked workstation:** clicks land on `LockApp`, activation fails, and UWP apps never expose
  UIA. Check for `LockApp` before blaming the tool. UIA read and type on Win32 apps such as
  Notepad still work.
- **Store-stub launches** (`notepad.exe`, `calc.exe`) report "no window observed". The real
  window belongs to another PID, so follow the launch with `window_management find`.

## Safety

- Production servers get read-only `SELECT` statements, unless the user asks for a write in
  the current turn.
- Confirm the intended window by title, process or handle before targeted input.
- A click success means dispatched, not verified. Verify the outcome before the next step.
