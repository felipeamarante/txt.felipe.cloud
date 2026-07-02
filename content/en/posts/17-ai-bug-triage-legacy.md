+++
title = "I Pointed an AI Agent at a 20-Year-Old Codebase and Told It to Triage Bugs"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-03-24"
categories = [
    "AI",
]
menu = "main"
+++

### "Nobody understands this code anymore. Can the AI?"

That was the actual question that started this. We have a dealer management system that is roughly twenty years old, written in PowerBuilder, with C#/.NET bolted on over the years and a SQL Server backend underneath. The people who wrote the original version are mostly gone. The documentation is a rumor.

Every senior engineer I know quietly dreads tickets that land on that system. So I did the obvious thing and pointed an AI coding agent at it.

Here is what I learned.

### Legacy code is where agents are weirdly the most useful

The hype says AI is great at greenfield projects. Spin up a fresh Next.js app, scaffold some CRUD, watch it fly. Sure. But that is also the place where you, the human, are fastest and happiest. You don't need much help there.

The place you actually need help is the place you hate: 200,000 lines of PowerBuilder that nobody has read since 2009.

And here is the thing. The agent does not care that it is PowerBuilder. It does not sigh when it opens a `.srw` file. It does not have twenty years of trauma associated with this codebase. It reads the code that exists, follows the calls across languages, and reports what it finds. The absence of documentation, which paralyzes a new human hire for weeks, barely slows it down, because it was never going to read the docs anyway. There are no docs.

That asymmetry is the whole story.

### The bug-triage agent

The first experiment was a triage pipeline. A ticket lands in Jira, the agent picks it up and runs six steps:

1. Read the ticket.
2. Decide if the ticket is even complete. (Spoiler: half of them are not.)
3. Check whether this was already fixed in a later version.
4. Search the codebase for the root cause.
5. Diagnose it and propose a concrete fix.
6. Report back to Jira.

![The six-step bug triage pipeline, with confidence-based routing](/images/17-triage-pipeline.svg)

It talks to Jira and to the code repositories through MCP (Model Context Protocol), so it is not screen-scraping anything. It reads tickets and code the same way a person with the right access would.

The part I am proudest of is step 2, which leads to the next point.

### Confidence-based routing, or "stop inventing fixes"

Every triage run ends with a confidence score from 1 to 5. It's really a measure of how far the agent actually got, and it decides what it's allowed to do next.

```
1 · Insufficient info   → push back: ask for repro steps, a version, a real error.
2 · Narrowed it down    → post a partial analysis; flag what's still unclear.
3 · Root cause found    → hand the diagnosis to a developer.
4 · Fix proposed        → a concrete fix, or a PR.
5 · Already fixed       → point at the version that resolved it, and close.
```

This one design choice is what made the thing usable instead of dangerous.

An LLM with no guardrails will confidently "fix" a ticket that says "sometimes the screen is weird." It will write a beautiful, plausible, completely fictional patch. Routing on confidence flips that default. When the ticket is garbage, the agent's job is to say so politely and bounce it back, not to hallucinate a hero fix.

A bot that pushes back on a vague ticket is doing real work. A bot that invents a fix for a vague ticket is creating future incidents.

### Round two: a deprecated system with no git remote

Encouraged, I tried the same idea on a second, older DMS. This one was built in WinDev on an Oracle backend, and it is officially deprecated, which made the constraints interesting:

- There is no upstream git remote. The "repository" is local code drops, tagged in git as they arrive. So there is nowhere to open a pull request.
- There were, again, no docs.

The pull request problem I solved by having the agent post its fix as a code snippet inside a Jira comment instead. Less elegant, but a reviewer gets the exact change to copy, with the reasoning attached. It also writes a JSONL audit log of every run, so when someone asks "why did the bot say that," there is a record.

For the missing docs, I hand-wrote a small module map and a French-to-English glossary (the codebase is bilingual in the worst possible way) and fed it in as context. That little file did more for accuracy than any amount of prompt tuning. The agent does not need perfect docs. It needs a map of the neighborhood.

### Codebase archaeology

The third experiment dropped the bug fixing entirely and just asked: can an agent document a system nobody understands?

I pointed one at 38 undocumented Java repositories. Twenty-plus years of tech debt, cryptic module names, cross-repo dependencies, zero documentation. I let it ingest the code plus a pile of external Word and Excel files that had accreted over the years.

It produced:

- A per-module "passport" document. One page per module: what it does, what it talks to, what touches it.
- A glossary decoding the cryptic short codes that are scattered everywhere.
- A system overview with a dependency graph (rendered in Mermaid, because of course).
- And a list of things it was not sure about.

That last bullet is the real unlock.

### The honesty is the product

A confident, wrong documentation file is worse than no documentation, because someone will trust it and act on it. The single most valuable behavior I got out of these experiments was the agent explicitly flagging its own uncertainty: "This module appears to handle invoicing, but I found two call paths I could not fully trace, and one short code I could not decode. Verify before relying on this."

That sentence is gold. It turns the agent from a confident liar into a useful junior who knows the limits of what they figured out. You can build on top of an honest "I'm not sure." You cannot build on top of a fluent lie.

### Be honest: these are POCs

I am not going to oversell this. These are proofs of concept, not magic. They make mistakes. They need a human reviewing the output, especially on the high-confidence fixes. The glossary and module map needed real human effort up front. And none of this replaces understanding your own systems.

But for the specific, miserable job of triaging bugs and mapping ancient code that no living human wants to touch, the agents earned their keep. They do the dreaded archaeology, flag what they are unsure about, and refuse to invent fixes for nonsense tickets.

For twenty-year-old PowerBuilder, that is more than I expected. Worth trying.
