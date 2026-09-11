<img src="/api/images/session-process-excalidraw-98f9634b.svg" alt="session-process.excalidraw" width="447">

# Coding Session Tracking and Reporting

Definitions / Background

- either manual or AI-assisted (agentic)
- humans; pairs

Purpose: what's this for? (see below for reports, etc.)
- Presume local for the individual user

Where Do Session Metrics Live?
- Local database (or sequence of record dump files?)
- Default: on (

Task Linking:
- Is a task appropriate to link to a coding session? Answer: yes, appropriate.
- How do we do that, and when? At start? At an arbitrary point? End? 
- A Jira subtask is one sort we should support.
- We support our own SpecDriven app tasks as well, preferably in URL form.

Metrics:

- overall time from open to close
- arbitrary key values per house rules
- link to specs used in the session
- PRs resulting from the session

Session work product:
- plans (a technical spec, like "plan.md" in Openspec, or plan output for some agents).

Agent metrics:

- which model, version , provider, effort (agent supplier, settings)
- token counts: input, cached, output
- local model or cloud model (e.g. the switchbox folks as a cloud proxy provider?)

"A (coding) session generates a metrics record."

What would be useful views of the metrics from coding sessions?
- snapshot of a session (what specs, ...)
    - company point of view: outward data capture "worked on X items for Y hours"
    - agent usage within a coding session; does it change over time? what's different?
- work product
    - PRs
    - new specs discovered and recorded during a session
- Reports supporting engineering approach to coding workflow
    - static analysis (CodeScene quality metrics)
    - downstream results (bugs?) - what's the join point from a session to its results (commit; PR?)
