# Loop — a daily kanban board

A single-file, local-first kanban board for recurring and one-off daily tasks. No login, no backend, no tracking — everything lives in your browser.

Built to run straight off GitHub Pages as one HTML file.

**Live:** https://bravowhiskybravo.github.io/Loop_kanban/

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
- **Calendar import (.ics)** — pull events from Google, Apple or Outlook Calendar into a list. Past events are skipped, with an adjustable forward window (default 90 days). Calendar events don't roll over by default; mark an event "sticky" to make it roll over. See [Importing a calendar](#importing-a-calendar-ics)
- **Sharing** — export a list and send the file; no accounts, no real-time collaboration in v1

## Tech stack

- Vanilla HTML/CSS/JS, no build step, no framework, no dependencies
- Storage: IndexedDB, with an automatic localStorage fallback if IndexedDB is unavailable or its connection drops
- Fonts: Permanent Marker + Kalam (handwritten/marker feel) for headings and task text, IBM Plex Mono for controls and metadata

## Importing a calendar (.ics)

Import → "Calendar file (.ics)". You get a preview first (target list, forward window, what will be added or skipped) and nothing changes until you confirm.

**How to get an .ics file** (menu names change between versions — if you can't find it, search "export <your calendar> to ics"):

- **Google Calendar** (on a computer, not the mobile app): Settings (gear) → Settings → Import & export → Export. You get a .zip with one .ics per calendar — unzip it and import the one you want.
- **Apple Calendar** (Mac): click the calendar in the sidebar → File → Export → Export… to save it as an .ics. iPhone/iPad can't export a file directly, so use a Mac.
- **Outlook** (classic desktop): File → Save Calendar, and save as iCalendar (.ics). Outlook on the web / new Outlook: Calendar → Settings → Shared calendars → Publish a calendar, then open the ICS link in a browser to download the file.

**What gets imported**

- Past events are skipped. Repeating events import from their next upcoming occurrence; ones that have ended are skipped. Multi-day events stay until they end.
- Only events within the forward window come in (default 90 days; blank or 0 = no limit). Your last value is remembered.
- Event times show as a tag on the card. Events with an explicit timezone use that timezone's clock time as written; UTC times convert to your local time.
- Re-importing the same calendar updates existing cards instead of duplicating them, and leaves your column and sticky choices alone.
- Simple repeats (daily, every X days, weekly with weekdays, monthly, yearly) become normal Loop recurrence. Complex rules like "second Tuesday" can't be converted: if still upcoming they import as a single event, otherwise they're skipped and counted in the preview.
- Calendar events don't roll over if missed. Open one and set "If missed" to "Roll over (sticky)" to change that. An untouched repeating event moves to its next occurrence instead of sitting on an old date.

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

Recurring items are flat, not template + instance: a card carries its own recurrence rule, and the next occurrence spawns automatically once the current one reaches Done. All occurrences of the same recurring item share a `seriesId`. `recurrence` is omitted entirely for one-off items. Moving a recurring item to a different list keeps it repeating — future occurrences just inherit that list's backfill/retention/end-of-day settings instead of its original list's. Items imported from a calendar also carry optional `source`, `externalId` (the event's UID), `sticky` and `eventTime` fields, and their `recurrence` may include an `until` date.

## Known limitations (v1)

- **No push notifications.** Static hosting can't wake the app in the background — reminders, and the built-in timer alert, only fire while the tab is open.
- **No real-time collaboration.** Sharing is export/import only; two people editing the same list won't sync live.
- **Browser storage isn't guaranteed forever.** Safari and some mobile browsers can evict local storage if the site goes unvisited for a long stretch. Export a JSON backup periodically if your data matters.
- **CSV/text exports are one-way.** Only JSON round-trips back into the app.
- **Calendar import is a file import, not a live sync.** Re-import the .ics to refresh.

## Backlog / ideas for later

- Native push notifications via a wrapped app (Capacitor/Tauri) or a small backend
- Real-time collaborative lists (would need a sync layer — breaks the pure local-first model, so deliberately deferred)
- Custom columns beyond To do / Doing / Done
- Series-level edit scope (edit this occurrence vs. all future occurrences — currently editing always applies forward)
- CSV/text import
- Google Calendar direct connect (sign-in instead of exporting a file)


