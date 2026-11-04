# Excalidraw support

**Assigned:** David

**Created by:** vydra

**Updated by:** vydra

Excalidraw is an open-source proejct for creating diagrams.

We should be able to use it from inside of our editor.

## S1: Create new Excalidraw diagram @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

If Excalidra support is enabled in user settings, show Excalidra icon.

Click on the icon to open Excalidaraw pane and use it to create the diagram.

Link to the diagram will be embedded into the feature file similar to how we handle images.

## S2: Edit Excalidraw diagram @v1 [published]

**Created by:** RomanWright

**Updated by:** RomanWright

Given a link to an Excalidraw diagram in a text file, user is able to edit the existing diagram.

After the user saves the editted diagram, overwrite the existing diagram shown in that text file.

## S3: Import Excalidraw file @v1 [proposed]

**Created by:** vydra

**Updated by:** vydra

Given a diagram saved as a `.excalidraw` file (the format excalidraw.com and the Excalidraw desktop app save), user is able to bring it into the editor.

In the Excalidraw pane, an Import button lets the user pick a `.excalidraw` file; its drawing replaces what is on the canvas (after confirming, if the canvas is not empty) and, for a new diagram, the diagram takes the file's name. Insert then embeds it like a diagram drawn in the pane.

The editor's image picker also accepts `.excalidraw` files: picking one opens the pane with it loaded, ready to insert.

A file that holds no Excalidraw drawing is reported, and the canvas is left as it was.
