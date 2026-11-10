# Loop — Task Tracker

A single-file, local-first kanban board for recurring and one-off daily tasks. Entries live in cookies, no account needed. 

Built to run off GitHub Pages as one HTML file. (link below) 

**Live:** https://bravowhiskybravo.github.io/Loop_task_tracker/

## Features

- **Kanban columns** — To do / Doing / Done per list, double-tap or drag-and-drop a card to move it
- **Move between lists** — drag a card onto a different list's column, or pick a new list/column from the item's edit view.
- **Card order** — drag a card up or down within a column (or between cards in another list) to place it exactly where you want.
- **Reorder lists** — use a list's ⋯ menu to move it left or right.
- **Multi-select** — bulk move, delete, or clear the rollover flag on several cards at once.
- **Search & filter** — the header search box matches titles, notes and labels across every list; the ☀ button shows only cards scheduled for today or overdue.
- **Undo** — a brief notification with an undo action follows deletes, column moves, list deletes, and destructive imports.

- **Recurrence** — repeat items daily, every X days, weekly (with specific weekdays), monthly, or yearly. A repeating item is a single card for good: when it's done and due again (the next day for a daily item), the same card resets to To do for its next date. No duplicates (hopefully).
- **Rollover** — one-off items left in To do or Doing past end-of-day get flagged and carried into the next day. Repeating items default to *not* rolling over (a missed card just resets for its next date); open the item and set **If missed → Roll over** to change that per item
- **Notes** — each item has a notes box that only shows in the edit view (right-click a card). Cards with notes show a small dark dot, useful for additional info, optional use. 
- **Labels & colors** — give a card an optional label with its own color, and change the To do / Doing / Done card colors in Settings.
- **Retention (per list)** — keep completed one-off items forever, or auto-clear them after a set number of days. Repeating items are exempt from the rule. 
- **Watch a web address** — a card can watch a URL and come back to To do when its value changes. See [Watching a web address](#watching-a-web-address)


- **Clock & timer** — live date/time in the top-left corner; click it to start a countdown (presets or custom minutes). Ends with a visual flash, an in-page beep, and a notification. Survives a page refresh. Note: only alerts while the tab is open
- **Export** — whole board or single list, as JSON (full fidelity), CSV, or plain text.
- **Import** — JSON only, with three explicit modes: **View** (preview, no changes), **Merge** (add into existing board, with prompts if list names collide), **Replace** (wipe and reload).
- **Calendar import (.ics)** — pull events from Google, Apple or Outlook Calendar into a list. Past events are skipped, with an adjustable forward window (default 90 days). Calendar events don't roll over by default; set an event's **If missed** toggle to Roll over to change that. See [Importing a calendar](#importing-a-calendar-ics)
- **Backup reminders** — Loop asks the browser to keep its data permanently and nudges you to export a backup after a set number of days or edits. See [Backup reminders](#backup-reminders)
- **Keyboard shortcuts** — see [Keyboard shortcuts](#keyboard-shortcuts)
- **Help** — the **?** button in the header opens this README in a new tab.


## Tech stack

- HTML/CSS/JS all you need is your browser. 
- Storage: IndexedDB, with an automatic localStorage fallback if IndexedDB is unavailable or its connection drops
- Fonts: Permanent Marker + Kalam for headings and task text, IBM Plex Mono for controls and metadata



## Watching a web address

Open a card's edit view (right-click) and fill in **Watch a web address** with an `https://` URL. An optional **match pattern** (a regular expression) narrows what is compared: the first capture group, or the whole match if there is no group. Press **Test now** to see the current value or a clear error before saving.

- Loop checks when it opens, when you return to the tab, and about every 5 minutes while open (each card at most every ~30 minutes). The first check only records a baseline.
- When the value changes, a card in **Done** moves back to **To do**, gets a "● changed" marker, and you see a notification. A card in Doing stays put but gets the marker. Moving the card to Done clears it, and the newest value becomes the new baseline.
- Changes that happen while Loop is closed are caught the next time it opens, because Loop compares against the last value it stored.
- With no pattern the whole response is compared, which can false-alarm on busy pages. For JSON, a pattern such as `"tag_name":\s*"([^"]+)"` watches just one field.
- The site must allow browser access (CORS). If it doesn't, the card shows a "⚠ watch" tag. A small helper on a server you control is the workaround.

## Backup reminders

Everything lives in your browser, so Loop asks the browser to keep its storage permanently and reminds you to back up. After the number of days or edits set in **Settings → Backup reminder** (default 14 days or 100 edits), a notification offers three choices:

- **DO IT** — exports the whole board as JSON
- **LATER** — adds a "Back up Loop" card to your first list (removed automatically after your next backup) and pauses the reminder for 3 days
- **FORGET** — dismisses it and pauses the reminder for a day

## Keyboard shortcuts

| Key | Action |
| --- | --- |
| `/` | Focus search |
| `n` | New item in the first list |
| `l` | New list |
| `s` | Toggle select mode |
| `t` | Set a timer |
| `?` | Open help (this README) |
| `Esc` | Close a dialog, or clear the search |

Shortcuts are ignored while you're typing in a field or while a dialog is open.

## Importing a calendar (.ics)

Import → "Calendar file (.ics)". You get a preview first (target list, forward window, what will be added or skipped) and nothing changes until confirmation.

**How to get an .ics file** (menu names change between versions — if you can't find it, search "export <your calendar> to ics"):

- **Google Calendar** (on a computer, not the mobile app): Settings (gear) → Settings → Import & export → Export. You get a .zip with one .ics per calendar — unzip it and import the one you want.
- **Apple Calendar** (Mac): click the calendar in the sidebar → File → Export → Export… to save it as an .ics. Note: iPhone/iPad can't export a file directly.
- **Outlook** (classic desktop): File → Save Calendar, and save as iCalendar (.ics). Outlook on the web / new Outlook: Calendar → Settings → Shared calendars → Publish a calendar, then open the ICS link in a browser to download the file.

**What gets imported**

- Past events are skipped. Repeating events import from their next upcoming occurrence. Multi-day events stay until they end.
- Only events within the forward window come in (default 90 days; blank or 0 = no limit). Your last value is remembered.
- Event times show as a tag on the card. Events with an explicit timezone use that timezone's clock time as written; UTC times convert to your local time.
- Re-importing the same calendar updates existing cards instead of duplicating them, and leaves your column and roll-over choices alone.
- Simple repeats (daily, every X days, weekly with weekdays, monthly, yearly) become normal Loop recurrence. Complex rules like "second Tuesday" can't be converted: if still upcoming they import as a single event, otherwise they're skipped and counted in the preview.
- Calendar events don't roll over if missed, which can be chaged in card edit. A repeating calendar event that was missed resets to its next occurrence instead of sitting on an old date.



## Data model (JSON export shape)

```json
{
  "meta": { "appVersion": "1.0", "exportedAt": "..." },
  "settings": {
    "defaultRetentionMode": "keep | days",
    "defaultRetentionDays": 30,
    "defaultEodTime": "00:00"
  },
  "lists": [
    {
      "id": "list_...",
      "name": "Daily Chores",
      "retentionMode": "keep | days",
      "retentionDays": 30,
      "eodTime": "00:00",
      "columns": ["todo", "doing", "done"],
      "items": [
        {
          "id": "item_...",
          "seriesId": "series_...",
          "title": "Write README.md",
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

Repeating items are flat: a card carries its own recurrence rule, and there is exactly one card per series (all cards of a series share a `seriesId`; if older data or an import contains several, they're merged into the newest on load, with an undo notification). `recurrence` is omitted entirely for one-off items. Moving a repeating item to a different list keeps it repeating under that list's retention/end-of-day settings. Items can also carry optional `notes` and `sticky` (true = roll over if missed). Items imported from a calendar additionally carry `source`, `externalId` (the event's UID) and `eventTime`, and their `recurrence` may include an `until` date. Other optional item fields: `watch` (url, pattern and last-seen state), `label` and `labelColor`, and `pos` (manual order within a column). Settings also store card colors and the backup-reminder state. Older exports that still contain backfill fields load fine; the setting is no longer used.


## Known limitations

- **No push notifications.** Static hosting can't wake the app in the background.
- **No real-time collaboration.** Sharing is export/import only; two people editing the same list won't sync live.
- **Browser storage isn't guaranteed forever.** Safari and some mobile browsers can evict local storage if the site goes unvisited for a long stretch. **Export a JSON backup periodically if your data matters.**
- **CSV/text exports are one-way.** Only JSON round-trips back into the app.
- **Calendar import is a file import, not a live sync.** Re-import the .ics to refresh.
- **URL watchers need browser access and an open tab.** Sites that don't allow cross-origin requests can't be watched directly, and checks only run while Loop is open.
