# Discuss how image markup undo history should work

**Author:** Jeffrey Miller

**Date:** 2026-10-27

**Kind:** Observation

When working with the new spec image markup feature, we noticed that old images were being persisted, multiplying the binary load of the specs repository.

We should consider how to persist spec markup history, or whether it is sufficient for a single Excalidraw session for markup to be the boundary of the undo history.

We can keep the current implementation and limit the undo history for an image based on the old versions of marked-up images and choose how many old versions keep when many edits are done.

W
