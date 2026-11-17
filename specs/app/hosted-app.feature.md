# hosted-app

SpecDriven is a single-user app: one settings file, one GitHub token, one
data repo. The hosted app keeps it that way and puts a small gateway in
front, so a team can use it from a browser without installing anything. The
gateway signs people in with an email address and a password, keeps each
person's secrets encrypted, and runs one app server — a worker — per person,
with its own folders and its own credentials. It is deployed as a Railway
service from the app's repository (docs/deployment-railway.md in the code
repo); the public demo (plugins.feature.md) is another service built from
the same image.

Because several people's workers share one container, a hosted worker runs
no code a person supplies and reaches nothing outside that person's folder.
Like the desktop app, the hosted app links to the source of the running
version (app/open-source-license.feature.md).

## S1: A hosted worker runs no code and reaches nothing outside the person's folder @v1 [proposed]

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

[test: %s %s is refused in a hosted worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L58 ]
[test: Settings refuse %j in a hosted worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L95 ]
[test: the state says hosted and offers no local agents, folder chooser or IntelliJ : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L84 ]
[test: the agent pane lists no sessions rather than refusing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L78 ]
[test: other settings still change in a hosted worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L106 ]
[test: the git engine is never gh in a hosted worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L111 ]
[test: the user shown is the gateway's account, not the container's OS user : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L162 ]

## S2: A password link works once @v1 [proposed]

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
[test: tokens are stored as SHA-256 hex : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L116 ]

## S3: A person's recent data repos survive a redeploy @v1 [proposed]

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

