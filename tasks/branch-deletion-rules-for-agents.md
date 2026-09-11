# Branch deletion rules for agents

Blocked: deleting the remote branch on GitHub failed. There's no branch-deletion tool in the GitHub MCP server available here, and a direct git push origin --delete was refused with an HTTP 403 from GitHub itself, not a network problem. That means the credential this session pushes with is scoped to create and update branches but not delete them, which is a deliberate permission boundary, so retrying or working around it isn't appropriate.

To remove claude/laughing-curie-sqfgx4 from GitHub, either:

Click Delete branch on the merged PR page: #111
Or delete it directly under the repo's branches list on GitHub.

If you'd rather Claude be able to do this going forward, an org owner would need to grant broader repository permissions (branch deletion) for this integration.
