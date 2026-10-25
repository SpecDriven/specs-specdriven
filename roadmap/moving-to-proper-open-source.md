# moving to proper open source

Consider similar to https://github.com/mastra-ai/mastra

License: AGPL 3.0 see https://choosealicense.com/licenses/agpl-3.0/

## How we accept contributions

We are unable to accept PR requests due to both security liability and resource constraints.

## Specs contributions

We enourage our users to propose specs into the specs-specdriven repo.

### Proposal process

- User must log in with a valid GitHub account.
- We will restrict number of proposed specs per user per day to address DDOS risks.
- Proposals by users who are not contributors will be written to GitHub via a separate GH developer token we can set via an env var
- Proposals as a GitHub issue type? Maybe?

## Deployment architecture

Create a separate mode in our app and a separate service in Railway

## Dependencies

Implement capturing spec creator and last updator.

## Implementation

see docs/deployment-railway.md for related feature implementation

## Status

- Phase 1, licensing and contribution policy: specs/app/open-source-license.feature.md
- Phase 2, the spec-proposals service (GitHub sign-in, add-only proposals
  committed to main as `[proposed]`, daily quota, bot token):
  specs/app/spec-proposals.feature.md — deployed as its own Railway service
  (docs/deployment-railway.md "Spec proposals service" in the code repo)

Reference companies:

https://github.com/mastra-ai/mastra
