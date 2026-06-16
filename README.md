# recLLMend

A small media tracker that's nothing but markdown files and a coding agent reading and writing them. A tidy, git-tracked record of everything you've watched, played and read, with notes on why each one worked for you or not.
It's Andrej Karpathy's "LLM wiki" idea, pointed at media instead of notes.

## How it works

A handful of files, each with one job:

- `taste-profile.md` is the model of your taste. The agent reads it first, every time, and keeps it current. It's yours, and the upstream template never overwrites it. (On a fresh copy you create it from the template below.)
- `taste-profile.template.md` is the blank starting version. It belongs to the upstream repo, so you can pull in improvements without them clobbering your real profile.
- `log/` holds one append-only entry per thing you've finished (or bounced off of), split into a file per medium.
- `backlog.md` is a holding area for things without a verdict yet — what you own but haven't rated, what you're partway through, and the picks the agent has recommended but you haven't tried. Tell it you're _not interested_ in something and it's set aside so it won't be suggested again. Nothing here is scored; an item only becomes a log entry once you actually react to it. (On a fresh copy you create it from `backlog.template.md` when first needed, like the profile.)
- `raw/` is where exports from IMDb, Goodreads, your Steam library and the like land when you import them. Your log is the record that matters, so `raw/` just keeps the latest export per source — the agent diffs it against your log to pull in what's new.
- `AGENTS.md` is the schema and the rules the agent follows. It's the file doing the heavy lifting. `CLAUDE.md` and `GEMINI.md` are symlinks to it, so Claude Code, Codex and Gemini CLI all work out of the box with one set of instructions to maintain.

Day to day it's a loop. You tell your coding agent of choice what you consumed and how it felt, it logs an entry and adjusts your profile, and when you want something for the evening you ask and it reasons from the notes instead of the genre.

## What you can say

You're talking to an agent, not a CLI, so phrasing is loose — but these are the things it's set up to do:

- **`log: ...`** — record something you finished, and how it felt: _"log: finished Disco Elysium, 9/10, the writing carried systems I'd normally bounce off."_ It files the entry and updates your profile.
- **Bounced off something?** Say so — _"bailed on Elden Ring after ~6 hours, the open-world drift lost me."_ A did-not-finish is logged as its own thing, and where you quit is often a stronger signal than any rating.
- **`recommend ...`** — ask for a pick: _"recommend me something short and weird for tonight."_ It reasons from your profile rather than genre, and (if it can browse) will surface recent or off-the-beaten-path options and check they're real before suggesting them. It also remembers what it suggested, so it won't repeat itself and can follow up later.
- **`what's on my list`** / **`what have you recommended`** — see the picks it suggested that you haven't gotten to yet, with how long they've been waiting. It'll also nudge you about aging ones when you ask for something new or report finishing something — so "that book you recommended a couple weeks back" still works in a fresh session.
- **`queue ...`** / **`I'm playing/reading ...`** / **`what's in my backlog`** — park something you own or are partway through _without_ rating it; it graduates to a real log entry once you have a genuine reaction.
- **`what's my taste in <medium>`** — a read-back of your profile for film, TV, games or books.
- **`ingest <file>`** — bulk-import an export (IMDb, Goodreads, Steam, …): it reads the file, pulls everything new into your log, refreshes your profile, and tidies the export into `raw/`. Works the same for your first import and for an updated export months later.
- **`resync`** — force a full rebuild of your profile from the log. You rarely need it; the agent reconciles on its own as the log grows. It's here for when you want it explicitly.

## Which model

recLLMend is only as good as the reasoning behind it — writing an honest structural _why_, inferring cross-cutting taste axes, verifying picks against the web so it doesn't recommend things that don't exist, and periodically rebuilding your profile from the whole log. That's reasoning-heavy work, so run it on a **flagship, frontier-tier model** (e.g. Claude Opus or Sonnet, a top-tier GPT, or a Gemini Pro-class model). A **small/fast tier** (Haiku-, Flash-, or mini-class) tends to give shallow *why*s, weaker cross-domain inference, and more hallucinated recommendations — the failure mode that erodes trust fastest. Everything is just markdown, so you can switch to a stronger model anytime and run `resync` to rebuild the profile.

## Setup

Your taste data is personal, so keep your own copy private. Clone this repo into a fresh private repo that still shares history with it, which is what lets you pull future changes cleanly:

```sh
# 1. Clone the blueprint
git clone https://github.com/venhaus/recllmend.git my-recllmend
cd my-recllmend

# 2. Create an empty PRIVATE repo on GitHub (e.g. <your-username>/my-recllmend), then point origin at it
git remote rename origin upstream          # the public blueprint stays as "upstream"
git remote add origin https://github.com/<your-username>/my-recllmend.git   # <your-username> = your GitHub login
git push -u origin main

# 3. Create your live profile from the template
cp taste-profile.template.md taste-profile.md
git add taste-profile.md && git commit -m "Start my taste profile"
```

Then open your coding agent in the folder. Either talk to the agent to fill in `taste-profile.md`, or drop your exports into `raw/` and tell it to ingest them. From there you just talk to it, using the commands above.

It's plain markdown the whole way down. Read it with `cat`, edit it in any editor, diff it in git, render it on GitHub.

## Keeping your copy up to date

Because your repo is a clone rather than a template copy, it shares history with this one, so pulling later changes is just a normal merge:

```sh
git fetch upstream
git merge upstream/main
```

This stays painless because of a simple split. This repo only ever touches `AGENTS.md` (and its `CLAUDE.md` / `GEMINI.md` symlinks), `README.md` and `taste-profile.template.md`; your copy only touches `taste-profile.md`, `log/` and `raw/`. The two never edit the same files, so merges don't conflict and your taste data never leaks back upstream.

## License

MIT. Do whatever you want with it.
