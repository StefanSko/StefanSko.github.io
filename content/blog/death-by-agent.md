---
title: "Death by Agent"
date: 2026-10-25
---

Math is being run over by AI. Barely a week goes by without any major publication of an LLM having solved some century-old mathematical puzzle.

And as a software engineer (like me!) you would have been living under a rock if you weren't exposed to the following catchphrase:


> "Coding is practically solved ..."
>
> — Boris Cherny, [“Inside Claude Code With Its Creator Boris Cherny,” Lightcone](https://youtu.be/PQU9o_5rHC4?t=2692)


Now, of course there is some clear association between math and computer science.
And while there are some great agentic breakthroughs in coding as well, one cannot really be completely without doubt. Reviewless workflows without a human in the loop are not a thing yet.
And since code can be created on the press of a button, that means we are reviewing all the time, lines after lines making sure no slop comes through.
A sisyphean task! (Because Slop DOES come through!)

So what's different?

For the mathematical breakthroughs to appear like this at scale, the proofs increasingly get encoded in a computer program. So far so similar.

In math we have a machine checking the proof against fixed propositions. For coding to be considered solved in the same sense we would need to prove our program against the right proposition. Usually in form of a specification. Potentially laid out concretely in a set of tests the model can develop and check against.
That raises the question whether we can get there that easily though. Now I personally think we have two problems (at least)! Both are orthogonal to any Reinforcement Learning schemes or goals you can hillclimb your model against!

## Problem 1: the languages we use

Philip Wadler argues that if we ever encounter aliens, and they know about logic and propositions, then they will likely also know about lambda calculus. [^wadler]
This is based on a proven direct correspondence between math and programming. The so-called Curry-Howard isomorphism:

- logical propositions (e.g. ‘this list is sorted’) correspond to types
- proofs correspond to programs
- simplifying a proof corresponds to evaluating a program

But, it is a correspondence between logic and... yeah: Lambda Calculus (not just that but a typed version, because types are actually logical propositions..., well, ok?). That is clearly academic territory. You can go along in your programming career just fine without ever getting in touch with those concepts. 
Not without surprise, the purest version of this you'll find at the core of such languages like Lean that as we have seen turn out to be great for proving mathematical theorems in our new agentic age. The types in those languages enforce much stronger promises, helping to actually DO let the LLM hill-climb to an eventual proof.

But that's not usually what *our* programs provide!

Consider the following crude implementation of a Priority-Queue in Python:

```python
from bisect import insort


class PriorityQueue:
    def __init__(self) -> None:
        self._items: list[int] = []

    def push(self, item: int) -> None:
        insort(self._items, item)

    def items(self) -> list[int]:
        return list(self._items)
```

Now consider a client using this library having the following derived helper to return the top 3 elements according to their priority:

```python
def top3(pq: PriorityQueue) -> list[int]:
    return pq.items()[:3]
```

With the implementation in mind we can call `items()` happily and be sure it is always sorted.  

A few days later a senior dev comes around the corner and thinks to optimize the thing by changing the implementation to a heap. This is a sensible thing to do. `insort(...)` is O(n)! God forbid! 

```python
import heapq

def push(self, item: int) -> None:
    heapq.heappush(self._items, item)
```

Now, however, you see a few tests breaking along the way:
The list is not guaranteed to be sorted anymore. So you put a `sorted(...)` around `_items` and return it:

```python
def items(self) -> list[int]:
    return sorted(self._items)
```

Everything green again, phew! Crisis averted!

A few days later a staff dev comes around and reminisces about his Java days and the API documentation of a PriorityQueue:

> Returns an iterator over the elements in this queue. The iterator does not return the elements in any particular order.
>
> — [Java `PriorityQueue.iterator()` API documentation](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/PriorityQueue.html#iterator())

So, why should it be sorted (in fact it increased complexity unwillingly)? We can remove the sorted, no? He removes the test as overspecified citing the Java doc. And so it goes, etc. pp. 
No step in itself was wrong though. They were locally defensible. In the meantime our consumer having written `top3(...)`, relied upon sortedness. So changing it, broke their code downstream!

The code could have been clearer from the beginning:

```python
from typing import NewType

SortedInts = NewType("SortedInts", list[int])

def items(self) -> SortedInts:
    ...
```

They could have gotten around the fact there is no native Sorted-like type in Python by just defining one.
That would have realistically reduced the chances of someone breaking it effectively. But it is only a label. Nothing checks that `push()` actually keeps it sorted.

It does not really represent a real proposition in the logical sense that a language like Lean can check against and prove we have a sorted list, like:

```lean
{ l : List Int // l.Sorted (· ≤ ·) }
```

So unlike for math, for the most part we don't really use languages that allow for the sound and crisp definition of a proposition. 

But, even if we all wrote Lean tomorrow, we’d still have to know what to prove.
Which brings us to Problem 2!

## Problem 2: the world we live in

This whole thing is basically Hyrum's Law and ofc there is a great xkcd for that:

[![xkcd: Workflow](https://imgs.xkcd.com/comics/workflow.png)](https://xkcd.com/1172/)

When CPython got a new and efficient dict implementation in 3.6 suddenly insertion order was preserved. It was touted as an implementation detail people should not rely upon!
Fast forward: People relied...! It got noticed, got discussed, and got turned into a promise: from 3.7 onwards, insertion order is officially part of the language spec.[^dict]

This is zooming in on a broader and philosophical question that Peter Naur asks in his paper "Programming as Theory Building"[^naur] from 1985: can the act of programming be characterized by just typing out the text that corresponds to the source code? 
Naur emphatically argues against that! He is quite explicit and visual by referring to programs lacking an underlying theory held by the programmers of how the code relates to the real world problem it intends to solve as: *dead*. For a library holding our Priority Queue that also implies knowing about your clients in the real world.

The state of events I described of multiple devs jumping in, changing around those implementation details, and potentially breaking `top3(...)` downstream would have signalled something to our dev team. It would have revealed a gap in knowledge about how this method is supposed to relate to the real world where it is actually used. 3 people held three different ideas; 3 distinct theories.

Optimally, as a result, all of those devs would have gathered to build up a common understanding on which invariant would make most sense. To do this in a sensible way, there would have to be some notion of where this existing method is used. Who relies on which invariant and why? They would have documented it. Now, such a problem could have been marked as avoidable in the future. 

The program would be back to creating useful outputs for our client library again. Phew! 

Naur nevertheless would have deemed it as near dead already. 

According to him we see a loss of theory routinely during handover of code from Team A to Team B with no dev being around any longer holding the original theory.
A common argument in such a situation: the documentation sucks! While a good documentation undoubtedly is helpful, Naur argues it is by far not sufficient.

Raw code and the docs are just a statement about what is! Both lack the whole selection process lingering behind the surviving pieces that we observe: 
- a test surviving just because it never failed; or because it is actually crucial? 
- an ugly piece of code surviving because it is incomprehensible and thus feared; or because it was the result of an urgent fix holding everything together?
- clean modular code that is well designed; or just rarely touched because it sits in a peripheral part of the program?
The code alone cannot disclose the reasoning. Rarely do we see it in docs either!

For considering the program to be well and alive, the team would be required to show the ability to constructively respond to a change of requirements coming from the real world. They would need to relate every new requirement to the existing code and establish a sense of similarity: where it belongs and how much would have to change!

Whether they used docs or pinned the promise down by using NewType, they merely documented one proposition. The list of propositions potentially entering the system is infinite though. And holding the theory is being able to account for that by being ready for the next one.

When comparing with the rigidity enforced by mathematical propositions, the languages we code in, as well as the outside world we are modelling seem much looser. Like with a good book the program we see is just the tip of the iceberg. All characters might have a life on their own still in the author's head. And we can't know from just reading the text.

## The agents that kill

Now enter our agentic age: manual code edits seem to be becoming a thing of the past. Agents do it for us! To the point that people argue exactly that: why look at the code at all? Let's move up the abstraction ladder already!

And to their credit, those agents have gotten impressively good! The probability of an obvious blunder; a dysfunctional code edit is small. This in and of itself is an astonishing achievement. Who would have thought this possible just 2 years ago? But our two problems still don't go away!

Problem 1:
Moving up the abstraction ladder implies trusting interfaces and types. But in the languages we use many interfaces can be consistent with our implementations.[^koppel] Even sth like `SortedInts` is just a label. They may as easily change it back the way our staff dev did. That being said, it will certainly help our agents. Maybe not a coincidence that languages with a richer type system like Rust, while being performant at the same time are experiencing a surge? 

Problem 2:
This one might actually get worse in our new day and age.
As we have seen from Hyrum’s law, parts of the code’s meaning are decided outside of it. Yet what the agents usually work with comes almost entirely from within: the code, the tests (often written by the same agent within the same session), a prompt, a design doc.
Now you could say: sure, but the agent can grep! If top3(...) lived in the same repo, the agent could find it and infer the invariant. But it doesn’t: it lives in a client library. No amount of grepping gets you there!
Prompts and design docs surely will help. And there are some great attempts that try to tackle these shortcomings by providing guidance on how to inject design/architecture/product information into the agent’s context.[^horthy] But they are still exactly the written-down artifact that Naur argued is insufficient.
We are still missing all programs the agents could have written but did not... and the reasons for why they didn’t.

The questions from before: the test that never failed, the ugly piece of code, the cleanly designed but non-critical module? An agent has no one to resolve those questions with, yet does so anyway: Imputing from all the code the model saw during training!

If theory cannot be encoded in written artifacts, then each change of session / model iteration / thinking levels is akin to a Handover from Team A to Team B. The very situation that Naur defined as the death of a program. In fact even within a session we have no real grasp whether internally the agents ever form any theory about the program it writes. Or whether a continuation might treat a previous promise fitting a certain theory as incidental and change it around on a whim (or vice versa).

Another parallel development: speed vs quantity. In our priority queue saga we have one fatal edit at human speed. A long running agent can have numerous locally defensible edits to the code within one afternoon! That is a whole different blast radius.

Some evidence:
The SlopCodeBench authors measure how code quality develops over time in code bases authored by agents. They report:

> Quality degrades across checkpoints, with structural erosion rising in 77% of trajectories and verbosity in 75.5%.
>
> — Gabriel Orlanski et al., *SlopCodeBench*[^slopcodebench]

Another finding: explicit quality guidance made the initial code cleaner, but did not slow degradation of the code. Naur wouldn't be surprised!
A thousand little deaths of our programs! Every time we let them go loose. Pretty much predicted from a theory of 1985!

Now after all this: Is Coding really solved?
We have breakthroughs in math by letting agents go after fixed logical propositions.
Whether `items()` should be sorted is a proposition no one wrote down, though. It depends on who uses `top3()` and why. It can change over time. An innocent looking implementation detail leaking out into the real world through all carefully crafted abstraction details. So "stop looking at the code" might not be the end game. Because of the languages we use; because of the world we live in. Even though everything might get transformed in this day and age: we gonna need to stay creative (I am sure a similar argument can be made for math!). That's good news!

[^naur]: Peter Naur, [“Programming as Theory Building”](https://doi.org/10.1016/0165-6074(85)90032-8), *Microprocessing and Microprogramming* 15 (1985), pp. 253–261.

[^slopcodebench]: Gabriel Orlanski et al., [“SlopCodeBench: Benchmarking How Coding Agents Degrade Over Long-Horizon Iterative Tasks”](https://arxiv.org/abs/2603.24755v2), first submitted March 25, 2026; quotation from the abstract of the May 7 revision (v2).

[^koppel]: Jimmy Koppel, “‘Modules Matter Most’ for the Masses”, *Path-Sensitive* (blog), March 30, 2023. Koppel points out that the relation between implementations and interfaces is many-to-many: an implementation might satisfy many possible interfaces, so the intended one can’t be recovered from the code alone.

[^wadler]: Philip Wadler, ["Propositions as Types"](https://doi.org/10.1145/2699407), *Communications of the ACM* 58, no. 12 (2015).

[^dict]: The [Python 3.6 release notes](https://docs.python.org/3/whatsnew/3.6.html#new-dict-implementation), and the [python-dev mailing list](https://mail.python.org/pipermail/python-dev/2017-December/151283.html).

[^horthy]: Dex Horthy, [“Why Software Factories Fail”](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/wsff.md) (HumanLayer). Orthogonal to Naur, Horthy’s take is less about theory and more about incentives. Models are trained with RL against verifiers, and tests give a fast pass/fail signal. His advice: keep humans in the loop before the code gets written: a product artifact, a system architecture one, a program design which sketches types, signatures, and call stacks, etc.
