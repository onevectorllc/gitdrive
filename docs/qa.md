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

## Q8 · 2026-10-26 · Stage 1
**Question:** (comment on "LFS") "what is LFS ?"
**Answer:** Git Large File Storage, a git add-on GitHub supports. The repo keeps only small pointer files, while the real content of large files lives in separate GitHub storage and is downloaded only when opened, so the repo stays small and fast. Costs: GitHub bills LFS storage and bandwidth beyond a free allowance, and libgit2 has no LFS support, so GitDrive needs its own LFS client (the main risk).

## Q9 · 2026-10-26 · Stage 1
**Question:** (comment on "LFS") "keep LFS in backlog for v1.1"
**Answer:** LFS (AC-9) moves to v1.1, and file locking (AC-11) moves with it because it uses the LFS lock API. v1 stores files in plain git with a size cap (new AC-9a: warn above 25 MB, refuse over 50 MB). The LFS spike is deferred.

## Q10 · 2026-10-26 · Stage 1
**Question:** (comment on "GitHub repo" in option 3.1) "github repo"
**Answer:** Read as choosing option 3.1 A and answering open question 1: v1 supports GitHub repos only. The engine still talks to a generic HTTPS git remote so other hosts can be added later.

## Q11 · 2026-10-26 · Stage 2
**Question:** (comment on "users restore their own removed files") "only admin can restore files"
**Answer:** Only repo admins (GitHub admin or maintain) can restore from Trash, including a user's own removed files. Editors and viewers see the Trash without a Restore button. Updated design S-7 and S-8 and D-8.

## Q12 · 2026-10-26 · Stage 2
**Question:** (reply to the three open items: delete goes to Trash, macOS 14 minimum, accepted security risks) "ok"
**Answer:** Recorded all three: delete goes to Trash like remove, minimum macOS is 14, and the three risks in security.md are accepted.
