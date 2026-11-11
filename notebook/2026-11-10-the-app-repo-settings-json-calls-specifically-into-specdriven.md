# The app repo settings.json calls specifically into SpecDriven

**Author:** jmeowmeow

**Date:** 2026-11-10

**Kind:** Observation

When deciding to detach a repo from SpecDriven workflow, it's worth considering how the app repo points to specific SpecDriven scripts.

These should be cleaned up, or the dependencies should be modified so that the app repo can be removed from the workflow without changes.

Context: hooks related to age
