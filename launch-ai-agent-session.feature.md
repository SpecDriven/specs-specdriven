# Start Coding Session

SpecDriven Start Coding Session Feature

Workflow: UI for starting the session.

System Behavior: Collect events, major events (start session, finish session); collect telemetry, send integration events.

Coding Session Entity
Start Session Event

* UUID
* Name
* Start Date and Time
* Type and version of the coding agent
* Session Report, consisting of:
* (nullable) End Date and Time (because Crash Only Software) 
* Token usage, input, cached, and output. (per the usual standards for LLM supplieers).
* Summary of the session.
* Linked artifacts - Git PR, Git commits, Issue(s)


Agent's Notion of a Coding Session vs. SpecDriven's Notion of a Coding Session
 - coding agents: "resume a session"
 - SpecDriven: ?

![specdriven-agent-dispatch-notes.excalidraw](/api/images/specdriven-agent-dispatch-notes-excalidraw-ffc27c60.svg)

## Launch Session Scenario Outline @v1 [proposed]

1. User opens a feature file.
2. User selects the whole feature, and all of its specs, or selects one or more specs from the feature.
3. User requests a coding session.
4. SpecDriven app provides a session dialog.
5. The session dialog shows the selected specs.
6. The user may provide a name for the coding session.
7. The user confirms starting the session or cancels the session.
8. If the user does not provide a name, SpecDriven app fills in an automatically generated session name like "Session 2026-08-20 15:30:30 PDT".
9. SpecDriven app prepares a coding session prompt by combining the session name first, followed by the feature title, followed by each spec selected by the user, including the spec name and body.
10. SpecDriven app copies the coding session prompt to the system clipboard.
