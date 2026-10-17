# Which SDD will win ?

**Author:** David Vydra

**Date:** 2026-10-17

**Kind:** Observation

responding to https://www.linkedin.com/posts/valentinajemuovic_spec-driven-development-when-ai-writes-the-share-7516752379958980608-as03/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAAABVcgBkX-l5A6YvINj1jYKbSqdF_2DzHg

I've been a promoter of test-driven development for well over 20 years. What I've learned in the process is that while the industry agreed that tests are mandatory, writing tests before code was a "road less travelled" for the vast majority of developers. Furthermore, the industry split between practitioners who have learned how to write useful and impactful tests and those who only cared about the code coverage number in order to merge their PRs.

I see the same dynamics play out in spec-driven development as we collectively try to figure out what that practice entails. There is mounting evidence that most developers prefer "chat-oriented development" -- that is they plan with their agent in a conversational style and then let it generate the code and tests. Needless to say that hitting 90% code coverage is assumed in this process. Assuming the right architectural guidance, frontier agents+models can do a very decent job using this approach and employers are expecting significant gains in developer productivity for their investment in AI.

At the same time, we are seeing general decrease in quality -- lots of articles have covered this trend. So how can SDD help?

I think that asking folks to write specs before code will end in the same result as test-driven -- only a fraction will ever do it consistently. However, asking developers to include specs with tests that are traceable to tests is something that may become a standard practice. Specs can be reviewd for accuracy by PMs and Testers and AI agents can keep specs and tests traceable and prevent the drift that killed most similar efforts in the past. I've been building a tool at https://specdriven.app to make this approach as enjoyable as possible. The fine folks at https://codespeak.dev/ seem to agree.

Would love to hear your thought on this topic.

==

I’ve been a promoter of test-driven development for well over 20 years. What I’ve learned in the process is that while the industry agreed that tests are mandatory, writing tests before code remained a “road less traveled” for the vast majority of developers. Furthermore, the industry split into practitioners who learned how to write useful, impactful tests and those who only cared about the code coverage number needed to merge their PRs.

I see the same dynamics playing out in spec-driven development (SDD) as we collectively try to figure out what that practice entails. There is mounting evidence that most developers prefer “chat-oriented development” — that is, they plan with their agent in a conversational style and then let it generate the code and tests. Needless to say, hitting 90% code coverage is assumed in this process. Given the right architectural guidance, frontier models and agents can do a very decent job with this approach, and employers are expecting significant gains in developer productivity from their investment in AI.

At the same time, we are seeing a general decrease in quality — lots of articles have covered this trend. So how can SDD help?

I think asking folks to write specs before code will end with the same result as test-driven development: only a fraction will ever do it consistently. However, asking developers to include specs that are traceable to tests is something that may become standard practice. Specs can be reviewed for accuracy by PMs and testers, and AI agents can keep specs and tests in sync, preventing the drift that killed most similar efforts in the past. I’ve been building a tool at https://specdriven.app to make this approach as enjoyable as possible. The fine folks at https://codespeak.dev/ seem to agree.

Would love to hear your thoughts on this topic.

What changed and why:

“Needless to say that hitting...” → “Needless to say, hitting...” The “that” is ungrammatical here.
“that is they plan” → “that is, they plan” needs the comma.
“a general decrease” was missing its article.
“the same result as test-driven” was missing “development”; “end with” is more idiomatic than “end in.”
“include specs with tests that are traceable to tests” was tangled (tests traceable to tests). I read your intent as “specs that are traceable to tests,” but adjust if you meant something else.
“reviewd” → “reviewed”; “thought” → “thoughts.”
“SDD” is used before it’s defined, so I introduced the acronym on first mention.
“--” → em dashes, and “agents+models” → “models and agents.”
“Testers” lowercased to match “developers” elsewhere; “the industry split into” is a bit cleaner than “split between.”
Added commas before “and employers” and “and AI agents,” since each joins two independent clauses.
“travelled” is British spelling; I switched to “traveled” since the rest of the text is American, but either is fine if you’re consistent.
