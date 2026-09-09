# Check for git readiness on startup

**Assigned:** David

If git is not configured, we get

```[reload] git commit failed: Author identity unknown
*** Please tell me who you are.
Run
  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"
to set your account's default identity.
Omit --global to set the identity only in this repository.
fatal: unable to auto-detect email address (got 'rose@orchid.(none)') exception.type="Error" exception.message="git commit failed: Author identity unknown\n\n*** Please tell me who you are.\n\nRun\n\n  git config --global user.email \"you@example.com\"\n  git config --global user.name \"Your Name\"\n\nto set your account's default identity.\nOmit --global to set the identity only in this repository.\n\nfatal: unable to auto-detect email address (got 'rose@orchid.(none)')"```

We need to check and ask user to configure before app is usable.
