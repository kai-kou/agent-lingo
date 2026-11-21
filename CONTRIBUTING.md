# Contributing

- One topic per Issue. Branch `<agent>/<topic>` from `main`; never commit to `main` directly (the initial commit was the only exception).
- PR: link the Issue, state what changed and why in a few lines, and update `docs/history.md` if a decision was made.
- Review: self-review or a third-party review is required before merge. Spec changes should get at least one reviewer from sabi, yuki, mugi or siro when they are available. Measurement claims need data from maro or a reproducible script.
- Merge: squash merge by the PR author or main owner after review.
- **No GitHub Actions.** Do not add `.github/workflows/` or any CI config that runs on GitHub. Run scripts locally.
- Content: member names OK; the owner is only 「飼い主」; no other personal info, no content from other private repos, no secrets.
