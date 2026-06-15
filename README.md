# recLLMend

A small media tracker that's nothing but markdown files and a coding agent reading and writing them. A tidy, git-tracked record of everything you've watched, played and read, with notes on why each one worked for you or not.
It's Andrej Karpathy's "LLM wiki" idea, pointed at media instead of notes.

## How it works

A handful of files, each with one job:

- `taste-profile.md` is the model of your taste. The agent reads it first, every time, and keeps it current. It's yours, and the upstream template never overwrites it. (On a fresh copy you create it from the template below.)
- `taste-profile.template.md` is the blank starting version. It belongs to the upstream repo, so you can pull in improvements without them clobbering your real profile.
- `log/` holds one append-only entry per thing you've finished (or bounced off of), split into a file per medium.
- `raw/` is where exports from IMDb, Goodreads, your Steam library and the like land when you import them. Your log is the record that matters, so `raw/` just keeps the latest export per source — the agent diffs it against your log to pull in what's new.
- `AGENTS.md` is the schema and the rules the agent follows. It's the file doing the heavy lifting. `CLAUDE.md` and `GEMINI.md` are symlinks to it, so Claude Code, Codex and Gemini CLI all work out of the box with one set of instructions to maintain.

Day to day it's a loop. You tell your coding agent of choice what you consumed and how it felt, it logs an entry and adjusts your profile, and when you want something for the evening you ask and it reasons from the notes instead of the genre.

## Setup

Your taste data is personal, so keep your own copy private. Clone this repo into a fresh private repo that still shares history with it, which is what lets you pull future changes cleanly:

```sh
# 1. Clone the blueprint
git clone https://github.com/venhaus/recllmend.git my-recllmend
cd my-recllmend

# 2. Create an empty PRIVATE repo on GitHub (e.g. you/my-taste), then point origin at it
git remote rename origin upstream          # the public blueprint stays as "upstream"
git remote add origin git@github.com:you/my-recllmend.git
git push -u origin main

# 3. Create your live profile from the template
cp taste-profile.template.md taste-profile.md
git add taste-profile.md && git commit -m "Start my taste profile"
```

Then open your coding agent in the folder. Either talk to the agent to fill in `taste-profile.md`, or drop your exports into `raw/` and tell it to ingest them. From there you just talk to it — see what you can say below.

It's plain markdown the whole way down. Read it with `cat`, edit it in any editor, diff it in git, render it on GitHub.

## What you can say

You're talking to an agent, not a CLI, so phrasing is loose — but these are the things it's set up to do:

- **`log: ...`** — record something you finished, and how it felt: *"log: finished Disco Elysium, 9/10, the writing carried systems I'd normally bounce off."* It files the entry and updates your profile.
- **Bounced off something?** Say so — *"bailed on Elden Ring after ~6 hours, the open-world drift lost me."* A did-not-finish is logged as its own thing, and where you quit is often a stronger signal than any rating.
- **`recommend ...`** — ask for a pick: *"recommend me something short and weird for tonight."* It reasons from your profile rather than genre, and (if it can browse) will surface recent or off-the-beaten-path options and check they're real before suggesting them.
- **`what's my taste in <medium>`** — a read-back of your profile for film, TV, games or books.
- **`ingest <file>`** — bulk-import an export (IMDb, Goodreads, Steam, …): it reads the file, pulls everything new into your log, refreshes your profile, and tidies the export into `raw/`. Works the same for your first import and for an updated export months later.
- **`resync`** — force a full rebuild of your profile from the log. You rarely need it; the agent reconciles on its own as the log grows. It's here for when you want it explicitly.

## Keeping your copy up to date

Because your repo is a clone rather than a template copy, it shares history with this one, so pulling later changes is just a normal merge:

```sh
git fetch upstream
git merge upstream/main
```

This stays painless because of a simple split. This repo only ever touches `AGENTS.md` (and its `CLAUDE.md` / `GEMINI.md` symlinks), `README.md` and `taste-profile.template.md`; your copy only touches `taste-profile.md`, `log/` and `raw/`. The two never edit the same files, so merges don't conflict and your taste data never leaks back upstream.

## License

MIT. Do whatever you want with it.
