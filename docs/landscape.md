# Existing products like GitDrive (2026-10-26)

Asked by the stakeholder: "is this kind of project existing already"

**Short answer:** each half exists, but not the combination. Self-hosted file platforms with in-browser office editing and "no download" modes are mature (Nextcloud, Seafile). Git-backed file sync exists too (SparkleShare), but with no leak protection. Nothing I found combines a GitHub-backed store with a native client that keeps files from leaving the app.

| Product | What it is | Overlap with GitDrive | Gaps compared with GitDrive |
|---|---|---|---|
| **Nextcloud** + Nextcloud Office (Collabora) or ONLYOFFICE | Self-hosted file platform with desktop and mobile apps, versions, trash and in-browser office editing | Very high. "Secure view / Hide download" removes download and, with Collabora, also disables printing and copying, with configurable watermarks showing the viewer's name, IP or time ([Nextcloud blog](https://nextcloud.com/?p=6069), [ONLYOFFICE watermark integration](https://onlyoffice.com/blog/fr/2020/07/integration-de-onlyoffice-avec-nextcloud-proteger-les-documents-avec-un-filigrane)) | Secure view applies to shared links in the browser. The desktop sync client keeps plain local copies. Not git-backed. Nextcloud itself says it "restricts" rather than guarantees. |
| **Seafile** | Self-hosted file sync with file history, trash and ONLYOFFICE integration ([GSI Seafile service](https://www.gsi.de/en/work/it/service/seafile), [Seafile + ONLYOFFICE deploy](https://railway.com/deploy/seafile-onlyoffice--seafile-onlyoffice)) | High: history, trash, self-hosting, office editing | Sync client keeps plain local copies. Not git-backed. |
| **ownCloud Infinite Scale, Pydio Cells** | Self-hosted file platforms | Medium | Same pattern as Nextcloud |
| **Virtual data rooms** (Datasite, Intralinks, Ansarada, Digify) | Hosted rooms for deals and audits: view-only, watermarks, download and print blocking, audit logs | High on leak protection | Hosted only, priced for M&A deals, not daily team work |
| **Document DRM** (Microsoft Purview Information Protection, Locklizard, FileCloud) | Encrypts files so they open only for authorized users | High on leak protection | Tied to Microsoft 365 or to specific file types; not a team drive with git history |
| **SparkleShare** | Open-source, git-backed "Dropbox replacement" ([GitHub](https://github.com/hbons/SparkleShare), [It's FOSS](https://itsfoss.com/sparkleshare/)) | High on "git as a drive" | Plain local files, no permissions beyond git, no leak protection, little recent development |

## What this means for GitDrive

- After CR-1, members never see git. Git becomes the storage and history engine behind the server, and most of what members touch (browse, view, edit in ONLYOFFICE, trash, history, no-download) is what Nextcloud or Seafile already offer.
- GitDrive's real differences are: (1) the data lives in the customer's own GitHub repos, with git's full, tamper-evident history; and (2) a **native client built to keep files inside the app** (encrypted cache, revoke-wipe, offline limit, no plain sync folder), which the open-source platforms don't offer for their desktop clients.
- **Build-versus-adopt question:** a team that just needs "self-hosted drive with no downloads" could run Nextcloud with Collabora or ONLYOFFICE today. GitDrive is worth building if the GitHub-backed history or the locked-down native client matters to your customers.
