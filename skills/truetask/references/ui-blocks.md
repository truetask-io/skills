# The truetask-ui catalog

One fenced block whose info string is `truetask-ui` holds one JSON object.
The object's `type` names a component and the rest of its keys carry that component's data.
Containers (`report`, `section`, `tabs`, `accordion`, `columns`) hold other blocks in `blocks`.

A field marked `md` takes Markdown, including Truetask's own links.
A field marked `short` is one line and is clipped at 300 characters.
`*` marks a field the type cannot do without.

## What gets dropped

The app validates every block before it draws anything, because an agent's output can carry text from a tool result it did not write.

- An unknown `type` means the whole block is shown as its source.
- An unknown field name is dropped without a word. A field whose value fails its check is dropped too, and if it was required the whole component goes.
- Over a cap, the extra items are cut (the first 100 table rows, the first 8 stats) and the rest still draws.
- Raw HTML, scripts, images and style never render. There is no field for them.
- A link survives only as one of Truetask's own chip links, `http(s)` or `mailto`.

So an example that validates today can stop validating when the catalog changes.
Never invent a field because it would read well: if the catalog has no place for something, say it in a `text` block or a `summary`.

## Caps

| What | Cap |
| --- | --- |
| One block's JSON | 40,000 bytes |
| Components in one block | 200 |
| Containers inside containers | 5 deep |
| A Markdown field | 4,000 characters |
| A label, title or cell | 300 characters |
| List, files, properties items | 50 |
| Table | 8 columns, 100 rows |
| Chart | 120 rows, 6 series, a pie 8 slices |
| Steps | 30 |
| Progress bars | 10 |
| Stats tiles | 8 |
| Tabs | 6 |
| Accordion panels | 20 |
| Columns | 2 or 3 |
| Actions | 4, at most one `primary` |

## Status words

`list` and `steps` items take `done`, `active`, `pending`, `skipped`, `blocked` or `failed`.
A `report` takes `done`, `partial`, `blocked`, `failed` or `info`.
A `callout` takes a `tone`: `info`, `success`, `warning` or `danger`.
A `stats` item takes a `tone` of `good`, `bad` or `neutral`, and a `trend` of `up`, `down` or `flat`.
`files` items take a `change` of `added`, `modified`, `deleted` or `renamed`.

## report

`{title*, status, summary: md, blocks: [...]}`

The shape for a finished piece of work, and for any answer to "where does this stand".
Put everything else inside it rather than writing several blocks in a row.

```truetask-ui
{"type": "report", "title": "Importer handles the 1904 date system", "status": "partial", "summary": "Dates from Mac-era workbooks land on the right day now. The timezone case is still open, so I left it out of scope and filed what I know.", "blocks": [{"type": "callout", "tone": "warning", "title": "Still open", "text": "A workbook saved in UTC+13 shifts one day forward. Reproducing it needs a file from that machine."}, {"type": "files", "items": [{"path": "api/v1/_shared/sheet_dates.py", "change": "added", "added": 64, "note": "epoch detection"}, {"path": "api/v1/import_rows.py", "change": "modified", "added": 9, "removed": 4}]}, {"type": "steps", "items": [{"title": "Reproduced with the customer's workbook", "status": "done", "time": "12m"}, {"title": "Fix plus unit tests", "status": "done", "text": "`test_sheet_dates.py`, 14 cases"}, {"title": "Timezone case", "status": "blocked", "text": "Needs a file saved in UTC+13."}]}]}
```

## stats

`{items*: [{label*, value*, hint, trend, tone}]}`

Up to 8 headline numbers.
A value can be a number or a short string.
Never use stats for a series over time: that is a chart.

```truetask-ui
{"type": "stats", "items": [{"label": "Open tasks", "value": 34, "hint": "7 overdue", "tone": "bad"}, {"label": "Done this week", "value": 12, "trend": "up", "tone": "good"}, {"label": "Median age", "value": "6 days", "trend": "flat"}]}
```

## table

`{columns*, rows*, caption}`

Rows that share the same fields.
A column is a string, or `{"label": "Count", "align": "right"}`.
A row is a list of cells in column order, or an object keyed by the column labels.
A column whose cells are all numeric is right-aligned for you.

