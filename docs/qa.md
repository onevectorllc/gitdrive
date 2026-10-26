# Q&A log

## Q1 · 2026-10-26 · Stage 1 (revised in Q6)
**Question:** "does this neccessary in this project scope? should I have a seperate project to work on server backup?"
**Answer:** Server backup is not needed for v1 and is out of the v1 plan. Keep it in this project as later work, because it reuses the core engine and the repo layout; a separate project only makes sense if it becomes its own product and codebase.

## Q2 · 2026-10-26 · Stage 1 (revised in Q16)
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

## Q13 · 2026-10-26 · Stage 2 (revised in Q16)
**Question:** "what if member access the repo on github ?"
**Answer:** That's allowed: edits and uploads on GitHub are normal commits and sync into GitDrive, and deletes still show in Trash. Admin-only restore can be bypassed (accepted risk). The real danger is a force-push, so GitDrive checks that the default branch blocks force-pushes and deletion and warns admins if not, and pauses syncing without touching local files if history is rewritten anyway (new AC-16, design S-4 and S-5).

## Q14 · 2026-10-26 · Stage 2
**Question:** "explore  other edge cases that can compromise gitdrive"
**Answer:** Listed 37 edge cases in edge-cases.md, grouped into untrusted content, GitHub refusals, app save behaviour, the user's Mac, people and access, and supply chain. Most are handled in v1 through new AC-17 to AC-19 and design S-16. Three need sign-off as accepted risks (E-30 to E-32), plus a 60-second quiet window for office files.

## Q15 · 2026-10-26 · Stage 2
**Question:** "it is not acceptable for member to keep local copy"
**Answer:** E-30 is no longer an accepted risk. No app can fully guarantee removal (a member can copy files while they still have access, or clone with plain git), but GitDrive can remove its own copy automatically. Options put to the stakeholder: auto-wipe on revoke plus a 14-day offline limit; the same plus no local history (recommended); or a GitDrive server that brokers all GitHub access.

## Q16 · 2026-10-26 · Stage 2
**Question:** "i won't accept the case where member has direct access to git repo as well"
**Answer:** Members will reach drives only through a GitDrive server that alone holds GitHub access. This is change request CR-1 (docs/cr/CR-1.md): a git proxy server, GitDrive accounts, current-files-only clones, revoke-wipe and an offline limit. Revises Q2 and Q13; supersedes D-1 and D-5 once approved.

## Q17 · 2026-10-26 · Stage 2
**Question:** "we should just allow app open thru gitdrive to prevent file leaking"
**Answer:** Possible, but editing is the hard part: any file handed to Word or Excel can be saved elsewhere. Options put to the stakeholder: A, encrypted storage with in-app preview, editing via temporary copies (reduces but doesn't prevent leaks); B, view and edit only inside GitDrive with an embedded office editor hosted on the GitDrive server, with copy, print, download and screenshots blocked (recommended; replaces AC-3 and limits offline editing); C, keep the plain folder and rely on MDM. Folds into CR-1.

## Q18 · 2026-10-26 · Stage 2
**Question:** "* member sign-in by email code * Offline limit: 14 days, view-only -? ues * are we making mac app or web app? * admin can export"
**Answer:** Recorded: members sign in with an email code; the offline limit is 14 days, view-only; admins can export (members can't). Members get a native Mac app, because the leak protections (screenshot exclusion, Keychain-protected encrypted cache, locked-down print and save) need native controls a browser can't give. Admins get a web console. A member web app could come later with weaker protection.

## Q19 · 2026-10-26 · Stage 2
**Question:** "* we do self-host"
**Answer:** The GitDrive server is self-hosted, packaged as a Docker Compose bundle (GitDrive server, office editor, Postgres, TLS reverse proxy) for one Linux machine, with SMTP configured for email sign-in codes. The same bundle works for hosting your own team or for customers hosting their own.

## Q20 · 2026-10-26 · Stage 2
**Question:** "what else can compromise gitdrive?"
**Answer:** Added E-38 to E-51 for the CR-1 design: server compromise, email and admin account takeover, screen recording (window exclusion blocks screenshots but isn't reliable against recording, so add visible watermarks), Accessibility scraping, editor leaks, clock rollback, editor vulnerabilities, self-hosting misconfiguration, malicious uploads, GitHub org owners' direct access, unattended Macs and server outages. Key proposals: admin approval for new devices, passkeys for admins, and watermarks.

## Q21 · 2026-10-26 · Stage 2
**Question:** "is this kind of project existing already"
**Answer:** Each half exists but not the combination (docs/landscape.md). Nextcloud and Seafile already offer self-hosted drives with ONLYOFFICE or Collabora, history, trash and no-download modes with watermarks, but their desktop clients keep plain local copies and they aren't git-backed. SparkleShare is git-backed but has no leak protection. GitDrive's differences are GitHub-backed history and a locked-down native client.

## Q22 · 2026-10-26 · Stage 2
**Question:** "is it meaningful to work on this project?"
**Answer:** It depends on the audience. For your own team: no, since Nextcloud or Seafile with ONLYOFFICE, no-download mode, watermarks and managed Macs covers most needs in days. As a product: there's a niche (regulated small teams wanting a self-hosted, leak-resistant drive), but after CR-1 git is hidden, the locked-down Mac app is the differentiator, and scope is large. Recommended next step: a one-week validation (pilot Nextcloud + ONLYOFFICE, talk to 3 to 5 prospective buyers) before writing code; a smaller path is a locked-down Mac client on top of Nextcloud. Asked who GitDrive is for.
