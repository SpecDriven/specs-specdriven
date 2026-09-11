![session-process.excalidraw](/api/images/session-process-excalidraw-98f9634b.svg)# Coding Session Tracking and Reporting

Definitions / Background

- either manual or AI-assisted (agentic)
- humans; pairs

Purpose: what's this for?

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

Agent metrics:

- which model, version , provider, effort (agent supplier, settings)
- token counts: input, cached, output
- local model or cloud model (e.g. the switchbox folks as a cloud proxy provider?)
