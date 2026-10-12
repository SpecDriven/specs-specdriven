# Aha! and Figma links

A feature, and each of its scenarios, can link to the product record and the
design behind it: an Aha! feature, epic, requirement or idea, and a Figma
file, prototype or frame. Like the related pull requests
(external-links/related-pull-requests.feature.md), the links are bookkeeping
around the spec, not the spec: they are stored in the feature's Automerge
document only (datastore/automerge-store.feature.md) and never written into
the `.feature.md`.

Each tool is a setting of its own, off by default, so a team that doesn't use
Aha! or Figma never sees either. Each link is a url and an optional title.

## Settings switch each tool on @v1 [proposed]

- Given: Settings → Aha! & Figma
- When: the user turns **Aha! links** or **Figma links** on
- Then: features and scenarios offer that tool's links: a chip in the
  feature header, a field in the scenario editor, a list on the scenario card
- and both are off unless turned on; off is stored as an absent key
- and turning a tool off hides its chip, field and lists; links already
  recorded stay in the documents and come back when it is turned on again

[test: are off unless explicitly turned on : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L310 ]
[test: each switch round-trips through the settings file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L317 ]
[test: turning a tool off keeps the links already recorded : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L342 ]
[test: each tool is on only when /api/state says true : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinkField.test.tsx#L69 ]

## Link Aha! and Figma to a feature @v1 [proposed]

- Given: a feature is open and the tool is switched on
- When: the user presses the tool's chip in the feature header
- Then: a popover lists the feature's links with a ✕ each, and an add row —
  the url and an optional title
- and the chip reads the count — "2 Figma links" — or "Link Figma" when there
  are none
- and adds and removes are stored on the feature's document, and the
  markdown is left alone

[test: links a scenario, then the feature, answering with the editor's view : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L289 ]
[test: the chip counts the links, or offers to link one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L134 ]
[test: a scenario by its document id; the feature itself is target null; a missing one is null : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L167 ]

## Link Aha! and Figma to a scenario @v1 [proposed]

- Given: a scenario is open in the editor pane and has been saved at least
  once, so the document holds an entry for it
- When: the user pastes a url into the "Aha!" or "Figma" field and presses Add
- Then: the link is stored on the scenario's entry in the feature's document,
  each tool in its own list beside the pull requests
- and the `.feature.md` is not touched
- and the fields are there for a draft and a published scenario too
- and a url that is not http or https cannot be added, nor one the list
  already holds; a url outside aha.io or figma.com is added with a warning,
  since an Aha! ideas portal can sit on a custom domain

[test: the document holds the links, the file does not, the editor sees them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L198 ]
[test: each tool is its own list, beside the pull requests : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L153 ]
[test: a draft scenario can carry links too : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L213 ]
[test: a scenario the document lacks is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L220 ]
[test: an unknown tool, a bad url and an unknown scenario are refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L299 ]
[test: list the links with a remove button each, and an add row named for the tool : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinkField.test.tsx#L31 ]
[test: before the scenario is in the document the field waits, and says so : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinkField.test.tsx#L44 ]
[test: only an http(s) url the list does not already hold can be added : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L126 ]
[test: a url on another site is still a link, but not the tool's own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L117 ]
[test: the hint warns about a url on another site, a repeat, or something that is no url : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinkField.test.tsx#L58 ]
[test: only aha and figma are kinds : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L140 ]
[test: linking Aha! or Figma is not an edit autosave has to write : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinkField.test.tsx#L76 ]

## The scenario card lists its Aha! and Figma links @v1 [proposed]

- Given: a scenario carries Aha! or Figma links and the tool is switched on
- When: the user looks at its card
- Then: an "Aha!" and a "Figma" list after the pull requests show each link,
  opening it in a new tab
- and a link reads as its title; without one, as the Aha! reference number
  (APP-123, APP-E-12, APP-I-7) or the Figma file name taken from the url
- and a scenario with none shows no list

[test: each by its title or the name in its url, linking to it in a new tab : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinkField.test.tsx#L15 ]
[test: a scenario with none shows no list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinkField.test.tsx#L25 ]
[test: the label is the title, else the name from the url, and never empty : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L107 ]
[test: the name an Aha! url gives away is its reference number : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L92 ]
[test: the name a Figma url gives away is the file's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L100 ]

## The links survive everything the markdown does @v1 [proposed]

- Given: a scenario or feature carries Aha! or Figma links
- When: the scenario is saved or published; a teammate's plain-markdown push
  is absorbed as drift; or the scenario is moved to another feature
- Then: the links are still there afterwards
- and two peers linking different records to the same scenario at the same
  time end with both

[test: a save that edits the body : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L234 ]
[test: publishing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L241 ]
[test: an outside edit absorbed as drift : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L249 ]
[test: a scenario moved to another feature takes them along : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L258 ]
[test: two peers linking different records concurrently end with both : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L178 ]
[test: the projection never carries them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/resourceLinks.test.ts#L186 ]
