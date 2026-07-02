+++
title = "Hiring for AI Leverage, Not Lines of Code"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2026-06-02"
categories = [
    "AI",
]
menu = "main"
+++

### "Can you reverse a linked list on the whiteboard?"

In 2026, that question is dead. I helped design a hiring loop for a senior+ engineering role recently, and the first thing we threw out was the coding puzzle.

Here is the uncomfortable truth nobody wants to say out loud: AI wrote 80% of the production code I shipped last quarter. Not the boilerplate. Real features. So why am I still grading candidates on whether they can hand-write a binary search faster than the person before them? That signal stopped mapping to job performance a while ago.

The question that actually matters now is different. Not "can you write the feature?" but "can you build the system that lets AI keep writing the features after you've moved on?"

### What we were really testing for

The role was a senior IC, the kind of person who sets the technical direction for a whole product line. For that level, the bottleneck is not typing speed. It never was. It's judgment, leverage, and the ability to multiply everyone else (including the machines).

So we stopped asking the candidate to be the smart person in the room. We asked them to build the thing that makes a dumber room smart.

We called it AI leverage. Can you stand up the scaffolding so that an AI agent, six months from now, can extend this product without you holding its hand?

### The format: a small slice, one day

Here's the take-home we landed on. Build a small slice of a realistic domain app. The candidate picks the module. Inventory, scheduling, billing, whatever they want, as long as it's a believable piece of a real product.

But the app is just the gate. It's pass/fail, and the bar is low. Does it run, is it not garbage, fine, move on.

Two hard requirements, because they're cheap to state and they quietly filter out the hand-wavers: the slice has to be **deployed to a free tier** (Vercel, Supabase, a free-tier EC2, whatever) and reachable at a URL, and it needs **basic auth**. If you can't get your own small slice deployed and gated, the scaffolding conversation is moot.

The app is not what we grade. We grade the AI-engineering scaffolding around it:

1. **AI rules and prompts.** A real agent instruction file (a good `CLAUDE.md`, or whatever your tool uses). Does it encode the conventions, the gotchas, the architecture, so an agent doesn't reinvent the wheel every prompt?
2. **Reusable skills and subagents.** Did they factor out the repeated work into things an agent can call? Or is it one giant prompt and a prayer?
3. **Day-2 agents.** Feature generation, bug triage, ops. The stuff a team actually lives with after the MVP ships.
4. **Verification and eval loops.** How does the system know the AI did the right thing? Tests, evals, guardrails. This is the part that separates the people who have actually run AI in production from the people who watched a demo.

The MVP takes a couple hours. The scaffolding is where a senior engineer spends their day, and it's where the real signal lives.

### The part everyone is quietly panicking about

Take-home challenges are broken. Everybody knows it. You hand someone a problem, they hand it to an AI, the AI hands you a polished solution, and you learn nothing about the human. The whole industry is pretending this isn't happening.

So we added a live day-2 twist.

At the presentation, the candidate walks us through what they built. Then we hand them an unseen task. Add a status field. Fix this bug. Extend the UI to do X. And we watch them do it, live, in front of us, using their own scaffolding.

This does two things at once, and both are huge.

First, it kills the cheating problem dead. It does not matter who or what wrote the take-home. If the scaffolding is genuinely good, the candidate will fly through the live task because their own system carries them. If they faked it, they freeze. You can see it in real time. No more guessing whether the polished PDF was actually theirs.

Second, it proves the system delivers, not just demos. A lot of AI tooling looks incredible on the happy path and falls apart the moment reality deviates from the script. The day-2 task is reality deviating from the script. If their `CLAUDE.md` and their subagents and their eval loop actually make the unseen change easy, that is the entire job, demonstrated live.

It's the best forty-five minutes of signal I've gotten from an interview in years.

### The rubric (and we hand it over up front)

No mind games. Candidates get the rubric before they start. Here's the weighting we used:

```
AI leverage & artifact quality   45%
Day-2 extensibility (live)       25%
Engineering judgment             20%
Communication                    10%
```

The biggest slice is the scaffolding. The second biggest is whether it actually held up live. Together that's 70% of the score riding on "did you build levers that work," which is exactly where we wanted it.

Sharing the rubric up front matters for more than fairness. It tells you something. A candidate who reads "45% AI leverage" and still spends the whole day polishing the app instead of the scaffolding has just answered a question you didn't even have to ask.

And everyone gets a comparable-but-fresh day-2 task. Same difficulty, different specifics, so nobody can pass the question to the next candidate in the hallway. Fair and bias-resistant, by design. On our side there's a matching interviewer guide: 1-to-5 scoring anchors for each rubric line so two interviewers land in roughly the same place, and a small bank of day-2 tasks calibrated to the same difficulty.

### The takeaway, if you're designing interviews

Stop grading the artifact a senior engineer produces by hand. In 2026, that's the cheapest thing they make.

Grade the leverage. Can they build the system that keeps producing after they leave the room? Can they make a fresh, unseen change land in front of you using nothing but the scaffolding they built? That's the job now. That's the whole job.

The whiteboard binary search told you someone crammed for the interview. The live day-2 task tells you whether they can actually make AI carry a product forward. One of those is worth your time.

Redesign your loop accordingly. You're welcome.
