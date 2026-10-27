# Images in scenarios

A scenario body can embed images — a screenshot pasted from the clipboard or
an image file picked from disk. Images live in the data repo like everything
else, so they sync through git with the features that reference them. The
`![...](/api/images/<name>)` references resolve inside the app; on GitHub the
files are browsable under `.images/`.

- In the scenario editor pane's Write tab, pasting clipboard content that
  contains an image uploads it and inserts `![alt](/api/images/<name>)` at
  the caret. Plain text pastes are untouched.
- The 🖼 toolbar button opens the OS file picker (images only, multiple
  allowed); each chosen file uploads and inserts the same way.
- Uploaded images are stored as files under `.images/` at the data repo root.
  The dot-folder keeps them out of the sidebar's folder tree while git tracks
  them; each upload is committed and pushed like a feature edit.
- Stored names are content-addressed: `<slugified stem>-<content hash>.<ext>`.
  Uploading the same bytes twice reuses the existing file, and a served name
  never changes content, so responses carry immutable cache headers.
- Supported types: png, jpg, jpeg, gif, webp, svg, avif — anything else is
  refused, as are empty files and files over 10 MB.
- The Preview tab (and every rendered scenario body) shows the image, scaled
  down to fit the pane width.
- While an upload runs the toolbar shows "Uploading…"; a failed upload shows
  the error in the toolbar and inserts nothing.

Sizing. A pasted screenshot lands at its pixel size — on a HiDPI display
twice as big as it looked on screen — so an image can be sized in place.

- In an editable Preview tab every image carries a resize handle at its
  bottom-right corner, shown on hover or keyboard focus (always shown on a
  touch screen). Published bodies, and rendered bodies outside an editor
  such as the scenario cards, show images without one.
- A width badge ("320 px") shows beside the handle on hover, focus and while
  dragging.
- The width is kept in the body as `<img src="…" alt="…" width="…">`, which
  any markdown viewer honours, so the image shows at that size everywhere
  the text is rendered. Only the width is written: a `height` the tag
  carried is dropped so the image keeps its proportions.
- Widths are whole pixels, no narrower than 32 px and no wider than the
  pane's content width.
- Going back to the natural size turns the tag into `![alt](src)` again —
  unless the alt contains `]` or the url contains `)`, which the markdown
  form cannot carry; then the tag stays, without a width.
- Each handle sizes its own image: the nth image on screen edits the nth
  image reference in the body, whatever code blocks, quotes or tables lie
  between.

## S1: no sizing where the body cannot be edited @v1 [proposed]

- Given: a published (read-only) scenario body, or a rendered body outside
  an editor such as a scenario card
- When: the user views or hovers an image in it
- Then: the image shows at the width the body gives it, with no resize
  handle and no width badge

[test: is not on a published (read-only) body : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageSizing.test.tsx#L32 ]
[test: is not on a preview outside an editor : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageSizing.test.tsx#L39 ]

## S2: paste an image from the clipboard @v1 [proposed]

- When: the user pastes clipboard content containing an image into the body
- Then: the image is uploaded to `.images/` in the data repo, committed and
  pushed
- and `![alt](/api/images/<name>)` is inserted at the caret

## S3: repeated upload of the same image @v1 [proposed]

- When: an image with bytes already stored under `.images/` is uploaded again
- Then: the existing file is reused — same name, no new commit

## S4: restore an image's natural size @v1 [proposed]

- Given: a sized image in an editable Preview
- When: the user double-clicks its resize handle, or presses Backspace or
  Delete with the handle focused
- Then: the width is removed from the body, and the reference reads
  `![alt](src)` again
- and where the alt contains `]` or the url contains `)`, the `<img>` tag
  stays, without a width
- Otherwise: the image is already at its natural size, and the body is
  unchanged

[test: Backspace and Delete ask for the natural size : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageSizing.test.tsx#L77 ]
[test: the natural size is the markdown form again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/markdown.test.tsx#L207 ]
[test: an alt or url the markdown form cannot carry keeps the tag : https://github.com/SpecDriven/specdriven-app/blob/main/tests/markdown.test.tsx#L212 ]
[test: leaves the text alone when there is no such image, or no such width : https://github.com/SpecDriven/specdriven-app/blob/main/tests/markdown.test.tsx#L229 ]

## S5: size an image by dragging its handle @v1 [proposed]

- Given: an image in the Preview tab of an editable body
- When: the user drags the handle at the image's bottom-right corner
- Then: the image follows the pointer live, with the badge naming its width,
  held between 32 px and the pane's content width
- and on release the width is written into the body as
  `<img src="…" alt="…" width="…">`, through autosave like any other edit
- Otherwise: the user releases at the width the image started at, or the
  drag is cancelled, and the body is unchanged

[test: is on every image of an editable preview : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageSizing.test.tsx#L23 ]
[test: keeps a width between the floor and the room, in whole pixels : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageSizing.test.tsx#L90 ]
[test: turns the markdown form into a sized tag : https://github.com/SpecDriven/specdriven-app/blob/main/tests/markdown.test.tsx#L191 ]
[test: targets the nth image only : https://github.com/SpecDriven/specdriven-app/blob/main/tests/markdown.test.tsx#L195 ]
[test: re-sizes a tag, dropping a height so the proportions hold : https://github.com/SpecDriven/specdriven-app/blob/main/tests/markdown.test.tsx#L201 ]
[test: quotes in the alt survive the round trip : https://github.com/SpecDriven/specdriven-app/blob/main/tests/markdown.test.tsx#L217 ]
[test: a width is whole pixels, at least one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/markdown.test.tsx#L224 ]
[test: reads CRLF as the preview does : https://github.com/SpecDriven/specdriven-app/blob/main/tests/markdown.test.tsx#L236 ]

## S6: size an image from the keyboard @v1 [proposed]

- Given: the resize handle of an image in an editable Preview has keyboard
  focus — it is a slider, announced with the image's alt and its width in px
- When: the user presses an arrow key — Right or Up to widen, Left or Down
  to narrow
- Then: the width changes by 10 px, or by 50 px with Shift held, and is
  written into the body as with a drag
- and Home sizes the image to 32 px, End to the pane's content width
- Otherwise: the user presses any other key, and the handle leaves it to
  the page

[test: arrows nudge the width; Shift takes bigger steps : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageSizing.test.tsx#L59 ]
[test: stops at the floor and at the pane's width : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageSizing.test.tsx#L68 ]
[test: other keys are not the handle's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageSizing.test.tsx#L82 ]
[test: names an unnamed image plainly : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageSizing.test.tsx#L45 ]

## S7: unsupported file is refused @v1 [proposed]

- When: the uploaded file is not a supported image type, is empty, or exceeds
  10 MB
- Then: nothing is stored and the toolbar shows the reason

## S8: upload an image file @v1 [proposed]

- When: the user clicks 🖼 and picks one or more image files
- Then: each file is uploaded and inserted like a pasted image
