# Loop task tracker — a daily kanban board

A single-file, local-first kanban board for recurring and one-off daily tasks. No login, no backend, no tracking.

Built to run straight off GitHub Pages as one HTML file.

**Live:** https://bravowhiskybravo.github.io/Loop_task_tracker/

## Features

- **Kanban columns** — To do / Doing / Done per list, double-tap or drag-and-drop a card to move it
- **Move between lists** — drag a card onto a different list's column, or pick a new list/column from the item's edit view — no delete-and-retype needed
- **Clock & timer** — live date/time in the top-left corner; click it to start a countdown (presets or custom minutes). Ends with a visual flash, an in-page beep, and a toast. Survives a page refresh, but — like notifications — only alerts while the tab is open
- **Recurrence** — repeat items daily, every X days, weekly (with specific weekdays), monthly, or yearly
- **Rollover** — items left in To do or Doing past end-of-day get flagged and carried into today automatically
- **Backfill (per list, toggleable)** — if you don't open Loop for a few days, recurring items either catch up one instance per missed day (backfill on) or jump straight to today (backfill off)
- **Retention (per list)** — keep completed items forever, or auto-clear them after a set number of days
- **Multi-select** — bulk move, delete, or clear the rollover flag on several cards at once
- **Undo** — a brief toast with an undo action follows deletes, column moves, list deletes, and destructive imports
- **Export** — whole board or single list, as JSON (full fidelity), CSV, or plain text
- **Import** — JSON only, with three explicit modes: **View** (preview, no changes), **Merge** (add into existing board, with prompts if list names collide), **Replace** (wipe and reload)
- **Sharing** — export a list and send the file; no accounts, no real-time collaboration in v1

## Tech stack

- Vanilla HTML/CSS/JS, no build step, no framework, no dependencies
- Storage: IndexedDB, with an automatic localStorage fallback if IndexedDB is unavailable or its connection drops
- Fonts: Permanent Marker + Kalam (handwritten/marker feel) for headings and task text, IBM Plex Mono for controls and metadata

## Deploying to GitHub Pages

1. Drop `kanban.html` into a repo (rename to `index.html` if you want it at the root of the Pages URL)
2. Settings → Pages → set the source branch/folder
3. Done — it's fully static, nothing else to configure

You can also just open the file directly in a browser (`file://`) for local use, though IndexedDB behaves more reliably served over `http(s)`.

## Data model (JSON export shape)

```json
{
  "meta": { "appVersion": "1.0", "exportedAt": "..." },
  "settings": {
    "defaultRetentionMode": "keep | days",
    "defaultRetentionDays": 30,
    "defaultBackfillEnabled": true,
    "defaultEodTime": "00:00"
  },
  "lists": [
    {
      "id": "list_...",
      "name": "Daily Chores",
      "retentionMode": "keep | days",
      "retentionDays": 30,
      "backfillEnabled": true,
      "eodTime": "00:00",
      "columns": ["todo", "doing", "done"],
      "items": [
        {
          "id": "item_...",
          "seriesId": "series_...",
          "title": "Take out trash",
          "column": "todo | doing | done",
          "scheduledFor": "YYYY-MM-DD",
          "completedAt": null,
          "recurrence": {
            "type": "daily | everyXDays | weekly | monthly | yearly",
            "interval": 1,
            "daysOfWeek": null,
            "anchorDate": "YYYY-MM-DD"
          },
          "rollover": { "isRolledOver": false, "rolledFrom": null }
        }
      ]
    }
  ]
}
```

Recurring items are flat, not template + instance: a card carries its own recurrence rule, and the next occurrence spawns automatically once the current one reaches Done. All occurrences of the same recurring item share a `seriesId`. `recurrence` is omitted entirely for one-off items. Moving a recurring item to a different list keeps it repeating — future occurrences just inherit that list's backfill/retention/end-of-day settings instead of its original list's.

## Known limitations (v1)

- **No push notifications.** Static hosting can't wake the app in the background — reminders, and the built-in timer alert, only fire while the tab is open.
- **No real-time collaboration.** Sharing is export/import only; two people editing the same list won't sync live.
- **Browser storage isn't guaranteed forever.** Safari and some mobile browsers can evict local storage if the site goes unvisited for a long stretch. Export a JSON backup periodically if your data matters.
- **CSV/text exports are one-way.** Only JSON round-trips back into the app.

## Backlog / ideas for later

- Native push notifications via a wrapped app (Capacitor/Tauri) or a small backend
- Real-time collaborative lists (would need a sync layer)
- Custom columns beyond To do / Doing / Done
- CSV/text import
