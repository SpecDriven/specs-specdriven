# Launch AI Agent Session

## Assemble Selected Specs for Coding Session to Clipboard @v1 [proposed]

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
