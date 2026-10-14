# hosted-app

SpecDriven is a single-user app: one settings file, one GitHub token, one
data repo. The hosted app keeps it that way and puts a small gateway in
front, so a team can use it from a browser without installing anything. The
gateway signs people in with an email address and a password, keeps each
person's secrets encrypted, and runs one app server — a worker — per person,
with its own folders and its own credentials. It is deployed as a Railway
service from the app's repository (docs/deployment-railway.md in the code
repo); the public demo (plugins.feature.md) and the spec proposals service
(app/spec-proposals.feature.md) are other services built from the same
image.

Because several people's workers share one container, a hosted worker runs
no code a person supplies and reaches nothing outside that person's folder.
Like the desktop app, the hosted app links to the source of the running
version (app/open-source-license.feature.md).

## A hosted worker runs no code and reaches nothing outside the person's folder @v1 [proposed]

- Given: the app running as a person's hosted worker
- When: anything asks it to run a local coding agent, install, update or
  remove a plugin, add a plugin folder or a marketplace, open the native
  folder chooser or IntelliJ, use the GitHub CLI engine, sign in to GitHub
  in the app, open a data repo that is not a GitHub url, or point live sync
  at a server
- Then: it answers 403, "not available in the hosted app"; only built-in
  plugins run
- and the app offers none of these: no agent targets, no folder chooser,
  no IntelliJ, and the agent pane lists no sessions
- and every other setting still changes as it does on the desktop
- and the user the app shows is the person's account email, not the
  container's operating-system user

[test: %s %s is refused in a hosted worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L54 ]
[test: Settings refuse %j in a hosted worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L89 ]
[test: the state says hosted and offers no local agents, folder chooser or IntelliJ : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L81 ]
[test: the agent pane lists no sessions rather than refusing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L75 ]
[test: other settings still change in a hosted worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L102 ]
[test: the git engine is never gh in a hosted worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L107 ]
[test: the user shown is the gateway's account, not the container's OS user : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L136 ]

## A password link works once @v1 [proposed]

- Given: an emailed link to choose a password, for sign-up or a reset
- When: the person opens it and saves a password
- Then: the link is used up, and opening it again does nothing
- and a password the rules refuse leaves the link valid for another try:
  at least 12 characters, not a common password, not the email address's
  name
- and asking for a new link retires the older ones
- and the link's token travels in the address's `#fragment`, never reaches
  a server log, and is stored only as a hash; passwords are stored as
  argon2id hashes

