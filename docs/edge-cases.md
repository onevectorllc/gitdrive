# GitDrive edge cases (Stage 2)

Asked by the stakeholder: "explore other edge cases that can compromise gitdrive". Date: 2026-10-26.

Each case lists what goes wrong, how bad it is, and what v1 does. **Severity:** High means data loss, a security hole or a stuck drive; Medium means a confusing or degraded result; Low means cosmetic or rare. **Plan:** *v1* is handled in v1 (covered by the new AC-17 to AC-19), *Risk* is an accepted risk that needs sign-off, and *Later* goes to the backlog.

## 1. Untrusted content from teammates (AC-17)

Anything in the repo was written by someone else, possibly from a compromised account.

| # | Case | Severity | v1 handling | Plan |
|---|---|---|---|---|
| E-1 | **Symlink in the repo** pointing outside the drive (for example to `~/.ssh` or `~/Library/LaunchAgents`), so a later write lands outside the drive | High | Symlinks are checked out as small text files, never as real links. Writes are refused for any path that resolves outside the drive folder. | v1 |
| E-2 | **Path tricks**: `..` segments, absolute paths, names like `.git` or `.GIT` | High | Rejected on checkout and on commit. That file shows as "can't sync" with the reason. | v1 |
| E-3 | **Case clash**: `Budget.xlsx` and `budget.xlsx` both exist on GitHub. macOS folders ignore case, so one silently overwrites the other (a known class of git bugs). | High | Detected before checkout. The second file is saved as `budget (case conflict).xlsx`, and the user is told. Case-only renames made on the Mac are committed as real renames. | v1 |
| E-4 | **Unicode lookalike names**: the same accented name stored two ways (NFC vs NFD, common between Mac and Windows) shows as two files or keeps re-syncing | Medium | Names are normalized to NFC when committing and compared after normalization. | v1 |
| E-5 | **A teammate uploads malware**, for example a `.app` or a script. Files GitDrive downloads skip macOS Gatekeeper, because only browsers and similar apps mark files as downloaded. | High | Every file GitDrive receives from GitHub gets the macOS quarantine flag, so Gatekeeper checks it the first time it's opened. The executable bit is never set on files from GitHub. | v1 |
| E-6 | **Disguised file names** using right-to-left override characters (`invoice‮fdp.app` displays as `invoiceppa.pdf`) | Medium | Control and bidirectional characters are shown escaped, and a warning badge appears. | v1 |
| E-7 | **Repo config that runs code**: `.gitattributes` filters, `.gitmodules` submodules, hooks | High | GitDrive never runs filters, hooks or submodule fetches. Hooks aren't cloned by git anyway. Submodules show as an empty "unsupported" folder. | v1 |

## 2. GitHub refusing or changing things (AC-18)

| # | Case | Severity | v1 handling | Plan |
|---|---|---|---|---|
| E-8 | **Secret scanning push protection** blocks a push because someone dropped a file containing a password or API key. Without handling, the drive retries forever and nothing else syncs. | High | The engine recognizes the rejection, names the file, and offers "Remove from sync". This rewrites only the local commits that haven't been pushed yet, then the rest syncs. | v1 |
| E-9 | **Push too big**: someone drags in 3 GB of photos. GitHub rejects a single push over 2 GB. | High | Large batches are split into several commits and pushes of at most about 500 MB each. Progress shows in S-07. | v1 |
| E-10 | **Repo archived, deleted, renamed or transferred**, or the GitHub App uninstalled | Medium | Renames and transfers follow GitHub's redirect and update the drive. Archived becomes read-only (S-08). Deleted or uninstalled pauses the drive and keeps local files. | v1 |
| E-11 | **Organization requires SAML SSO** and the token isn't authorized for it | Medium | The SSO error is detected, and the app shows "Authorize GitDrive for your organization" with GitHub's link. | v1 |
| E-12 | **Default branch renamed** (for example `master` to `main`) | Low | The default branch is re-read on every fetch and tracked automatically. | v1 |
| E-13 | **API rate limits** at 50 users | Medium | Git fetches don't count against the REST limit. REST calls (permissions, branch rules) use ETags and run every 10 minutes or less often. The app backs off when GitHub asks it to. | v1 |
| E-14 | **GitHub outage** | Low | Same as offline: changes queue (AC-8). | v1 |

## 3. How apps save files (AC-19)

| # | Case | Severity | v1 handling | Plan |
|---|---|---|---|---|
| E-15 | **Autosave storms**: Word, Excel and Pages autosave every few seconds while someone types, so a 5-second quiet window makes dozens of commits. Each one adds a full copy of the binary file and bloats the repo. | High | Office and iWork files use a longer quiet window (60 s after the last write). Unpushed local commits for the same file are squashed into one before pushing. | v1 |
| E-16 | **Safe-save dance**: apps write a temp file, delete the original, then rename. This looks like delete plus add, which would break history and send the file to Trash. | High | Changes are grouped within the quiet window, and a delete followed by a re-create of the same path counts as one modify. Temp and lock files (`~$*`, `.~lock*`, `*.sb-*`, `*.tmp`) are ignored. | v1 |
| E-17 | **Half-written files**: a large photo is still copying when the quiet window ends | Medium | A file is committed only once its size and modified time have been stable across two checks. | v1 |
| E-18 | **Sync loop**: GitDrive writes a file it pulled, FSEvents reports the change, and GitDrive commits it back | High | The engine records its own writes (path and content hash) and ignores the matching events. | v1 |
| E-19 | **Delete versus edit**: one person removes a file while another edits it | High | The edit wins. The file comes back with the edit, and the remover is notified. | v1 |
| E-20 | **Same new name**: two people add `Report.docx` in the same folder at the same time | Medium | Treated like a same-file conflict: one becomes a conflicted copy (AC-7). | v1 |
| E-21 | **File versus folder**: one person creates file `Plans` and another creates folder `Plans/` | Low | The file is renamed `Plans (conflicted copy).ext` and the user is told. | v1 |
| E-22 | **Files hidden by the repo's `.gitignore`**: the user expects a file to sync but it never does | Medium | The browser shows a "Not synced: ignored by drive rules" badge. | v1 |

