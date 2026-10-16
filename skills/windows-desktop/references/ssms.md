# SSMS (WPF) quirks

SQL Server Management Studio is a WPF app with a very large UIA tree. These rules keep the
agent from stalling on it.

## Finding controls

- `ui_find` on the whole window often times out. Click toolbar buttons by window-relative
  coordinates from a window screenshot instead.
- To search inside the app, find an observed container first, then search within it.

## Running queries

- **New Query** opens on the login's default database, often `master`. Start every pasted
  batch with `USE <db>;`.
- Paste the batch with the clipboard and Ctrl+V. Never type it key by key.
- One parse error fails the whole batch before anything runs. A bad paste is harmless, but
  read the Messages tab before trusting an empty grid.

## Reading results

- Results below the visible grid need a scroll or a narrower query. Re-running a smaller query
  beats scrolling.
- A results grid cut off at the edge means the target needs more width. Step the layout to
  40/60, then 30/70.

## Production servers

Run read-only `SELECT` statements only, unless the user asks for a write in the current turn.
