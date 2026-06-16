# recLLMend

> Operating manual for any coding agent working in this repo. `CLAUDE.md` and `GEMINI.md` are symlinks to this file, so Claude Code, Codex, and Gemini CLI all read the same instructions. Edit this file, never the symlinks.

A personal media tracker and recommendation engine built as plain markdown, queried and maintained by a coding agent.
Just text files an agent reads and writes. Based on the Karpathy "LLM wiki" pattern.

## What this repo is

- `AGENTS.md` (this file) — the schema and the rules the agent follows. The load-bearing file; `CLAUDE.md` and `GEMINI.md` symlink to it.
- `taste-profile.md` — the model of the user's taste. Small, always read first. The agent maintains this. It's the live, user-owned file; if it doesn't exist yet, this is a fresh copy of the blueprint — copy `taste-profile.template.md` to `taste-profile.md` before the first interview (see Conventions).
- `taste-profile.template.md` — the pristine starting template, owned by the upstream blueprint. Never fill it with real data; it exists so blueprint improvements can flow down without touching the user's live profile.
- `log/` — append-only consumption entries, sharded by medium (`log/film.md`, `log/tv.md`, `log/games.md`, `log/books.md`). One entry per work.
- `backlog.md` — a holding area for items that aren't taste signal _yet_: owned-but-unrated, currently in progress, recommended-but-not-yet-tried, queued, or abandoned without a verdict. Explicitly **not** the taste model — no ratings are invented here. It exists so this data isn't lost or force-fit into the log; an item graduates to a real log entry only when a genuine reaction arrives. User-owned; if it doesn't exist yet, copy `backlog.template.md` to `backlog.md` on first need.
- `backlog.template.md` — the pristine starting template for `backlog.md`, owned by the upstream blueprint. Never fill it with real data; like the profile template, it exists so blueprint improvements flow down without touching the user's live backlog.
- `raw/` — exports you've imported (IMDb CSV, Goodreads export, etc.), landed here by `ingest`. The log is the source of truth, so `raw/` only needs the latest export per source; don't hand-edit them, and let new exports supersede old ones.

## Core loop

Both halves below start by reading `taste-profile.md`, and reading it means running the staleness check under "Keeping the profile honest" first — that's what makes reconciliation happen on its own, without the user asking.

When the user reports something they consumed — finished or bounced off ("watched X, 8/10, the structure-as-content thing landed"; "bailed on Y three hours in, the systems never cohered"):

1. Identify the work before logging it. Resolve the title to a specific, real work (medium, year, creator); if it's ambiguous, a likely typo, or matches several works, browse to disambiguate and confirm with the user rather than guessing. Logging the wrong work — or a phantom one — silently corrupts the log and everything built on it, so this is the step you never skip. Then make sure you actually understand what the work _is_ and how it's built; if you don't, look it up (web if available) so you can read the user's reaction correctly.
2. Append a log entry to the correct medium shard, using the entry schema below. A bounce is a normal entry with `rating: bounced` and a `bailed` point. Use your understanding of the work to write a `why` that names the real structural feature the user is reacting to — but the reaction and rating are theirs: interpret and sharpen what they said, never invent or overrule it. If their read diverges from the received take on the work, log their read; the divergence is itself signal.
3. If the reaction reveals or shifts a taste signal, update `taste-profile.md` accordingly. Note material updates briefly; don't rewrite the whole profile for one data point. Bounces are high-value here: they calibrate the medium's friction tolerance and often belong under "Confirmed bounce-offs."
4. Tell the user what you logged and any profile change, then continue the conversation.

When the user asks for a recommendation:

1. Read `taste-profile.md` (always) plus the relevant medium shard(s). Do NOT read the entire log if only one medium is in scope. Also scan `log/` + `backlog.md` for the candidates you're weighing, so you never recommend something already consumed, bounced, parked, or previously suggested — including cross-platform (a work may have been played/read off the export).
2. Reason from the _why_ fields and the medium-specific model, not surface genre. The point of this tool over Letterboxd-style services is reasoning about structural fit across whatever signals actually drive this person — friction tolerance is the classic example, but tone, novelty appetite, and commitment cost matter just as much. But a single-user, content-based model like this _will_ narrow into a filter bubble if you only maximize fit, so weigh picks as fit × _unexpectedness_, not fit alone: reserve at least one pick as a deliberate stretch — relevant but genuinely surprising relative to the log. Treat a novelty appetite as licence to explore, not just a box to match.
3. If you can browse the web, use it to get past the limits of your training data: surface recent releases and genuinely off-the-beaten-path candidates the user probably hasn't hit. **Verify by checking, not recalling** — an LLM's unaided self-check is unreliable, so confirm every non-obvious pick against the web (it exists, with the right year / form / creator) _and_ against `log/` + `backlog.md` (not already consumed, bounced, or suggested), and put forward only what survives. Search expands and fact-checks the candidate set; it does not import popularity — and your own sense of ranking is biased by list position and crowd popularity, so let the structural-fit reasoning decide, not the order or the buzz. If you can't browse, lean on what you're confident about and flag that picks may be dated or unverified.
4. Give a small number of high-confidence picks with the reason each should land. Flag the bet level honestly. Never invent a title or its details — a smaller list you can vouch for beats a confident hallucination. Record each pick in `backlog.md` as a `recommended` item (date, bet level, one-line predicted fit) so it isn't re-suggested and the prediction can be checked against the real reaction when one lands.

