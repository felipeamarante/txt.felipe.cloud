+++
title = "Ensemble Code Review: Many AI Reviewers, One Judge"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-05-05"
categories = [
    "DevTools",
]
menu = "main"
+++

### "Just have the AI review the PR" sounds great until you read the comments

Code review is the bottleneck nobody wants to admit to. It's slow, it's inconsistent, and the quality depends entirely on who happened to be free that afternoon. So the obvious move in 2026 is to throw an LLM at it.

I tried that. It was a disaster.

Point one model at a diff and ask it to review, and it will happily dump twenty comments. Half are nitpicks about formatting your linter already caught. A few are confidently wrong. One or two are actually useful, buried somewhere in the middle. Developers learn within a week that the bot cries wolf, and then they stop reading it entirely. Worse than useless: it's noise with a robot's confidence.

So I rebuilt the whole thing around a different idea.

### The problem isn't the reviewers. It's that nobody filters them.

Here's the thing about AI reviewers: more of them actually gives you better coverage. One model misses the SQL injection but catches the race condition. Another flags the missing null check. A security-focused persona sees things a general one glosses over. Diversity helps.

The catch is that diversity also multiplies the noise. Five reviewers means five times the nitpicks, five times the false positives, five chances to be confidently wrong. If you pipe all of that straight to the PR, you've made the spam problem worse, not better.

In a human team you'd never do this. You don't post every junior's raw notes to the author. Someone senior reads the feedback, throws out the wrong stuff, and says the one thing that matters.

That's the pattern. Many eyes, one judge.

### The design: fan out, then adjudicate

The system has two distinct phases, and the second one is where all the value lives.

**Phase one: fan out.** A roster (just a YAML file) defines which reviewers run on a given diff. Each one is a separate AI persona with its own focus: security, performance, correctness, style, whatever you want. They run as parallel async jobs, and each reviewer explores the actual repo independently. Not just the diff lines, the surrounding code, the imports, the call sites. Each one emits its findings as structured data, not prose. A finding has a file, a line, a severity, a short title, the rationale, and a concrete fix.

**Phase two: adjudicate.** This is the part that makes it work.

First, the raw findings get merged by vote. If three reviewers independently flag the same line, that's a strong signal. If one reviewer mentions something nobody else noticed, that's weaker, but it's kept as a candidate.

Then a single strong model, the Review Lead, takes over. I use Claude Opus for this one because the judge needs to be smarter than the reviewers. The Lead gets the merged candidate findings AND the real diff, and its only job is to adjudicate:

- Drop the false positives. The reviewer claimed a null pointer, but the variable is guarded three lines up. Gone.
- Drop the irrelevant. Correct, but it's about code the diff didn't touch. Gone.
- Rank what survives by how many reviewers flagged it, then severity, then how concrete the fix is.
- Write exactly ONE comment.

One review. One GitHub comment. The signal, none of the sludge.

![Ensemble review flow: a diff fans out to independent reviewers, votes merge, a judge drops false positives and emits one advisory comment](/images/20-ensemble-review.svg)

### Advisory only, on purpose

The bot never blocks a merge. Ever.

This is a deliberate trust choice, not a technical limitation. The moment an AI reviewer can block your PR, two bad things happen. People start gaming it, and people start resenting it. An advisory reviewer that's right 90% of the time is a gift. A blocking reviewer that's wrong 10% of the time is a daily tax that the whole team will route around within a month.

So it advises. The humans still decide. The bot earns trust by being useful, not by holding the merge button hostage.

### The boring parts that make it real

A toy runs in a notebook. Production has constraints, and the constraints are most of the work.

**Data residency.** Orchestration is Node.js on AWS Lambda, deployed with SAM. Every bit of inference runs on Bedrock in-region, in our case eu-west-1. The code and the diffs never leave the region. For a European company reviewing proprietary code, that's not a nice-to-have, it's the difference between "yes" and "legal said no."

**Least privilege.** Two separate tokens, never one. A read-only org token that can clone and read repos, and nothing else. A separate review-scoped token that can post the one comment, and nothing else. If either leaks, the blast radius is tiny.

**Webhook security.** Every incoming webhook gets its HMAC signature validated before anything runs. Access is membership-gated, so a random fork-and-PR from the internet doesn't kick off a review on your dime.

**An audit trail.** Every run logs its metrics (which reviewers ran, what they found, what the Lead kept or killed) to a dedicated branch. That log is gold. It's how you tell which reviewers are pulling their weight and which ones just generate noise, and it's how you tune the roster over time instead of guessing.

### What I'd tell you before you build your own

The instinct is to make the reviewers smarter. Resist it. The reviewers can be merely good. The leverage is in the judge.

A mediocre panel with a sharp arbiter beats a panel of geniuses with no filter, every time. The arbiter is what turns "twenty comments, half wrong" into "one comment, worth reading." Spend your best model, your best prompt, and your most careful evaluation there.

Many eyes find the bugs. One judge makes them worth listening to.

Worth building.
