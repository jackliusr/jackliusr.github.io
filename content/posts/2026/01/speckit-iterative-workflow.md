---
title: "spec-kit: iteration workflow in a feature"
date: 2026-02-02T20:10:00+08:00
categories:
- tech
tags:
- spec-kit
- sdd
---

# Building with LLMs: How Spec-Kit Changed My Workflow

Over the past three years, I’ve used LLMs—such as ChatGPT, Copilot, and Claude—primarily as *assistants* to generate code snippets for very specific requirements. They were useful, but always in a limited way: small pieces of logic, isolated helpers, or syntax scaffolding.

What I **didn’t** do until recently was build an entire system *around* an LLM.

The main reason was simple: most of my real work lives inside internal frameworks, legacy systems, and opinionated architectures. General-purpose LLM tools weren’t a great fit for those constraints. They could help locally, but they didn’t shape the system.

That changed last week.

Using **spec-kit** fundamentally shifted how I think about building software with LLMs. Instead of treating the model as a code generator, spec-kit positions it as a **workflow engine**—one that participates in requirements clarification, specification refinement, and implementation planning.

In this post, I’ll share how I iteratively built a small spec-kit–based system, and why the iteration model matters more than “getting it right the first time”.

---

## Requirements Are Never Right the First Time

Anyone who’s worked on real systems knows this:

- Requirements evolve  
- Specifications inherit ambiguity from requirements  
- The “first draft” is almost never correct  

Specifications are not a static artifact—they’re a living document. Treating them as final is usually where things go wrong.

When you run: /speckit.specify, spec-kit creates a new feature specification. In practice, that initial output is *rarely* exactly what you want. And that’s okay.

The mistake is assuming the workflow must be strictly linear.

---

## The Traditional Workflow

The conventional spec-kit flow looks like this:

```mermaid
flowchart LR
    specify --> plan --> task --> implement
```

This mirrors classic software delivery thinking:

1. Specify the feature

1. Plan the approach

1. Break it into tasks

1. Implement

This works well only if each step is already correct—which almost never happens in reality.

## Embracing an Iteration Workflow

Instead of restarting from scratch when something feels off, I use an iteration-first workflow.

In this approach, I freely modify any artifact:

* spec.md

* plan.md

* tasks.md

Then I ask spec-kit to continue from that change, rather than rerunning the entire pipeline.

My go-to prompt looks like this:

```
/speckit.analyze I changed [spec.md|plan.md|task.md], do things accordingly.
```

Why this works:

* I can’t guarantee consistency between my edits and existing artifacts

* spec-kit re-analyzes context holistically

* Each phase stays loosely coupled but logically aligned

## Iterative State Model

Conceptually, I think of the workflow like this:

```mermdaid
stateDiagram-v2
    [*] --> Specify
    Change_Spec --> Plan
    Specify --> Plan
    Change_Plan --> Task
    Plan --> Task
    Change_Task --> Implement
    Task --> Implement
    Implement --> [*]

```

Key idea:

Any artifact change becomes a new entry point into the workflow.

You’re no longer locked into a strict forward-only pipeline. Instead, you move forward from wherever clarity improves.

## Why This Changed My Perspective

Spec-kit isn’t just about generating specs—it enforces thinking in phases while still allowing iteration.

For the first time, I felt like:

* The LLM was collaborating on system design, not just code

* Iteration was a first-class concept, not a workaround

* Legacy and internal constraints could be respected incrementally

This is the missing piece I hadn’t found in other LLM tools.

## Closing Thoughts

LLMs are great at producing answers.
Spec-kit is good at supporting process.

Once you accept that specifications are never perfect, an iterative workflow stops being a compromise—and becomes the strategy.

If you’re working with real systems, not toy demos, that difference matters.