## Keeping the profile honest

`taste-profile.md` is a cache of the log, and caches drift. The per-entry updates in the core loop are deliberately small, so over time the profile skews recent and accrues hypotheses the log has since outgrown. Two things keep it honest without a manual rewrite:

- **Anchor every signal.** A claim should cite its evidence — a log entry or two, a _contrast or pattern across entries_, or an _explicit stated preference_ — and sit at the right confidence: settled patterns in the per-medium sections, untested ones under "Open questions / hypotheses to test." Some of the most useful signals are cross-cutting axes anchored to a contrast (e.g. loving a solo puzzle but disliking the same thing made competitive) rather than to one title; that's fine as long as the evidence is named and falsifiable, so a resync can still check it.
- **Reconciliation marker.** The profile header carries `Last reconciled: YYYY-MM-DD (N entries)`. Read it whenever you read the profile.

Resync is automatic — the user should never have to ask for it. You already read `taste-profile.md` at the start of every interaction, so checking its marker is on the hot path. Make it a mandatory first step, not an option:

- **Staleness check (every interaction).** Count current log entries — `grep -c '^### ' log/*.md` summed across shards — and compare to the marker's N. If the log has grown ~20 entries past it, resync _before_ answering or logging.
- **Contradiction check (at log time).** If a freshly logged entry clashes with a signal already in the profile, resync — or at minimum correct the contradicted signal — then continue, and tell the user.

"resync" stays available as a command the user _can_ invoke, but that's a manual override of an automatic process, not the trigger the system leans on.

A resync is a deliberate heavy pass, like ingest: read every shard, rebuild the taste model from scratch, diff it against the current profile, and tell the user what moved and why — hypotheses promoted to settled, signals demoted or dropped, new patterns found. Then rewrite `taste-profile.md`, prune it back toward the soft ~120-line tripwire (the goal is no redundancy with the log, not a line count), and update the marker. Only `taste-profile.md` changes; the log is append-only and is never rewritten during a resync.

## Seeding and re-ingesting

**First run (empty log).** Two ways in, converging on the same thing — an honest first draft of the model:

- _From an export:_ ingest it (rules below), then run a short taste interview. Surface the patterns the data actually shows — clusters of high ratings, outliers, things abandoned — and ask a few pointed questions about _why_ they hold. Seed `taste-profile.md` from the answers, not the raw ratings: the numbers tell you where to dig, the reasoning is what you record.
- _From scratch:_ with no export, interview directly — a handful of works they love and ones they bounced off, across the media they care about, always chasing the _why_.

Either way the first profile is mostly hypotheses, not settled signal — park tentative reads under "Open questions / hypotheses to test" and let the log confirm or kill them. Set the reconciliation marker when you finish.

**Re-ingesting (returning user, updated export).** Exports are cumulative, so a fresh dump mostly repeats what's already logged. Your baseline is the log, not the old export — diff the new export against the log, never against a previous export:

- Ingest only the _delta_: skip works already logged unchanged, append works that are new, and for a work whose rating changed, edit its entry in place and note the change in the `why` (the re-rate rule). Match works by title + year + medium.
- File the export into `raw/` under a clear, dated name (e.g. `raw/imdb-2026-06-15.csv`), replacing the previous one. No need to hoard superseded exports: the log is the durable record, the export is reproducible from the source any time, and git keeps old versions if you ever want them. Just never hand-edit an export — it's a faithful machine drop.
- Don't re-run the first-run interview. After a sizable delta, run a `resync` so the profile reflects the new data, and briefly ask about anything striking in what's new.

