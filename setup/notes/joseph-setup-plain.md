# Joseph's AI setup, in plain language

Source: https://rascaltwo.github.io/ai-setup/rascal-ai-setup-tour/
Repo: https://github.com/RascalTwo/ai-setup
Read on 2026-09-08. 16 "stops" on a guided map.

## The self-improving loop (what Ryan asked about first)

Joseph's first attempt was **`retro`**: you ran it by hand at the end of a chat,
it wrote a report on what went wrong. It failed:

- You had to remember to run it. Eventually nobody did — it sat dead for 3.5
  months before anyone noticed.
- Each report covered one chat only. Nothing added up across chats, so no
  problem ever looked important enough to fix.
- It wrote 89 suggestions. Zero were ever merged.
- Its filter for "what counts as a problem" only caught 20-64% of real problems,
  and never looked at the AI's own messages at all - so it missed 60 of 91.

**`dream`** replaced it. Same goal, four fixes:

1. **Automatic.** Runs every night on a Mac timer. Can't be forgotten.
2. **Across all chats at once.** Consolidating is the whole point - a mistake
   made 14 times is obviously worth fixing; made once, it isn't.
3. **Grabs too much on purpose, then narrows.** The cheap first pass keeps every
   real user message plus every AI sentence that sounds like an admission
   ("I assumed...", "I missed..."). Then one Claude call scores what matters.
4. **Groups by root cause using free local math.** Similar complaints get
   bundled without paying for more AI calls.

Output: each bundle becomes **a branch you can review** - real proposed edits,
flagged if two proposals collide. A dashboard ranks them by
how-often x how-bad. **Nothing applies itself.** A human approves. It also nags
you with "last run: N days ago" so it can never die quietly again.

## Foundation (stops 1-5)

- One public repo is the single source of truth for rules, skills, and settings.
- An installer script points the live config at the repo, instead of copying, so
  editing the repo changes the live setup instantly. It's safe to re-run and
  won't overwrite real files.
- One rules file serves both Claude Code and Codex. Two filenames, same content.
- Skills are the main capability layer.
- Review agents are written once and generated into each tool's format.

## The leverage (stops 6-10)

- **Memory lives outside any one AI tool** (`basic-memory`), so both agents read
  the same notes. Rule: search notes before guessing. Write rule: if the work is
  already producing a document, knowledge goes in the document; if nothing is
  being written down, it goes in memory. Never both.
- **Small free local models** for cheap grunt work (reading screenshots).
- **The rules file governs behavior, not style.** See glossary: evidence ladder,
  CLI-first, delegation threshold, Given/When/Then, laziest correct solution.
- **`r2-sdlc`** = his repeatable path from "idea" to "pull request":
  understand -> design -> build test-first -> simplify -> review.
  `r2-gauntlet` runs ~14 specialist reviewers and merges them into ONE ranked
  report so you don't read 14 reports.

## Terminal layer (stops 11-13)

He runs many Claude sessions at once, so his terminal is set up to manage a
crowd rather than one conversation.

- **Ghostty** - his terminal app, configured by text file so it's version
  controlled. He re-enabled the alert sound so an agent can get his attention.
- **herdr** - splits one window into many panes, one agent each, and *knows what
  agents are* - it tracks each as idle / working / blocked / done and shows in a
  sidebar which one needs him. Replaces guessing.
- **Four homegrown add-ons**, triggered by hooks (a hook = "run this script
  automatically when X happens"):
  - every image an agent touches gets saved and is browsable with a hotkey
  - a hotkey opens a new tab already running Claude in the current folder
  - when a session ends, a local model reads it and renames the tab something
    meaningful instead of "7"
  - a tracker that answers "which of my sessions are stuck?"

## Provenance and upkeep (stops 14-16)

"Provenance" = where things came from and who owns what. Three concerns:

1. **Layering.** Public repo + private work repo + machine-local repo, kept
   separate, one installer, all merging into one flat live folder. Company
   skills never touch the public repo.
2. **Publishing on purpose.** Real day-to-day history lives on a private branch.
   The public branch is rebuilt each time as a single fresh commit with no
   history, so nothing leaks through old commits. A guard script blocks any push
   that would break that rule. Publishing is a decision, never a side effect.
3. **Self-repair.** `dream`, above.

## Portable core for Ryan (descending value per effort)

1. A rules file (`AGENTS.md`) - highest leverage, no infrastructure.
2. A `dream`-style improvement loop.
3. Memory with an explicit write rule.
4. An installer / overlay split - only once there's enough to keep in sync.
