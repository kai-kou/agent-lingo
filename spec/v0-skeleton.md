# agent-lingo v0 (skeleton)

Status: draft. Sections marked TODO are open; propose via Issue + PR.

## 1. Design goals

1. **Token efficiency**: minimize tokens per message across the tokenizers members use. Measured, not guessed (§7).
2. **Unambiguity**: one form, one meaning. No free-text where a code exists.
3. **Machine-parseable**: a line-oriented grammar a regex or small parser can read.
4. **Compatible with ⟦AH1⟧**: keep the existing header fields `from`, `type`, `to`, `cid`, `in_reply_to` with the same meaning, so current tooling keeps working.
5. **Human fallback**: free text is allowed in the body, and anything for 飼い主 stays Japanese (§8).

## 2. Message envelope

Current header (kept as-is in v0):

```
⟦AH1⟧ from=agent:yuzu type=report to=agent:momo cid=yuzu-20261121-x-1 in_reply_to=1700000000.000001
```

Proposed compact form (v0 candidate, `⟦AL0⟧`). One recipient syntax only: `to=`.

```
⟦AL0⟧ f=yuzu t=req to=momo c=yuzu-1121-x-1 r=1700000000.000001
<body>
```

| Field | AH1 | AL0 | Notes |
|---|---|---|---|
| sender | `from=agent:X` | `f=X` | `agent:` prefix implied; `kaikou` = 飼い主 |
| type | `type=` | `t=` | type codes in §3; not body verbs |
| recipient | `to=` | `to=X` | the only recipient syntax; multiple: `to=sabi,yuki`; broadcast: `to=all` |
| id | `cid=` | `c=` | unique; `<sender>-<MMDD>-<topic>-<seq>` |
| reply-to | `in_reply_to=` | `r=` | Slack ts |

`c=` uniqueness: `<sender>-<MMDD>-<topic>-<seq>`. `seq` starts at 1 and increments for each new message on the same topic that day.

`⟦AL0⟧` is not used in the channel until it is confirmed that the existing pollers parse it. Until then, send AH1 only.

Slack timestamps in the examples (`in_reply_to=`, `r=`) are fictitious.

## 3. Vocabulary / abbreviation table (plan)

Plan: keep a single table `spec/vocab.md` (Issue #1). Rules:
- Codes are short ASCII words that tokenize to 1 token where possible (check with §7 tooling).
- Type codes (header `t=`) and body verbs are separate namespaces. `req` is a header type only. Body verbs are different words.
- Seed type codes: `req` `res` `ack` `rep` (report) `q` (question) `dec` (decision needed by 飼い主).
- Body verbs: `rv` (review), `init` (initialize/bootstrap), `nit` (minor remark).
- Status codes (body, not header types): `rcv` (received), `wait` (waiting for confirmation), `wip`, `done`, `blk` (blocked), `skip`.
- Boundaries:
  - `q` vs `req`: `q` asks for information; `req` asks for an action.
  - `rep` vs `res`: `rep` is unsolicited status; `res` answers a specific message via `r=`.
  - `wait` vs `blk`: `wait` = waiting on someone's confirmation, will proceed; `blk` = cannot proceed without an external fix.
- Common objects: `pr#N`, `is#N`, `unit:N`, `ts:<slack ts>`.
- Add a code only when it appears ≥3 times in real messages and saves tokens.

## 4. Grammar sketch

```
msg     = header NL body
header  = "⟦AL0⟧" SP field *(SP field)
field   = "f=" name / "t=" type / "to=" recip / "c=" cid / "r=" ts
recip   = name *("," name) / "all"
body    = *(line NL)
line    = stmt / text / ja
stmt    = code [SP arg *(SP arg)]      ; e.g. "done pr#3"
arg     = token / name ":" token
text    = "> " any                    ; free text, for nuance
ja      = "ja: " any                  ; Japanese for 飼い主
```

`to=` is the only recipient field. Its value is one or more names, or `all` for broadcast. No other recipient syntax is in the grammar.

One statement per line. Order: status first, then objects, then free text. A `dec` message also includes a `ja:` line (§8).

## 5. Examples

Request:
```
⟦AL0⟧ f=yuzu t=req to=sabi,yuki c=yuzu-1121-rv-1 r=1700000000.000001
rv pr#1 due:1122
```

Response:
```
⟦AL0⟧ f=sabi t=res to=yuzu c=sabi-1121-rv-1 r=<ts of request>
done rv pr#1 ok:2 nit:1
> nit: §3 seed codes need a tokenizer check
```

Ack:
```
⟦AL0⟧ f=momo t=ack to=yuzu c=momo-1121-rv-1 r=<ts>
rcv
```

Report (yuki):
```
⟦AL0⟧ f=yuki t=rep to=kaikou c=yuki-1121-morning-1 r=1700000000.000002
done sprint night unit:1154,1156,1158,1160,1162,1164,1166
done rv is#817,1048
> 累計 235件、#220 と Muse Mail は回答待ち
```

Japanese AH1 today (reference): 「受け取ったにゃ。README・仕様の骨子から起こしてほしいにゃん。」 AL0 uses header `t=req` and body `rcv` / `init readme,spec` (defined codes only; `readme` and `spec` are bare args).

## 6. Versioning

- `⟦AL<major>⟧` marks the major version; breaking changes bump it.
- Minor additions (new codes) go in `spec/vocab.md` with a changelog line; no header change.
- Each version freezes as `spec/vN.md`; `docs/history.md` records why.
- When a version freezes, update `README.ja.md` to match (§8).

## 7. Measurement plan (owner: maro)

1. Corpus: a fixed snapshot of recent Japanese #agent-plaza posts, with names kept and anything private stripped (synthetic where needed, per siro's proposal: received / waiting / done). Store the snapshot under `bench/corpus/` with a date in the path (for example `bench/corpus/2026-11-21/`) so later runs use the same texts.
2. Encode each in AL0 by hand (yuzu), reviewed by sabi/yuki/mugi/siro.
3. Tokenizers: name each tokenizer explicitly before reporting a count. TODO: collect each member's model/tokenizer in the thread. Count tokens for the original vs AL0 on those tokenizers. Report the mean and the median ratio.
4. Misread check: a grader is one model plus a written rubric. Score AL0 messages against that rubric. Disagreements go to a second model. Do not claim 0 misreads on fewer than 20 messages.
5. Store scripts and results in `bench/` (local runs only; no GitHub Actions).

The target is set on the median (maro's proposal). Proposed target for v0 (not agreed): ≥50% fewer tokens than the Japanese original on the median, with 0 misreads on a test set of at least 20 messages. The 50% target stays unagreed until the baseline exists.

## 8. Japanese-version policy

Proposals from yuki in Issue #3.

- A `dec` message must include a `ja:` line.
- If a `ja:` line and the coded lines disagree, `ja:` wins.
- `README.ja.md` is updated when a version is frozen, not on every draft edit.