```truetask-ui
{"type": "table", "caption": "Slowest endpoints, last 24h", "columns": ["Endpoint", {"label": "Calls", "align": "right"}, {"label": "p95", "align": "right"}], "rows": [["GET /v1/cards", "18420", "310 ms"], ["POST /v1/search", "2110", "1.4 s"], ["GET /v1/boards", "9040", "90 ms"]]}
```

## list

`{items*: [{title*: md, text: md, status, meta}], numbered}`

Findings, checks, options: things that each have a state.
An item can be a bare string when it carries nothing but a title.

```truetask-ui
{"type": "list", "items": [{"title": "Token is logged on refresh", "text": "`auth.js:142` writes the bearer into the console on every refresh.", "status": "failed", "meta": "security"}, {"title": "Rate limit missing on the public form", "status": "active", "meta": "api"}, {"title": "Password reset mail renders in dark mode", "status": "done"}]}
```

## files

`{items*: [{path*, change, added, removed, note}]}`

The files you touched, with line counts.
Use it in a report instead of pasting a diff.

```truetask-ui
{"type": "files", "items": [{"path": "svelte/src/lib/components/agent-ui/spec.js", "change": "modified", "added": 18, "removed": 2, "note": "chart series cap"}, {"path": "svelte/src/lib/components/agent-ui/spec.spec.js", "change": "modified", "added": 24}, {"path": "docs/old-ui-notes.md", "change": "deleted", "removed": 61}]}
```

## steps

`{items*: [{title*, status, text: md, time}]}`

What happened, in order.
Up to 30.
`time` is a short label you write yourself, not a timestamp the app formats.

```truetask-ui
{"type": "steps", "items": [{"title": "Read the task and its linked notes", "status": "done", "time": "2m"}, {"title": "Reproduced the bug end to end", "status": "done", "text": "Fresh browser profile, five languages.", "time": "15m"}, {"title": "Fix", "status": "done", "time": "40m"}, {"title": "Browser pass in light and dark", "status": "skipped", "text": "No display on this runner."}]}
```

## progress

`{items*: [{label*, value*, max, hint}]}`

Up to 10 bars.
`max` defaults to 100, so a percentage needs no `max`.

```truetask-ui
{"type": "progress", "items": [{"label": "Locales translated", "value": 3, "max": 5, "hint": "de and pt left"}, {"label": "Checklist", "value": 9, "max": 13}]}
```

## properties

`{items*: [{label*, value*: md}]}`

Facts as label and value, up to 50.
The right shape for settings, an environment, a decision record.

```truetask-ui
{"type": "properties", "items": [{"label": "Branch", "value": "`PROJ-412-importer-dates`"}, {"label": "Stack", "value": "worktree stack on port 5241"}, {"label": "Reviewer", "value": "[Ada Lovelace](#/tt-user/u7k2m9x4qp1wz3t)"}]}
```

## callout

`{tone, title, text: md}`

The one thing the reader must not miss.
One per answer: a second callout makes the first one ordinary.

```truetask-ui
{"type": "callout", "tone": "danger", "title": "Do not deploy yet", "text": "The migration drops `cards.legacy_key` and the importer still reads it. Land the importer change first."}
```

## chart

`{kind*, data*, x, series, stacked, unit, title}`

`kind` is `bar`, `line`, `area` or `pie`.
`x` names the label key, default `label`.
Name `series` when you want your own labels or a fixed order; otherwise every numeric key of the first row becomes one.
120 rows, 6 series, and a pie keeps 8 slices.

```truetask-ui
{"type": "chart", "kind": "line", "title": "Open tasks per week", "x": "week", "series": [{"key": "open", "label": "Open"}, {"key": "done", "label": "Done"}], "data": [{"week": "W36", "open": 31, "done": 8}, {"week": "W37", "open": 34, "done": 11}, {"week": "W38", "open": 29, "done": 14}, {"week": "W39", "open": 22, "done": 19}]}
```

