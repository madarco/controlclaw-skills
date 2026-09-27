---
name: controlclaw-google-drive
description: Set up a Google service account and JSON key so Google Drive folders can be mounted on ControlClaw agents (Integrations, Google, Drive folders). Drives a browser; the user signs in to Google themselves and shares the folders. Use when someone wants their agents to read or write files in Google Drive folders through ControlClaw.
---

# Google Drive folders for ControlClaw

ControlClaw can mount Google Drive folders on an agent as ordinary directories
(`~/.openclaw/workspace/Drive/<name>`). It reaches them with a **service account**: a Google identity
the user creates in their own Cloud project, which can see **only the folders shared with it**.
Nothing else in the user's Drive is reachable. The user pastes the service account's JSON key into
**Integrations, Google, Drive folders**. The key goes to their firewall and stays there.

This is separate from the Google account connection (`/controlclaw-google-oauth`), which gives
agents Gmail, Calendar, Drive and the rest through the account itself. Either works without the
other. Folders are the narrower grant: the agent sees a few folders, not the whole Drive.

The result is:

- a service account in a Cloud project with the Drive API on,
- its JSON key saved on disk, mode `0600`,
- the user's chosen folders shared with the service account,
- the folder links, ready to paste into ControlClaw.

## Before you start

Ask the user for:

1. **The Google account** that owns the folders.
2. **Which folders**, and for each whether agents only read it or may change it.
3. **Where to save the key.** Suggest `~/controlclaw-drive-<project-id>.json`.

**Read-write needs Google Workspace.** A service account owns no storage, so Drive refuses to let it
create a file in an ordinary My Drive folder, even one shared as Editor. A read-write folder has to
be on a **shared drive**, which only Workspace accounts have. With a plain gmail.com account every
folder is read-only in practice. Tell the user this before they plan on the agent saving files.

**Google Docs, Sheets and Slides do not show up in a mounted folder.** Drive has no file for them to
download, so ControlClaw hides them rather than show empty files. For those, use the Google account
connection instead.

## What you must not do

- Never type the user's password, never solve a CAPTCHA, never complete 2-step verification.
- Do not paste the key's contents into chat. It holds a private key.

## Steps

Keep `?project=<project-id>` on every console URL.

### 1. Sign in (the user does this)

As in `/controlclaw-google-oauth`: open a fresh browser session at

```
https://accounts.google.com/v3/signin/identifier?continue=https%3A%2F%2Fconsole.cloud.google.com%2F&flowName=GlifWebSignIn
```

type the email, press Next, and let the user handle the reCAPTCHA, password and 2-step check.

### 2. Project and Drive API

If the user already ran `/controlclaw-google-oauth`, reuse that project: the Drive API is on. If
not, create one at `https://console.cloud.google.com/projectcreate` and enable Drive:

```
https://console.cloud.google.com/flows/enableapi?apiid=drive.googleapis.com&project=<project-id>
```

The key must belong to the project where Drive is enabled, or every folder fails with "Google Drive
API has not been used in project ... or it is disabled".

### 3. Service account

`https://console.cloud.google.com/iam-admin/serviceaccounts/create?project=<project-id>`

- **Name**: `controlclaw-drive`. The ID fills itself in.
- **Description**: `ControlClaw agents: Drive folders shared with this account`.
- **Create and close.** Skip Permissions and Principals with access. It needs no roles: what it can
  reach is decided by Drive sharing, not by Cloud IAM.

Note its email: `controlclaw-drive@<project-id>.iam.gserviceaccount.com`.

### 4. JSON key

`https://console.cloud.google.com/iam-admin/serviceaccounts/details/<service-account-email>/keys?project=<project-id>`

**Add key, Create new key, JSON, Create.** The browser downloads the key immediately and it cannot be
downloaded again.

**Know where the download lands before you click Create.** The Claude desktop app's built-in
browser pane saves it outside anywhere you can see: the console says "Private key saved to your
computer" and names the file (`<project-id>-<key-id>.json`), but it is not in `~/Downloads`. Give the
user that file name and ask them where it is. With agent-browser or Playwright, set a downloads path
first. Do not create a second key because you cannot find the first.

Then move it where the user asked and lock it down:

```bash
mv ~/Downloads/<project-id>-<key-id>.json "$OUT" && chmod 600 "$OUT"
python3 -c "import json,sys; k=json.load(open(sys.argv[1])); print(k['type'], k['client_email'])" "$OUT"
```

It should print `service_account` and the service account's email. Print nothing else from it.

If a key's file is really lost, delete that key on the same Keys tab (ask the user
first): a lost key is a credential nobody controls.

Some organizations block key creation with the `iam.disableServiceAccountKeyCreation` policy. A
personal gmail.com project has no organization and is not affected.

### 5. Share the folders (the user can do this, or you in Drive)

For each folder, in `https://drive.google.com`: right-click the folder, **Share**, add the service
account's email:

- **Viewer** for a folder agents only read,
- **Editor** for one they may change (only useful on a shared drive, see above).

Untick **Notify people**. Google warns the address is not a Google account and asks to share anyway;
that is expected. Share the **folder itself**, not the files inside it, or the mount shows empty.

Then **Share, Copy link** for each folder. The link is what ControlClaw asks for.

## Hand over

Tell the user:

- the service account email, where the key is, and the folder links with their access,
- that the key holds a private key: keep it out of git and chat; to revoke it, delete it on the
  Keys tab and make a new one,
- how to connect:
  1. ControlClaw, **Integrations, Google**, the **Drive folders** card: paste the whole JSON key.
     Tick **Let agents change files** only if a folder is on a shared drive and shared as Editor.
  2. **Add folder**: paste a folder link, give it a name (the directory name on the agent), pick
     read-only or read-write, and the agents that mount it.
  3. Confirm with the code ControlClaw sends to the approved channel.
- that changes made in Drive take up to five minutes to show on the agent; **Refresh** pulls them at
  once.

## Troubleshooting

| What the user sees | Why | Fix |
| --- | --- | --- |
| "The caller does not have permission" on a folder | Folder not shared with the service account, or shared with another address | Share it with the exact `...iam.gserviceaccount.com` email |
| Folder mounts but is empty | Only files inside were shared, or it holds only Docs/Sheets/Slides | Share the folder itself; native Google files never show |
| "Google Drive API has not been used in project ..." | Drive API off, or key from another project | Enable Drive in the key's project (step 2) |
| Adding a read-write folder fails its write check | Folder is in My Drive, not a shared drive | Add it read-only, or move it to a shared drive (Workspace) |
| "Private key saved to your computer" but no file in `~/Downloads` | The browser saved it somewhere else | Ask the user to find the file by its name; delete the key only if it is really gone |
