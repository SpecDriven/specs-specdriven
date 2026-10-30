# Project Management (Jira, Linear, etc)

## S1: Add link at spec level @v1 [published]

Each spec (scenario) can have its own link.

## S2: Add link to feature @v1 [published]

Add link at feature level

## S3: Publishing links into projected feature (.md) files - Jira @v1 [published]

Given: user settings 'Publish external links' is ON
Write format: [<program short name>:<ticket id>](<web link)
Example: [Jira:ABC-123](https:jira.com/ABC-123)

## S4: Several tickets on a feature @v1 [proposed]

A feature is often tracked under more than one ticket — an epic and its
stories, or one story per release — so the feature carries a list of them;
a scenario still carries one.

- Given: project management is switched on in Settings and a feature is open
- When: the user presses the "Ticket" chip in the feature header
- Then: a popover lists the feature's tickets, each a link to the tool with
  a ✕, and an add row — the url, with the tool and ticket id filled in when
  the url gives them away — with an Add button and the count, "2 of 5
  tickets linked"
- and the chip reads "Link ticket" with none, the one ticket's own label
  ("Jira:ABC-123", with the open-in-new-tab icon beside it) with one, and
  the count ("3 tickets") with more
- and a feature links up to 5 tickets: at five the add row gives way to a
  line saying so; a ticket already in the list — the same url, or the same
  id in the same tool, case aside — is refused under the fields
- and each add or remove is saved at once, as a whole list, through the
  tickets route: the markdown gets one `[Jira:ABC-123](url)` line per
  ticket, together under the heading, while "Publish external links" is on,
  and none while it is off — the document holds them either way
- and a file edited by hand with more than five, or with a repeat, is read
  whole: the repeat is dropped, the rest are kept and shown, and the add row
  waits until there is room
- and a document from before the list, with its single ticket, reads as a
  list of one; the first change to the tickets moves it to the list
- and a feature imported from Jira (external-links/jira-wiki-import.feature.md)
  starts with the issues it was read from as its tickets, at most five

[test: a feature carries several tickets, one line each, in order, and writes them back together : https://github.com/SpecDriven/specdriven-app/blob/main/tests/externalLinks.test.ts#L565 ]
[test: a repeated ticket is dropped on parse, the first kept; a scenario still takes one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/externalLinks.test.ts#L578 ]
[test: the list is set whole — reconcile replaces it, an empty target clears it, a no-op leaves the heads : https://github.com/SpecDriven/specdriven-app/blob/main/tests/externalLinks.test.ts#L592 ]
[test: an older document's single link reads as a list of one, and the first change retires it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/externalLinks.test.ts#L608 ]
[test: a three-way merge takes the side that changed the list, ours when both did : https://github.com/SpecDriven/specdriven-app/blob/main/tests/externalLinks.test.ts#L635 ]
[test: with the setting off, the editor's view folds every ticket back; drift absorbs a hand-added one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/externalLinks.test.ts#L646 ]
[test: the chip reads the one ticket, or counts them, or offers to link one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/externalLinks.test.ts#L664 ]
[test: a repeat, or a sixth, cannot join the list : https://github.com/SpecDriven/specdriven-app/blob/main/tests/externalLinks.test.ts#L671 ]
[test: sets the list — the document holds it, the file one line each, the view has them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureTickets.test.ts#L73 ]
[test: refuses a sixth, a repeat, a malformed ticket and a qTest or Confluence link : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureTickets.test.ts#L106 ]
[test: a document from before the list is migrated by the first change : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureTickets.test.ts#L118 ]
[test: sets the tickets, answering with the editor's view : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureTickets.test.ts#L142 ]
[test: lists the tickets as links with a remove button each, and an add row with the count : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ticketLinkField.test.tsx#L12 ]
[test: at five the add row gives way to the count and how to make room : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ticketLinkField.test.tsx#L35 ]
[test: the Jira issues as tickets, the sources under the title, each scenario proposed and numbered in the agent's order, with its body and where it came from : https://github.com/SpecDriven/specdriven-app/blob/main/tests/jiraWikiImportPlan.test.ts#L131 ]
[test: with Publish external links off the file stays bare and the view still has them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureTickets.test.ts#L88 ]
[test: the description and the rest of the feature survive, since the server's fresh view is what is saved : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureTickets.test.ts#L97 ]
[test: a malformed body, a non-http url, a sixth and a repeat are 400s with a reason : https://github.com/SpecDriven/specdriven-app/blob/main/tests/featureTickets.test.ts#L152 ]
[test: with none, only the add row; with the tool set in Settings, the url alone is asked for : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ticketLinkField.test.tsx#L25 ]
[test: the server's refusal shows under the fields, and saving disables the buttons : https://github.com/SpecDriven/specdriven-app/blob/main/tests/ticketLinkField.test.tsx#L45 ]
