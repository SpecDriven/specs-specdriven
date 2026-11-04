# Pomodoro

**Created by:** vydra

**Updated by:** vydra

Not all code should be written by AI agents.

The Pomodoro technique times the work of the person at the keyboard. It is a
feature of the app, not a kind of session: one timer belongs to the person,
however many AI sessions are running in parallel.

- The timer is **off until asked for**: a switch in Settings, device-local
  like Vim mode and the theme. Off, there is no bar, no `p` key and no
  entries in the command palette.
- It is started in **one click** — the **Start a pomodoro** button at the foot
  of the sidebar, the `p` key, the rail's timer chip, or the command palette.
  No name, no dialog.
- There is **no session record**. The timer lives in memory for as long as the
  app is open; a reload leaves no pomodoro running. Nothing is written to disk
  but the device-local preferences. This is the deliberate cost of keeping the
  timer out of the data repo, which is pushed to git.
- The intervals are **Focus**, **Short break** and **Long break** (1–180
  minutes; the long break every 1–12 pomodoros). The defaults are 25/5/15
  minutes, a long break every 4.
- A pomodoro is *indivisible*: it either runs out (`completed`) or is voided
  (`abandoned`). It is never paused. A break may be cut short — also
  `abandoned`. Abandoned time still counts as time spent; only completed work
  intervals count as pomodoros.
- After every break comes focus; after a completed pomodoro comes the short
  break, or the long one when the cycle is full; after a voided pomodoro, the
  pomodoro again.
- The clock is never a counter: what remains is the running interval's planned
  end minus the time now, so a laptop asleep for ten minutes lands on the
  right number.
- One timer for the person: starting a pomodoro while one runs changes
  nothing.
- The interval lengths, the sound and the tick are device-local preferences,
  reached from the bar's ⋯ menu. Changing them affects the **next** interval;
  the running one keeps what it started with.
- The bar is a lot of a narrow sidebar, so it is a pane rather than furniture:
  a **timer icon in the sidebar's header**, left of the coding agent's robot,
  opens and closes it, and closing it does not stop the timer.

## S1: a pomodoro cannot be paused, only abandoned @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the user presses "Abandon" during focus
- Then: the interval is recorded as abandoned, it fills no dot, its time still
  counts toward the focus so far, and the next interval offered is the
  pomodoro again
- Otherwise: there is no pause control — a pomodoro is indivisible

