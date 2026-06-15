# recLLMend

> Operating manual for any coding agent working in this repo. `CLAUDE.md` and `GEMINI.md` are symlinks to this file, so Claude Code, Codex, and Gemini CLI all read the same instructions. Edit this file, never the symlinks.

A personal media tracker and recommendation engine built as plain markdown, queried and maintained by a coding agent.
Just text files an agent reads and writes. Based on the Karpathy "LLM wiki" pattern.

## What this repo is

- `AGENTS.md` (this file) — the schema and the rules the agent follows. The load-bearing file; `CLAUDE.md` and `GEMINI.md` symlink to it.
- `taste-profile.md` — the model of the user's taste. Small, always read first. The agent maintains this. It's the live, user-owned file; if it doesn't exist yet, this is a fresh copy of the blueprint — copy `taste-profile.template.md` to `taste-profile.md` before the first interview (see Conventions).
- `taste-profile.template.md` — the pristine starting template, owned by the upstream blueprint. Never fill it with real data; it exists so blueprint improvements can flow down without touching the user's live profile.
- `log/` — append-only consumption entries, sharded by medium (`log/film.md`, `log/tv.md`, `log/games.md`, `log/books.md`). One entry per work.
- `raw/` — immutable source exports (IMDb CSV, Goodreads export, etc.). Never edit these; they exist for retracing.

## Core loop

Both halves below start by reading `taste-profile.md`, and reading it means running the staleness check under "Keeping the profile honest" first — that's what makes reconciliation happen on its own, without the user asking.

When the user reports something they consumed — finished or bounced off ("watched X, 8/10, the structure-as-content thing landed"; "bailed on Y three hours in, the systems never cohered"):

1. Append a log entry to the correct medium shard, using the entry schema below. A bounce is a normal entry with `rating: bounced` and a `bailed` point.
2. If the reaction reveals or shifts a taste signal, update `taste-profile.md` accordingly. Note material updates briefly; don't rewrite the whole profile for one data point. Bounces are high-value here: they calibrate the medium's friction tolerance and often belong under "Confirmed bounce-offs."
3. Tell the user what you logged and any profile change, then continue the conversation.

When the user asks for a recommendation:

1. Read `taste-profile.md` (always) plus the relevant medium shard(s). Do NOT read the entire log if only one medium is in scope.
2. Reason from the _why_ fields and the medium-specific model, not surface genre. The point of this tool over Letterboxd-style services is reasoning about structural fit across whatever signals actually drive this person — friction tolerance is the classic example, but tone, novelty appetite, and commitment cost matter just as much.
3. If you can browse the web, use it to get past the limits of your training data: surface recent releases and genuinely off-the-beaten-path candidates the user probably hasn't hit, and *verify* that anything non-obvious actually exists and has the attributes you're leaning on (year, form, premise) before putting it forward. Search expands and fact-checks the candidate set; it does not import popularity — the fit reasoning stays grounded in the taste model, or you've just rebuilt Letterboxd. If you can't browse, lean on what you're confident about and flag that picks may be dated or unverified.
4. Give a small number of high-confidence picks with the reason each should land. Flag the bet level honestly. Never invent a title or its details — a smaller list you can vouch for beats a confident hallucination.

## Keeping the profile honest

`taste-profile.md` is a cache of the log, and caches drift. The per-entry updates in the core loop are deliberately small, so over time the profile skews recent and accrues hypotheses the log has since outgrown. Two things keep it honest without a manual rewrite:

- **Anchor every signal.** A claim in the profile should name a log entry or two as its evidence and sit at the right confidence: settled patterns in the per-medium sections, untested ones under "Open questions / hypotheses to test." A claim you can trace is a claim a resync can check.
- **Reconciliation marker.** The profile header carries `Last reconciled: YYYY-MM-DD (N entries)`. Read it whenever you read the profile.

Resync is automatic — the user should never have to ask for it. You already read `taste-profile.md` at the start of every interaction, so checking its marker is on the hot path. Make it a mandatory first step, not an option:

- **Staleness check (every interaction).** Count current log entries — `grep -c '^### ' log/*.md` summed across shards — and compare to the marker's N. If the log has grown ~20 entries past it, resync *before* answering or logging.
- **Contradiction check (at log time).** If a freshly logged entry clashes with a signal already in the profile, resync — or at minimum correct the contradicted signal — then continue, and tell the user.

"resync" stays available as a command the user *can* invoke, but that's a manual override of an automatic process, not the trigger the system leans on.

A resync is a deliberate heavy pass, like ingest: read every shard, rebuild the taste model from scratch, diff it against the current profile, and tell the user what moved and why — hypotheses promoted to settled, signals demoted or dropped, new patterns found. Then rewrite `taste-profile.md`, prune it back toward the soft ~120-line tripwire (the goal is no redundancy with the log, not a line count), and update the marker. Only `taste-profile.md` changes; the log is append-only and is never rewritten during a resync.

