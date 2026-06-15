# recLLMend

A personal media tracker and recommendation engine built as plain markdown, queried and maintained by a coding agent.
Just text files an agent reads and writes. Based on the Karpathy "LLM wiki" pattern.

## What this repo is

- `taste-profile.md` — the model of the user's taste. Small, always read first. The agent maintains this. It's the live, user-owned file; if it doesn't exist yet, this is a fresh copy of the blueprint — copy `taste-profile.template.md` to `taste-profile.md` before the first interview (see Conventions).
- `taste-profile.template.md` — the pristine starting template, owned by the upstream blueprint. Never fill it with real data; it exists so blueprint improvements can flow down without touching the user's live profile.
- `log/` — append-only consumption entries, sharded by medium (`log/film.md`, `log/tv.md`, `log/games.md`, `log/books.md`). One entry per work.
- `raw/` — immutable source exports (IMDb CSV, Goodreads export, etc.). Never edit these; they exist for retracing.

## Core loop

When the user reports something they consumed ("watched X, 8/10, the structure-as-content thing landed"):

1. Append a log entry to the correct medium shard, using the entry schema below.
2. If the reaction reveals or shifts a taste signal, update `taste-profile.md` accordingly. Note material updates briefly; don't rewrite the whole profile for one data point.
3. Tell the user what you logged and any profile change, then continue the conversation.

When the user asks for a recommendation:

1. Read `taste-profile.md` (always) plus the relevant medium shard(s). Do NOT read the entire log if only one medium is in scope.
2. Reason from the _why_ fields and the medium-specific model, not surface genre. The point of this tool over Letterboxd-style services is reasoning about structural fit and per-medium friction tolerance.
3. Give a small number of high-confidence picks with the reason each should land. Flag the bet level honestly.

## Entry schema

Each log entry is one block:

```
### <Title> (<Year>)
- **medium**: film | tv | game | book
- **rating**: <n>/10   (unified 0–10 scale across every medium; see the rating-translation rule under Conventions for imports)
- **date**: YYYY-MM-DD   (date logged, or "backlog" for bulk-imported history)
- **why**: One or two sentences on the *structural* reason it landed or didn't — what about its form, pacing, or demands matched or clashed with the taste model. This field is the whole value of the repo. Never leave it empty; if the user didn't say, ask one short question rather than guessing. The *only* exception: a bulk-imported entry (`date: backlog`) whose source carries no reasoning may use the sentinel `(imported — no reason captured)`. Never use that sentinel for a freshly reported work.
```

## Conventions

- Append, don't reorder. Newest entries go at the bottom of a shard.
- Ratings are directional, not precise. This domain tolerates fuzziness; don't agonize over a point.
- Translate imported ratings to the unified 0–10 scale: 5-star sources (Goodreads, Letterboxd) ×2, preserving halves (4.5★ → 9); IMDb's 1–10 carries over as-is. Note the source scale in the ingestion summary so the conversion is auditable.
- If a work already has an entry and the user re-rates it, edit in place and note the change in the `why`.
- Keep `taste-profile.md` under ~120 lines. It's a summary that points into the log, not a second copy of it.
- Never invent a `why`. Ask — or, for backlog imports only, use the `(imported — no reason captured)` sentinel.
- Sharding rule: if a shard grows past a few hundred entries, split by decade (`log/film-2020s.md`). Not needed at the start.
- File ownership (keeps blueprint updates conflict-free): blueprint owns `CLAUDE.md`, `README.md`, and `taste-profile.template.md`; the user's copy owns `taste-profile.md`, `log/`, and `raw/`. Never write personal data into a blueprint-owned file, and on a fresh copy create the live `taste-profile.md` by copying the template rather than editing the template in place.

## Commands the user may use

- "log: ..." or natural report of consumption → run the core append loop.
- "recllmend ..." → run the recommendation loop (e.g. "recllmend me something short and weird tonight").
- "what's my taste in <medium>" → summarize from the profile + shard.
- "ingest <file>" → parse a `raw/` export into log entries, using the schema. This is the heavy operation; do it deliberately. Backlog entries take the `(imported — no reason captured)` sentinel rather than an invented `why`.
- First ingestion only: after parsing, run a brief taste interview. Surface the patterns the import actually reveals (clusters of high ratings, outliers, abandoned works) and ask a small number of pointed questions about *why* those patterns hold, then seed `taste-profile.md` from the answers. Build the profile from the user's reasoning, never from the raw ratings alone.