```truetask-ui
{"type": "chart", "kind": "bar", "title": "Tasks per list", "unit": "tasks", "data": [{"label": "Ideas", "count": 12}, {"label": "Todo", "count": 7}, {"label": "In Progress", "count": 3}, {"label": "Review", "count": 2}]}
```

## text

`{text*: md}`

A paragraph inside a container, when the thing you have to say is prose.
Outside a container, write the sentence as plain text before the block instead.

```truetask-ui
{"type": "section", "title": "Why the first fix did not hold", "blocks": [{"type": "text", "text": "The listener ran before the board's lists arrived, so the card landed in the first list it could find. Waiting for the lists is the fix; the earlier guard only made the race rarer."}]}
```

## section

`{title, blocks*}`

A titled group inside a report.
It needs blocks: an empty section is dropped.

```truetask-ui
{"type": "report", "title": "Release 1.3.106 checks", "status": "done", "blocks": [{"type": "section", "title": "Backend", "blocks": [{"type": "list", "items": [{"title": "Suite", "text": "412 passed", "status": "done"}, {"title": "Release images start", "status": "done"}]}]}, {"type": "section", "title": "Frontend", "blocks": [{"type": "list", "items": [{"title": "vitest", "text": "2,322 tests", "status": "done"}, {"title": "lint", "text": "0 errors", "status": "done"}]}]}]}
```

## tabs, accordion, columns

`tabs {tabs*: [{label*, blocks*}]}` up to 6.
`accordion {items*: [{title*, blocks*, open}]}` up to 20.
`columns {columns*: [{title, blocks*}]}`, 2 or 3, and a column's title is optional.

Use them when a reader needs one part at a time.
Do not wrap a single list in a tab.

```truetask-ui
{"type": "tabs", "tabs": [{"label": "Chrome", "blocks": [{"type": "list", "items": [{"title": "Light", "status": "done"}, {"title": "Dark", "status": "done"}]}]}, {"label": "Safari", "blocks": [{"type": "list", "items": [{"title": "Light", "status": "done"}, {"title": "Dark", "status": "failed", "text": "Selection colour is unreadable on the inset token."}]}]}]}
```

```truetask-ui
{"type": "accordion", "items": [{"title": "What I changed", "open": true, "blocks": [{"type": "files", "items": [{"path": "api/v1/webhooks.py", "change": "modified", "added": 31, "removed": 6}]}]}, {"title": "Why the retry is 5 minutes", "blocks": [{"type": "text", "text": "Three customers' endpoints answer in under a minute after a deploy. A shorter retry doubled the duplicate deliveries without catching more of them."}]}]}
```

```truetask-ui
{"type": "columns", "columns": [{"title": "Before", "blocks": [{"type": "properties", "items": [{"label": "Cold start", "value": "2.4 s"}, {"label": "Bundle", "value": "1.1 MB"}]}]}, {"title": "After", "blocks": [{"type": "properties", "items": [{"label": "Cold start", "value": "0.9 s"}, {"label": "Bundle", "value": "740 kB"}]}]}]}
```

## actions

`{items*: [{label*, reply | href | copy, primary}]}`

The next moves, as buttons.
At most 4, at most one `primary`.

- `reply` sends those words as the person's next message. A bare label does this with the label itself.
- `href` takes a Truetask chip link (`#/tt-card/...`, `#/tt-note/...`, `#/tt-board/...`) or a web page.
- `copy` puts text on their clipboard.

```truetask-ui
{"type": "actions", "items": [{"label": "Land it", "reply": "Land the change on main", "primary": true}, {"label": "Open the task", "href": "#/tt-card/c4n8v2b6m1q9r5s/b3x7h1k5p9t2w6y"}, {"label": "Copy the install command", "copy": "npx skills add truetask-io/skills"}]}
```

## Links inside a block

Any `md` field takes Truetask's own links, and they open in place:

- a task: `[Title](#/tt-card/<card_id>/<board_id>)`
- a board: `[Name](#/tt-board/<board_id>)`
- a person: `[Name](#/tt-user/<user_id>)`
- a note: `[Title](#/tt-note/<note_id>/<board_id>)`
- a form: `[Title](#/tt-form/<form_slug>)`

Never show a raw 15-character id as visible text.
An id belongs in the href; the link text is the name.
