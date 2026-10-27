# Image Editor Markup

When writing and evaluating specs containing images, adding markup annotations to an image allows a user to stay in the spec editor context. SpecDriven supports adding markup to an image in a spec using an integrated editor.

The integrated editor is Excalidraw (editor/excalidraw-support.feature.md) opened on the image. With the Excalidraw diagrams switch on in Settings, every image uploaded to the data repo offers **Mark up**: a button on the image on hover in an editable Preview, and a toolbar button while the caret is inside the image's link. The pane shows the image locked in place — it cannot be selected, moved or erased — with the Rectangle tool selected and every shape drawn red, bold, unfilled and sharp-cornered.

Save burns the markup into a PNG of the original's own pixel size — a shape running off the image is cut at its edge — uploads it, and points the image in the body at it, keeping its alt text and width. The PNG also carries the shapes and the original's url, so the markup stays editable. Cancel asks before discarding unsaved markup.

## S1: Add box markup around a section of an image in the spec. @v1 [proposed]

- **Given** an uploaded image in an editable body, open in the markup pane
- **When** the user drags across part of the image with the Rectangle tool
- **Then** a red, unfilled, sharp-cornered box surrounds that part
- **And** after Save the image in the body shows the box, at the image's own pixel size, with its alt text and width unchanged

[test: carries a markup button in the editable preview once the switch is on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/excalidrawPref.test.tsx#L178 ]
[test: is offered by the toolbar with the caret in its link : https://github.com/SpecDriven/specdriven-app/blob/main/tests/excalidrawPref.test.tsx#L203 ]
[test: any image stored in the data repo : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L63 ]
[test: is the original's stem, without its hash, marked as markup : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L87 ]
[test: falls back to 'image' : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L99 ]
[test: is stored as <stem>-markup-<hash>.png, the markup still in it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L103 ]

## S2: Add an arrow as markup on top of a spec image. @v1 [proposed]

![image](/api/images/image-d61bf2de.png)

- **Given** an image open in the markup pane
- **When** the user picks the Arrow tool (or presses `A`), presses where the arrow starts, drags, and releases where it should point
- **Then** an arrow runs from the press to the release, its head at the release point
- **And** its ends stay where they were dropped, not snapping to the image's edge or to a box
- **And** selecting the arrow shows a handle at each end, which moves that end

## S3: Allow undo of markup commands @v1 [proposed]

- **Given** markup drawn in the pane
- **When** the user presses Undo, or `Ctrl/⌘+Z`
- **Then** the canvas returns to how it was before the last command
- **And** Redo, or `Ctrl/⌘+Shift+Z`, applies it again
- **But** undo stops at the canvas as the pane opened it: the image, and any markup saved before, stay

## S4: Reopening a marked-up image keeps its markup editable @v1 [proposed]

- **Given** an image whose markup was saved
- **When** the user opens it in the markup pane again
- **Then** the original image is on the canvas, with no markup burnt in
- **And** the saved shapes are over it as shapes, to move, change or delete

[test: never piles the suffixes up : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L95 ]
[test: carry the PNG CRC : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L119 ]
[test: round-trip, just before IEND : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L123 ]
[test: replace the keyword's chunk and leave other keywords alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L131 ]
[test: are absent from a PNG without one, and from what is no PNG : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L139 ]
[test: refuse what is no PNG, and text beyond Latin-1 : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L146 ]
[test: encodes as ASCII and decodes to itself : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L153 ]
[test: travels in the PNG under its own keyword, not Excalidraw's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L159 ]
[test: is nothing when it is not one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L167 ]
[test: moves each element by the offset, its points along with it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L184 ]

## S5: Removing all markup restores the original image @v1 [proposed]

- **Given** a marked-up image open in the markup pane
- **When** the user deletes every shape and saves
- **Then** the image in the body points at the original again, and nothing is uploaded

## S6: Mark up is offered only on images it can change @v1 [proposed]

[negative]

- **Given** an Excalidraw diagram, an image from outside the data repo, or an image in a read-only (published) body — or the Excalidraw diagrams switch is off
- **When** the user hovers the image in Preview, or puts the caret in its link
- **Then** no Mark up button is shown

[test: not a diagram, which has its own editor : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L71 ]
[test: not an image from anywhere else, whose pixels the canvas cannot read : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L77 ]
[test: names only an image stored here as its original : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageMarkup.test.ts#L176 ]
[test: carries none while the switch is off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/excalidrawPref.test.tsx#L185 ]
[test: carries none on a published (read-only) body : https://github.com/SpecDriven/specdriven-app/blob/main/tests/excalidrawPref.test.tsx#L189 ]
[test: is never a diagram, nor an image from elsewhere : https://github.com/SpecDriven/specdriven-app/blob/main/tests/excalidrawPref.test.tsx#L196 ]
[test: is not offered by the toolbar while the switch is off : https://github.com/SpecDriven/specdriven-app/blob/main/tests/excalidrawPref.test.tsx#L210 ]