## 4. The user's Mac

| # | Case | Severity | v1 handling | Plan |
|---|---|---|---|---|
| E-23 | **Drive folder inside iCloud Drive, Dropbox or OneDrive** (for example the user moves it to Documents with iCloud Desktop and Documents on). Two sync tools fight over the same files. | High | On launch and when the folder moves, the app checks its location and refuses to sync from a cloud-synced folder, explaining why. | v1 |
| E-24 | **Disk full** during a clone or pull | Medium | Free space is checked before a pull. Files are written to a temp file and renamed, so a failure leaves the old version intact. | v1 |
| E-25 | **Mac sleeps or the app quits mid-push** | Low | Pushes either land completely or not at all on GitHub. The queue resumes on next launch. | v1 |
| E-26 | **Local git data corrupted** (disk error, an old Time Machine restore of the app's data) | Medium | An integrity check runs on launch. If it fails, the engine re-clones and then re-applies local file changes as new edits. | v1 |
| E-27 | **FSEvents drops events** during a burst of 100,000 files | Medium | On an overflow flag, the engine falls back to a full folder scan. | v1 |
| E-28 | **Very long names**: over 255 bytes, or deep paths | Low | Shown as "can't sync" with the reason. | v1 |
| E-29 | **Mac clock wrong**: commit times are off | Low | History is ordered by commit order, not by time. Times far in the future are flagged. | v1 |

## 5. People and access

| # | Case | Severity | v1 handling | Plan |
|---|---|---|---|---|
| E-30 | **A removed member keeps a full copy**: when someone leaves the team, their Mac still has every file and all of history. GitDrive can't remotely wipe it. | High | Once access is gone, GitDrive stops syncing and offers to delete the local copy. Nothing can force it. | **Risk** |
| E-31 | **Stolen or compromised Mac** with a signed-in GitDrive | High | Access tokens expire after 8 hours. An admin revokes the user on GitHub, which also kills the refresh token. The local copy is protected only by the Mac's own login and FileVault. | **Risk** |
| E-32 | **Faked authors**: anyone using plain git can put any name on a commit, so history could show "Alex" for a change Alex didn't make | Medium | History shows GitHub's "Verified" badge where a commit is signed. Signing GitDrive's own commits automatically needs more GitHub permissions and work. | **Risk** in v1, signing **Later** |
| E-33 | **Permission lowered with unsaved work**: an editor becomes a viewer while they have unpushed changes | Medium | Local commits are kept and the drive turns read-only. The user can export their changes as files to send to an admin. | v1 |
| E-34 | **Sensitive file added by mistake**: removing it only moves it to Trash, and it stays in history on every member's Mac | High | Already an accepted risk (security.md). The admin docs will explain how to purge it with GitHub's tools. | **Risk** (existing) |

## 6. Distribution and supply chain

| # | Case | Severity | v1 handling | Plan |
|---|---|---|---|---|
| E-35 | **A fake update** pushed to users | High | Sparkle verifies an EdDSA signature on every update, and the app is Developer ID signed and notarized. | v1 |
| E-36 | **A compromised dependency** in Rust or Swift | Medium | Versions are pinned, and `cargo audit` and `cargo deny` run in CI. Dependencies are kept to a minimum. | v1 |
| E-37 | **Corporate proxy that intercepts TLS** | Low | The macOS system trust store is used, so company root certificates work, and certificate checks are never turned off. | v1 |

## New acceptance criteria

- **AC-17 Untrusted content is safe.** Symlinks, path tricks, case clashes, Unicode lookalikes, disguised names and repo config can't write outside the drive, run code or silently overwrite a file. Files received from GitHub carry the macOS quarantine flag. (E-1 to E-7)
- **AC-18 GitHub refusals never jam a drive.** Push protection, oversized pushes, archived or deleted repos, SSO, an uninstalled app and rate limits each produce a clear message and a way forward, with no data loss, and other files keep syncing where possible. (E-8 to E-14)
- **AC-19 One edit is one clean change.** Autosaves, safe-saves, partial writes and GitDrive's own writes produce one commit per editing session, with no false deletes and no sync loops. (E-15 to E-18)

## Needs a decision from the stakeholder

1. Accept E-30 (removed members keep their copy), E-31 (stolen Mac until revoked) and E-32 (authors can be faked by plain-git users) as v1 risks?
2. Is a 60-second quiet window for office files (E-15) acceptable? It means a teammate sees an edit about a minute after the person stops typing.