**Large exports.** A first import can be hundreds of rows, but turning them into backlog entries is mechanical — parse, translate the rating, format, append the `(imported — no reason captured)` sentinel. Reach for the most reliable tool you have: if you can run code, a short throwaway script that emits the markdown beats hand-transcribing hundreds of entries and won't silently drop or duplicate rows. If you parallelize with subagents or workers instead, split the job by shard (one medium per worker) so no two writers touch the same file. The taste interview and profile seeding come afterward as a single pass over the whole set — that synthesis isn't parallelizable. Subagent/worker support varies by agent; skip it if your harness lacks it.

## Entry schema

Each log entry is one block:

```
### <Title> (<Year>)
- **medium**: film | tv | game | book
- **rating**: <n>/10   — or `bounced` if the user didn't finish it (unified 0–10 scale; see the rating-translation rule under Conventions for imports). A `bounced` entry must name its *kind* in the `why` (see the bounce-taxonomy convention): a bounce is not always a negative verdict.
- **date**: YYYY-MM-DD   (date logged, or "backlog" for bulk-imported history)
- **bailed**: where they quit — "20 min", "ep3", "~p.80", "act 2". Bounce entries only; omit it for anything finished. Where someone bails is the friction-tolerance signal, so capture it.
- **why**: The *structural* reason it landed, didn't, or made them bail — what about its form, pacing, or demands matched or clashed with the taste model. This is the whole value of the repo, so capture it as richly as the reaction warrants: often a sentence or two, but never flatten real nuance (a work can land and clash at once). The test is specificity, not length — say what the rating alone can't. Never leave it empty; if the user didn't say, ask one short question rather than guessing. End every `why` with a **provenance tag** (see Conventions): `(stated)`, `(inferred)`, or the sentinel. The *only* full-sentinel case: a bulk-imported entry (`date: backlog`) whose source carries no reasoning may use `(imported — no reason captured)`. Never use that sentinel for a freshly reported work.
```

A finished entry and a bounce, filled in:

```
### Example Film (2019)
- **medium**: film
- **rating**: 9/10
- **date**: backlog
- **why**: Structure was the content — the [specific formal device] made the viewer experience the theme rather than observe it. The kind of formal-invention-with-momentum that lands hardest. (stated)

### Example Game (2021)
- **medium**: game
- **rating**: bounced
- **date**: backlog
- **bailed**: ~4h in
- **why**: Friction bounce — strong premise, but the moment-to-moment systems never cohered into momentum and the busywork-to-payoff crossed the user's friction line. (stated)
```

## Conventions

