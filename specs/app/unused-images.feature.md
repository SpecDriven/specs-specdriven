# Unused images

Images pasted or uploaded into a body land as content-addressed files under
`.images/` in the data repo (editor/images.feature.md) and nothing ever
removes them: a screenshot whose line was deleted, one replaced by a better
one, the original a markup save re-pointed the body away from
(image-editor-markup.feature.md), or every image of a deleted feature stays
in the repo and in every clone. The Images page finds them and lets the user
delete the ones they do not want.

- Its own page in the editor pane at `/images`, opened from the user menu or
  the command palette (`g i`) like Data.
- A stored image is referenced when any text file in the data repo names its
  url, `/api/images/<name>` — in a markdown image, an `<img>` tag or plain
  prose; url-encoded or not — outside hidden folders. Features count as the
  editor sees them, so a draft scenario that lives only in the Automerge
  document keeps its images. The scan is textual on purpose: a reference the
  renderer would not show still protects the file.
- Two tabs: Unreferenced, the images no file names, with a checkbox on each;
  and All, every stored image with how many files use it. Both show
  thumbnails — the images themselves, served as a body shows them — with the
  name and size under each. A summary line counts them and their bytes.
- Clicking a thumbnail opens the image beside the grid at full width with
  its size, the files that reference it (a feature as a link to it), and
  for an unreferenced one a Delete button.
- Deleting is a `git rm` of the files, committed and pushed in one commit
  like any other change; a confirmation names what goes and warns that a
  body still linking one would show a broken image.
- Refresh rescans the data repo.

## S1: List the unreferenced images @v1 [proposed]

- Given: images stored under `.images/`, some named by a feature, a page or
  another text file, some by a draft scenario the document alone holds, and
  some by nothing
- When: the user opens the Images page
- Then: the Unreferenced tab shows the images no file names, as thumbnails
  with their name and size, and says how many of the stored images they
  are and how much space they take
- and the All tab shows every stored image with the number of files that use
  it, or "unreferenced"

[test: maps each image to the files that reference it, draft scenarios included : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageCleanup.test.ts#L134 ]
[test: finds stored names in markdown images, <img> tags, encoded urls and prose : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageCleanup.test.ts#L110 ]
[test: only names storeImage generates are stored names : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageCleanup.test.ts#L125 ]
[test: a repo with no images lists none; stray files in .images/ are not images : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageCleanup.test.ts#L150 ]
[test: lists the unreferenced images first, selects them all and deletes them after asking : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imagesPage.test.tsx#L116 ]
[test: a thumbnail opens the image with where it is used; the All tab shows every image : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imagesPage.test.tsx#L189 ]
[test: nothing stored, and a server error, each say so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imagesPage.test.tsx#L232 ]
[test: Images has its own url : https://github.com/SpecDriven/specdriven-app/blob/main/tests/route.test.ts#L345 ]
[test: image routes match by method : https://github.com/SpecDriven/specdriven-app/blob/main/tests/api.test.ts#L46 ]

## S2: Delete the selected images @v1 [proposed]

- Given: the Unreferenced tab, with some images ticked — one by one, or all
  with "Select all"
- When: the user clicks Delete and confirms
- Then: the files are removed from `.images/`, committed and pushed in one
  commit, and the grid and the tab counts drop them
- Otherwise: the user declines the confirmation, and nothing is sent

[test: removes unreferenced images from .images/ : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageCleanup.test.ts#L160 ]
[test: removes the images in one commit, pushed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageCleanup.test.ts#L167 ]
[test: refuses names that are not stored images, or have no file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageCleanup.test.ts#L199 ]
[test: list the images with their references; delete the unreferenced; 409 for a referenced one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageCleanup.test.ts#L212 ]
[test: a delete needs a non-empty list of names : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageCleanup.test.ts#L235 ]
[test: lists the unreferenced images first, selects them all and deletes them after asking : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imagesPage.test.tsx#L116 ]
[test: a declined confirmation sends nothing; a refusal reloads and keeps the image : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imagesPage.test.tsx#L165 ]

## S3: A referenced image is never deleted @v1 [proposed]

- Given: an image the page listed as unreferenced that a file has named
  since — a body edited, a feature restored
- When: the user deletes it
- Then: the server scans afresh, refuses the whole request naming the image
  and its files, deletes nothing, and the page reloads so the image leaves
  the Unreferenced tab

[test: refuses an image a file still references, and deletes nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageCleanup.test.ts#L189 ]
[test: list the images with their references; delete the unreferenced; 409 for a referenced one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imageCleanup.test.ts#L212 ]
[test: a declined confirmation sends nothing; a refusal reloads and keeps the image : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imagesPage.test.tsx#L165 ]

## S4: The demo browses but cannot delete @v1 [proposed]

- Given: the public demo (public-demo.feature.md), whose data repo everyone
  shares
- When: a visitor opens the Images page
- Then: both tabs and the preview work, with no checkboxes and no Delete

[test: the demo browses but cannot delete : https://github.com/SpecDriven/specdriven-app/blob/main/tests/imagesPage.test.tsx#L222 ]
