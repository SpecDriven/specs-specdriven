# custom-attributes

A team tags its features and scenarios with attributes of its own, such as
the business rules a spec depends on: `[br:young-driver-surcharge:on]` or
`[br-sub:young-driver-surcharge:under-21:1.5]`. The team defines the kinds
of attribute it uses in the data repo's `custom-attributes.txt`, which is
shared through git like the specs. Attributes are bookkeeping around a spec,
like feature flags (app/feature-flags.feature.md): a published scenario's
attributes may change without a new version.

## S1: Attribute lines sit under the heading @v1 [proposed]

- Given: a feature or a scenario in a `.feature.md`
- When: a line right under its `#` or `##` heading holds nothing but
  `[<key>:<name>…:<value>]` tokens, e.g. `[br:max-claims:3] [line:auto]`
- Then: those are its attributes, in the order written: the last segment
  is the value and the ones before it are names
- and the key is lower case, starting with a letter, and is never `ff`,
  `test`, `http` or `https`; a segment holds no `[`, `]`, `:` or
  line break and no space at either end, inner spaces being allowed
- and the line may sit in any order among the flag, ticket link and
  authorship lines, may mix attribute and flag tokens, and several such
  lines add up
- and a token inside a sentence, a malformed token, an `[ff:…]` token that
  is not a valid flag, or such a line further down stays ordinary text
- and an exact duplicate is dropped; the same attribute with another value
  is kept
- and a save writes them right after the flag line, in order, starting a
  new line wherever the key changes
- and what the definitions say never changes how a file is read

[test: attribute lines under the headings are lifted into attributes and written back as they were : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L67 ]
[test: a line may mix flag and attribute tokens, among the link and authorship lines in any order : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L85 ]
[test: text that only looks like a token stays text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L105 ]
[test: an exact duplicate is dropped, the same attribute with another value is kept : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L135 ]
[test: only attributes the token can carry are written : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L143 ]

## S2: Attributes are checked against their definitions @v1 [proposed]

- Given: the data repo defines custom attributes
- When: I open a feature
- Then: the feature header shows a chip per attribute of the feature, and
  each scenario card a row of chips for its own, labelled by their
  definition, an `on` value green and an `off` value red
- and a chip whose attribute does not fit is amber and its tooltip says
  why: the key is not defined, it is not used where it is (a feature-only
  attribute on a scenario), it has the wrong number of names, a value is
  not one of the words or not a number, or a `from` name is not used by
  any attribute it refers to
- and the same attribute given twice with two values is a warning on both
- When: the data repo defines no attributes at all
- Then: the chips are labelled by key and warn about nothing

