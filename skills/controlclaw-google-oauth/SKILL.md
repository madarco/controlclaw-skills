---
name: controlclaw-google-oauth
description: Set up a Google Cloud project and OAuth client for a Google account, so it can be connected to ControlClaw (Integrations, Google) and shared with agents for Gmail, Calendar, Drive, Contacts, Sheets and Docs. Drives a browser; the user signs in to Google themselves. Use when someone wants to connect a Google account to ControlClaw and has no OAuth client JSON yet.
---

# Google OAuth client for ControlClaw

ControlClaw does not ship its own Google OAuth app. Each organization brings an OAuth client from a
Google Cloud project it owns, and drops its client JSON into **Integrations, Google, Set up**. This
skill creates that project and client in the user's Google account, in a browser you drive.

The result is:

- a Google Cloud project with the six APIs ControlClaw uses turned on,
- a consent screen whose sign-ins do not expire: **Internal** for a Google Workspace organisation, or
  **External** and published **In production** (unverified) for anyone else,
- a **Web application** OAuth client with ControlClaw's redirect URI,
- the client JSON saved on disk, mode `0600`.

It takes about 10 minutes, most of it waiting for the console.

## Before you start

Ask the user for:

1. **The Google account** (e.g. `someone@gmail.com`). It will own the project and be the account
   ControlClaw connects to.
2. **Where to save the client JSON.** Suggest `~/controlclaw-google-oauth-<project-id>.json`.
3. **Extra redirect hosts**, only if they test ControlClaw from somewhere other than
   `controlclaw.com` (a dev tunnel host, for example). Most people have none.
4. **Internal or External**, unless the account is a `gmail.com` (or `googlemail.com`) address,
   which can only be External; do not ask then. Otherwise ask: "Does this account belong to a
   Google Workspace organisation, and will only accounts in that organisation sign in to this
   app?" Yes means **Internal**; no, or not sure, means **External**. Internal has no Testing mode,
   no 7-day sign-out and no "unverified app" warning, so steps 5 and 7 are skipped.

Use a **fresh browser session** (a new tab in your own browser pane, or a new agent-browser
session), not the user's everyday browser, so the project lands in the right account.

## What you must not do

- Never type the user's password, never solve a CAPTCHA, never complete 2-step verification. Fill the
  email, press Next, and hand the browser to the user.
- Ask before ticking "I agree to the Google API Services: User Data Policy". That is the user
  accepting Google's terms.
- Ask before **Publish app**. It changes who can sign in to the app.
- Do not paste the client secret into chat. Read it from the page and write it straight to the file.

## Steps

Console URLs take `?project=<project-id>`; keep it on every URL after step 2, or the console may act
on another project.

### 1. Sign in (the user does this)

Open:

```
https://accounts.google.com/v3/signin/identifier?continue=https%3A%2F%2Fconsole.cloud.google.com%2F&flowName=GlifWebSignIn
```

Type the email, press Next. Google usually shows a reCAPTCHA ("Verify it's you") on a fresh
browser. Tell the user to tick it, enter the password, finish any 2-step check, and say when the
Cloud console has loaded. Wait for them.

A first-time account lands on "Try Google Cloud with $300 in free credits". Ignore it; no billing
is needed for any of this.

### 2. Create the project

`https://console.cloud.google.com/projectcreate`

Name it e.g. `ControlClaw <Family or Org>`. The project ID is derived from the name and shown under
the field (`controlclaw-moma` for `ControlClaw MoMa`); note it. Parent resource "No organization" is
right for a gmail.com account; a Workspace account should leave it on its organisation, which
Internal needs. Create, and wait about 10 seconds until the console switches to the
new project's dashboard.

### 3. Turn on the APIs

One URL enables all six:

```
https://console.cloud.google.com/flows/enableapi?apiid=gmail.googleapis.com,calendar-json.googleapis.com,drive.googleapis.com,people.googleapis.com,sheets.googleapis.com,docs.googleapis.com&project=<project-id>
```

Next, then Enable. Wait until the page says "You have successfully enabled". These map to
ControlClaw's services: Gmail, Calendar, Drive, Contacts (People API), Sheets, Docs. ControlClaw
checks each ticked service on connect, and a service whose API is off fails the connection with
"This Google account cannot use everything you asked for", so enable all six even if the user only
wants some today.

### 4. Consent screen

`https://console.cloud.google.com/auth/overview/create?project=<project-id>`

Tour pop-ups ("Click on the menu anytime", "Is this a production environment?") cover the form on a
new account; close them first.

- **App name**: `ControlClaw <Family or Org>`. The user sees it on Google's sign-in screen.
- **User support email**: the account (pick it from the dropdown).
- **Audience**: what the user chose before you started. **Internal** only lets accounts in the
  Workspace organisation sign in, and the grant does not expire. **External** lets any Google
  account sign in, and starts in Testing (steps 5 and 7). A gmail.com account only offers
  External. If Internal is greyed out, the project has no organisation parent: use External and
  tell the user why.
