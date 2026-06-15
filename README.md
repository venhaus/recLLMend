# recLLMend

A personal media tracker and recommendation engine that is just a few markdown files read and maintained by a coding agent (e.g. Claude Code). If the tooling vanished tomorrow, you'd still have a clean, greppable, git-versioned record of everything you've watched, played, and read — and why you felt the way you did about it.

It's an instance of Andrej Karpathy's "LLM wiki" pattern, specialized for media.

## Why this exists

Recommendation services pattern-match on surface genre and "users like you." They can't reason about why something landed for you — that a film's structure was the point, that you'll tolerate friction in a game you'd never tolerate in a book. This repo stores the why and lets an agent reason over it. The model of your taste is the product; the list is just the substrate.

## How it works

taste-profile.md — the model of your taste. The agent reads this first, every time, and keeps it updated.
log/ — append-only entries, one per work, sharded by medium.
raw/ — your immutable source exports (IMDb, Goodreads, etc.), for bulk import and retracing.
CLAUDE.md — the schema and conventions the agent follows. The load-bearing file.

The loop: tell the agent what you consumed and how it felt → it appends an entry and updates your profile → ask it for recommendations → it reasons from the why, not the genre.

## Setup

Click Use this template to make your own repo (make it private — this is personal data).
Clone it locally.
Open Claude Code in the folder.
Fill in taste-profile.md by talking to the agent, or drop exports into raw/ and say "ingest" them.
From then on: "log: ..." to record, "recllmend ..." to get picks.

Everything stays plain markdown. Read it with cat, edit it in any editor, version it with git, render it on GitHub.

## License

MIT. Do whatever you like.