[test: an attribute is checked against its definition, and the names used are suggested : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributesDsl.test.ts#L145 ]
[test: labelled by their definition, an on value green and an off one red : https://github.com/SpecDriven/specdriven-app/blob/main/tests/attributesField.test.tsx#L34 ]
[test: one that does not fit is amber and says why : https://github.com/SpecDriven/specdriven-app/blob/main/tests/attributesField.test.tsx#L45 ]
[test: labelled by key and warning about nothing when the repo defines none, and nothing when there are none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/attributesField.test.tsx#L64 ]

## S3: Attributes are edited on the feature and in the scenario editor @v1 [proposed]

- Given: the data repo defines attributes for features or for scenarios
- When: I open the feature header's attributes chip
- Then: a popover lists the feature's attributes, each with a control for
  its value and a ✕, and an add row; every change is saved on the spot
- When: I edit a scenario
- Then: the editor pane has an "Attributes" field that works the same,
  saved with the scenario
- and a value's control follows its definition: a switch for `on|off`, a
  select for fixed words, a field for a part to fill in (a number one
  with a decimal keypad), and a select with "Other…" for both
- and the add row picks a kind of attribute among those defined for
  features, or for scenarios, and suggests the names the data repo already
  uses; a name that must refer to another attribute suggests that one's
- and the add row refuses a part left empty, a name no attribute it refers
  to uses yet, an attribute already there, and a value that must be a
  number but is not
- and an attribute is added after the others of its key
- and an attribute no definition covers shows as written, with its ✕

[test: a row per attribute with a control for its value and a remove button, and an add row : https://github.com/SpecDriven/specdriven-app/blob/main/tests/attributesField.test.tsx#L74 ]
[test: with no definitions for that scope, it says where to add them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/attributesField.test.tsx#L102 ]
[test: adding puts it after the others of its key; changing and removing touch only that one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/attributesField.test.tsx#L115 ]
[test: an empty part, a name not used yet, one already there or a value that is not a number is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/attributesField.test.tsx#L134 ]
[test: an attribute change is something for the editor's autosave to save : https://github.com/SpecDriven/specdriven-app/blob/main/tests/attributesField.test.tsx#L148 ]
[test: each part gets the control that fits it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributesDsl.test.ts#L124 ]

## S4: Attributes change on a published scenario without a new version @v1 [proposed]

- Given: a published scenario
- When: I add, change or remove one of its attributes
- Then: the change is saved, and the scenario keeps its version and stays
  published
- and its text still may not change
- and a published body an older app recorded with an attribute or flag
  line still in its text does not refuse later saves of the feature

[test: an attribute on a published scenario changes without a new version : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L255 ]
[test: a published body recorded with its attribute line still in it does not block a save : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L208 ]

## S5: Attributes travel with the document and merge as one value @v1 [proposed]

- Given: the Automerge document behind a feature
  (datastore/automerge-store.feature.md)
- When: a feature file with attributes is imported, edited or projected
- Then: the document holds the feature's and each scenario's attributes, a
  draft's included, and the projection writes them back
- and when two sides change a list of attributes, the side that changed it
  wins, ours when both did, as with the flags
- and the feature listing carries each feature's and scenario's attributes

[test: the document carries the attributes and projects them back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L167 ]
[test: a three-way merge keeps the side that changed the attributes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L182 ]
[test: a draft's attributes reach the editor from the document : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L281 ]
[test: the listing carries the feature's and the scenarios' attributes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L327 ]

## S6: Custom attributes are defined in Settings @v1 [proposed]

- Given: Settings → Custom attributes, for the open data folder
- When: I write a definition, a pattern starting in the first column
  followed by indented properties:

  ```
  [br:<rule>:on|off|<value>]
    label: Business rule
    scope: feature, scenario
    description: A business rule the spec depends on
  ```

- Then: it is saved as I type in the data folder's `custom-attributes.txt`
- and in a pattern the key comes first, each `:`-separated part after it is
  a name, and the last part is the value; a part lists alternatives split
  by `|`, each a fixed word or at most one slot: `<name>` takes any text,
  `<name:number>` a number, and `<name from k>` a name some `[k:…]`
  attribute in the data folder already uses
- and `label` defaults to the key, `scope` to both features and
  scenarios, and `description` is help text; whole lines starting with
  `#` are comments
- and the keys `ff`, `test`, `http`, `https`, `draft`, `proposed`,
  `published` and `saved` are reserved
- and the editor counts the problems and lists each by its line, saying
  what was expected, and sums up what the file defines
- and a definition with a problem is left out, a bad property is ignored,
  a second definition of a key is left out, and the file is saved anyway
- and an empty file offers to start from an example defining business
  rules and business sub-rules

[test: the example defines business rules and sub-rules with no problems : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributesDsl.test.ts#L24 ]
[test: a pattern holds words and slots, and the properties default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributesDsl.test.ts#L36 ]
[test: every problem has its line and column, and says what was expected : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributesDsl.test.ts#L52 ]
[test: a reference must name another defined attribute : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributesDsl.test.ts#L87 ]
[test: a definition with a problem is left out, a bad property is ignored, a second definition is left out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributesDsl.test.ts#L98 ]
[test: the example has no problems and its definitions are summed up : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributesEditor.test.tsx#L19 ]
[test: each problem is listed by line and described by the text box : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributesEditor.test.tsx#L29 ]
[test: an empty file offers the example : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributesEditor.test.tsx#L40 ]
[test: the definitions are read with the listing and written by Settings : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L298 ]

## S7: Filter features and scenarios by attribute @v1 [proposed]

- Given: features and scenarios that carry attributes
- When: I open the sidebar's Filter menu
- Then: it has an entry per attribute key in use, labelled by its
  definition, offering "Any …", each attribute by its names, and each of
  those with each value it is set to
- When: I pick one
- Then: features that carry it, or have a scenario that does, read as
  matches in the tree; the open feature shows only the scenarios that
  carry it, a scenario carrying its feature's attributes too
- and it combines with the status and assignee filters, shows as a chip
  that clears it, and an open feature with no matching scenario says so

[test: a scenario matches by its own attributes or its feature's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/treeFilter.test.ts#L40 ]
[test: a feature matches when it or a scenario of it carries the attribute, and by assignee : https://github.com/SpecDriven/specdriven-app/blob/main/tests/treeFilter.test.ts#L53 ]
[test: the menu has an entry per key with any, each attribute, and each with its value : https://github.com/SpecDriven/specdriven-app/blob/main/tests/treeFilter.test.ts#L65 ]
[test: the active filter reads by its label : https://github.com/SpecDriven/specdriven-app/blob/main/tests/treeFilter.test.ts#L80 ]
[test: custom attributes changed in the open feature carry over too : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioNav.test.ts#L142 ]

## S8: Search finds attributes @v1 [proposed]

- Given: a feature or a scenario carrying `[br:good-driver-discount:on]`
- When: I search for "good driver" (search.feature.md)
- Then: it is among the hits, ranked above a mention in body text
- and when only its attributes matched, the snippet shows their line

[test: a custom attribute finds the feature or scenario carrying it, with its line as the snippet : https://github.com/SpecDriven/specdriven-app/blob/main/tests/search.test.ts#L74 ]

## S9: The definitions file is shared with the team @v1 [proposed]

- Given: `custom-attributes.txt` at the root of the data repo
- When: the app syncs (datastore/automerge-store.feature.md)
- Then: the file is committed and pushed with the specs, and a teammate's
  definitions arrive with the pull and reach Settings and the editors
- and a save from Settings folds in a change the file got on disk since it
  was loaded, rather than overwriting it
- and when two teammates change the file at once, the pull merges both
  changes instead of stopping the sync on a conflict

[test: a save folds in a disk edit Settings has not shown : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L316 ]
[test: two teammates adding definitions at once merge on the next sync : https://github.com/SpecDriven/specdriven-app/blob/main/tests/customAttributes.test.ts#L342 ]
