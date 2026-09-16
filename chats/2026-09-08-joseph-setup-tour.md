# Chat: distilling Joseph's AI setup tour

Started 2026-09-08. Raw log: `2026-09-08-joseph-setup-tour.raw.jsonl`

Tool calls are summarized as one line; only human and assistant text is kept in full.

## **Ryan**

My coworker Joseph has this incredible AI setup tour where he walks through installing certain tools, what skills and rules he sets up, and how he uses agents in his daily work. The link can be found here: https://rascaltwo.github.io/ai-setup/rascal-ai-setup-tour/

I need your help to distill this into something that I would use. He mentioned to me a few things that piqued my interest, like a "retrospective" skill that he runs that analyzes the chat and builds into existing skills and rules or creates new ones. There a ton of other examples, so let me know at a high level what's available and let's step through everything bit by bit until I have a full understanding of his setup.

---

## **Claude**

I'll pull up the tour page and read through it first.

---

> _tool: ToolSearch_ select:WebFetch

> _tool: WebFetch_ https://rascaltwo.github.io/ai-setup/rascal-ai-setup-tour/

## **Claude**

The summary is high-level; let me dig for the specifics, especially the retrospective skill.

---

> _tool: WebFetch_ https://rascaltwo.github.io/ai-setup/rascal-ai-setup-tour/

## **Claude**

The page is likely a JS-driven deck — WebFetch only sees the shell. Let me load it in the browser.

---

> _tool: mcp__Claude_Browser__navigate_ https://rascaltwo.github.io/ai-setup/rascal-ai-setup-tour/

> _tool: mcp__Claude_Browser__get_page_text_

## **Claude**

Good — that's stops 1–5. Let me advance the deck to get the rest.

---

> _tool: mcp__Claude_Browser__find_ Next

> _tool: mcp__Claude_Browser__browser_batch_

> _tool: mcp__Claude_Browser__javascript_tool_

> _tool: mcp__Claude_Browser__javascript_tool_

> _tool: mcp__Claude_Browser__javascript_tool_

## **Claude**

Found the retrospective story — stop 16. Let me get the rest of it.

---

> _tool: mcp__Claude_Browser__javascript_tool_

> _tool: WebFetch_ https://rascaltwo.github.io/ai-setup/skill-dream/

## **Claude**

I read the full tour (the deck loads stops 6–16 in JS, so I pulled them from the page source) plus the dedicated `dream` skill page. Here's the map.

## First, about that "retrospective" skill

The thing Joseph described exists, but the version he mentioned is **dead and was replaced**. Worth knowing before you copy it:

- **`retro`** (the original): manually invoked, wrote a per-session markdown report of what went wrong. It produced **89 proposals over two months, landed zero of them**, then went dormant for 3.5 months before anyone noticed. Its keyword prefilter had 20–64% recall and captured *zero* assistant turns — it missed 60 of 91 actual problems.
- **`dream`** (the replacement): runs **nightly via launchd, unprompted**, across *all* sessions rather than one. Pipeline is deliberately cheap:
  1. **Prefilter (local, over-collects)** — every substantive human turn + any assistant segment making first-person past-tense claims ("I incorrectly assumed…").
  2. **Score (one Claude call per batch)** — the only expensive step.
  3. **Cluster (local)** — `nomic-embed-text` on Ollama + arithmetic, not another LLM call.
  4. **Synthesize** — one cluster → one **git branch** with full diffs and conflict badges.
  5. **`dream-queue` dashboard** — the human gate. Ranked by recurrence × blast radius. Nothing auto-applies. A proposal that rewrites `dream` itself is just another branch to review.

The design lesson is the valuable part: *the retro failed because it was manual, per-session, unaggregated, and failed silently.* `dream` fixes exactly those four things.

## The rest of the setup, at altitude

**Foundation (stops 1–5)** — a public repo `github.com/RascalTwo/ai-setup` as single source of truth; a Bun installer (`bun install.ts`) that's idempotent, repairs symlinks, and refuses to clobber real files; `AGENTS.md` as the canonical rules file with `CLAUDE.md` pointing at the same bytes; a skills catalog; reviewer subagents authored once in Ruler, emitted to both Claude and Codex formats.

