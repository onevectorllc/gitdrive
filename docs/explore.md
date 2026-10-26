# GitDrive: Stage 1 (Explore)

Status: **draft, waiting for Explore gate approval**
Date: 2026-10-26 · Size: **Large** (new system, all four stages)

## 1. Requirement (stakeholder's words)

> "Regular usecase: gitdrive will start with a mac app. We can add/remove/delete/browse/history/open files (later we can use iOS, ipad, or web)"

> "Advanced usecase: backup" … "backup mode: I mean a server can connect to it to upload backup files"

> "it also available on Android later"

> "can we have more than one user to access the drive?"

> "I suppose this can be used in middle team size"

**Restated:** GitDrive turns a private git repo into a shared Drive. Version 1 is a Mac app where a team of about 10 to 50 people can browse, open, add, remove and delete files, and see and restore each file's history. Every save becomes a git commit. iOS, iPad, Android, web and server backup come later, and v1 must not block them.

## 2. Scope

**In v1:** Mac app, GitHub as the only host, multi-user access through GitHub, conflict handling, large files, offline queue.

**Later (designed for, not built):** iOS and iPad, Android, web, server backup uploads, other git hosts, per-folder permissions.

## 3. Options and recommendation

### 3.1 Where the drive lives

| Option | Pros | Cons |
|---|---|---|
| **A. GitHub repo (recommended)** | No server to run; accounts, teams, permissions and audit come free; private repos are cheap | Repo size soft limit (a few GB); LFS storage is billed; permissions are per repo, not per folder |
| B. Self-hosted git (Gitea, GitLab) | Full control, no LFS billing | Someone has to run and secure a server |
| C. Our own GitDrive server | Per-folder permissions, real-time sync | Much bigger build, we own uptime and security |

Recommend **A** for v1, with the engine talking to "a git remote over HTTPS" so B can be added later without a rewrite.

### 3.2 Core engine (shared across platforms)

| Option | Pros | Cons |
|---|---|---|
| **A. Rust + libgit2 (git2-rs), bindings via UniFFI (recommended)** | One engine for Mac, iOS, Android; libgit2 is mature and handles clone, fetch, push over HTTPS | No built-in Git LFS or partial clone, so we write an LFS client |
| B. Rust + gitoxide | Pure Rust, fast, active | Push support still less mature than libgit2 |
| C. Swift + SwiftGit2 | Simplest for Mac | Apple only, Android needs a second engine |
| D. Shell out to the `git` CLI | Full git features | Not available on iOS or Android, sandboxing issues |

Recommend **A**, keeping the git layer behind our own interface so we can swap in gitoxide later.

### 3.3 Mac app shape

| Option | Pros | Cons |
|---|---|---|
| **A. SwiftUI window first, then Finder integration (recommended)** | Fast to ship and test; Finder integration reuses the same engine | Two UI surfaces to maintain |
| B. Finder integration (File Provider) only | Feels most like Google Drive | Harder to build and debug; history and conflict UI still need a window |

### 3.4 Sync model

Full local clone of each drive, kept in the app's container. Saves wait about 5 seconds, then go out as one commit. On push rejection, the engine fetches, replays local commits on top and retries. Same-file conflicts keep the remote version as the main file and save the local one as `<name> (<user>'s conflicted copy <date>).<ext>`. Offline changes queue and sync on reconnect. Optional per-file locks use the Git LFS locking API.

Mobile apps will later need on-demand downloads rather than full clones. That is out of v1 but noted as a risk below.

### 3.5 Sign-in and permissions

A GitHub App with the device-flow sign-in, so users never paste tokens. Tokens are stored only in the macOS Keychain. Access is managed through a GitHub organization and teams; read-only collaborators get a read-only drive.

## 4. Acceptance criteria (v1)

- **AC-1** A user signs in with GitHub and picks any private repo they can access as a drive.
- **AC-2** The app lists folders and files; after the first clone, a 10,000-file drive opens a folder in under 2 seconds.
- **AC-3** Opening a file uses its default macOS app. A saved change is committed and pushed within 15 seconds when online.
- **AC-4** Adding (including drag and drop), renaming and deleting files or folders each create a commit attributed to that user.
- **AC-5** A history panel shows each file's versions (author, time, message), and any version can be restored as a new commit.
- **AC-6** Two users saving different files at the same time both succeed, with no error shown to either.
- **AC-7** Two users saving the same file at the same time both keep their work: one version stays as the file, the other is saved as a conflicted copy, and the user is notified.
- **AC-8** Changes made offline are queued and synced on reconnect without data loss.
- **AC-9** Binary file types (images, PDF, office files, media, archives) and any file over 10 MB are stored through Git LFS, transparently to the user.
- **AC-10** A user with read-only access can browse and open but not change files; the UI disables edits and the engine refuses them.
- **AC-11** A user can lock a file; other users see it as read-only until it is unlocked.
- **AC-12** The core engine has no Apple-only dependency and compiles for an Android target in CI.
- **AC-13** Credentials are stored only in the Keychain, never in plain files or logs.
- **AC-14** The drive appears in the Finder sidebar and supports browse, open, add and delete there (File Provider). *Could be moved to v1.1.*

## 5. Risks

- **Git LFS in Rust:** libgit2 has no LFS support, so we must write or adopt an LFS client. Needs a spike.
- **Repo growth:** GitHub repos slow past a few GB. Mitigation: LFS for big files, a size budget and guidance to split drives.
- **Mobile storage:** full clones won't suit phones. Later mobile apps may need on-demand fetching via the GitHub API.
- **Push contention:** a busy team means frequent rejected pushes. Mitigated by batching and automatic retry; to be load-tested with simulated users.
- **App Store sandbox:** the distribution choice affects file access and Finder integration.

## 6. Suggested spikes (optional, before the gate)

1. **LFS upload and download** from Rust against a GitHub repo.
2. **Rust engine to Swift** via UniFFI, called from a small SwiftUI app.
3. **File Provider extension** backed by a local clone.

## 7. Open questions for the stakeholder

1. Is GitHub the only host for v1? *(open)*
2. ~~What should the code repo be called?~~ **Answered:** "repo is created already" → `onevectorllc/gitdrive`.
3. ~~Distribution?~~ **Answered:** direct download, "this is first step". The app ships signed and notarized outside the Mac App Store; App Store can follow later.
4. Should Finder integration (AC-14) be in v1 or v1.1? *(open)*
5. ~~What kinds of files?~~ **Answered:** "usually docs, photos, regular files used by office person". See 7.1.

### 7.1 What the answers change

- **Large files (AC-9):** office docs and photos are binary and can't be merged, and photos add up fast. Instead of a single 50 MB cutoff, store common binary types (images, PDF, video, audio, archives) in LFS always, plus anything over 10 MB. Office files (docx, xlsx, pptx) also go to LFS. This keeps the git history small.
- **Locking (AC-11):** since office files can't be merged, locking matters more. The app should offer "lock while editing" when an office file is opened.
- **Direct download:** no App Store sandbox limits for v1; needs a Developer ID certificate, notarization and an auto-update channel (Sparkle).
