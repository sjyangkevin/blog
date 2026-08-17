---
date: '2026-08-16T22:28:48-04:00'
draft: false
title: 'When Code Became Cheap, Engineering Became the Hard Part'
summary: 'What building a 59,000-line project with Claude Code taught me about designing context, constraints, and feedback loops for coding agents.'
tags: ['AI-assisted Development', 'Harness Engineering', 'Software Engineering', 'Claude Code']
---

## A data engineering project

I used to work on a data engineering team where we defined data quality together with the business stakeholders who knew the data best. Those expectations became rules that ran alongside the pipelines, with alerts sent when something failed.

As the number of datasets we managed grew, we started to see a different set of operational challenges.

Alerts worked well for individual failures, but it became harder to see the bigger picture: which issues were most urgent, which failures kept recurring, which datasets were well covered, and where the monitoring gaps were. Much of that context was spread across pipelines, emails, and people's knowledge.

I wanted a single viewport that brought that information together: data assets, quality checks, SLAs, breaches, and monitoring gaps.

It was useful enough to want, but large enough that I probably would not have built it myself within a reasonable timeframe. Then, coding agents got good, and suddenly the project felt possible.

## Why vibe coding was such an easy thing to start

For a long time, the biggest cost of a side project was not understanding what to build, but the gap between having an idea and turning it into working software.

You could be completely clear on the problem and still decide not to build it, because you knew how many evenings it would take to go from a blank repository to something usable. AI coding assistants such as Claude Code changed that calculation.

I set myself a simple constraint: I would build the entire platform with Claude Code, and I would not write any application code myself.

At first, the experience was almost disorienting in how fast it moved.

I would describe a page or a feature. Claude would implement it. I would run it, notice something off, describe the issue, and within seconds there would be a new version.

Describe. Run. Observe. Adjust.

The loop was extremely tight. The distance between thinking of something and seeing it working almost disappeared. What would previously have taken days of incremental work could now emerge in a single session.

When the codebase is small, the design space is still fluid, and most decisions are local. In that environment, coding agents are usually effective. The mistake I made was assuming this experience would continue to hold as the system grew. It didn’t.

After about twenty-five days of active development, the project had grown to more than 59,000 lines across the stack, and my conversations with Claude had accumulated roughly 9.8 million tokens.

The platform "worked". But by that point, writing code was no longer the hard part.

## The first thing that broke was my ability to judge the output

Early in the project, my prompts often embedded the solution directly. For example, "following the DAMA-DMBOK framework, evaluate data quality using core measurable characteristics known as dimensions, which determine if data is fit for its intended use". "Implement aggregation logic to generate a score for each dimension". I thought I was being efficient.

The problem was not that Claude could not implement my decisions. It was that it could implement my bad decisions just as efficiently as my good ones. **AI changes the cost of a bad decision**.

When writing code is slow, a questionable design creates friction early. You spend enough time implementing it that you may notice the problem along the way. When implementation is extremely fast, a weak assumption can propagate through models, tests, and code, before you have had much time to reconsider it. Speed does not only accelerate good engineering. It accelerates whatever decision you made.

So I changed the conversation. Instead of prompting "Use X to implement Y." I started giving Claude the constraints and asking it to propose multiple approaches.

What needs to be true?

What are the trade-offs?

How would this behave under high volume of data quality checks across many datasets?

That changed my role. I was no longer primarily producing the solution. I was evaluating solutions. And that led to one of the more surprising lessons from the project: **system design knowledge became more valuable, not less**.

It is easy to imagine that once an AI can generate an implementation, knowing how to build software matters less. I experienced the opposite.

If the agent can give you three plausible architectures in a minute, the scarce skill is no longer producing plausible architectures. It is knowing which one you should reject.

## Then the project became too large for the conversation

A different problem appeared gradually. Claude started to struggle with the growing size of the codebase.

As the project expanded, more and more context was required for it to decide what to do next. It reads through a large amount of existing code to understand the current state, or to reveal the system design decisions.

This made every new action increasingly expensive in tokens, because a large portion of the conversation was spent reconstructing what already existed, such as how the system was structured, what conventions we had established, and why certain decisions had been made.

At the same time, starting a fresh session did not help much. Each session began cold, and I would again spend time re-explaining the state of the project before any meaningful work could happen. I realized the core issue was that the system had **no efficient way to manage and surface the right context at the right time**.

I had been treating context as something the model should implicitly retain or rediscover. Instead, it needed to be **explicitly structured and maintained inside the repository itself**.

I started creating specifications, architecture decision records (ADRs), and mermaid diagrams, to answer questions such as "what did we choose?", "what alternatives did we reject?", "why?", and "what are the consequences?". Ask Claude Code to commit with messages including context that help someone to reason when encountering the code weeks later could understand what changed and why. Also, I tried to stop when the repository itself was in a coherent state: the change made sense, tests were passing or the failure was documented, and the next step could be reconstructed without needing access to the previous conversation.

A clean session boundary is cheaper. A session that ends halfway through a refactor leaves the next agent trying to reverse-engineer both what the code does and what the previous agent intended to make it do.

## Then I became the bottleneck

The next failure mode was less comfortable. Claude could produce more code in an hour than I could meaningfully review in several. At first I tried to compensate by reading faster. That is not a strategy. You cannot out-read a code generator.