**The actual leverage (stops 6–10)**
- **`basic-memory` MCP** as shared persistent notes instead of any one agent's proprietary memory. Rule: search it *before* guessing. Write rule: if the work already produces markdown, knowledge goes there; if nothing's being written down, it goes in memory — never both.
- **Ollama helpers** — local vision (`gemma4:e4b`) before spending cloud vision tokens; `qwen2.5-coder:7b` for knowledge graphs.
- **`AGENTS.md` as an operating contract**, not a style guide. Three layers: inputs get cleaned (voice-homophone confirmation, an evidence ladder memory→tools→web→recall), work gets routed (skills-and-MCP-before-ad-hoc-scripts, CLI-first tool hierarchy, an explicit delegation threshold), outputs get safer (Given/When/Then verification proportional to risk, "laziest correct solution" bias).
- **`r2-sdlc`** — story → design → TDD → simplify → layered review, with `r2-gauntlet` collapsing the whole reviewer panel into one ranked report.

**Terminal layer (11–13)** — Ghostty configured by file; **herdr** replacing tmux (treats coding agents as first-class objects with `idle`/`working`/`blocked`/`done` lifecycle); and four homegrown hook-driven glue tools: `ttyimgspool`, `claude-tab`, `herdr-autolabel` (a `Stop` hook that renames tabs via local Ollama), `fleet`.

**Provenance (14–16)** — three repos collapsing into one flat catalog via `--overlay` so company skills never enter the public repo; `private/trunk` carrying real history while `main` is always a single parentless squashed commit, enforced by a pre-push guard; and `dream`.

**Skills** — 35 owned, plus 8 external packs by reference (`external-skills.json` → `npx skills`): Matt Pocock's planning pipeline (`handoff`, `grilling`, `wayfinder`), `ponytail` (simplicity bias), 14 `ln-*` background review auditors, `graphify`, `find-skills`, `skill-creator`.

## What I'd suggest distilling

Most of this is load-bearing for *his* job (Okta/Ping/Terraform/Confluence wrappers, provider porting) and worthless to you. The genuinely portable core is roughly four things, in descending value-per-effort:

1. **`AGENTS.md` as an operating contract** — highest leverage, zero infrastructure, works today.
2. **A `dream`-style nightly improvement loop** — the thing you actually asked about.
3. **Persistent memory with an explicit write rule** — you already have a file-based memory dir; his rule about "never both" is the useful bit.
4. **A deterministic installer + overlay split** — only matters once you have enough to keep in sync.

Where do you want to start? My recommendation is **#1 (the rules contract)**, because `dream` is only as useful as the rules it can propose edits to — without a real `AGENTS.md`, a retro loop has nothing to write into. But if the nightly loop is what's exciting you, we can build a simplified version of that first and let the rules accrete from it.

---

## **Ryan**

Okay first thing's first, I need to clarify that I'm not as smart as Joseph. There are a lot of words, terms, and phrases that I don't understand and some that you even mentioned that go over my head. I'll list a few of them and I need your help to understand first what you meant and what Joseph means by a lot of things as we progress.

Here's a list of things from your message just now that I don't get:

* "landed zero of them" - specifically the "landed" term
* "prefilter", "recall"
* "launchd"
* "over-collects"
* "nomic-embed-text"
* "Ollama + arithmetic"
* "conflict badges"
* "recurrence x blast radius"
* "repairs symlinks, and refuses to clobber real files"
* "canonical rules"
* "knowledge graphs"
* "operating contract"
* "evidence ladder memory→tools→web→recall"
* "skills-and-MCP-before-ad-hoc-scripts, CLI-first tool hierarchy, an explicit delegation threshold"
* "Given/When/Then verification proportional to risk, "laziest correct solution" bias"
* The whole bullet point about r2-sdlc
* Might do me some good to explain the whole Terminal layer section more
* "Provenance" and its entire section, too


Use this list to start and do two things: explain each bullet point in a way I'll understand, and rewrite your message using this simplified language, then we'll proceed with this in mind.

Also, explain to me the value of an AGENTS.md file, and I'd like to work together with you to use Hermes Agent as my main agent with Honcho as one of my memory providers and probably Notion (which I already have), Obsidian, or Mem to store memory per project or subject. I know this is a lot, take inspiration from Joseph's AI setup to save our place in a folder within this ai root folder I created for you.

