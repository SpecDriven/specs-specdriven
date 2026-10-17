# Which SDD will win ?

**Author:** David Vydra

**Date:** 2026-10-17

**Kind:** Observation

responding to https://www.linkedin.com/posts/valentinajemuovic_spec-driven-development-when-ai-writes-the-share-7516752379958980608-as03/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAAABVcgBkX-l5A6YvINj1jYKbSqdF_2DzHg


I’ve been a promoter of test-driven development for well over 20 years. What I’ve learned in the process is that while the industry has agreed that tests are mandatory, writing tests before code has remained a “road less traveled” for the vast majority of developers. Furthermore, the industry split into practitioners who learned how to write useful, impactful tests and those who only cared about the code coverage number needed to merge their PRs.

I see the same dynamics playing out in spec-driven development (SDD) as we collectively try to figure out what that practice entails. There is mounting evidence that most developers prefer “chat-oriented development” — that is, they plan with their agent in a conversational style and then let it generate the code and tests. Needless to say, hitting 90% code coverage is assumed in this process. 

Given the right architectural guidance, frontier models and agents can do a solid job with this approach, and employers are expecting significant gains in developer productivity from their investment in AI.



I think asking folks to write specs before code will end up with the same result as test-driven development: only a fraction will ever do it consistently. However, asking developers to include specs that are traceable to tests is something that should be a standard practice. Agents will generate the first draft of the specs, and developers will vet them along with the automated tests. Additionally, specs should be reviewed for accuracy by PMs and testers. AI agents can keep specs and tests in sync, preventing the drift that killed most similar efforts in the past. I’ve been building a tool at https://specdriven.app to make this approach as enjoyable as possible. The fine folks at https://codespeak.dev/ seem to agree.

Would love to hear your thoughts on this topic.
