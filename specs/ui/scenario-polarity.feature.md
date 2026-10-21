# Scenario polarity

A scenario is positive — the happy path — or negative — the failure or edge
it guards. The marker is bookkeeping around the scenario, like a feature
flag or its ticket link, not part of its text: a published scenario's
polarity may change without a new version. The app shows and flips it only
while the "Positive and negative scenarios" setting is on; the markdown
keeps it either way, and a Confluence page groups the scenarios by it.

## A negative marker sits under the heading @v1 [proposed]

- Given: a scenario in a `.feature.md`
- When: a line right under its `##` heading holds nothing but `[negative]`
- Then: the scenario is negative
- and the line may sit in any order among the flag, attribute, ticket link
  and authorship lines, and reads in any case
- and a second such line, one further down, or `[negative]` inside a
  sentence stays ordinary text, as does one under the feature's `#` heading
- and a save writes the marker back as the first line under the heading,
  `[negative]` in lower case
- and the document carries it like a feature flag, so a draft's marker
  reaches the editor and the projection writes it back

[test: a negative line under the heading is lifted into negative and written back as the first line under it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarity.test.ts#L56 ]
[test: the marker mixes with flag, link and authorship lines in any order; a second one, or one further down, stays text : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarity.test.ts#L69 ]
[test: the document carries the marker and projects it back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarity.test.ts#L98 ]
[test: reconcile sets and clears the marker, and a no-op stays one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarity.test.ts#L105 ]
[test: a draft's marker reaches the editor from the document : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarity.test.ts#L201 ]

## A proposal cannot mark a scenario negative @v1 [proposed]

- Given: the spec proposals service (app/spec-proposals.feature.md)
- When: a visitor proposes a scenario whose text would read back as the
  negative marker
- Then: the proposal is refused, like one that would add a flag line

[test: a body that would mark the scenario negative is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/proposals.test.ts#L109 ]

## A published scenario's marker changes without a new version @v1 [proposed]

- Given: a published scenario
- When: its marker is set or cleared
- Then: the file and the document take the change at the same version
- and its body still may not change

[test: flipping a published scenario's marker is bookkeeping, no new version, the body still immutable : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarity.test.ts#L172 ]

## A scenario without the marker is positive @v1 [proposed]

- Given: a scenario with no `[negative]` line
- Then: it is positive — the marker is binary, and only the negative side
  is ever written
- and a feature file from before the marker reads and writes back unchanged

[test: a scenario without the marker is positive, and today's files round-trip unchanged : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarity.test.ts#L91 ]

## The Confluence page groups scenarios by polarity @v1 [proposed]

- Given: the "Positive and negative scenarios" setting is on
- When: I publish a feature to Confluence (external-links/confluence-wiki.feature.md)
- Then: after the description the page holds a "Positive" section with the
  positive scenarios in scope, then a "Negative" section with the negative
  ones, each in the feature's order
- and each scenario heading carries a green "Positive" or red "Negative"
  status lozenge after its number
- and a section with no scenario in scope is left out
- and the numbers and anchors stay those of the whole feature
- When: the setting is off
- Then: the page is one run of scenarios, as before, whether or not any
  carries the marker

[test: with the setting off the page is unchanged, marker or not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L175 ]
[test: with it on, a Positive section then a Negative one, each heading with its lozenge after the number : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L180 ]
[test: a section with no scenario in scope is left out, and the numbers stay the feature's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluenceStorage.test.ts#L194 ]
[test: with the Positive and negative scenarios setting on the page is split by polarity : https://github.com/SpecDriven/specdriven-app/blob/main/tests/confluencePublish.test.ts#L326 ]

## The pill shows and flips the marker @v1 [proposed]

- Given: the "Positive and negative scenarios" setting is on
- When: I open a feature
- Then: each scenario card shows a small dot next to its title, green for
  positive and red for negative
- When: I click the dot on a card
- Then: the marker flips and is saved on the spot, the dot changing colour
  with it, a published scenario included
- and the card's dot stands down while the editor pane is open, as Move
  up and Move down do
- When: I edit a scenario
- Then: the editor pane's head shows a pill reading Positive or Negative in
  the same colours, and a click flips the draft's marker, saved with the
  scenario
- When: the setting is off
- Then: no dot or pill shows, and a `[negative]` line already in a feature file
  stays as written

[test: withScenarioPolarity marks one scenario and leaves the rest alone; positive drops the key : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarityClient.test.tsx#L21 ]
[test: the pill reads Positive in green or Negative in red and says what a click does : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarityClient.test.tsx#L33 ]
[test: on a card the polarity is only a dot, coloured the same and still saying what a click does : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarityClient.test.tsx#L45 ]
[test: the autosave signature follows the marker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarityClient.test.tsx#L56 ]

## The setting is off by default @v1 [proposed]

- Given: Settings
- Then: "Positive and negative scenarios" is a switch, off unless turned on
- and it is kept in the settings file, off stored as no entry
- and `PUT /api/state` refuses anything but a boolean for it

[test: the Positive and negative scenarios setting is off by default and round-trips through the settings file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarity.test.ts#L210 ]

## Two teammates flipping the marker end with one value @v1 [proposed]

- Given: two copies of a feature from the same base
- When: one side flips a scenario's marker and the other leaves it
- Then: the merge takes the side that flipped it
- When: both sides flip it differently
- Then: ours wins
- and a flip is an edit: against the other side deleting that scenario it
  is a conflict

[test: a three-way merge keeps the side that flipped the marker, ours when both did; a flip is an edit against a delete : https://github.com/SpecDriven/specdriven-app/blob/main/tests/scenarioPolarity.test.ts#L123 ]