---

> _tool: Bash_ Inspect the ai root folder

## **Claude**

Empty folder — clean slate. Let me set up a place to keep our notes, then walk through everything.

---

> _tool: Bash_ Create setup folder and glossary

> _tool: Bash_ Write plain-language tour notes and decisions log

## **Claude**

Nothing here is about smarts — it's vocabulary. Joseph has been living in this for months and named things for himself; I then repeated his names back to you as if they were common knowledge. That's on me. I've saved a glossary to [glossary.md](notes/glossary.md) so we can keep adding to it.

---

# Part 1: What those terms mean

**"landed zero of them"** — "Landed" is slang for "actually got merged into the real thing." He wrote 89 suggestions; none were ever accepted. They existed as documents and died as documents.

**"prefilter"** — A cheap first pass that throws out obvious junk before the expensive step runs. Like skimming envelopes for anything that might be a bill before opening any of them.

**"recall"** — A score. *Of all the real problems that existed, what fraction did the filter actually catch?* 20% recall means it found 1 in 5 and missed the rest. (Unrelated to "recall" meaning "remember" — confusingly, that meaning shows up later in the evidence ladder.)

**"over-collects"** — Deliberately grabbing too much in that first pass, on purpose. His old filter was picky and missed 60 of 91 real problems. The new one is greedy: hand 500 maybes to the next step rather than miss 60 real ones. Cheap step is generous, expensive step is picky.

**"launchd"** — macOS's built-in "run this thing at this time" scheduler. The Mac equivalent of a cron job. It's what makes `dream` run nightly whether or not anyone remembers.

**"nomic-embed-text"** — The name of one small AI model. Its job: turn a sentence into a list of numbers, arranged so that sentences meaning similar things get similar numbers. It runs on your own Mac, free.

**"Ollama + arithmetic"** — **Ollama** is an app that runs AI models locally on your Mac instead of calling a cloud API — free, private, weaker than Claude, fine for grunt work. Once your sentences are lists of numbers, finding which ones are similar is just comparing numbers. Plain math. So grouping "these 14 complaints are the same problem" costs him nothing.

**"conflict badges"** — A warning label on a proposal: *this edit overlaps with that other edit, you can't cleanly accept both.*

**"recurrence × blast radius"** — The ranking formula. *Recurrence* = how many times this problem happened. *Blast radius* = how much damage when it does. Multiply, sort, fix the top. A tiny annoyance that happens 40 times can outrank a disaster that happened once.

**"repairs symlinks, refuses to clobber real files"** — A **symlink** is a shortcut/pointer. His live config folder isn't a real folder, it's a pointer at the repo — so editing the repo instantly changes the live setup, no copying, no drift. *Repairs* = if a pointer is broken or aimed wrong, the installer silently re-aims it. *Clobber* = overwrite and destroy; if the installer finds a real file where it wanted to put a pointer, it stops rather than delete your work.

**"canonical rules"** — The one official copy everything else defers to. He has one rules file, and the second filename is just a pointer at the same bytes, so the two can't drift apart and disagree.

**"knowledge graphs"** — Notes stored as things-and-connections ("Ryan → works on → Project X → uses → Postgres") instead of paragraphs. Lets you ask "what else touches Postgres?" and get an answer.

**"operating contract"** — Rules about *how the agent should behave and decide*, not about code formatting. Not "use tabs not spaces" but "check your notes before you guess." Contract because it's binding, not advisory.

**"evidence ladder: memory → tools → web → recall"** — Where to look for a fact, in order of trustworthiness. **1. Memory** — your own saved notes. **2. Tools** — something that reads live reality (query the database, check the API). **3. Web/docs.** **4. Recall** — the model just remembering. Last, because that's where confident wrong answers come from. The point is to climb *down* the ladder only when forced.

**"skills-and-MCP-before-ad-hoc-scripts"** — *Ad-hoc* = invented on the spot for one use, then thrown away. *Skill* = a saved procedure. *MCP* = the standard plug for connecting an AI to an outside service (Notion, Jira, a database); an "MCP server" is one such plug. Rule: before writing a throwaway script, check whether a saved skill or a connected tool already does it — otherwise you re-solve the same problem forever.

