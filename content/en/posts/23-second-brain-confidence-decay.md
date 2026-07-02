+++
title = "My Second Brain Forgets on Purpose"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-06-16"
categories = [
    "AI",
]
menu = "main"
+++

### "Wait, did I actually decide that, or did I just write it down once and never check again?"

I hit that question about once a week. I work across a dozen projects, make a hundred small decisions, and dump them all into notes. Then six months later I open a note that confidently tells me something that stopped being true in March.

That's the real problem with notes. They don't lie on purpose. They just freeze. The fact was correct when you wrote it, nobody updated it, and now it sits there with the same crisp confidence it had on day one. Most note systems only know how to accumulate. They never forget, and they never argue with themselves.

So I built a personal knowledge base that does both. It forgets on purpose, and it keeps its own contradictions in the open. This is a personal experiment, not a product, but the idea has held up better than I expected.

### The unlock isn't "AI takes notes"

Everyone has tried pointing an AI at their notes. It works for a week. Then the agent spawns three slightly different pages about the same project, overwrites a decision you actually cared about, and starts hallucinating with total confidence because nothing ever pushed back on it.

The fix isn't a smarter model. It's a stricter contract. I give the agent a small set of rules it is not allowed to break, and the whole thing stops rotting.

### Three layers, all plain files

No database. Just Markdown, YAML frontmatter, and Git for versioning. Deliberately boring.

1. **`/sources`** is the immutable audit trail. Raw inputs go here and never get edited: meeting transcripts, exported boards, voice dumps, scratch notes. This is the ground truth. If a fact in the wiki is ever in doubt, I can trace it back to the exact file it came from.

2. **`/wiki`** is the agent-editable layer. One entity per page: a project, a person, a decision, a daily note. Each page carries frontmatter that makes it auditable.

3. **`WIKI.md`** is the contract. It's the rulebook the agent has to obey every time it touches the wiki. This file is the actual product. Everything else is just storage.

![Three-layer knowledge base: sources feed the wiki, WIKI.md governs it, and confidence decays from high to medium to low when a page goes unconfirmed](/images/23-decay-diagram.svg)

Here's what a wiki page header looks like:

```yaml
---
title: "Migration off the legacy DMS"
type: decision
last_confirmed: 2026-09-12
confidence: high
sources:
  - sources/2026-09-12-arch-review.md
  - sources/2026-08-30-voice-dump.md
---
```

That `confidence` field is the whole point, and I'll get to it.

### Rule one: compound, don't duplicate

When a new fact shows up, the agent has to find-or-create the matching entity page and add a dated bullet under `## Updates`. It is never allowed to spawn a near-duplicate page. So a project doesn't end up as "Project X", "Project X (notes)", and "ProjectX-final". It ends up as one page that grows a timeline.

```markdown
## Updates
- 2026-09-12: Decision confirmed in arch review. Going with the phased cutover.
- 2026-08-30: Leaning toward a phased cutover, still nervous about the data layer.
```

You can read a page top to bottom and watch the thinking evolve. That alone fixed half my note mess.

### Rule two: keep both sides of a contradiction

This is my favorite rule. When a new fact conflicts with an old one, the agent does not overwrite. It keeps both, with dates.

```markdown
## Updates
- 2026-09-12: Cutover is now phased over two quarters.
- 2026-06-01: Cutover planned as a single big-bang weekend.
```

Most systems would silently replace the old line and you'd never know it changed. I want the opposite. Radical transparency. When something contradicts what I believed three months ago, I want to see the seam, not a smooth lie. Half the time the contradiction itself is the interesting part.

### Rule three: forget on a schedule

Here's where it stops being a note app and starts feeling like memory.

A nightly job runs and does three things:

- **Archive stale pages.** Anything older than 45 days with no open issues gets moved to an archive folder. Out of the way, still in Git, still searchable, just not pretending to be current.
- **Flag broken wikilinks.** If a page points at something that got renamed or archived, it gets flagged instead of silently dangling.
- **Decay confidence.** Any page not re-confirmed in 21+ days drops a level: `high` becomes `medium`, `medium` becomes `low`.

That last one is the star. Think about how human memory actually works. You don't trust a six-month-old recollection the way you trust something from yesterday. You hedge. You say "I think it was around X." My knowledge base does the same thing now. A fact that nobody has touched in three weeks visibly gets less trustworthy, instead of sitting there falsely crisp.

So when I open a page and see `confidence: low, last_confirmed: 2026-04-02`, I read it completely differently. It's not "this is true." It's "this was true, go check." Stale facts age out in the open instead of quietly lying to me.

### The bit that wrote this post

There's one more workflow I didn't expect to love: voice capture. The agent samples my own phrasing from the raw sources into a small style file. So when it drafts something, it sounds like me, not a generic assistant that opens every paragraph with "In today's fast-paced world."

Which is the honest disclosure here: the first draft of this post came out of that exact pipeline. I fed it my voice file and my own notes about the system, it gave me something that sounded like me, and I edited from there. Slightly recursive. I'm fine with it.

### Why files, and why this is enough

People ask why I didn't reach for a real database or one of the fancy second-brain apps. Because the boring stack does everything I need. Markdown is readable in fifty years. YAML frontmatter is queryable with five lines of script. Git gives me a perfect audit trail for free, every change, every contradiction, every decay step, with a timestamp and a diff. The AI agents just run the auto-organize and auto-forget workflows on top. Nothing is locked in a proprietary box.

Is it perfect? No. The agent occasionally files something under the wrong entity, and I still have to skim the contradiction log. But it has stopped me from trusting at least three "facts" that had quietly gone wrong. That's a good trade.

The point was never to build an AI that takes notes. Plenty of those exist. The point was to give the AI a contract strict enough that the knowledge it keeps gets more honest over time, not less. Knowledge should get less trustworthy when nobody reconfirms it. Just like ours does.

If your notes have ever lied to you with a straight face, try making them forget. You're welcome.
