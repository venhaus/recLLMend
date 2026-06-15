# recLLMend

A small media tracker that's nothing but markdown files and a coding agent (I use Claude Code) reading and writing them. No app, no database. If the agent vanished tomorrow I'd still have a tidy, greppable, git-tracked record of everything I've watched, played and read, with notes on why each one worked for me or didn't.

It's Andrej Karpathy's "LLM wiki" idea, pointed at media instead of notes.

## Why I built it

The big recommendation services are good at "people who liked X also liked Y" and not much else. They don't know that I loved a film mostly for how it was put together, or that I'll happily grind through a slow, fiddly game but bail on a slow book by page 30. So this repo keeps the *why*, and the agent reasons over that rather than genre tags. What I actually care about is the model of my taste; the list of titles is just where it gets stored.

## How it works

A handful of files, each with one job:

- `taste-profile.md` is the model of my taste. The agent reads it first, every time, and keeps it current. It's mine, and the upstream template never overwrites it. (On a fresh copy you create it from the template below.)
- `taste-profile.template.md` is the blank starting version. It belongs to the upstream repo, so I can pull in improvements without them clobbering my real profile.
- `log/` holds one append-only entry per thing I've finished, split into a file per medium.
- `raw/` is where I dump exports from IMDb, Goodreads and the like, left untouched for bulk imports and for going back to the source later.
- `CLAUDE.md` is the schema and the rules the agent follows. It's the file doing the heavy lifting.

Day to day it's a loop. I tell the agent what I watched and how it felt, it logs an entry and adjusts my profile, and when I want something for the evening I ask and it reasons from the notes instead of the genre.

## Setup

Your taste data is personal, so keep your own copy private. A fork won't do here, since forks of a public repo stay public. Instead, clone this into a fresh private repo that still shares history with it, which is what lets you pull future changes cleanly:

```sh
# 1. Clone the blueprint
git clone https://github.com/venhaus/recllmend.git my-taste
cd my-taste

# 2. Create an empty PRIVATE repo on GitHub (e.g. you/my-taste), then point origin at it
git remote rename origin upstream          # the public blueprint stays as "upstream"
git remote add origin git@github.com:you/my-taste.git
git push -u origin main

# 3. Create your live profile from the template
cp taste-profile.template.md taste-profile.md
git add taste-profile.md && git commit -m "Start my taste profile"
```

Then open Claude Code in the folder. Either talk to the agent to fill in `taste-profile.md`, or drop your exports into `raw/` and tell it to ingest them. After that it's "log: ..." to record something and "recllmend ..." when you want a pick.

It's plain markdown the whole way down. Read it with cat, edit it in any editor, diff it in git, render it on GitHub.

## Keeping your copy up to date

Because your repo is a clone rather than a template copy, it shares history with this one, so pulling later changes is just a normal merge:

```sh
git fetch upstream
git merge upstream/main
```

This stays painless because of a simple split. This repo only ever touches `CLAUDE.md`, `README.md` and `taste-profile.template.md`; your copy only touches `taste-profile.md`, `log/` and `raw/`. The two never edit the same files, so merges don't conflict and your taste data never leaks back upstream.

## License

MIT. Do whatever you want with it.