**"CLI-first tool hierarchy"** — Preference order for *how* to touch a system: 1) command line, 2) browser automation if it's genuinely a web thing, 3) clicking the real desktop UI, 4) anything else. Earlier is faster, more reliable, and easier to verify.

**"explicit delegation threshold"** — A stated rule for when to hand work to a second agent. Broad independent research producing lots of output → delegate. One specific fact you need for your very next edit → just look it up yourself, because handing it off costs more than it saves.

**"Given/When/Then verification proportional to risk"** — *Given/When/Then* is a way to write a test as a sentence: **Given** a starting state, **When** I do X, **Then** Y should happen. It forces a checkable claim instead of "looks good to me." *Proportional to risk* = match the checking to the stakes — a typo fix gets a glance, payment code gets real tests.

**"laziest correct solution bias"** — A deliberate default: the smallest change that genuinely works. Reuse what's already in the codebase, use built-in features, don't invent a framework. "Laziest" is a compliment here; **correct** is doing the load-bearing work.

**"r2-sdlc"** — SDLC = Software Development Life Cycle, the path from idea to shipped code. "r2" is just Joseph's initials-ish tag on his own version. It's a saved 5-step recipe: understand the request → design → build test-first → simplify → review. `r2-gauntlet` is the companion: he has ~14 specialist reviewer agents (one for tests, one for architecture, one for performance...), and rather than read 14 reports, the gauntlet runs them all and merges the findings into one ranked list.

**The Terminal layer** — Context: he runs *many* Claude sessions simultaneously, so his terminal is built to manage a crowd, not a conversation.
- **Ghostty** — his terminal app. Configured by editing a text file rather than clicking preferences, so the config can live in version control like everything else. He re-enabled the alert sound so an agent can ping him.
- **herdr** — splits one window into many panes, one agent per pane. The special part: it *knows what an agent is*, tracking each as idle / working / blocked / done, with a sidebar showing which one needs him. Replaces scanning panes hoping to spot one waiting.
- **Four homegrown add-ons** wired to **hooks** (a hook = "automatically run this script whenever X happens"): every image an agent touches gets saved and is browsable with a hotkey; a hotkey opens a new tab already running Claude in the current folder; when a session ends a local model reads it and renames the tab something meaningful instead of "7"; and a tracker answering "which of my sessions are stuck?"

**"Provenance"** — Literally "where a thing came from." In his setup it covers three worries:
1. **Layering** — he has a public repo, a private work repo, and a machine-local one. Separate repos, one installer, all merging into one live folder. Company secrets can never accidentally end up in the public one.
2. **Publishing on purpose** — his real day-to-day history lives on a private branch. The public branch gets rebuilt each release as a *single fresh commit with no history*, so nothing leaks via old commits. A guard script blocks any push that would violate that. Publishing is a decision, never a side effect.
3. **Self-repair** — that's `dream`.

---

# Part 2: The same message, rewritten

Full version saved at [joseph-setup-plain.md](notes/joseph-setup-plain.md). The short form:

### The self-improving loop you asked about

His first attempt, **`retro`**, is dead. You ran it by hand at the end of a chat and it wrote up what went wrong. It failed four ways: you had to remember to run it (eventually nobody did — it sat dead 3½ months); each report covered one chat only, so nothing added up and no problem ever looked big enough to fix; it produced 89 suggestions and none were merged; and its filter for "what counts as a problem" caught only a fifth to two-thirds of real problems and never read the AI's own messages at all.

**`dream`** replaced it with those four fixes: it runs automatically on a nightly timer; it looks at *all* chats at once so a mistake made 14 times is visibly worth fixing; its first pass grabs too much on purpose (every real user message, plus every AI sentence that sounds like an admission — "I assumed…", "I missed…") and then one Claude call picks what matters; and it groups similar complaints by free local math instead of more AI calls.

Each group becomes a branch of proposed edits you can read and accept or reject, flagged when two proposals collide. A dashboard sorts them by how-often × how-bad. **Nothing applies itself** — a human approves everything. And it nags you with "last run: N days ago" so it can never quietly die again.

### Everything else, briefly

