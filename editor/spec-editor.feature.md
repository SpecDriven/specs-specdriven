# Spec Editor

Spec editor will be separate pane that will replace the feature pane with '<' icon to go back to the feature.

## Create New Spec @v1 [published]

Add scenario.
This opens the editor dialog for the new spec.
The new spec editor dialog has a title for the user to fill in.
The new spec starts in draft mode.
The draft spec is persisted in Automerge.
The user can use Delete which removes the draft spec.
The escape key or the close-editor widget will leave the scenario in its autosaved state. Like the Settings dialog, the bottom of the spec dialog will have a note to use Esc to exit.
Two buttons available: "Propose" and "Publish".

## Delete Published Spec @v1 [published]

Given: deletion of published specs is turned on in user settings
When: click on Delete button
Then: a confirmation dialog pops up with: "Danger Zone! Confirm that you want to delete a Published" spec.
The user can click "Confirm" button or the "Cancel" button.
If confirmed, the spec is deleted.
