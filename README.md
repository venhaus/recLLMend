# recLLMend

A personal media tracker and recommendation engine that is just a few markdown files read and maintained by a coding agent (e.g. Claude Code). If the tooling vanished tomorrow, you'd still have a clean, greppable, git-versioned record of everything you've watched, played, and read — and why you felt the way you did about it.

It's an instance of Andrej Karpathy's "LLM wiki" pattern, specialized for media.

## Why this exists

Recommendation services pattern-match on surface genre and "users like you." They can't reason about why something landed for you — that a film's structure was the point, that you'll tolerate friction in a game you'd never tolerate in a book. This repo stores the why and lets an agent reason over it. The model of your taste is the product; the list is just the substrate.

## How it works

taste-profile.md — the model of your taste. The agent reads this first, every time, and keeps it updated. It's yours; the blueprint never touches it. (Created on first setup by copying taste-profile.template.md.)
taste-profile.template.md — the pristine starting template, owned by the blueprint so improvements can flow down to your copy without clobbering your real profile.
log/ — append-only entries, one per work, sharded by medium.
raw/ — your immutable source exports (IMDb, Goodreads, etc.), for bulk import and retracing.
CLAUDE.md — the schema and conventions the agent follows. The load-bearing file.

The loop: tell the agent what you consumed and how it felt → it appends an entry and updates your profile → ask it for recommendations → it reasons from the why, not the genre.

## Setup

Your real taste data is personal, so your copy must be a **private** repo. A GitHub fork won't do — forks of a public repo stay public. Instead, duplicate this repo into a private one that still shares git history, so you can pull future blueprint improvements cleanly:

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

Then open Claude Code in the folder and fill in `taste-profile.md` by talking to the agent, or drop exports into `raw/` and say "ingest" them. From then on: "log: ..." to record, "recllmend ..." to get picks.

Everything stays plain markdown. Read it with cat, edit it in any editor, version it with git, render it on GitHub.

## Keeping your copy in sync with the blueprint

Because your private repo shares history with the blueprint (it's a clone, not a template), pulling later improvements is an ordinary merge:

```sh
git fetch upstream
git merge upstream/main      # or: git rebase upstream/main
```

This stays conflict-free by design via **file ownership**: the blueprint only ever changes `CLAUDE.md`, `README.md`, and `taste-profile.template.md` — files your copy doesn't edit. Your personal data lives in `taste-profile.md`, `log/`, and `raw/`, which the blueprint never touches. So upstream merges update the schema and conventions without ever colliding with (or leaking) your taste.

## License

MIT. Do whatever you like.