- Append, don't reorder. Newest entries go at the bottom of a shard.
- Series vs. work granularity: one entry per work by default, but collapse a series into a single entry when the user's verdict is blanket across it ("the whole Uncharted series, ~7"), and split it into per-work entries when their reactions diverge. The rule scales down too — log a standout episode on its own only when it carries a distinct reaction.
- Ratings are directional, not precise. This domain tolerates fuzziness; don't agonize over a point.
- A bounce (didn't finish) is signal, not a gap: log it with `rating: bounced` and a `bailed` point. Where someone bails — and what they push through — is the main evidence for each medium's friction tolerance.
- Bounce taxonomy — name the _kind_ at the start of the bounce's `why`, because they are different signals: **disengagement** (lost interest before forming a verdict), **friction** (too long / hard / grindy — the classic friction-tolerance bounce), **contextual** (mood, genre-satiation, or sequencing — _not_ about the work itself, so don't over-read it as durable taste), or **dislike** (an active negative verdict — but a verdict usually belongs as a low score, not a bounce). Distinguishing these is what makes the friction model trustworthy. A bounce always requires having _started_ the work and quit; deciding not to try something at all (sight-unseen "not interested") is **not** a bounce — it's a `dismissed` backlog item (see the backlog convention).
- `backlog.md` is a parking lot, not the log: record owned-but-unrated, in-progress, queued, recommended, abandoned-without-verdict, and dismissed items there with an optional status (`owned` / `playing` / `queued` / `recommended` / `abandoned` / `dismissed`). For a `recommended` item, also jot the date, your bet level, and the one-line predicted fit, so the recommender can be held to account. Never invent a rating to force something into the log — move an item into a real log entry only when a genuine reaction (a rating + why, or a bounce) actually arrives; for a `recommended` one, note in the new entry's `why` whether the prediction held (recommendation calibration).
- Dismissal vs. bounce: if the user says they're **not interested** in a backlog/recommended item _without having engaged with it_, that is a sight-unseen rejection, not a bounce (a bounce needs actual engagement and a bail point) and not a log entry (nothing was consumed). Mark it `dismissed` with the reason and stop suggesting it. If the reason is taste-revealing ("too anime", "a competitive MMO"), fold _that_ into `taste-profile.md` as a stated anti-preference; for a `recommended` item, treat the miss as recommendation calibration (the bet was wrong at the premise stage). `abandoned` is the _passive_ version (drifted off, no opinion); `dismissed` is the _active_ "no". These backlog statuses are low-stakes optional tags, not a taxonomy to agonize over: if you're unsure which fits, don't sweat it — both suppress the item from future suggestions, and the only thing the choice governs is whether it's okay to _gently_ resurface the item later (`abandoned` may be; `dismissed` is a closed door). Recommendations persist across sessions and carry a date, so a later session can resurface them by name and age — when the user asks for something new or reports finishing a work, gently follow up on aging `recommended` items ("the book I suggested ~2 weeks ago — did you get to it?") and close the loop on their answer. Surface, don't nag.
- Translate imported ratings to the unified 0–10 scale: 5-star sources (Goodreads, Letterboxd) ×2, preserving halves (4.5★ → 9); IMDb's 1–10 carries over as-is. Note the source scale in the ingestion summary so the conversion is auditable.
- If a work already has an entry and the user re-rates it, edit in place and note the change in the `why`.
- `taste-profile.md` is a summary that points into the log, never a second copy of it. Richness is the point — a detailed, multi-axis model is this product's whole value, so never drop a real distinction just to be brief. What you cut is _redundancy and staleness_, not nuance: prune sections that merely restate log entries or repeat each other, and claims the log no longer supports. Treat ~120 lines _of model content_ (the signals themselves, not the marker or section headers) as a soft "time to prune redundancy / resync" tripwire, not a budget; a longer profile made entirely of distinct, anchored, falsifiable signals is fine.
- Never invent a `why`. Ask — or, for backlog imports only, use the `(imported — no reason captured)` sentinel.
- Tag `why` provenance so resyncs and recommendations can weight it: `(stated)` = the user's own reasoning; `(inferred)` = your structural read from their patterns, offered for confirmation; `(imported — no reason captured)` = the backlog-import sentinel. Stated outranks inferred outranks imported. An `(inferred)` claim is a hypothesis, not fact — surface it for the user to confirm, and a resync should re-examine every `(inferred)` `why` rather than trusting it.
- Identify before you log. Resolve typos and ambiguous titles to a real, specific work, and understand what it is before writing the entry. A misidentified entry poisons the log at its source and pins the user's real reaction to the wrong work — worse than any bad recommendation, because everything downstream trusts the log. Research the work (objective); never fabricate the reaction (subjective).
- Never recommend a work you can't vouch for. Verify off-canon or recent picks — that they exist and match what you're claiming — or flag them as unverified. A hallucinated recommendation is the one unrecoverable error here: it teaches the user the tool can't be trusted.
- Sharding rule: if a shard grows past a few hundred entries, split by decade (`log/film-2020s.md`). Not needed at the start.
- File ownership (keeps blueprint updates conflict-free): the blueprint owns `AGENTS.md` (and its `CLAUDE.md` / `GEMINI.md` symlinks), `README.md`, `taste-profile.template.md`, and `backlog.template.md`; the user's copy owns `taste-profile.md`, `log/`, `raw/`, and `backlog.md`. Never write personal data into a blueprint-owned file, and on a fresh copy create the live `taste-profile.md` and `backlog.md` by copying their templates rather than editing the templates in place.

## Commands the user may use

- "log: ..." or natural report of consumption → run the core append loop.
- "queue ..." / "add to backlog ..." / "I'm playing/reading <X>" → record in `backlog.md` with a status; do not invent a rating. Promote to the log when a real reaction lands.
- "not interested in <X>" / "skip that rec" → mark the backlog/recommended item `dismissed` (with the reason); stop suggesting it, and if the reason is taste-revealing, fold it into `taste-profile.md`. Not a log entry — nothing was consumed.
- "what's in my backlog" → summarize `backlog.md`.
- "what have you recommended" / "what's on my list" → summarize the `recommended` items in `backlog.md` (with bet level and how long they've been waiting).
- "recommend ..." → run the recommendation loop (e.g. "recommend me something short and weird tonight").
- "what's my taste in <medium>" → summarize from the profile + shard.
- "resync" → reconcile `taste-profile.md` against the full log (see "Keeping the profile honest"). The heavy maintenance op; do it deliberately.
- "ingest <file>" → the end-to-end import: read the export, reconcile it into the log (delta only — see "Seeding and re-ingesting"), seed or `resync` the profile from it, and file the export into `raw/` under a clear dated name. The heavy operation; do it deliberately. Imported backlog entries take the `(imported — no reason captured)` sentinel rather than an invented `why`.
