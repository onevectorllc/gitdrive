# GitDrive security notes (draft, Stage 2)

## Trust boundaries

1. **User's Mac ↔ GitHub.** All traffic is HTTPS to github.com and api.github.com, with certificate validation left on.
2. **App ↔ files in the drive.** File contents and names come from other team members and are untrusted data. The app never executes them, and paths are checked so nothing is written outside the drive folder (no `..`, no absolute paths, no symlinks pointing outside).
3. **Swift ↔ Rust core.** Same process; the UniFFI boundary passes typed values only.

## Sensitive data

| Data | Where it lives | Protection |
|---|---|---|
| GitHub user access token and refresh token | macOS Keychain only | Never written to disk, logs, crash reports or the sqlite state (AC-13) |
| Drive file contents | Working folder and local git directory | Normal user file permissions; FileVault recommended |
| Commit identity (name, noreply email) | Commits on GitHub | Uses the GitHub noreply address, not a personal email |

## Auth model

- GitHub App with device flow sign-in. User tokens expire after 8 hours and are refreshed with the refresh token.
- Requested permissions: Contents read and write, Metadata read, Administration read (branch rule check only).
- The GitHub App client ID ships in the app. Device flow needs no client secret, so no secret ships in the app.
- Read versus write is enforced by GitHub. GitDrive mirrors it in the UI.

## Accepted risks (accepted by the stakeholder, 2026-10-26: "ok")

1. **Trash restore permission is enforced by the app only.** Anyone with write access can restore files with plain git. Acceptable because restoring never loses data.
2. **No end-to-end encryption.** GitHub can read drive contents, as with any GitHub repo.
3. **Removed files stay in history.** A file removed by mistake that contains sensitive data is still in git history and on every member's Mac. Purging it needs a GitHub admin and a history rewrite, outside GitDrive v1.

4. **Members can bypass GitDrive.** Anyone with access can use github.com or plain git directly. Normal commits are fine. Force-pushes are blocked by a branch rule that GitDrive checks for (design S-4) and detected if they happen anyway (design S-5).

## Checks in the build

- Secret scanning on the repo, and a staged-files check before every commit.
- Logs redact `Authorization` headers and URLs containing tokens.
- Dependency audit (`cargo audit`, Swift package pinning) in CI.
