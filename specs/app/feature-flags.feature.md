# feature-flags

A feature or a scenario may name the feature flags that gate it in the
product being specified, written `[ff:<name>:on|off]` in the markdown. Flags
are bookkeeping around a spec, like its ticket link, not part of its text: a
published scenario's flags may change without a new version. The app shows
and edits them only while the "Feature flags" setting is on; the markdown
keeps them either way.

## S1: A flag line sits under the heading @v1 [proposed]

- Given: a feature or a scenario in a `.feature.md`
- When: a line right under its `#` or `##` heading holds nothing but
  `[ff:<name>:on|off]` tokens, e.g. `[ff:new-checkout:on] [ff:card-v2:off]`
- Then: those are its flags, in the order written
- and the line may sit in any order among the ticket link and authorship
  lines, and several such lines add up; a name given twice keeps its first
  value
- and `on` and `off` read in any case and are written in lower case
- and a name starts with a letter or digit and holds only letters, digits,
  `.`, `_` and `-`
- and a token inside a sentence, a malformed token, or a flag line further
  down stays ordinary text
- and a save writes the flags back as one line, right under the heading

[test: flag lines under the headings are lifted into flags and written back as they were : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlags.test.ts#L53 ]
[test: flag lines mix with the link and authorship lines in any order, and several lines add up : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlags.test.ts#L66 ]
[test: a token inside text, a malformed token or a flag line further down stays text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlags.test.ts#L85 ]
[test: flag names the token syntax can carry : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlags.test.ts#L97 ]

## S2: Flags are edited on the feature and in the scenario editor @v1 [proposed]

- Given: the "Feature flags" setting is on
- When: I open a feature
- Then: the feature header shows a chip per flag of the feature, and each
  scenario card a chip per flag of its scenario, an on flag green and an off
  flag red, its OFF beside the editors' switches red too
- When: I open the header's flags chip
- Then: a popover lists the feature's flags, each with an on/off switch and
  a ✕, and takes a new one by name with its own on/off switch, so the flag
  is added already on or off; every change is saved on the spot
- When: I edit a scenario
- Then: the editor pane has the same "Feature flags" field, and its changes
  are saved with the scenario
- and the add row's switch starts on, and a name that is not allowed or
  already there is refused with a message

[test: one chip per flag, saying whether it is on, an off flag red : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlagsField.test.tsx#L23 ]
[test: nothing when there are none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlagsField.test.tsx#L31 ]
[test: a switch and a remove button per flag, and an add row : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlagsField.test.tsx#L38 ]
[test: adding appends the flag, on or off as set; toggling and removing touch only that flag : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlagsField.test.tsx#L52 ]
[test: a name the token cannot carry, or one already there, is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlagsField.test.tsx#L62 ]
[test: a flag change is something for the editor's autosave to save : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlagsField.test.tsx#L71 ]

## S3: Flags change on a published scenario without a new version @v1 [proposed]

- Given: a published scenario
- When: I add, toggle or remove one of its flags
- Then: the change is saved, and the scenario keeps its version and stays
  published
- and its text still may not change

[test: a flag on a published scenario changes without a new version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlags.test.ts#L156 ]

## S4: Flags travel with the document and merge as one value @v1 [proposed]

- Given: the Automerge document behind a feature
  (datastore/automerge-store.feature.md)
- When: a feature file with flags is imported, edited or projected
- Then: the document holds the feature's and each scenario's flags, a
  draft's included, and the projection writes them back
- and when two sides change a list of flags, the side that changed it wins,
  ours when both did, as with a ticket link

[test: the document carries the flags and projects them back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlags.test.ts#L103 ]
[test: a three-way merge keeps the side that changed the flags : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlags.test.ts#L110 ]
[test: a draft's flags reach the editor from the document : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlags.test.ts#L183 ]

## S5: The Feature flags setting shows or hides them @v1 [proposed]

- Given: the "Feature flags" setting in Settings → Specs is off, as it is
  by default
- When: I open a feature whose file has flag lines
- Then: no flag chips or flag editors show
- and saving the feature keeps its flag lines exactly as they were
- When: I turn the setting on
- Then: the chips and editors appear
- and the setting is saved in the settings file like the others, off
  stored as an absent key

[test: the Feature flags setting is off by default and round-trips through the settings file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureFlags.test.ts#L191 ]
