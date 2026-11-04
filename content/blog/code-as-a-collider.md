---
title: "Code as a Collider"
date: 2026-11-04
description: "The code we read is what survived. Its meaning lives in what didn't."
---

Agents now read our code as if it contained the whole story, and write new code by retrofitting a theory onto it.

The code we observe is not a random sample, though. It is the result of a selection process with many layers, and at each one several competing causes decide what survives. In causal inference, a variable with several causes is called a collider. Look only at the survivors and you are conditioning on it. Causes that had nothing to do with each other suddenly appear correlated. That is collider bias.

Your PM comes to you (a dev) with a sky-high vision. Nobody really knows yet what it means, possibly not even the PM. In the old days (like last year), that vision would have been refined and negotiated over the course of multiple sprints. During those sprints, many layers of code would have been added, and some of them peeled off again. That is the first layer of selection.

Once PM and dev agreed on a candidate, it would have been released to a test audience and compared against our baselines in an A/B test. That is the second layer. Again, one can easily imagine the first such iteration not surviving contact with reality, so the code would have been modified once more. Out of urgency and release pressure, ugly rule-based layers would be dropped onto an initially cleanly designed code base. Eventually something gets released that the PM can still somewhat sell to the higher-ups as vaguely resembling the initial vision, while the devs are not totally frustrated by the design and code quality. That compromise is the third layer.

The final product was specified actively, while being iterated on. And the reasoning for why it turned out a certain way is not in the final artifact but in the negatives: the code that could have been written but wasn't, and the reasons why not. Without the negatives, we cannot tell our ugly rule-based layer apart from either sloppy engineering or an essential part of our product produced under pressure. Neither can we know whether a passing unit test passes because it protects an important invariant or just because nothing important ever puts it to the test.

All of those negatives usually live only as implicit knowledge, shared within a team of individuals. Naur would call it the theory of the program.[^naur]

In our shiny new agentic age, agents don't know about the code that could have been written but wasn't, or the reasoning for why. That is fine for impressive-looking translations from language A to language B, even down to assembly, or for tasks that can be specified with mathematical precision. The messy kind of product discovery described above looks difficult to me, though. Agents do iterate, and a long-running loop produces plenty of negatives. They just happen where nobody can see them, and nobody carries them into the next session. From the outside it looks like waterfall to us.

We stopped doing waterfall, though.

[^naur]: Peter Naur, [“Programming as Theory Building”](https://doi.org/10.1016/0165-6074(85)90032-8), *Microprocessing and Microprogramming* 15 (1985), pp. 253–261. For what this means for coding agents, see [Death by Agent](/blog/2026/10/death-by-agent/).
