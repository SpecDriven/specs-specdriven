# Image Support

## Rationale
Specs can include images, which are good for communicating structure, domain context, or software behavior to both team members and coding agents.

## Image Storage
Images are typically binary files which are inefficient to store within Git source repositories, which are best for text files with compact differences.

SpecDriven should choose an image storage solution which can support:
* Effectively communicating a need in the UI.
* Effectively describing structure or domain context.
* Efficiently using storage space for the current image and history.