[test: a link works once, and a weak password does not use it up : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L95 ]
[test: only the newest link works, and links are stored hashed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L103 ]
[test: password rules — length, common passwords, the email's name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L161 ]
[test: passwords are stored as argon2id hashes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L88 ]
[test: tokens are stored as SHA-256 hex : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L107 ]

## A person's recent data repos survive a redeploy @v1 [proposed]

- Given: a signed-in person whose worker has a data repo open, and a
  gateway with a datastore (Accounts survive a redeploy)
- When: the service is redeployed without its volume, so their folder and
  settings file are gone, and they use the app again
- Then: their new worker opens the repo they had open, with their other
  recent repos listed after it, instead of asking for a repo url
- and the gateway notes a worker's repos every minute while it runs, when
  it stops and after a crash, keeps them per account, and pushes them
  with the datastore; deleting the account deletes them
- and only GitHub repos are kept, in canonical form, once each, at most
  ten
- and a worker that already has a settings file keeps its own: the gateway
  never overwrites it, never follows one that is a symlink, and does not
  read one over 1 MB

[test: only GitHub repos are kept, in canonical form, once each, at most ten : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L45 ]
[test: the list is stored per user, reports a change, and goes with the account : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L53 ]
[test: a worker's settings give the open repo first, then its recent ones; no file says nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L69 ]
[test: a settings file that is a symlink is not followed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L78 ]
[test: a worker with no settings gets the remembered repos; one with settings keeps its own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L89 ]
[test: after the volume is lost, the user's next worker opens the repo they had open : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L100 ]
[test: a snapshot read into a fresh database gives back accounts, roles, allowed emails, secrets that still open, recent repos and live sessions : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDatastore.test.ts#L86 ]

## A worker cannot reach another person's folder @v1 [proposed]

- Given: a person controls their data repo's contents, symlinks included
- When: their worker follows a link to another person's folder, to the
  accounts database or to the gateway's process
- Then: the operating system refuses it: each worker runs as a user id of
  its own, without supplementary groups and with no way to gain
  privileges, and its folder belongs to that user id alone
- and a data repo's `code-repo` path is ignored in a hosted worker, so it
  cannot point into another person's folder
- and outside the hosted image, where the gateway cannot change user ids,
  the workers share its user and the gateway logs a warning

[test: each worker runs as its own uid, owning its folder alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L128 ]
[test: the worker command drops to the user's uid with no way back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L169 ]
[test: a data repo's code-repo path is ignored, so it cannot point into another user's folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L146 ]

## Accounts survive a redeploy @v1 [proposed]

- Given: the gateway has a GitHub token, SPECDRIVEN_DATASTORE_TOKEN, that
  can read and write the datastore repo (SpecDriven/hosted-app unless
  SPECDRIVEN_DATASTORE_REPO names another)
- When: people sign up, sign in, change a password or save a token, and the
  service is then redeployed without its volume
- Then: they sign in again with the same password, still signed in where
  their session had not expired, and their sealed tokens still open
- and the repo holds one JSON file per table: accounts with password
  hashes and roles, allowed emails, secrets still sealed under the master
  key, recent data repos, and session hashes without IP address, user
  agent or last-seen time
- and a change worth keeping is pushed within 30 seconds, a session's
  sliding expiry only on shutdown, and everything once more before the
  gateway exits
- and a database that already has accounts keeps them and pushes them over
  the repo's copy
- and a repo the gateway cannot read stops it from starting, so an empty
  account list is never pushed over the real one
- and the token never reaches a worker or a command line

[test: a snapshot read into a fresh database gives back accounts, roles, allowed emails, secrets that still open, recent repos and live sessions : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDatastore.test.ts#L86 ]
[test: an empty database is filled from what another gateway pushed; one with accounts keeps its own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDatastore.test.ts#L120 ]
[test: a repo pushed before the roles is read as plain users and allowed emails, and saved in today's form : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDatastore.test.ts#L143 ]
[test: a sliding session expiry alone is not pushed until a forced save; a new session is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDatastore.test.ts#L175 ]
[test: a repo that cannot be read stops the restore, and nothing is ever pushed over it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDatastore.test.ts#L200 ]
[test: a push that loses a race to another writer is redone on top of it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDatastore.test.ts#L209 ]
[test: the token and repo come from the environment; no token means no datastore : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDatastore.test.ts#L67 ]
[test: the token reaches git through the credential helper's environment, not argv : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDatastore.test.ts#L78 ]
[test: the worker environment carries the user's folders and secrets, not the gateway's keys : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L53 ]

## Admins give and take the admin role @v1 [proposed]

- Given: a signed-in admin on the admin page, which lists every account
  with its role
- When: they make another account an admin, or an admin a user
- Then: the change applies on that person's next request, with no new
  sign-in
- and the last admin cannot stop being one, so there is always an admin:
  the page says to make someone else an admin first
- and a role is admin or user, nothing else, and an account that does not
  exist is refused
- and a new account is a user, and a database from before the roles keeps
  its accounts as users and its invites as allowed emails

[test: an admin gives and takes the admin role, which applies at once, but never from the last admin : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L427 ]
[test: accounts start as users : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L57 ]
[test: a database from before the roles keeps its accounts as users and its invites as allowed emails : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L62 ]

## Admins keep the allowed emails on the admin page @v1 [proposed]

- Given: a signed-in admin on the admin page
- When: they add an address (`name@example.com`) or a whole domain
  (`@example.com`) to the allowed emails
- Then: that address, or any address in that domain, can sign up (Sign up
  with an email address)
- and the page lists each entry with the admin who added it
- and an entry is trimmed and lowercased, and anything that is neither an
  address nor @ and a domain is refused
- and adding an address can email it an invitation to the sign-up page,
  unless it already has an account; a domain is never emailed
- and taking an entry off the list stops new sign-ups only: an account
  already made keeps working

[test: an admin opens the admin page and sees the allowed emails and the users : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L373 ]
[test: an admin allows an address, which is invited and can sign up, and takes it off again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L390 ]
[test: a domain on the list lets every address in it sign up, and is never emailed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L418 ]
[test: an allowed entry is an address or @ and a domain, trimmed and lowercased : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L48 ]
[test: an invite goes only to an address without an account : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L114 ]

## Change the password on the account page @v1 [proposed]

- Given: a signed-in person on the account page
- When: they give their current password and a new one
- Then: the new password is saved, every other session of the account
  ends, and this browser stays signed in
- and without the right current password nothing changes

[test: changing the password needs the current one and ends every session : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L150 ]
[test: changing the password signs this browser in again and ends the others : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L337 ]

## Each person gets a worker of their own @v1 [proposed]

- Given: two signed-in people
- When: each uses the app
- Then: each request reaches that person's own worker, with its own
  settings, folders and environment, over a private socket; the gateway's
  session cookie is not passed on
- and the gateway serves the app's pages and its bundled assets itself,
  built once; the API, plugin bundles and every write go to the worker
- and request bodies pass through and streamed answers stream
- and a worker starts on its person's first request, stops after 30 idle
  minutes, and restarts whenever its person changes a secret or resets
  the password
- and an account id that is not a UUID never becomes a folder path

[test: signed in, the app's pages come from the gateway and its API from the worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L213 ]
[test: two users reach two different workers : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L222 ]
[test: request bodies pass through and streamed answers stream : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L232 ]
[test: two users get two workers with separate settings, folders and environments : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L84 ]
[test: restart stops the worker; the next request starts a fresh one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L109 ]
[test: idle workers are stopped by the sweep : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L118 ]
[test: user ids that are not UUIDs never become paths : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L80 ]
[test: page paths get the shell : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAppShell.test.ts#L26 ]
[test: bundled assets are served immutable, and nothing outside the build is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAppShell.test.ts#L34 ]
[test: the API, plugin bundles and writes are the worker's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAppShell.test.ts#L42 ]

## Only admins open the admin page @v1 [proposed]

- Given: a signed-in person who is not an admin
- When: they open `/admin` or call the `/auth/admin/*` API
- Then: both answer 403, the page with the gateway pages' strict content
  security policy
- and nobody else can add to the allowed emails: the old invite API is
  gone
- and signed out, `/admin` sends the browser to sign-in and comes back
  after, and the API answers 401

[test: an admin gives and takes the admin role, which applies at once, but never from the last admin : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L427 ]
[test: signed out, the admin page leads to sign-in and its API refuses : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L366 ]
[test: a new account is a user, with no admin API, no admin page and no invites to send : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L355 ]

## Requests from anywhere else are refused @v1 [proposed]

- Given: the gateway answering at its public url
- When: a request names another host, or a write comes from another site
- Then: it is refused before it reaches a worker; the gateway's own
  `/auth/*` API passes the same Host, Origin and Content-Type checks as the
  app's
- and the gateway's own pages carry a strict content security policy, are
  never framed and never cached
- and the health check answers without a Host check or a session, for the
  hosting platform

[test: a request for another host name is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L147 ]
[test: a cross-site write is refused before it reaches the worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L241 ]
[test: the sign-in pages carry a strict CSP and are never framed or cached : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L189 ]
[test: the health check answers without a Host check or a session : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L142 ]

## Reset a forgotten password @v1 [proposed]

- Given: the forgot-password page
- When: someone enters an email address
- Then: the page says "check your email" whether or not the account
  exists, and the email goes out in the background so the response time
  does not tell either
- and the reset link works once, for 30 minutes
- and saving the new password ends every session of the account, restarts
  its worker, emails a "your password was changed" notice and signs this
  browser in

[test: a forgotten password gets a reset link that expires in 30 minutes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L115 ]
[test: forgot sends nothing for an unknown address, and sign-up of a taken one sends a reset : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L126 ]
[test: a reset ends every session and sends a change notice : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L135 ]
[test: forgot answers the same for known and unknown addresses : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L276 ]
[test: a reset through the API ends the old session and restarts the worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L284 ]

## Sign in and out @v1 [proposed]

- Given: an account with a password
- When: its owner signs in with it
- Then: the browser gets a session cookie and opens the app; Sign out on
  the account page ends the session
- and a wrong password signs nobody in, and repeated wrong passwords are
  refused for a while, per address and per IP, by limits that survive a
  restart
- and the cookie is `HttpOnly`, `SameSite=Lax` and, over https, `Secure`
  with the `__Host-` prefix; the server keeps only a hash of it
- and a session ends after 14 days unused, and after 30 days whatever its
  use
- and signed out, the app's pages send the browser to sign-in and its API
  answers 401

[test: login, logout and a wrong password : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L254 ]
[test: repeated wrong passwords are rate limited : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L266 ]
[test: the rate limiter allows max attempts per window, then refuses until it passes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L211 ]
[test: the session cookie is HttpOnly, SameSite=Lax, and __Host- Secure over https : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L200 ]
[test: a session is stored as a hash, slides with use and ends at 30 days : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L176 ]
[test: an unused session expires after 14 days : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L192 ]
[test: tokens are stored as SHA-256 hex : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L107 ]
[test: signed out, the app redirects pages to sign-in and refuses the API : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L180 ]

## Sign up with an email address @v1 [proposed]

- Given: the gateway's sign-up page
- When: someone enters an email address
- Then: if the address, or its domain, is on the allowed emails an admin
  keeps on the admin page, whatever its case, the gateway emails a link
  that chooses the account's first password; any other address gets
  nothing
- and the account has no password until that link is used, so nobody can
  claim an address they cannot read
- and an address that already has an account gets a password reset link
  instead
- and the page answers the same for every address, so it never tells who
  has an account or who may sign up
- and addresses are trimmed and lowercased, and anything that is not an
  email address is refused

[test: sign-up emails a link to an allowed address or domain, and nothing to others : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L60 ]
[test: an address put on the list later can sign up, and is matched whatever its case : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L70 ]
[test: an address cannot sign in until its link sets a password : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L78 ]
[test: forgot sends nothing for an unknown address, and sign-up of a taken one sends a reset : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L126 ]
[test: emails are trimmed and lowercased; non-emails are refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L170 ]
[test: sign-up, set password, and the session reaches the user's worker without the gateway cookie : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L201 ]

## The Admin item under Settings shows for admins only @v1 [proposed]

- Given: a signed-in person using the hosted app
- When: they open the user menu
- Then: an admin sees Admin under Settings, which opens the admin page;
  anyone else does not
- and the account page links to the admin page for admins only
- and the role comes from the gateway's `/auth/me`, not from the worker

[test: an admin opens the admin page and sees the allowed emails and the users : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L373 ]
[test: a new account is a user, with no admin API, no admin page and no invites to send : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L355 ]

## The master key is rotated without losing secrets @v1 [proposed]

- Given: a deployment with stored secrets
- When: the operator sets a new master key and moves the current one to
  the previous keys, then redeploys
- Then: at start the gateway re-encrypts every secret under the new key,
  and from then on the secrets need only the new key
- and a master key is 32 bytes, kept as a Railway variable and never on
  the volume; in production the gateway refuses to start without one

[test: rotating the master key re-seals old rows, which then need only the new key : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L80 ]
[test: a master key must be 32 bytes, and production requires one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L47 ]

## The original admin is set up on a database with no admin @v1 [proposed]

- Given: a gateway starting on a database with no admin, after the
  datastore restore (Accounts survive a redeploy), so a fresh volume takes
  the repo's accounts and admins first
- When: it starts
- Then: the original admin written into the code (`david@vydra.net`)
  becomes the admin, with the password chosen for it, which the code keeps
  only as an argon2id hash
- and the account is created if needed; if it exists, its password is
  replaced and its sessions and emailed links end
- and a database that has an admin, whoever it is, keeps it, and nothing
  changes; since the last admin cannot lose the role, this happens once

[test: a database without an admin makes the original admin one, with the chosen password : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L80 ]
[test: an existing original account becomes the admin with the chosen password, its old sign-ins ended : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L93 ]
[test: a database that has an admin, whoever it is, keeps it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L105 ]

## Tokens and keys are kept sealed @v1 [proposed]

- Given: a signed-in person on the account page
- When: they save a GitHub token or an agent API key
- Then: a GitHub token is checked with GitHub first, and the account it
  belongs to becomes the identity that signs their commits
- and every secret is encrypted with AES-256-GCM under the deployment's
  master key, bound to its owner and its name, so a copy of the database
  reveals nothing and a secret moved to another person or name does not
  open
- and the page shows each secret's name and last four characters, and for
  GitHub the account, but never the value: the browser can only write a
  secret, replace it or remove it
- and agent key names are short lowercase words, and `github` is reserved
- and secrets are opened only when the gateway starts that person's
  worker, and reach it as environment variables built from scratch: the
  GitHub token, the agent keys, and none of the gateway's own keys
- and the worker only reads the GitHub sign-in it was given: signing in,
  out or to another account there is refused, and the app points to the
  account page instead

[test: the GitHub token is checked with GitHub, stored sealed, shown by hint, and restarts the worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L297 ]
[test: agent keys are saved, listed by hint and removed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L313 ]
[test: one user's secrets are not another's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L329 ]
[test: a sealed secret opens with the same key and associated data : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L23 ]
[test: a ciphertext moved to another user or name, or altered, does not open : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L30 ]
[test: a different master key cannot open the secret : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L40 ]
[test: the store keeps ciphertext only and lists hints, never values : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L52 ]
[test: agent key names are short lowercase slugs, and github is reserved : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L98 ]
[test: the worker environment carries the user's folders and secrets, not the gateway's keys : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L53 ]
[test: the env token backend reads the gateway's record and refuses to change it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L114 ]
[test: agent keys arrive as SPECDRIVEN_AGENT_KEY_<NAME> variables : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L124 ]
