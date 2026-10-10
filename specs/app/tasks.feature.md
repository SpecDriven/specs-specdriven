# Tasks

Simple task manager suitable for a small team.

Support deep link just like Features.

Integrate with Google Calendar

## A bare folder url opens that tab @v1 [proposed]

- Given: the Tasks setting is on and the folder holds at least one task
- When: the user opens `/tasks` — no task after the folder — from the address
  bar, a shared link or a reload
- Then: the Tasks tab opens on the first task it lists, the same one `g t` and
  the collapsed rail's Tasks icon go to, and the address bar canonicalises to
  that task's own url, so a reload reopens it
- and `/roadmap`, `/notebook` and `/models` behave the same way for their own
  tabs, the notebook opening on its newest entry
- and a folder that holds nothing, or whose tab is switched off, goes home
  rather than opening an empty pane

## Add task to Google Calendar @v1 [proposed]

- Given: a task is open, and "Google Calendar" is switched on in Settings —
  off by default, stored like the Tasks switch, and off there is no calendar
  button on any task
- When: the user presses "Add to calendar" in the task's header
- Then: a small dialog proposes how long the task will take — asked of the
  local `claude` CLI in one short turn, with no tools and no session record,
  when the CLI is on PATH; otherwise a picker of 15m, 30m, 1h, 2h, 4h and 8h
  — and a start time, the next quarter hour
- When: the user adjusts either and presses "Add to Google Calendar"
- Then: the estimate is written on the task as one metadata line under its
  heading, `**Estimate:** 1h 30m`, beside the assignment and the status, and
  Google Calendar opens in the browser on a prefilled event: the task's name,
  the start and the end the estimate gives, and the task's text with a link
  back to it — nothing is written to Google without the person's say-so
- Otherwise: the estimate reads `30m`, `2h` or `1h 30m`; a line the app
  cannot read as one, `**Estimate:** two days`, stays in the body as ordinary
  text rather than being lost, and a task that already carries an estimate
  proposes that one instead of asking again

[test: the Google Calendar setting is off unless explicitly turned on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/state.test.ts#L192 ]
[test: the estimate route is 404 while Google Calendar is switched off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/estimate.test.ts#L151 ]
[test: the estimate route asks Claude once, with no tools, and answers in minutes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/estimate.test.ts#L125 ]
[test: the estimate route is 503 without a Claude CLI, 404 for a task that is not there, and 502 when Claude fails : https://github.com/SpecDriven/specdriven-app/blob/main/tests/estimate.test.ts#L159 ]
[test: a turn the API refused carries its own words : https://github.com/SpecDriven/specdriven-app/blob/main/tests/estimate.test.ts#L186 ]
[test: a proposal that is not a number is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/estimate.test.ts#L196 ]
[test: falls back to the presets when no proposal comes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/addToCalendarForm.test.tsx#L47 ]
[test: says when Claude proposed the estimate, and asks while it thinks : https://github.com/SpecDriven/specdriven-app/blob/main/tests/addToCalendarForm.test.tsx#L39 ]
[test: offers the stored estimate, a start time, and the Google Calendar link : https://github.com/SpecDriven/specdriven-app/blob/main/tests/addToCalendarForm.test.tsx#L27 ]
[test: an estimate that does not parse disables the link : https://github.com/SpecDriven/specdriven-app/blob/main/tests/addToCalendarForm.test.tsx#L59 ]
[test: the default start is the next quarter hour : https://github.com/SpecDriven/specdriven-app/blob/main/tests/calendar.test.ts#L78 ]
[test: the Google Calendar link carries the title, UTC start and end, and the details : https://github.com/SpecDriven/specdriven-app/blob/main/tests/calendar.test.ts#L60 ]
[test: an estimate parses minutes, hours and both, and formats back canonically : https://github.com/SpecDriven/specdriven-app/blob/main/tests/calendar.test.ts#L20 ]
[test: an estimate outside one minute to a week, or in words, is not one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/calendar.test.ts#L38 ]
[test: the Estimate line is read and written like the assignment's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/calendar.test.ts#L49 ]
[test: a page parses its Estimate line in any order, and writes it before the status : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tasks.test.ts#L162 ]
[test: a page with an estimate it cannot read keeps the line as body : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tasks.test.ts#L173 ]
[test: a task's estimate is saved and cleared through its document : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tasks.test.ts#L227 ]

## The tasks folder is created and synced @v1 [proposed]

- Given: the Tasks setting is on — it is off unless explicitly turned on
- When: a sync tick runs and the data folder has no 'tasks' folder
- Then: the 'tasks' folder is created, once; a folder that already exists is
  left as it is, its pages untouched
- and while the setting is off, a sync tick never creates it
- and every sync tick commits and pushes every file in the folder, each
  task's document with it, so a peer merges later edits to the task
- and only 'tasks' and the paths under it count as task paths

[test: the Tasks setting is off unless explicitly turned on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tasks.test.ts#L64 ]
[test: ensurePageFolder creates the tasks folder once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tasks.test.ts#L71 ]
[test: a sync tick creates the tasks folder only when the setting is on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tasks.test.ts#L80 ]
[test: a sync tick commits and pushes every file in the tasks folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tasks.test.ts#L95 ]
[test: tasks paths are the folder and what is under it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/tasks.test.ts#L114 ]

## Transform non-complient title to allow save of file @v1 [proposed]

While adding task for 'Explore https://storytime.team/',
encountered system inability to save it.

remove non-complient chars so the task can be named.

Examples

| input                           | acceptable name        |
| ------------------------------- | ---------------------- |
| Explore https://storytime.team/ | Explore storytime.team |
| @@ do this  &&                  | do this                |
| _____do this____                | do this                |
