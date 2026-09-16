# Glossary

Plain-language definitions of terms that came up while studying Joseph's setup.
Add to this whenever a new bit of jargon shows up.

## From the tour

**landed** — Slang for "actually got merged into the real code." A proposal that
was written but never merged did not "land." Joseph's old retro tool wrote 89
suggestions and merged zero of them.

**prefilter** — A cheap first pass that throws out obviously irrelevant stuff
before the expensive step runs. Like skimming a stack of mail for anything that
might be a bill, before actually opening envelopes.

**recall** (as a measurement) — Of all the real problems that existed, what
percent did the filter actually catch? 20% recall = it found 1 in 5 and missed
the other 4. (Different from the everyday meaning of "recall" = remembering.)

**over-collect** — Deliberately grabbing too much in the prefilter, on purpose.
Better to hand 500 maybes to the scoring step than to miss 60 real problems.
Cheap step is generous; expensive step is picky.

**launchd** — macOS's built-in "run this at a set time" scheduler. The Mac
version of a cron job. It runs `dream` every night whether or not you remember.

**Ollama** — An app that runs AI models on your own Mac instead of calling a
cloud API. Free after download, private, no per-token cost, but weaker than
Claude. Good for grunt work.

**nomic-embed-text** — One specific small model you run in Ollama. It turns a
sentence into a list of numbers ("embedding") where similar meanings produce
similar numbers.

**Ollama + arithmetic (clustering)** — Once sentences are lists of numbers, you
can group similar ones by comparing numbers — plain math, no AI call. That's how
`dream` finds "these 14 complaints are all the same problem" for free.

**conflict badge** — A warning label on a proposal meaning "this edit overlaps
with another proposal, you can't accept both cleanly."

**recurrence x blast radius** — The ranking formula for proposals.
*Recurrence* = how many times this problem happened. *Blast radius* = how much
it affects when it goes wrong. High on both = fix it first.

**symlink** — A shortcut/pointer file. `~/.claude/skills` isn't a real folder,
it's a pointer to the real one in the repo. Edit the repo, the live setup
changes instantly — no copying.

**repairs symlinks** — If a pointer is broken or aimed at the wrong place, the
installer silently re-aims it.

**refuses to clobber real files** — "Clobber" = overwrite and destroy. If the
installer finds a real file where it wanted to put a pointer, it stops instead
of deleting your work.

**canonical** — The one official copy that everything else defers to. Joseph has
one rules file; `CLAUDE.md` is just a pointer to it, so the two can never
disagree.

**knowledge graph** — Notes stored as things-and-connections ("Ryan → works on →
Project X → uses → Postgres") instead of paragraphs. Lets you ask "what touches
Postgres?"

**operating contract** — Rules about *how the agent should behave and decide*,
not about code style. Not "use tabs" but "check your memory before you guess."

**evidence ladder (memory -> tools -> web -> recall)** — Where to look for a fact,
in order, most-trustworthy first: 1) your saved notes, 2) real tools that read
live data, 3) web/docs, 4) last resort, the model's memory (most likely to be
confidently wrong).

**skills-and-MCP-before-ad-hoc-scripts** — Before writing a throwaway script,
check whether a saved skill or connected tool already does it. Stops you from
re-solving the same problem forever.

**MCP** — Model Context Protocol. The standard plug for connecting an AI to an
outside service (Notion, Jira, a database). An "MCP server" is one such plug.

**ad-hoc** — Made up on the spot for one use, then thrown away.

**CLI-first tool hierarchy** — Preference order for *how* to touch a system:
1) command line, 2) browser automation if it's genuinely a web thing,
3) clicking the actual desktop UI, 4) anything else. Earlier = faster and more
reliable.

**delegation threshold** — The rule for when to hand work to a second agent.
Big independent research with long output = delegate. One specific fact you need
for the very next edit = just look it up yourself.

**Given/When/Then** — A way of writing a test as a sentence.
*Given* a starting state, *When* I do X, *Then* Y should happen. Forces a
checkable claim instead of "looks good to me."

**proportional to risk** — Match the amount of checking to the stakes. Typo fix
gets a glance; payment code gets real tests.

**laziest correct solution** — Bias toward the smallest change that genuinely
works: reuse what's already in the codebase, use built-in features, don't build
frameworks nobody asked for.

**provenance** — The paper trail of where something came from. Here it means the
rules about which repo owns what, what's private, and how something becomes
public on purpose rather than by accident.