- **Contact email**: the account; press Enter so it becomes a chip.
- **Finish**: stop and ask the user to confirm the User Data Policy checkbox. Then tick it,
  Continue, Create. The page shows "OAuth configuration created!".

### 5. Test user (External only)

Skip this for Internal.

`https://console.cloud.google.com/auth/audience?project=<project-id>`

Add users, type the account, Enter, Save. **Check the Test users table afterwards.** Save sometimes
closes the panel without saving ("No rows to display"); if so, add it again. It only matters while the
app is in Testing, but step 7 may be skipped.

### 6. OAuth client

`https://console.cloud.google.com/auth/clients/create?project=<project-id>`

- **Application type**: Web application.
- **Name**: `ControlClaw (controlclaw.com)`. Only shown in the console.
- **Authorized JavaScript origins**: none.
- **Authorized redirect URIs**: `https://controlclaw.com/oauth/callback`, plus one per extra host
  the user gave you (`https://<host>/oauth/callback`). Google must be able to reach it, so a
  `*.localhost` host does not work.
- Create.

The "OAuth client created" dialog is **the only time the secret is shown**. Do not close it until
the file is written.

The dialog's "Download JSON" button saves into the browser's download folder, which you may not be
able to reach. It is simpler to read the client ID (ends in `.apps.googleusercontent.com`) and the
secret (starts with `GOCSPX-`) from the dialog (the page's accessibility tree has both, on the
"Copy to clipboard" buttons) and write the file yourself, in the format Google's download uses:

```bash
umask 077
python3 - "$OUT" <<'EOF'
import json, sys
json.dump({"web": {
  "client_id": "<client-id>",
  "project_id": "<project-id>",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_secret": "<secret>",
  "redirect_uris": ["https://controlclaw.com/oauth/callback"],
}}, open(sys.argv[1], "w"))
EOF
chmod 600 "$OUT"
```

Check the file parses and has all seven keys, then close the dialog with OK. If the file is lost
later, the secret cannot be viewed again: the client's page has "Add secret" to make a new one.

### 7. Publish (External only, ask first)

Skip this for Internal: an Internal app has no Testing mode and nothing to publish.

In **Testing**, Google throws the grant away after 7 days and the user must "Sign in again" in
ControlClaw every week. Publishing fixes that. Explain this and ask before doing it.

1. `https://console.cloud.google.com/auth/branding?project=<project-id>`: set
   - Application home page `https://controlclaw.com`
   - Privacy policy link `https://controlclaw.com/privacy`
   - Terms of service link `https://controlclaw.com/terms`

   `controlclaw.com` is already an authorized domain (the redirect URI added it). Save; the page
   shows "Branding changes saved!".
2. `https://console.cloud.google.com/auth/audience?project=<project-id>`: **Publish app**, Confirm.
   Status becomes **In production**.

The External app stays **unverified**, which is fine for a family or a small team: Google shows an "unverified
app" screen at sign-in, and an unverified app can be granted by at most 100 accounts over its
lifetime. Do not submit it for verification; that is a weeks-long review for an app one account
uses.

## Hand over

Tell the user:

- the project ID, the client name, the redirect URI(s), and where the JSON is,
- that the JSON holds a secret: keep it out of git and chat,
- how to connect it:
  1. ControlClaw, **Integrations, Google, Set up**; drop in the JSON.
  2. Tick the services the agents need.
  3. Sign in with Google as the account. For an External app, on "Google hasn't verified this
     app", click **Advanced**, then **Go to ControlClaw ... (unsafe)**; an Internal app skips that
     screen. Allow every permission listed.
  4. Confirm with the code ControlClaw sends to the approved channel, then grant the connection to
     an agent.
- that Google says new client settings take "5 minutes to a few hours"; an early `redirect_uri_mismatch`
  or `invalid_client` usually just needs a wait.

## Troubleshooting

| What the user sees | Why | Fix |
| --- | --- | --- |
| "This Google account cannot use everything you asked for. \<Service\>: ... has not been used in project ... or it is disabled" | That service's API is off | Enable it (step 3 URL), wait a minute, connect again |
| `Error 403: access_denied` "has not completed the Google verification process" | App is in Testing and the account is not a test user | Add the test user (step 5) or publish (step 7) |
| `Error 400: redirect_uri_mismatch` | The host ControlClaw is on is not a redirect URI on the client | Add `https://<host>/oauth/callback` to the client, wait a few minutes |
| Connection stops working after a week | External app still in Testing | Publish (step 7), then "Sign in again" on the Google card |
| `Error 403: org_internal` | The app is Internal and the account signing in is outside its Workspace organisation | Sign in with an account in the organisation, or switch the audience to External (Audience page, **Make external**) and do steps 5 and 7 |
| Google says the app "already has access to N permissions" | The account connected before; Google merges the old grant with the new request | Nothing; continue |