**Foundation** — one public repo holds his rules, skills, and settings. An installer points his live config *at* that repo instead of copying, so editing the repo changes the live setup instantly. Safe to re-run, won't destroy your files. One rules file serves both his AI tools.

**The leverage** — Memory lives outside any single AI tool so both his agents read the same notes, with a hard rule: search notes before guessing, and if the work is already producing a document put the knowledge *there*, otherwise in memory, **never both**. Small free local models do the cheap grunt work. His rules file governs behavior rather than style. And he has one saved recipe for going from idea to finished code, plus a way to run 14 reviewers and get one merged report.

**Terminal** — a setup for herding many simultaneous agents: a config-file-driven terminal, a pane manager that tracks each agent's status, and four small scripts that fire automatically on events.

**Ownership and upkeep** — public/private/local repos kept strictly separate; publishing to the public one is a deliberate act with a guard script enforcing it; and `dream` keeps the whole thing honest.

**Worth stealing, in order:** (1) a rules file, (2) a `dream`-style loop, (3) memory with a clear write rule, (4) an installer — only once you have enough to keep in sync.

---

# Part 3: Why an AGENTS.md is worth having

Here's the problem it solves. Right now, every time you start a chat with me, I show up with zero knowledge of how you like to work. You correct me — "don't add comments everywhere," "check the docs before you assert that," "ask me before you touch the database." Then the chat ends, and the next one starts from zero. You re-teach the same lessons forever.

`AGENTS.md` is a plain text file of those lessons that gets loaded into every session automatically. That's it. It's not magic — it's just the difference between correcting someone daily and writing it down once.

Four specific things it buys you:

1. **Corrections become permanent.** Say it once, it holds.
2. **It's the thing a `dream` loop writes into.** This is why I recommended starting here. A nightly improvement loop with no rules file has nowhere to put what it learns. Rules file first, loop second — the loop is the machine that grows the file.
3. **It's readable and editable by you.** Not a hidden setting. You can open it, disagree with a line, and delete it.
4. **It's portable.** `AGENTS.md` is becoming a cross-tool convention — Claude Code, Codex, and others read it. That's why Joseph keeps one file with a second name pointing at it: switch tools, keep your rules.

The trap to avoid: people write a 400-line manifesto of things that sound wise, and the agent follows none of it. Good rules are specific, few, and earned — each one traceable to a real time something went wrong. Which, again, is exactly what the nightly loop is for.

---

# Part 4: Before we build — two things I need from you

I've logged your wants in [decisions.md](notes/decisions.md). But I don't want to build on a guess:

**"Hermes Agent"** — I'm not confident what this is. I know Nous Research's *Hermes* family of open models, but "Hermes Agent" as a *main agent* to replace Claude Code sounds like it might be a specific product I'm not certain about. Can you send a link or tell me where you came across it? If I guess wrong here, everything downstream is wrong.

**Honcho** — I believe you mean Plastic Labs' Honcho, a memory/personalization service that builds a model of the user across sessions. Two questions: is that the one, and do you have an account yet?

And one design question we should settle early, because it prevents a mess later — **you named four possible memory stores** (Honcho, Notion, Obsidian, Mem). Joseph's whole memory rule is *never put the same knowledge in two places*, because then neither is trustworthy. So rather than wiring up four, tell me what you actually want each for. My instinct, subject to your correction:

- **Honcho** — facts about *you*: how you work, preferences, corrections. The stuff that would otherwise live in `AGENTS.md`.
- **Obsidian** — per-project working notes, because it's local plain-text markdown and I can read and write it directly with no API.
- **Notion** — the things other humans need to see.
- **Mem** — I'd cut it unless you're already using it and love it; it overlaps heavily with the other three.

Answer those and we'll start building — I'd begin with the `AGENTS.md`, drafted from how you actually work rather than from a template.

---

## **Ryan**

Before we do anything, let's save all of our work here so I can continue on another computer. I created a GitHub repository at github.com/rjrjensen/atelier. I want to save this entire chat history, too. Can you save all of this there?

---

> _tool: Bash_ Locate session transcript and check GitHub auth

## **Claude**

The transcript is on disk and you're logged into GitHub. I'll convert the raw session log into a readable markdown transcript (keeping the raw file too), then init the repo and push.

---

> _tool: Bash_ Export session log to readable markdown