## Entry schema

Each log entry is one block:

```
### <Title> (<Year>)
- **medium**: film | tv | game | book
- **rating**: <n>/10   — or `bounced` if the user didn't finish it (unified 0–10 scale; see the rating-translation rule under Conventions for imports)
- **date**: YYYY-MM-DD   (date logged, or "backlog" for bulk-imported history)
- **bailed**: where they quit — "20 min", "ep3", "~p.80", "act 2". Bounce entries only; omit it for anything finished. Where someone bails is the friction-tolerance signal, so capture it.
- **why**: The *structural* reason it landed, didn't, or made them bail — what about its form, pacing, or demands matched or clashed with the taste model. This is the whole value of the repo, so capture it as richly as the reaction warrants: often a sentence or two, but never flatten real nuance (a work can land and clash at once). The test is specificity, not length — say what the rating alone can't. Never leave it empty; if the user didn't say, ask one short question rather than guessing. The *only* exception: a bulk-imported entry (`date: backlog`) whose source carries no reasoning may use the sentinel `(imported — no reason captured)`. Never use that sentinel for a freshly reported work.
```

A finished entry and a bounce, filled in:

```
### Example Film (2019)
- **medium**: film
- **rating**: 9/10
- **date**: backlog
- **why**: Structure was the content — the [specific formal device] made the viewer experience the theme rather than observe it. The kind of formal-invention-with-momentum that lands hardest.

### Example Game (2021)
- **medium**: game
- **rating**: bounced
- **date**: backlog
- **bailed**: ~4h in
- **why**: Strong premise, but the moment-to-moment systems never cohered into momentum and the busywork-to-payoff crossed the user's friction line — a textbook friction-tolerance bounce.
```

## Conventions

- Append, don't reorder. Newest entries go at the bottom of a shard.
- Ratings are directional, not precise. This domain tolerates fuzziness; don't agonize over a point.
- A bounce (didn't finish) is signal, not a gap: log it with `rating: bounced` and a `bailed` point. Where someone bails — and what they push through — is the main evidence for each medium's friction tolerance.
- Translate imported ratings to the unified 0–10 scale: 5-star sources (Goodreads, Letterboxd) ×2, preserving halves (4.5★ → 9); IMDb's 1–10 carries over as-is. Note the source scale in the ingestion summary so the conversion is auditable.
- If a work already has an entry and the user re-rates it, edit in place and note the change in the `why`.
- `taste-profile.md` is a summary that points into the log, never a second copy of it. Richness is the point — a detailed, multi-axis model is this product's whole value, so never drop a real distinction just to be brief. What you cut is *redundancy and staleness*, not nuance: prune sections that merely restate log entries or repeat each other, and claims the log no longer supports. Treat ~120 lines *of model content* (the signals themselves, not the marker or section headers) as a soft "time to prune redundancy / resync" tripwire, not a budget; a longer profile made entirely of distinct, anchored, falsifiable signals is fine.
- Never invent a `why`. Ask — or, for backlog imports only, use the `(imported — no reason captured)` sentinel.
- Never recommend a work you can't vouch for. Verify off-canon or recent picks — that they exist and match what you're claiming — or flag them as unverified. A hallucinated recommendation is the one unrecoverable error here: it teaches the user the tool can't be trusted.
- Sharding rule: if a shard grows past a few hundred entries, split by decade (`log/film-2020s.md`). Not needed at the start.
- File ownership (keeps blueprint updates conflict-free): the blueprint owns `AGENTS.md` (and its `CLAUDE.md` / `GEMINI.md` symlinks), `README.md`, and `taste-profile.template.md`; the user's copy owns `taste-profile.md`, `log/`, and `raw/`. Never write personal data into a blueprint-owned file, and on a fresh copy create the live `taste-profile.md` by copying the template rather than editing the template in place.

## Commands the user may use

- "log: ..." or natural report of consumption → run the core append loop.
- "recommend ..." → run the recommendation loop (e.g. "recommend me something short and weird tonight").
- "what's my taste in <medium>" → summarize from the profile + shard.
- "resync" → reconcile `taste-profile.md` against the full log (see "Keeping the profile honest"). The heavy maintenance op; do it deliberately.
- "ingest <file>" → parse a `raw/` export into log entries, using the schema. This is the heavy operation; do it deliberately. Backlog entries take the `(imported — no reason captured)` sentinel rather than an invented `why`.
- First ingestion only: after parsing, run a brief taste interview. Surface the patterns the import actually reveals (clusters of high ratings, outliers, abandoned works) and ask a small number of pointed questions about *why* those patterns hold, then seed `taste-profile.md` from the answers. Build the profile from the user's reasoning, never from the raw ratings alone. Set the reconciliation marker to today's date and the post-ingest entry count when you finish.
