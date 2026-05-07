---
layout: post
title: "Stop Paying the Review Tax"
date: 2026-05-06
categories: [ai, workflow]
---

{% comment %}
Source draft: writing/drafts/versions/review-tax.v4.md
{% endcomment %}

Most people think the problem with AI coding is that models make mistakes. That’s not the real problem. The real problem is that the model is fast and trust is slow, so you “save time” generating code and then you pay it all back reviewing, debugging, and cleaning up. The faster the model gets, the worse that tax becomes, because the amount of output grows faster than your ability to confidently say “this is correct.” People sometimes call that payback the Review Tax.

I hit a wall with this for a simple reason: I don’t read code. I’m not saying that to be cute, and I’m not using it as a brand. It’s just a constraint. If “read every line” is the only trust mechanism, then I’m stuck. So I had to find a different control surface—one that lets me move fast without pretending I can verify by scanning files.

Here’s the shift: I stopped trying to trust the output, and I started trying to trust the process. That sounds abstract, but it becomes very concrete once you decide what “correct” means before the agent writes anything, and once you require evidence at the end that the system still behaves the way you intended.

## The moment it clicked

I recently ran what would normally be a multi-day sprint for a professional developer. The model itself estimated 6–12 days. I got through it in a few hours. This shift helped me move roughly 20x faster. If I tried to “be responsible” by reading everything it produced, I would have erased the speedup immediately—not because I’m lazy, but because review doesn’t scale when the output is huge, and because most of the time you’re not just looking for bugs, you’re trying to reconstruct intent.

So I didn’t review the way people expect. I verified at a higher level. I stopped verifying the lines and started verifying the system.

That distinction matters because review is not one thing. It has levels:

- **Line-level review** asks: does this code look right?
- **Behavior-level review** asks: does it do what the user wanted?
- **Architecture-level review** asks: did it do that without damaging the system?

My point is not that line-level review is useless. It is that line-level review stops scaling as the primary control when AI can generate large changes quickly. The control has to move up: define the behavior, define what must not change, and require evidence that those things still hold.

## The default workflow doesn’t scale

Most AI coding workflows still look like: generate first, trust later. You ask for a change, the agent produces a lot of code, and then you try to regain trust by reading what it did. That made sense when output was small. It stops making sense when output is large, because now the bottleneck is not “writing,” it’s “audit.” And audits are expensive for a reason: you’re not just checking for mistakes; you’re reverse engineering decisions that weren’t written down anywhere.

## The framing that helped me

I use a boring analogy. A building inspector doesn’t certify a building by watching every hammer swing. They certify it by checking it against requirements: codes, permits, inspections, acceptance criteria. They still care about details; they just don’t create trust by staring at everything. They create trust by designing checks that catch the important failures.

That’s what I want in AI coding: a way to say “this change is acceptable” without turning my day into a manual audit.

## The workflow: decisions → invariants → gates

This is the simple version of what I do. Before the agent touches anything, I write a short decision memo. Engineers call these ADRs (Architecture Decision Records), but it’s just a record of what we’re doing, why we’re doing it, and what must not change. The point is not documentation for the sake of documentation; the point is to force clarity before implementation.

Then I translate that memo into a small list of invariants. An invariant is a rule that must stay true. For example: if `metadata.taskFamily` is missing, the selector must fail closed; no file I/O outside `/temp`; all external calls go through one adapter module. The reason to write these down is simple: they turn “correct” into something you can test.

Then I enforce gates that return pass/fail. Tests, assertions, static checks, permission boundaries, golden-file comparisons—anything is fine as long as the output is not “looks good.” It has to be evidence. If the gate fails, the work is not done. The loop becomes straightforward: fix the failure, re-run the gate, repeat until it passes.

That is how I get speed without pretending I can review code.

## The strongest objection (and it’s valid)

The strongest pushback is: “Most failures aren’t syntax. The code runs, but it doesn’t do what you want.” Yes. That is why I care about outcome-level checks. But there’s a catch: you can’t test what you didn’t define. If your decision memo is vague, your invariants will be vague, and you’ll drift faster. This workflow doesn’t remove thinking. It moves thinking earlier.

## Limits (especially if you can’t read code)

Invariants only catch what you wrote down. They won’t catch what you forgot. And some issues are hard to encode: performance regressions, security problems, long-term maintainability. So if you can’t read code, you need a policy for evidence. I ask the agent for artifacts I can actually verify: test output, invariant pass/fail reports, logs that demonstrate key flows, and a short “what changed and why” summary in plain English. I also keep changes small and scoped, I make risky rules fail closed, and if something is mission-critical I narrow the surface area until the AI can work in small, verifiable steps with stronger gates around each one. That’s not a failure; that’s governance.

This is also why I think the workflow is probably easier for a real programmer, not harder. A programmer can quickly identify the occasions where debugging is required. That makes it much easier to spot the truly tricky situations too—especially architectural drift, which is one of the hardest things for a non-coder to detect.

## What I changed after trying to explain this

The first time I tried to explain this idea, I used the wrong language. I used terms I picked up from AI conversations, and people heard buzzwords instead of logic. So here’s the clean version: this isn’t about trusting the model more. It’s about trusting the system. “Done” has to be explicit, and drift has to be detectable.

## The takeaway

AI coding is already fast. Trust is the bottleneck. If your only trust mechanism is reading code line-by-line, the Review Tax will eat your speed. If you define constraints up front and require evidence at the end, you can move faster without gambling.
