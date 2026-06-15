# recLLMend

A personal media tracker and recommendation engine built as plain markdown, queried and maintained by a coding agent.
Just text files an agent reads and writes. Based on the Karpathy "LLM wiki" pattern.

## What this repo is

- `taste-profile.md` — the model of the user's taste. Small, always read first. The agent maintains this.
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
- **rating**: <n>/10   (or n/5 if that's the user's scale for this medium — keep it consistent within a shard)
- **date**: YYYY-MM-DD   (date logged, or "backlog" for bulk-imported history)
- **why**: One or two sentences on the *structural* reason it landed or didn't — what about its form, pacing, or demands matched or clashed with the taste model. This field is the whole value of the repo. Never leave it empty; if the user didn't say, ask one short question rather than guessing.
```

## Conventions

- Append, don't reorder. Newest entries go at the bottom of a shard.
- Ratings are directional, not precise. This domain tolerates fuzziness; don't agonize over a point.
- If a work already has an entry and the user re-rates it, edit in place and note the change in the `why`.
- Keep `taste-profile.md` under ~120 lines. It's a summary that points into the log, not a second copy of it.
- Never invent a `why`. Ask.
- Sharding rule: if a shard grows past a few hundred entries, split by decade (`log/film-2020s.md`). Not needed at the start.

## Commands the user may use

- "log: ..." or natural report of consumption → run the core append loop.
- "recllmend ..." → run the recommendation loop (e.g. "recllmend me something short and weird tonight").
- "what's my taste in <medium>" → summarize from the profile + shard.
- "ingest <file>" → parse a `raw/` export into log entries, using the schema. This is the heavy operation; do it deliberately.