[test: only GitHub repos are kept, in canonical form, once each, at most ten : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L48 ]
[test: the list is stored per user, reports a change, and goes with the account : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L56 ]
[test: a worker's settings give the open repo first, then its recent ones; no file says nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L72 ]
[test: a settings file that is a symlink is not followed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L112 ]
[test: a worker with no settings gets the remembered repos; one with settings keeps its own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L123 ]
[test: after the volume is lost, the user's next worker opens the repo they had open : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L142 ]
[test: a snapshot read into a fresh database gives back accounts, roles, allowed emails, secrets that still open, recent repos and live sessions : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayDatastore.test.ts#L86 ]

## S4: A worker cannot reach another person's folder @v1 [proposed]

- Given: a person controls their data repo's contents, symlinks included
- When: their worker follows a link to another person's folder, to the
  accounts database or to the gateway's process
- Then: the operating system refuses it: each worker runs as a user id of
  its own, without supplementary groups and with no way to gain
  privileges, and its folders belong to that user id alone
- and a data repo's `code-repo` path is ignored in a hosted worker, so it
  cannot point into another person's folder
- and outside the hosted image, where the gateway cannot change user ids,
  the workers share its user and the gateway logs a warning

[test: each worker runs as its own uid, owning its folders alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L217 ]
[test: the worker command drops to the user's uid with no way back, under its caps : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L588 ]
[test: a data repo's code-repo path is ignored, so it cannot point into another user's folder : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L172 ]

## S5: Accounts survive a redeploy @v1 [proposed]

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
[test: the worker environment carries the user's folders and secrets, not the gateway's keys : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L68 ]

## S6: Admins give and take the admin role @v1 [proposed]

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

[test: an admin gives and takes the admin role, which applies at once, but never from the last admin : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L450 ]
[test: accounts start as users : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L57 ]
[test: a database from before the roles keeps its accounts as users and its invites as allowed emails : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L62 ]

## S7: Admins keep the allowed emails on the admin page @v1 [proposed]

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
  already made keeps working until an admin deletes it (Admins delete an
  account)

[test: an admin opens the admin page and sees the allowed emails and the users : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L396 ]
[test: an admin allows an address, which is invited and can sign up, and takes it off again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L413 ]
[test: a domain on the list lets every address in it sign up, and is never emailed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L441 ]
[test: an allowed entry is an address or @ and a domain, trimmed and lowercased : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L48 ]
[test: an invite goes only to an address without an account : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L114 ]

## S8: Admins delete an account @v1 [proposed]

- Given: a signed-in admin on the admin page, which lists every account
- When: they choose Delete on another account, and confirm
- Then: that person is signed out at once, everywhere
- and their worker stops, with every process of their uid
- and their folder on the service is deleted, and only then the account,
  with its sessions, links, GitHub account and recent repos
- and it is gone from the list; the allowed entries they added stay
- and the address can sign up again only while it is allowed
- and nobody deletes their own account there, and the last admin stays
- and if their folder cannot be deleted, the account is kept, signed out,
  and the page says to try again

[test: an admin deletes an account, which is signed out and loses its worker and folder before the account goes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L474 ]
[test: when an account's folder cannot be deleted, the account stays, signed out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L504 ]
[test: an admin may delete another account, but not their own, a missing one or the last admin : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L131 ]
[test: deleting an account takes everything tied to it, and its allowed entries stay without who added them : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAdmin.test.ts#L145 ]
[test: a deleted account's folder goes once its worker is stopped, and never while it runs : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySpawner.test.ts#L240 ]
[test: deleting an account ends its uid's processes and deletes its folder, and touches no one else's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L387 ]

## S9: Change the password on the account page @v1 [proposed]

- Given: a signed-in person on the account page
- When: they give their current password and a new one
- Then: the new password is saved, every other session of the account
  ends, and this browser stays signed in
- and without the right current password nothing changes

[test: changing the password needs the current one and ends every session : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L150 ]
[test: changing the password signs this browser in again and ends the others : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L360 ]

## S10: Each person gets a worker of their own @v1 [proposed]

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

[test: signed in, the app's pages come from the gateway and its API from the worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L230 ]
[test: two users reach two different workers : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L239 ]
[test: request bodies pass through and streamed answers stream : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L249 ]
[test: two users get two workers with separate settings, folders and environments : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L106 ]
[test: restart stops the worker; the next request starts a fresh one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L133 ]
[test: idle workers are stopped by the sweep : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L180 ]
[test: user ids that are not UUIDs never become paths : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L102 ]
[test: page paths get the shell : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAppShell.test.ts#L26 ]
[test: bundled assets are served immutable, and nothing outside the build is : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAppShell.test.ts#L34 ]
[test: the API, plugin bundles and writes are the worker's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAppShell.test.ts#L42 ]

## S11: Only admins open the admin page @v1 [proposed]

- Given: a signed-in person who is not an admin
- When: they open `/admin` or call the `/auth/admin/*` API
- Then: both answer 403, the page with the gateway pages' strict content
  security policy
- and nobody else can add to the allowed emails: the old invite API is
  gone
- and signed out, `/admin` sends the browser to sign-in and comes back
  after, and the API answers 401

[test: an admin gives and takes the admin role, which applies at once, but never from the last admin : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L450 ]
[test: signed out, the admin page leads to sign-in and its API refuses : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L389 ]
[test: a new account is a user, with no admin API, no admin page and no invites to send : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L378 ]

## S12: Requests from anywhere else are refused @v1 [proposed]

- Given: the gateway answering at its public url
- When: a request names another host, or a write comes from another site
- Then: it is refused before it reaches a worker; the gateway's own
  `/auth/*` API passes the same Host, Origin and Content-Type checks as the
  app's
- and the gateway's own pages carry a strict content security policy, are
  never framed and never cached
- and the health check answers without a Host check or a session, for the
  hosting platform

[test: a request for another host name is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L164 ]
[test: a cross-site write is refused before it reaches the worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L278 ]
[test: the sign-in pages carry a strict CSP and are never framed or cached : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L206 ]
[test: the health check answers without a Host check or a session : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L159 ]

## S13: Reset a forgotten password @v1 [proposed]

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
[test: forgot answers the same for known and unknown addresses : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L313 ]
[test: a reset through the API ends the old session and restarts the worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L321 ]

## S14: Sign in and out @v1 [proposed]

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

[test: login, logout and a wrong password : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L291 ]
[test: repeated wrong passwords are rate limited : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L303 ]
[test: the rate limiter allows max attempts per window, then refuses until it passes : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L211 ]
[test: the session cookie is HttpOnly, SameSite=Lax, and __Host- Secure over https : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L200 ]
[test: a session is stored as a hash, slides with use and ends at 30 days : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L176 ]
[test: an unused session expires after 14 days : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayAccounts.test.ts#L192 ]
[test: tokens are stored as SHA-256 hex : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L116 ]
[test: signed out, the app redirects pages to sign-in and refuses the API : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L197 ]

## S15: Sign up with an email address @v1 [proposed]

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
[test: sign-up, set password, and the session reaches the user's worker without the gateway cookie : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L218 ]

## S16: The Admin item under Settings shows for admins only @v1 [proposed]

- Given: a signed-in person using the hosted app
- When: they open the user menu
- Then: an admin sees Admin under Settings, which opens the admin page;
  anyone else does not
- and the account page links to the admin page for admins only
- and the role comes from the gateway's `/auth/me`, not from the worker

[test: an admin opens the admin page and sees the allowed emails and the users : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L396 ]
[test: a new account is a user, with no admin API, no admin page and no invites to send : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L378 ]

## S17: The master key is rotated without losing secrets @v1 [proposed]

- Given: a deployment with stored secrets
- When: the operator sets a new master key and moves the current one to
  the previous keys, then redeploys
- Then: at start the gateway re-encrypts every secret under the new key,
  and from then on the secrets need only the new key
- and a master key is 32 bytes, kept as a Railway variable and never on
  the volume; in production the gateway refuses to start without one

[test: rotating the master key re-seals old rows, which then need only the new key : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L78 ]
[test: a master key must be 32 bytes, and production requires one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L50 ]

## S18: The original admin is set up on a database with no admin @v1 [proposed]

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

## S19: The start screen leads to the account and admin pages @v1 [proposed]

- Given: a signed-in person in the hosted app with no specs repo open yet,
  or whose worker does not answer, so there is no user menu
- When: the app shows its start screen, asking for a specs repository url,
  or says the server cannot be reached
- Then: the screen shows the account's email and links to the account
  page, to the admin page for an admin, and to Sign out, so an admin can
  administer the app without any specs repo
- and a clone GitHub refuses, such as with "Write access to repository not
  granted", says the token on the account page cannot read and write the
  repository, and links to that page
- and a clone, pull or push with no usable token says to add one on the
  account page, never to sign in with GitHub in Settings

[test: an admin with no specs repo gets the account page, the admin page and Sign out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedStartNav.test.tsx#L9 ]
[test: a user who is not an admin gets no admin link : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedStartNav.test.tsx#L19 ]
[test: a token GitHub refuses points at the account page's token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedStartNav.test.tsx#L30 ]
[test: other errors and no error get no hint : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedStartNav.test.tsx#L46 ]
[test: a refused clone points at the account page's token, not Settings : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L179 ]

## S20: Tokens are kept sealed, and no AI agent's API key is kept @v1 [proposed]

- Given: a signed-in person on the account page
- When: they save a GitHub token
- Then: it is checked with GitHub first, and the account it belongs to
  becomes the identity that signs their commits
- and it is encrypted with AES-256-GCM under the deployment's master key,
  bound to its owner and its name, so a copy of the database reveals
  nothing and a secret moved to another person or name does not open
- and the page shows its last four characters and the GitHub account, but
  never the value: the browser can only write it, replace it or remove it
- and it is opened only when the gateway starts that person's worker, and
  reaches it in an environment built from scratch: the GitHub token and
  none of the gateway's own keys
- and the worker only reads the GitHub sign-in it was given: signing in,
  out or to another account there is refused, and the app points to the
  account page instead
- and the service keeps no AI agent's API key: the account page has
  nowhere to save one, and keys saved before are deleted when the
  database is migrated, so the service is not responsible for them

[test: the GitHub token is checked with GitHub, stored sealed, shown by hint, and restarts the worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L334 ]
[test: the account keeps no AI agent API keys, with nowhere to save one and none listed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L350 ]
[test: agent API keys saved before are deleted when the database is migrated, and the GitHub token stays : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L96 ]
[test: a sealed secret opens with the same key and associated data : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L26 ]
[test: a ciphertext moved to another user or name, or altered, does not open : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L33 ]
[test: a different master key cannot open the secret : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L43 ]
[test: the store keeps ciphertext only and lists hints, never values : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySecrets.test.ts#L55 ]
[test: the worker environment carries the user's folders and secrets, not the gateway's keys : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L68 ]
[test: the env token backend reads the gateway's record and refuses to change it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L118 ]

## S21: Signing in opens the page that was asked for, and never another site @v1 [proposed]

- **Given** someone signed out who opens a page of the app, or follows a
  sign-in link that names where to go next
- **When** they sign in
- **Then** the browser opens the page they asked for, with its query and
  fragment; with nothing named it opens the app
- **And** a sign-in link naming another site — outright, or dressed up as a
  path with a second slash, a backslash, or a tab or newline the browser
  would drop — opens the app instead, so a link someone was sent can never
  lead from a real sign-in to somebody else's page

[test: a same-site path is kept, with its query and fragment : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayNext.test.ts#L13 ]
[test: no value, or one that is not a path, goes to the app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayNext.test.ts#L19 ]
[test: a value the browser would load from another site goes to the app : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayNext.test.ts#L27 ]
[test: what is returned is the path the browser resolves, not the raw value : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayNext.test.ts#L39 ]

## S22: The gateway never writes inside a person's folder @v1 [proposed]

- **Given** a worker's folders, whose contents the person's data repo can
  shape, links included
- **When** the gateway prepares the folders, starts the worker, notes the
  worker's repos or stops it
- **Then** nothing left in those folders changes or shows a file outside
  them: a link put where the gateway would write or read leads nowhere
- **And** the person's top folder is the gateway's: the worker can list it,
  and cannot add, rename or replace the five folders in it — home, root,
  repos, run and tmp — which are the worker's alone
- **And** the worker sets itself up as it starts: its account as git's
  author, and the remembered repos when it has no settings file (A person's
  recent data repos survive a redeploy)
- **And** the gateway reads a worker's settings as that worker's user id,
  in a process that carries none of the gateway's own keys
- **And** a folder that turns out to be a link, or another user id's, stops
  that worker from starting; folders from before this rule are brought in
  line on each worker's next start

[test: links a worker leaves in its folders lead the gateway nowhere : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L281 ]
[test: a folder that is a link, or another uid's, stops the worker from starting : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L316 ]
[test: the worker names its account as git's author, in a file of its own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkerStart.test.ts#L35 ]
[test: a link where the git author goes is replaced, and what it points at is untouched : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkerStart.test.ts#L43 ]
[test: the worker writes the remembered repos into a settings file that is missing, and keeps one that is there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkerStart.test.ts#L53 ]
[test: a command the gateway runs for a worker carries none of the gateway's environment : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayLimits.test.ts#L126 ]

## S23: Stopping a worker ends everything it started @v1 [proposed]

- **Given** a worker that started other processes and let go of them
- **When** the worker is stopped — idle, restarted for a changed secret or
  a password reset, or the service shutting down — or is about to start
  again
- **Then** every process running as its user id ends with it, and no other
  person's process is touched
- **And** this holds when the worker had already exited by itself
- **And** a process that ends whatever is sent to end it is ended all the
  same

[test: stopping a worker ends every process of its uid, and no one else's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L344 ]
[test: only a worker's uid can have its processes ended : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L558 ]
[test: a process that ends whatever is sent to end it is ended all the same : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L541 ]

## S24: A repository's contents cannot make a hosted worker run code @v1 [proposed]

- **Given** a data repo that holds a symlink, a hook in its `.git` folder,
  or a remote that is not an http(s) url
- **When** a hosted worker clones it, pulls, commits or pushes
- **Then** a symlink is checked out as a plain file holding the link's
  target, on the system git and on the bundled engine alike; to git it is
  still that link, unchanged
- **And** no hook runs
- **And** a remote that is a local path, or that names a command to run, is
  refused: a hosted worker fetches over http and https only
- **And** a clone from before this rule has its symlinks replaced the same
  way when the worker starts, with its `.git` folder left alone
- **And** on a person's own machine nothing changes: a symlink is checked
  out as one, and hooks run

[test: the %s engine of a hosted worker checks a symlink out as a plain file : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedGit.test.ts#L61 ]
[test: outside the hosted app the %s engine still checks a symlink out as one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedGit.test.ts#L82 ]
[test: a hosted worker's git runs no hook a write into .git left there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedGit.test.ts#L94 ]
[test: a hosted worker's git fetches over http(s) only, never from a local path or a command posing as a url : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedGit.test.ts#L116 ]
[test: symlinks in the cloned repositories become plain files holding their target; .git is left alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkerStart.test.ts#L73 ]

## S25: One person cannot use up what the others need @v1 [proposed]

- **Given** several people's workers sharing one container and one volume
- **When** a worker starts, and every minute while it runs
- **Then** it starts under a cap on its processes, 512, and on the files
  each may hold open, 4096, with a temp folder of its own
- **And** no worker starts while the volume has less than 256 MB free; the
  person is told the service is out of disk space
- **And** where a disk budget is set, a worker whose folder holds more is
  stopped, its person is told how much the folder holds, and it starts
  again once the folder is under the budget
- **And** where a memory budget is set, a worker holding more is stopped,
  and starts again on its person's next request
- **And** the service's variables set these numbers; the two budgets are off
  unless set

[test: a worker starts under the kernel's caps on processes and open files : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L568 ]
[test: limits come from the environment; what is unset or not a number keeps its default : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayLimits.test.ts#L40 ]
[test: a folder's size and the volume's free space are measured; what is not there is not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayLimits.test.ts#L55 ]
[test: no worker starts while the volume is nearly full : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayLimits.test.ts#L66 ]
[test: a worker whose folder is over its budget is stopped, and starts again once it is under : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayLimits.test.ts#L80 ]
[test: a worker holding more memory than its budget is stopped, and starts again when asked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayLimits.test.ts#L105 ]
[test: where there is no /proc entry, a process's memory is not known : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayLimits.test.ts#L119 ]

## S26: A worker's answer cannot act on the gateway's pages @v1 [proposed]

- **Given** a signed-in person whose worker answers a request
- **When** the gateway passes the answer on to the browser
- **Then** no cookie the worker set goes with it, so the session stays the
  one the gateway gave
- **And** an answer on the API, opened as a page, runs no script and loads
  nothing

[test: a worker's answer sets no cookie, and on the API cannot run as a page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayServer.test.ts#L260 ]

## S27: The part that answers the internet is not root @v1 [proposed]

- **Given** the hosted service, whose first process starts as root so that
  each person's worker can have a user id of its own
- **When** it starts
- **Then** that first process only readies the volume and starts, stops and
  reports on workers; it answers no web request and opens no network
  connection
- **And** the gateway — which answers the internet, signs people in and
  holds the master key — is started by it as a user id that is not root
- **And** the gateway keeps its database and the datastore's clone in a
  folder of its own on the volume; files an older volume kept beside the
  users' folders are moved into it on the first start
- **And** the gateway asks for a worker over a socket that only it can
  open, after saying a key made at that start; anything else that reaches
  the socket is cut off at its first line
- **And** nothing in such a request picks what root does: the user id must
  be one the gateway made, the user id to run as must be a worker's, never
  root's or the gateway's, and the worker's environment reaches no program
  that runs as root
- **And** a stop from the platform ends the gateway, then every worker
- **And** nothing a worker leaves for root to read can end the gateway's
  connection to it
- **And** on a developer's machine there is one process, and it says so

[test: started as root, the gateway that answers requests is not root, and a person's worker runs as their own uid : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySpawner.test.ts#L322 ]
[test: the spawner starts a worker only for a user id that is one and a uid that is a worker's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySpawner.test.ts#L57 ]
[test: the spawner takes an environment of plain names and text, and caps and a port that are numbers in range : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySpawner.test.ts#L77 ]
[test: the spawner's socket answers only a connection that says the key, and only messages of its own kind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySpawner.test.ts#L131 ]
[test: a wrong key is refused, a worker's end reaches the gateway, and a lost spawner is noticed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySpawner.test.ts#L190 ]
[test: a settings file filled to break the spawner's connection fails that one answer and nothing else : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySpawner.test.ts#L167 ]
[test: a worker started through the spawner's socket serves its person, and stops when asked : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySpawner.test.ts#L209 ]
[test: the gateway's folder on the volume takes in the database and datastore an older volume kept beside it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySpawner.test.ts#L277 ]
[test: root hands the gateway's folder to the gateway's uid once, and stays out of it afterwards : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySpawner.test.ts#L302 ]
[test: a worker's settings are read as text by one side and made sense of by the other : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayRepos.test.ts#L96 ]
[test: the launcher becomes the program it is told to, in its folder and with its environment alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L160 ]
[test: the worker command drops to the user's uid with no way back, under its caps : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L588 ]

## S28: A worker serves only its own gateway's requests @v1 [proposed]

- **Given** a worker, which owns the folder its socket is in and could put a
  link to another person's socket there
- **When** a request reaches a worker
- **Then** the worker answers only a request that carries the key the
  gateway made for that worker when it started it; any other is refused
- **And** so a request the gateway was led to send to another person's
  worker is refused there
- **And** a browser cannot bring a key of its own: the gateway's replaces it
- **And** every start of a worker gets a new key
- **And** the gateway, which is neither root nor the worker, reaches the
  worker's socket and nothing else in the person's folder; no other
  person's worker reaches the socket

[test: a worker refuses a request that does not carry its own key : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L145 ]
[test: the gateway's uid reaches a worker's socket; another worker's uid does not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L414 ]
[test: a hosted worker given a key answers only a request that carries it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L188 ]

## S29: The account pages have a host of their own @v1 [proposed]

- **Given** a service set up with an account url as well as its public url
- **When** a browser asks either host for something
- **Then** the account host serves sign-in, sign-up, password reset, the
  account page, the admin page and their API, and sends a request for
  anything else to the app's host
- **And** the app's host serves the app and each person's worker, sends a
  request for one of the account pages to the account host, and serves none
  of the account's API
- **And** the app's host says who is signed in — the email and the role —
  and no more of the account
- **And** a change sent to the account host from a page on the app's host is
  refused, with the person's cookies or without
- **And** the sign-in's cookie means nothing on the app's host, and the app
  host's cookie means nothing on the account host
- **And** email links lead to the account host
- **And** signing out on either host, or changing the password, ends the
  person's sessions on both
- **And** only sign-ins are copied to the datastore
- **And** without an account url everything stays on the one host, and the
  service says at start that the account pages share it

[test: each host serves its own half and points to the other for the rest : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L329 ]
[test: the app host's cookie reaches the worker and nothing else; the sign-in cookie means nothing on the app's host : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L189 ]
[test: script shown on the app's host cannot write to the account host, with the person's cookies or without : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L215 ]
[test: email links lead to the account host : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L357 ]
[test: signing out on either host ends both sessions, and so does a changed password : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L364 ]
[test: only sign-ins go into the datastore, where an app session read back would count as one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L415 ]
[test: with one host, the app's own session answers are the sign-in's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L517 ]
[test: the account url is an https url on another host than the app's, and needs the app's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L533 ]
[test: each of the gateway's three cookies belongs to one host, is hidden from script, and never reaches a worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L470 ]

## S30: A signed-in browser is handed to the app's host with a code that works once @v1 [proposed]

- **Given** a service with an account host, and a browser with no session on
  the app's host
- **When** the browser opens a page of the app
- **Then** it is sent to the account host, signs in there if it has not, and
  is sent back with a code; the app's host then gives it a cookie of its
  own and shows the page that was asked for
- **And** the code works once, for a minute
- **And** the code works only together with a secret the app's host left in
  that browser before sending it off, so a code made for one browser signs
  no other browser in
- **And** several tabs sent off together all come back signed in
- **And** a secret the app's host did not make itself is never used, so
  script on the app's host cannot put one of its own in the browser
- **And** where the browser lands is always a page on the app's host
- **And** the app's session ends when the sign-in it came from ends, and
  using the app keeps that sign-in alive, up to 30 days from signing in
- **And** a browser that cannot finish is shown a page to try again from,
  not sent round again

[test: signed out, the app's host sends a page to the account host to sign in, and refuses the API : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L142 ]
[test: a signed-in browser is handed over with a code and lands on the page it asked for, with a cookie of the app host's own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L166 ]
[test: a hand-over code works once, only with the secret in the browser it was made for, and for a minute : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L242 ]
[test: several tabs sent off together all come back signed in : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L297 ]
[test: a hand-over secret the gateway did not make is never used : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L278 ]
[test: where to land after a hand-over is a path on the app's host, whatever the link says : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L313 ]
[test: using the app keeps the sign-in alive, up to 30 days from signing in : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L393 ]

## S31: The app's pages run only the app's own script @v1 [proposed]

- **Given** a signed-in person, and specs whose markdown, images and
  diagrams a teammate can shape
- **When** the gateway serves one of the app's pages
- **Then** the page carries a policy under which only script served by the
  service's own host runs, and the page's one inline script by its hash
- **And** no text is run as script
- **And** the page sends nothing to another site and is framed by none; the
  one exception is the diagram editor's fonts, from the one address they
  are published at
- **And** the service's variables can send the policy as report-only, where
  the browser says what would be blocked and blocks nothing, or turn it off

[test: the app's pages carry a policy that runs only the app's own script : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L424 ]
[test: an inline script is hashed as the browser reads it, however its tags are written : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayHosts.test.ts#L496 ]

## S32: A worker is confined to its own folder and one way out @v1 [proposed]

- **Given** a worker running as its person's user id, on a kernel that lets
  a process narrow itself
- **When** the worker is started
- **Then** it can read the system's files and the app's code, and change
  neither
- **And** it can write in its own five folders and nowhere else — not in
  the shared temp folder, and not in a file anyone on the machine may write
- **And** it can read no file outside those, even one anyone may read
- **And** it can connect to one port, the gateway's outbound proxy, and
  listen on none; it can make no UDP socket
- **And** the proxy lets a connection through to GitHub and GitHub's API and
  refuses every other, saying so in the log; git and the app's own requests
  go through it
- **And** the worker can make no call that reaches another process's
  memory, a mount, a namespace or the kernel itself
- **And** everything the worker starts is held to the same
- **And** the proxy holds a limited number of connections at once, and one
  that does not say where it wants to go is closed
- **And** the service's variables can add hosts to the proxy's list, and can
  turn the confinement off, leaving the user id of its own

[test: a worker is confined to its own folders and the proxy's port, and gets only its own environment : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L452 ]
[test: the worker's environment sends git and its own requests through the gateway's proxy, when there is one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkers.test.ts#L166 ]
[test: a worker's plan gives it its own folders, the system and the app's code to read, and one port : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L33 ]
[test: the file rights follow the kernel's Landlock version, and the code is never writable : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L47 ]
[test: TCP is confined from Landlock 4, and only when there is a port to allow : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L72 ]
[test: the %s system-call filter lets a program's everyday calls through : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L79 ]
[test: the %s filter refuses the calls that reach other processes, mounts, namespaces and the kernel : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L86 ]
[test: the %s filter lets a thread or a child be made, never a namespace : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L100 ]
[test: the %s filter leaves a worker unix sockets and TCP, and no other kind : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L115 ]
[test: a call made as another architecture ends the process, and x86-64's second numbering is refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L149 ]
[test: workers may reach GitHub and its API, a stand-in the environment names, and what the operator adds : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayEgress.test.ts#L53 ]
[test: a tunnel to a host on the list is opened, and bytes pass both ways : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayEgress.test.ts#L70 ]
[test: a tunnel anywhere else is refused, and the refusal is logged : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayEgress.test.ts#L79 ]
[test: a plain http request goes through to a host on the list, one request a connection, and nowhere else : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayEgress.test.ts#L94 ]
[test: what is not a proxy request is turned away : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayEgress.test.ts#L114 ]
[test: the proxy holds a limited number of connections at once, and closes one it refused : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayEgress.test.ts#L123 ]
[test: git, told of the proxy through its environment, reaches the stand-in through it and nothing else : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayEgress.test.ts#L153 ]

## S33: The service says at each start what holds its workers @v1 [proposed]

- **Given** a platform whose kernel may or may not let a process narrow
  itself, or may do so for files and not for the network
- **When** the service starts
- **Then** it starts a throwaway process the way it starts a worker, has it
  try what a worker must be able to do and what it must not, and logs one
  line: the kernel, and which of files, network and system calls are held
- **And** what the kernel does not offer is left out and named in that
  line; the user id of its own still keeps people's files apart
- **And** a worker that could not do something it must is reported as
  broken in that line
- **And** `bun src/gateway/index.ts probe` prints the same line on any
  machine

[test: on this kernel, a confined process keeps to its folder and its one port : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L242 ]
[test: what holds a worker is said in one line, and so is what does not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L182 ]
[test: asking what this kernel offers changes nothing, and the answer names what is missing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L207 ]
[test: where the system has no way to confine a worker, nothing is applied and that is said : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L227 ]
[test: the confinement is on unless the service is told otherwise : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewaySandbox.test.ts#L233 ]

## S34: A person connects their GitHub account instead of pasting a token @v1 [proposed]

- Given: a hosted service that has a GitHub App of its own, and a signed-in
  person on their account page
- When: they press Connect GitHub and confirm at GitHub
- Then: the account page shows the GitHub account they connected, and a
  link to install the app on their specs repository
- and the gateway keeps who they are on GitHub, an id and a login, and no
  token: the one GitHub gave for the sign-in answers that one question and
  is revoked at once
- and a token they pasted before is deleted, and their worker starts again
  without one; until they connect, that token keeps working
- and no token can be pasted any more
- and a sign-in that did not set out from this browser, for this account,
  connects nothing: not a code someone else sent, not one whose way back
  was changed, and not one the person cancelled at GitHub
- and setting out for GitHub is something only the account page does: a
  link, or a page somewhere else that a signed-in browser is sent to,
  starts nothing; signed out, nothing sets out, and the way back leads to
  sign-in first
- and a GitHub account that is connected to one person's account here
  cannot be connected to another's until it is disconnected there, and the
  page says so
- and Disconnect removes the link and stops their worker; what stops a
  worker from the account page can be done only so often in an hour
- and who a person is on GitHub goes to the datastore with their account,
  so a redeploy without the volume still knows it
- and the app is set up with three of the service's variables, a client
  id, a client secret and a private key: with none of them people paste a
  token as before, and with only some the service refuses to start

[test: with the app, the account page offers connecting and takes no pasted token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubConnect.test.ts#L134 ]
[test: without the app nothing of it is served : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubConnect.test.ts#L145 ]
[test: signed out, nothing sets out for GitHub, and the way back leads to sign-in first : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubConnect.test.ts#L155 ]
[test: only the account page sets out for GitHub, never a link or another site's page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubConnect.test.ts#L165 ]
[test: a GitHub account that is one person's here cannot be connected to another's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubConnect.test.ts#L185 ]
[test: what stops a person's worker is limited, so it cannot be done without end : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubConnect.test.ts#L214 ]
[test: back from GitHub with its code, the person is who GitHub says, the pasted token is deleted, and the worker starts again : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubConnect.test.ts#L226 ]
[test: a code that did not come back to the browser that set out links nothing : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubConnect.test.ts#L248 ]
[test: GitHub's own return after the app was installed on a repository just shows the account page : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubConnect.test.ts#L277 ]
[test: disconnecting removes the link and stops the worker, which ends its tokens : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubConnect.test.ts#L285 ]
[test: the secret a browser carries to GitHub never reaches its worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubConnect.test.ts#L307 ]
[test: a sign-in says who the person is, and its token is revoked at once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L143 ]
[test: where the app is installed comes from GitHub's own description of it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L157 ]
[test: who a person is on GitHub is kept without a secret, goes to the datastore and comes back : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L271 ]
[test: none of the three variables means no app; some of them is a mistake that stops the start : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L107 ]
[test: the private key is read as the file, with its line ends escaped, or in base64 : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L114 ]

## S35: A worker gets a GitHub token for one repository, for an hour @v1 [proposed]

- Given: a person who has connected GitHub, whose worker is about to fetch
  from a repository or push to it
- When: the worker's git asks the gateway for that repository's token
- Then: the gateway makes one that lasts an hour and reaches that
  repository alone, and only when the app is installed on the repository
  and the person's GitHub account has access to it there
- and the token can write only if the person can: someone who can only
  read the repository gets a token that can only read it
- and what a person may do is asked of GitHub each time a token is made,
  by the account's id, so access taken away on GitHub is gone here within
  the hour, and a login that passed to somebody else gets nothing
- and the worker holds no GitHub token, in its environment or on disk: its
  git gets the token through a helper as it needs it, on the system's git
  and on the bundled engine alike, and the commits are the person's
- and with no token to give, the sync fails as it does with no sign-in,
  and the app says to connect GitHub and install the app on the repository;
  the refusal reads the same whether the app is not installed there or the
  person has no access, so asking tells nobody which repositories exist
- and the token is for the repository GitHub named and no other: a
  repository that was renamed or moved is not followed to its new place
- and a token is handed out again until its hour is nearly over, a refusal
  stands for a minute, and one person can make the gateway ask GitHub only
  so many times in ten minutes, however often their worker stops and
  starts; so can everyone together
- and a token for a repository the person can no longer reach is revoked
  when that is found out
- and when the worker stops, for any reason, the tokens it was given are
  revoked

[test: a worker's git opens the repository with a token it asked the gateway for, and the token ends with the worker : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkerGithub.test.ts#L128 ]
[test: with the service's app, a worker is told who the person is on GitHub and where to ask, and holds no token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayWorkerGithub.test.ts#L105 ]
[test: its own word is signed with the private key, for nine minutes, in the app's name : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L131 ]
[test: a token is for one repository, to write it when the person may write it there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L167 ]
[test: a person who can only read gets a token that can only read : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L215 ]
[test: no token where the app is not installed, the person has no access, or the login is now someone else's : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L224 ]
[test: a repository's name is owner/name and nothing that names another path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L121 ]
[test: no token for a person who has not connected GitHub : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L292 ]
[test: a token is handed out again until its hour is nearly over, then a new one is made and the old one ended : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L299 ]
[test: asking twice at once asks GitHub once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L313 ]
[test: a refusal stands for a minute, and a worker cannot make the gateway ask GitHub without end : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L321 ]
[test: everyone's workers together can ask GitHub only so often : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L346 ]
[test: a token whose repository the person can no longer reach is ended when that is found out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L366 ]
[test: the token handed out is for the repository GitHub named, by its id, and never for another : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L184 ]
[test: a refusal says the same whether the app is not installed or the person has no access : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L206 ]
[test: when a worker stops, the tokens it was given end, and one still being made is never handed out : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayGithubApp.test.ts#L382 ]
[test: git's helper answers git with the repository's token, and with nothing when there is none to ask for : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayTokenSocket.test.ts#L135 ]
[test: a real git clones and pushes with the token its helper fetched, which is nowhere in its environment : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayTokenSocket.test.ts#L147 ]
[test: the bundled engine asks the gateway the same way, with no git installed : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayTokenSocket.test.ts#L201 ]
[test: the repository is the first two parts of a url's path : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayTokenSocket.test.ts#L127 ]
[test: with the service's GitHub App, the sign-in is who the person is and holds no token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L130 ]
[test: what a failed sync says to do names the app where the service has one : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedMode.test.ts#L144 ]
[test: where the service has a GitHub App, the hint names the app and the connected account, not a token : https://github.com/SpecDriven/specdriven-app/blob/main/tests/hostedStartNav.test.tsx#L40 ]

## S36: Only a person's own worker can ask for their GitHub tokens @v1 [proposed]

- Given: several people's workers in one container, each with a user id of
  its own
- When: a worker's git asks for a token
- Then: it asks on a socket of that worker's own, in a folder that belongs
  to the gateway and that only that worker's user id may pass through:
  another worker's user id cannot reach the socket, and the worker itself
  can neither replace it nor put anything beside it
- and the gateway answers only when the other end of the connection is
  that worker's user id, as the kernel reports it: anyone else, root
  included, is hung up on before anything is read
- and which person is asking is never something the request says
- and one connection carries one request: what is not a request gets no
  token, and nothing after the first line is read
- and the socket goes when the worker stops, and a worker that asks a
  moment before its gateway listens waits for it

[test: the gateway's uid listens in a folder only that worker can enter, and answers that worker's uid alone : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayTokenSocket.test.ts#L244 ]
[test: the kernel says which uid is on the other end of a socket : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayTokenSocket.test.ts#L300 ]
[test: only the worker's uid is answered, and anyone else on the other end is hung up on unheard : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayTokenSocket.test.ts#L98 ]
[test: a worker asks for one repository and gets its token, or the reason there is none : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayTokenSocket.test.ts#L78 ]
[test: one request a connection, so what is not a request gets no token and nothing after the first line is read : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayTokenSocket.test.ts#L87 ]
[test: the socket goes with the worker, and one that is not there yet is waited for a moment : https://github.com/SpecDriven/specdriven-app/blob/main/tests/gatewayTokenSocket.test.ts#L114 ]

## S37: Start AI Session opens Claude Code with the prompt @v1 [proposed]

- Given: a signed-in person in the hosted app, with specs checked in the
  sidebar or single scenarios checked on their cards, or with only a
  feature open
- When: they click Start AI Session
- Then: no dialog opens and no agent runs on the service: a new browser tab
  opens on Claude Code on the web (claude.ai/code) with the prompt for what
  is checked — or, with nothing checked, for the open feature — in the
  link, so it arrives in Claude's box under the person's own claude.ai
  sign-in
- and the prompt gives the specifications in full and names no folder of
  the service: the work runs in the cloud, on the repository the person
  picks there
- and the note under the button says the tab is open, and to pick the
  repository and send
- and the prompt is copied to the clipboard as well: one too long for a
  link opens Claude Code without it, and the note says to paste it
- and a browser that blocks the new tab gets a link in the note that opens
  Claude Code with the prompt; a prompt too long for a link that could not
  be copied either opens nothing, and the note offers a Copy prompt button
- and with nothing checked and no feature open, nothing opens and the app
  says what to select
- and it works in Safari too, which opens a tab and copies only during the
  click: the tab is opened and the clipboard handed the prompt while the
  prompt is still being built

[test: the hosted app copies what is checked, else the open feature, else says what to do : https://github.com/SpecDriven/specdriven-app/blob/main/tests/startSessionDialog.test.tsx#L40 ]
[test: a text still on its way is copied once it arrives, through a ClipboardItem made at once : https://github.com/SpecDriven/specdriven-app/blob/main/tests/clipboard.test.ts#L40 ]
[test: a text that fails to arrive throws its own error, and nothing is copied : https://github.com/SpecDriven/specdriven-app/blob/main/tests/clipboard.test.ts#L49 ]
[test: a browser that refuses the ClipboardItem write gets the text the plain way : https://github.com/SpecDriven/specdriven-app/blob/main/tests/clipboard.test.ts#L55 ]
[test: the prompt rides in q with its newlines, spaces and ampersands encoded : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWeb.test.ts#L43 ]
[test: a prompt whose link would pass the cap gets no link : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWeb.test.ts#L51 ]
[test: an empty tab opens at once, without a handle on the app, and goes where it is sent : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWeb.test.ts#L81 ]
[test: a blocked pop-up is no tab : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWeb.test.ts#L99 ]
[test: in a browser the copy is asked for and the tab opened before the prompt is there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWeb.test.ts#L105 ]
[test: a prompt that fits its link opens Claude Code with it, copied or not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWeb.test.ts#L116 ]
[test: a prompt too long for a link is pasted, the tab on Claude Code without it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWeb.test.ts#L123 ]
[test: too long and not copied closes the tab and hands the prompt back to copy : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWeb.test.ts#L130 ]
[test: a blocked tab leaves a link, with the prompt when it fits and to paste into when not : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWeb.test.ts#L136 ]
[test: a prompt that fails to build closes the tab and throws its own error : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWeb.test.ts#L145 ]
[test: says the tab is open and what is left to do there : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWebNote.test.tsx#L15 ]
[test: a blocked tab leaves a link that opens Claude Code in a tab of its own : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWebNote.test.tsx#L28 ]
[test: a prompt that went nowhere offers one classed button to copy it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/claudeWebNote.test.tsx#L39 ]
[test: a session in the cloud is told the specs are below, and no folder of this machine : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agent.test.ts#L97 ]
[test: the prompt route builds the cloud prompt, with no folder of this machine in it : https://github.com/SpecDriven/specdriven-app/blob/main/tests/agentSession.test.ts#L313 ]