[test: abandon voids a pomodoro, skip cuts a break, and each refuses the other : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L231 ]
[test: work to begin with, and again after a voided pomodoro : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroRules.test.ts#L47 ]
[test: splits work from breaks, counts abandoned time, and only completed pomodoros : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroRules.test.ts#L87 ]

## S2: a pomodoro runs out and a break is offered @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the running Focus interval reaches its planned end
- Then: it is recorded as completed *at its planned end* rather than at the
  moment the app noticed, the chosen sound plays, the bar flashes, and a
  notice offers "Start break" — the button that begins the next interval

[test: a tick before the end only moves the clock; at the end the pomodoro completes at its planned end : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L178 ]
[test: closeInterval ends the open one at now or at the planned end : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroRules.test.ts#L120 ]
[test: flashes when told to, and the menu offers the settings and to stop : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L58 ]

## S3: interval lengths are remembered on this machine @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the user changes Focus, Short break, Long break or "Long break every"
  in the settings dialog and saves
- Then: the next time the dialog opens it holds those numbers, and the next
  interval runs on them
- Otherwise: a value is out of range or not a number, and it is clamped to
  1–180 minutes (1–12 for "Long break every") rather than lost

[test: defaults when nothing or junk is stored; each field clamps on its own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L99 ]
[test: load takes the stored preferences, and savePrefs keeps them for the next timer : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L110 ]
[test: the form's text becomes clamped preferences : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroSettings.test.tsx#L55 ]
[test: shows the stored preferences, and no preview for silence : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroSettings.test.tsx#L39 ]

## S4: one timer, however many sessions @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a pomodoro is already running
- When: the user starts a pomodoro again — the button, the `p` key or the
  palette
- Then: nothing changes: the running interval keeps its clock and its
  settings, and no second timer appears
- Otherwise: the running interval is what the person is doing; the AI sessions
  going in parallel have no timer of their own

[test: starting while one runs changes nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L170 ]
[test: appendInterval opens one, and refuses a second while it runs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroRules.test.ts#L114 ]
[test: only the last interval can be open : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroRules.test.ts#L104 ]

## S5: preview the sound that announces a break @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the user picks a sound in the settings dialog and presses the ▶ button
- Then: that sound plays at once
- Otherwise: the sound is "None", and the ▶ button is disabled

[test: offers the four lengths, the sounds with a preview, the tick, and Save : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroSettings.test.tsx#L20 ]
[test: shows the stored preferences, and no preview for silence : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroSettings.test.tsx#L39 ]

## S6: settings saved mid-pomodoro land on the next interval @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a pomodoro is running
- When: the user opens "Pomodoro settings…" from the bar's ⋯ menu and saves
  new lengths
- Then: the dialog says so, the running interval keeps the length it started
  with, and the change takes effect from the next interval
- Otherwise: no timer is running, and the next pomodoro simply starts on the
  new lengths

[test: while a pomodoro runs, says the change lands on the next one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroSettings.test.tsx#L47 ]
[test: load takes the stored preferences, and savePrefs keeps them for the next timer : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L110 ]
[test: come from the timer's settings : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroRules.test.ts#L38 ]

## S7: skip a break @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the user presses "Skip" during a short or long break
- Then: the break ends as abandoned and the bar offers the next pomodoro
- Otherwise: the interval is a pomodoro, and "Skip" is not offered — as
  "Abandon" is not offered during a break

[test: abandon voids a pomodoro, skip cuts a break, and each refuses the other : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L231 ]
[test: during a break — Skip, the break class, and a coffee : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L39 ]
[test: work after any break, and the cycle restarts after a long break of any outcome : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroRules.test.ts#L60 ]

## S8: start a pomodoro @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the Pomodoro is switched on in Settings
- When: the user presses "Start a pomodoro" in the bar under the sidebar's
  header, the `p` key, the rail's timer chip, or "Start a pomodoro" in the
  command palette
- Then: the first pomodoro is running at once, on the lengths last set, and
  the bar takes the place of the button
- Otherwise: there is nothing to fill in — no name and no dialog; a pomodoro
  is something started in the second the person decides to focus

[test: start unlocks audio, keeps the prefs, and opens the first pomodoro at once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L155 ]
[test: with no timer running, one button starts a pomodoro — no name, no dialog : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L86 ]
[test: the rail's idle chip is a timer icon that starts one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L104 ]

## S9: stop the timer @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: the user chooses "Stop Pomodoro" from the bar's ⋯ menu, or from the
  command palette
- Then: the timer switches off, a running interval goes with it, the window
  title's prefix goes, and the bar becomes the invitation to start one again
- Otherwise: no timer is running, and stopping changes nothing

[test: stop switches the timer off, and the title prefix goes with it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L250 ]
[test: nothing happens without a timer : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L261 ]
[test: flashes when told to, and the menu offers the settings and to stop : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L58 ]

## S10: the bar shows where the timer stands @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Then: the Pomodoro bar shows the phase, the clock counting down, a progress
  line, one dot per completed pomodoro in the current cycle, and the focus
  time so far
- Otherwise: the sidebar is collapsed, and the rail shows the clock alone as a
  chip that brings the sidebar back when clicked

[test: during focus — the phase, the clock, progress, dots, total, and Abandon : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L25 ]
[test: the rail chip is the clock alone, and a timer icon between intervals : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L74 ]
[test: fills one dot per completed pomodoro, capped at the cycle length : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroRules.test.ts#L78 ]
[test: between intervals — no clock, and the button that starts what comes next : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L48 ]

## S11: the header's icon opens and closes the bar @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: the Pomodoro is switched on
- Then: a timer icon sits in the sidebar's header, left of the coding agent's
  robot, and the bar opens directly beneath the header
- When: the user presses that icon, or the bar's own close button
- Then: the bar goes, and the clock, the progress line, the cycle's dots and
  the buttons go with it — the space is given back to the work
- and a running pomodoro keeps running: the header's icon carries its clock,
  coloured for a break, and the window title still counts down
- When: the user presses the icon again
- Then: the bar comes back as it was
- Otherwise: the sidebar is collapsed to the rail, which shows its chip
  either way

[test: closed and idle — a bare timer icon that opens the bar : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L122 ]
[test: open, the same icon closes the bar again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L135 ]
[test: while a pomodoro runs, the icon carries its clock : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L142 ]
[test: a break reads as one, and the flash carries over : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L148 ]
[test: the bar can be closed without stopping the timer : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroBar.test.tsx#L66 ]
[test: closes and stays closed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroPref.test.tsx#L74 ]
[test: is its own question — closing the bar leaves the feature on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroPref.test.tsx#L87 ]

## S12: the long break comes when the cycle is full @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: as many pomodoros have been completed since the last long break as
  "Long break every" allows
- When: the running pomodoro completes
- Then: the break offered is the long one, and the cycle's dots start again
  after it, whether it ran out or was skipped

[test: a short break after a pomodoro, the long one when the cycle is full : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroRules.test.ts#L52 ]
[test: work after any break, and the cycle restarts after a long break of any outcome : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroRules.test.ts#L60 ]
[test: honours the timer's own cycle length : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroRules.test.ts#L69 ]
[test: the next interval follows the rules, and the tick sounds only during focus when asked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L204 ]

## S13: the Pomodoro is controlled by a switch in user settings @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- Given: a device that has never chosen
- Then: the Pomodoro is **off**: no timer icon in the sidebar's header, no
  bar, no `p` key, and no "Start a pomodoro" or "Pomodoro settings…" in the
  command palette — a timer nobody asked for is a row of sidebar taken from
  the work
- When: the user turns "Pomodoro timer" on in Settings
- Then: the header's timer icon appears and the bar opens beneath it, so the
  switch shows what it just switched on
- and the choice is remembered on this device, like Vim mode and the theme,
  rather than in the settings file — it is a habit of the person at this
  keyboard, not a property of the project

[test: is off until switched on — a timer nobody asked for costs sidebar : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroPref.test.tsx#L50 ]
[test: is remembered on the device, and off is stored as a decision : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroPref.test.tsx#L54 ]
[test: reads off when storage is unavailable : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoroPref.test.tsx#L63 ]

## S14: the window title carries the clock @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

- When: an interval is running
- Then: the window title is prefixed with the time left and the phase, as in
  "24:59 · Focus"
- Otherwise: no interval is running, and the prefix goes

[test: the clock rounds up, the title carries the phase, durations read as minutes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L123 ]
[test: stop switches the timer off, and the title prefix goes with it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L250 ]
[test: derive from the open interval's planned end : https://github.com/SpecDriven/specdriven-app/blob/main/tests/pomodoro.test.ts#L136 ]
