# Developer-led coding session

Not all code should we written by AI agents.

We use the Pomodoro technique to implement a "human" coding session.

A developer-led session is timed by the Pomodoro technique, run by the person
at the keyboard, and recorded locally beside the AI session records — never in
the data repo, which is pushed to git.

- A session is started from the **Pomodoro** tab of the Start session dialog,
  beside the AI tab. It needs no specs selected and no code generation
  switched on.
- The intervals are **Focus**, **Short break** and **Long break**, their
  lengths set per session (1–180 minutes; the long break every 1–12
  pomodoros). The defaults are 25/5/15 minutes, a long break every 4.
- A pomodoro is *indivisible*: it either runs out (`completed`) or is voided
  (`abandoned`). It is never paused. A break may be cut short — also
  `abandoned`. Abandoned time still counts as time spent; only completed work
  intervals count as pomodoros.
- After every break comes focus; after a completed pomodoro comes the short
  break, or the long one when the cycle is full; after a voided pomodoro, the
  pomodoro again.
- The clock is never a counter: what remains is the running interval's planned
  end minus the time now, so a reload, or a laptop asleep for ten minutes,
  lands on the right number.
- One session at a time — the bar shows a single timer.
- The interval lengths, the sound and the tick are device-local preferences, a
  habit of the person at this keyboard; each session's record keeps a snapshot
  of the ones it started with.

## a pomodoro cannot be paused, only abandoned @v1 [proposed]

- When: the user presses "Abandon" during focus
- Then: the interval is recorded as abandoned, it fills no dot, its time still
  counts toward the session's focus so far, and the next interval offered is
  the pomodoro again
- Otherwise: there is no pause control — a pomodoro is indivisible

## a pomodoro runs out and a break is offered @v1 [proposed]

- When: the running Focus interval reaches its planned end
- Then: it is recorded as completed *at its planned end* rather than at the
  moment the app noticed, the chosen sound plays, the bar flashes, and a
  notice offers "Start break" — the button that begins the next interval

## an interval that ran out while the app was closed @v1 [proposed]

- Given: the app was closed, or the machine asleep, past the running
  interval's planned end
- When: the user comes back to the app
- Then: that interval is closed as completed at its planned end, quietly — no
  chime for a break that ended an hour ago — and the bar offers what comes
  next

## end the session @v1 [proposed]

- When: the user chooses "End session" from the bar's menu
- Then: a running interval is abandoned and its time counted, the session is
  closed with its totals, and the bar and the window title's prefix go
- Otherwise: the session has ended already, and ending it again changes
  nothing

## interval lengths are remembered on this machine @v1 [proposed]

- When: the user changes Focus, Short break, Long break or "Long break every"
  and starts a session
- Then: the next time the dialog opens it holds those numbers
- Otherwise: a value is out of range or not a number, and it is clamped to
  1–180 minutes (1–12 for "Long break every") rather than lost

## one session at a time @v1 [proposed]

- Given: a Pomodoro session is already running
- When: the user opens the Pomodoro tab of the Start session dialog
- Then: it says so, points at the timer in the sidebar, and offers to end that
  session rather than start a second one

## preview the sound that announces a break @v1 [proposed]

- When: the user picks a sound in the Pomodoro tab and presses the ▶ button
- Then: that sound plays at once
- Otherwise: the sound is "None", and the ▶ button is disabled

## skip a break @v1 [proposed]

- When: the user presses "Skip" during a short or long break
- Then: the break ends as abandoned and the bar offers the next pomodoro
- Otherwise: the interval is a pomodoro, and "Skip" is not offered — as
  "Abandon" is not offered during a break

## start a developer-led session @v1 [proposed]

- When: the user opens the Start session dialog, picks the **Pomodoro** tab,
  names the session and presses "Start Pomodoro"
- Then: the session begins with its first pomodoro already running, the
  Pomodoro bar appears at the foot of the sidebar, and the session's record
  keeps the interval lengths it started with
- Otherwise: the name is empty, and "Start Pomodoro" stays disabled

## the bar shows where the session stands @v1 [proposed]

- Then: the Pomodoro bar shows the phase, the clock counting down, a progress
  line, one dot per completed pomodoro in the current cycle, and the focus
  time so far
- Otherwise: the sidebar is collapsed, and the rail shows the clock alone as a
  chip that brings the sidebar back when clicked

## the clock survives a reload @v1 [proposed]

- When: the app is reloaded while a pomodoro is running
- Then: the bar comes back on the same interval showing the time that actually
  remains — not the time that remained when the page was left

## the long break comes when the cycle is full @v1 [proposed]

- Given: the session has completed as many pomodoros since its last long break
  as "Long break every" allows
- When: the running pomodoro completes
- Then: the break offered is the long one, and the cycle's dots start again
  after it, whether it ran out or was skipped

## the Pomodoro tab needs no specs and no code generation @v1 [proposed]

- Given: code generation is switched off in Settings and nothing is selected
  in the sidebar
- When: the user opens the Start session dialog
- Then: the AI tab is disabled and says why, while the Pomodoro tab starts a
  session as usual

## the window title carries the clock @v1 [proposed]

- When: an interval is running
- Then: the window title is prefixed with the time left and the phase, as in
  "24:59 · Focus"
- Otherwise: no interval is running, and the prefix goes
