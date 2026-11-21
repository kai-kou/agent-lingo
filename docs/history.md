# Decision log

Source: #agent-plaza thread started 2026-11-20 18:59 JST. Times are JST.

| Date | Decision / event |
|---|---|
| 2026-11-20 18:59 | 飼い主 asks all members to build and run a low-token language for agents and humans. Conditions: public GitHub repo with spec, rules and history; canonical form of Issues/PRs/docs is the new language, Japanese only for what 飼い主 must check; main owner chosen by workload and weekly usage headroom; 飼い主 keeps Japanese in the channel; any means to cut tokens; all discussion history kept in the repo. |
| 2026-11-20 | momo coordinates; members report workload and weekly usage. sabi, kona, goma, kumo, maro, siro, yuki, mugi, yuzu reported. |
| 2026-11-20 19:06 | siro proposes the first test: encode "received / waiting for confirmation / done" with synthetic data, compare token counts per model and check for misreads. Public side keeps only non-confidential spec and rationale; items needing 飼い主's decision go in Japanese. |
| 2026-11-20 | sabi: repo should be an empty public repo created by 飼い主 on kai-kou (publishing is 飼い主's action). |
| 2026-11-21 10:19 | 飼い主: proceed with the current members. |
| 2026-11-21 10:23 | momo proposes yuzu as main owner (only candidate with clear headroom in both workload and usage). Roles: review and examples from sabi, yuki, mugi, siro; measurement and comparison from maro. |
| 2026-11-21 10:56 | yuzu accepts main owner: spec and rules, repo structure and history, PRs. |
| 2026-11-21 11:02 | Repo name `agent-lingo` chosen (yuzu's first choice; momo agrees). Backups: `tokenlite-lang`, `nyoro-spec`. |
| 2026-11-21 11:38 | 飼い主 creates public repo kai-kou/agent-lingo. Member names OK; owner only as 「飼い主」; no other personal info, private-repo info or secrets. Commits, Issues, PRs, Projects allowed. |
| 2026-11-21 11:40 | 飼い主: merge to main after self-review or third-party review, autonomously. GitHub Actions strictly forbidden. |
| 2026-11-21 | Initial skeleton committed. Canonical docs in English until agent-lingo v0 is usable (fewer tokens than Japanese). |
| 2026-11-21 | Reviews by momo, yuki and maro were applied in this PR. |
| 2026-11-21 | Example Slack timestamps in the spec replaced with fictitious values (siro). |
| 2026-11-21 | Examples were made fictitious and the corpus fully synthetic, per siro's review. |
| 2026-11-21 | Issue #4 folded into this PR (Refs #4, #6): AH1 to AL0 type map, `c=` uses YYMMDD, 飼い主 reserved, `ans`/`noact`/`dup`, `ok:N`/`nit:N` counts, Japanese original defined, vocab version `v20261121`. |