If the quality of the project depended on me noticing every type mismatch, stale test, forgotten caller, corrupted interface, or lint violation in thousands of generated lines, then the system did not actually scale.

I had automated code generation while leaving verification manual. That just **moved the bottleneck directly onto me**. The solution was to change the environment so that as much verification as possible happened without requiring judgment.

The most important change was making standard code quality checks unavoidable. I wired static analysis, type checking, linting and unit tests into **pre-commit hooks**. Initially I thought of these as ordinary software-engineering hygiene. They became something more important.

Before that, I could tell Claude: "make sure the tests pass before committing." That is an instruction. Instructions can be forgotten. They can be misunderstood. They can also disappear with the conversation that contained them.

A pre-commit hook is different. In the normal development path, the repository itself rejects the commit when the checks fail. There is no interpretation involved. The test passes or it does not. The type checker exits successfully or it does not. The linter accepts the code or it does not. That distinction turned out to matter enormously.

I stopped asking the agent to remember quality and started making quality part of the environment. Better still, the feedback was machine-readable. When a commit failed because of a type error, Claude could read the error, change the implementation, run the check again and continue. Most of the time, I did not need to participate in that loop at all. That made me think differently about automation.

A rule living in my head creates work for me. A rule written in a prompt creates a request for the agent. A rule enforced by the repository creates a constraint.

Constraints are much more scalable. This does not mean tests or linters make code correct. They do not. They create a deterministic floor: **a set of basic conditions the code must satisfy before it is even eligible for human judgment**.

And that is exactly what I needed. The agent was producing code at machine speed. The first layer of review needed to operate at machine speed too.

## I started breaking work into smaller, verifiable pieces

Earlier, I would ask for an entire feature.

That sounds efficient until the result is hundreds or even a few thousand lines of code, and your only acceptance criterion is whether the page seems to work.

I started decomposing features into smaller units that could be implemented and validated in a single iteration. Each piece needed a concrete definition of done. If I could not explain how I would know that a task was complete, I stopped treating it as ready for delegation. This was another reversal I had not expected.

**The faster Claude became at implementation, the more important it became for me to slow down before implementation**.

**Smaller tasks meant shorter feedback loops**: I could verify correctness earlier, catch mistakes sooner, and adjust direction without paying the cost of a large, entangled change. A vague task can generate an enormous amount of plausible code. A precise task can generate a small amount of verifiable code. The second is usually faster in the end.

## There was one form of testing I refused to hand over

I still tested the actual product myself. I tried workflows as a user. I looked at pages and asked whether the information was actually useful. That remained human for a reason.

A coding agent can be extremely good at verifying whether an implementation matches a specification. But there is a different failure mode: The implementation can match the specification perfectly, and the specification can still be wrong.

Maybe the workflow is awkward. Maybe a feature behaves exactly as described and, once you use it, you realize you described the wrong feature. Static analysis cannot tell you that. A unit test cannot tell you that. And an agent that helped create the specification is not always the best entity to notice it either. That is where I still want human judgment.

## What I would do differently on day one

If I started the same project again, I would not begin with a giant prompt explaining exactly what I wanted Claude to build. I would build the environment, or the **harness**, first. 

I would review the design before implementation. I would keep important decisions in the repository instead of leaving them in conversations. I would establish deterministic quality gates early, before the codebase became large enough to depend on them. I would break work into small, independently verifiable increments. And I would decide deliberately which feedback loops could be automated and which still required human judgment.

None of these practices would make Claude generate code faster. Claude was already fast. They would make that speed easier to control. That is the broader lesson I took from the project: **AI does not remove the engineering bottleneck. It moves it**. Once implementation becomes cheap, other things become scarce: **decision quality**, **context**, **verification**, and **judgment**.

## Speed was the wrong thing to optimize

It is tempting to summarize the experiment with numbers: twenty-five development days, more than 59,000 lines of code, millions of tokens, and a full-stack platform built without me writing the application code.

Those numbers are impressive, but they describe the least interesting part of what changed.

Generating code was rarely the expensive part.

Rebuilding the wrong code was.

Recovering lost context was.

Finding interfaces that had drifted apart was.

Discovering that two perfectly reasonable abstractions represented the same concept was.

Trying to review changes faster than I could understand them was.

So the metric I care about now is **not how quickly code can be generated. It is how much of what gets generated survives**.

But even that only captures part of the shift.

The bigger change was that I could now attempt projects I would previously have dismissed as too large for one person. AI-assisted development did not just reduce the time required to implement an idea; it expanded the range of ideas that felt feasible to build.

And once that constraint changed, the work around implementation mattered much more.

The challenge became deciding what should be built before generating it, preserving enough context for the next session to continue coherently, turning repeatable checks into deterministic constraints, and keeping human attention focused on the decisions that could not be automated.

That changed how I think about working with coding agents.

The goal is not to supervise every line they produce. At some point, they can generate code faster than a person can meaningfully review it.

The goal is to design an environment in which they can work effectively without requiring constant supervision: one that gives them the right context, preserves important decisions, rejects predictable mistakes automatically, keeps changes small enough to verify, and leaves the human responsible for the parts that actually require judgment.

I started this project wondering how much software one person could build with AI-assisted development. I ended it with a different question:

What kind of engineering environment lets one person safely use a system that can generate more code than they can possibly review?

More broadly, the takeaway is not to “use AI to write code faster”, but to **build a system in which agents can produce software more reliably and safely at scale**.
