# Q&A log

## Q1 · 2026-10-26 · Stage 1 (revised in Q6)
**Question:** "does this neccessary in this project scope? should I have a seperate project to work on server backup?"
**Answer:** Server backup is not needed for v1 and is out of the v1 plan. Keep it in this project as later work, because it reuses the core engine and the repo layout; a separate project only makes sense if it becomes its own product and codebase.

## Q2 · 2026-10-26 · Stage 1
**Question:** "can we have more than one user to access the drive?"
**Answer:** Yes. Each user signs in with their own GitHub account and is added to the repo with read or write access; git records who changed what. Same-file conflicts are saved as a "conflicted copy". GitHub grants access per repo, not per folder, so restricted areas mean one repo per shared space.

## Q3 · 2026-10-26 · Stage 1
**Question:** "how to handle Simultaneous saves ?"
**Answer:** If different files changed, the engine fetches, replays the local commit on top and pushes again, retrying automatically. If the same file changed, the remote version stays and the local one is saved as a conflicted copy, and the user is notified. Saves are batched over a few seconds and queued offline, and files that can't be merged can be locked via Git LFS locks.

## Q4 · 2026-10-26 · Stage 1
**Question:** (comment on open question 5) "usually docs, photos, regular files used by office person"
**Answer:** Office files and photos are binary and can't be merged, so they go to Git LFS by type, plus any file over 10 MB (AC-9 revised from a 50 MB cutoff). Locking matters more, so the app offers "lock while editing" for office files.

## Q5 · 2026-10-26 · Stage 1
**Question:** (comment on open question 4) "yes, if this is possible, keep this for v1.1"
**Answer:** Yes, it's possible through a macOS File Provider extension, the same mechanism Google Drive and Dropbox use. AC-14 moves to v1.1; v1 ships the SwiftUI window, and the engine is designed so Finder integration can sit on top of it.

## Q6 · 2026-10-26 · Stage 1
**Question:** (comment on the backup requirement) "not in scope"
**Answer:** Server backup is now out of scope for GitDrive, not just deferred. Removed from the "later" list; revises Q1.

## Q7 · 2026-10-26 · Stage 1
**Question:** (comment on "remove" in the requirement) "admin can restore"
**Answer:** Removing a file is a soft delete: it goes to a Trash view and a repo admin can restore it with its full history, as a new commit (AC-15). Git keeps every version, so this is cheap. Open: whether "delete" is a permanent purge, and whether non-admins can restore their own files.
