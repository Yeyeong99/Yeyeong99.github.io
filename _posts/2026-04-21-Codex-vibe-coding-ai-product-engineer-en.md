---
title: "[AI Product Engineer] How I Use Codex for Vibe Coding: A First Step"
date: 2026-04-21 14:50:00 +09:00
last_modified_at: 2026-04-21 14:50:00 +09:00
categories: [AI Product Engineer]
tags: [Codex, Vibe Coding, AI Product Engineer, Agent, Product Development]
description: "How to use Codex as a product development partner across planning, implementation, testing, and documentation"
language: en
postid: 13
---

## Introduction

Around last October, when vibe coding started to become a real topic, I expected it to open a completely new world. I thought the annoying process of opening HTML, inspecting elements, and toggling CSS properties one by one in developer tools would finally be over. If AI could do that, I thought, maybe I could build an end-to-end product much faster than before.

So I tried to build a blog with that expectation. But the result was very different from what I imagined. Building the database, wiring CRUD flows, and making the whole thing work as a product did not magically resolve itself. After that experience, I became skeptical of vibe coding.

Then I had to build a demo agent in a very short time, within about three weeks. Without vibe coding, there was no realistic way to meet the deadline. So I tried again. This time, I used Codex, and it felt very different from my first attempt. As I built product features, fixed bugs, and organized documentation with it, my view changed. The most important part of vibe coding was not how fast I could make AI write code. It was how clearly I could explain the problem and break it into workable pieces.

Vibe coding, at least in my experience, is not about handing coding over to AI. It is closer to running a faster product development loop with AI: defining the problem, understanding the existing codebase, choosing an implementation direction, testing the result, and documenting what changed.

In that process, the developer's role moves closer to the product itself. I began to think of this as a first step toward working as an **AI Product Engineer**.

## Codex Feels More Like a Pair Engineer Than a Code Generator

When my first attempt at vibe coding failed, I had been using AI almost like a tool that would create the final result for me. I described the screen and the features I wanted, and I expected AI to design the structure, connect the database, and finish the CRUD flows. But once the task became even slightly complex, the context started to break. When something went wrong, it was hard to know where to look.

What changed this time was that I used Codex less as a final-output generator and more like a **pair engineer that explores and edits inside the existing codebase with me**. Codex can search across the codebase, identify related files, understand existing patterns, trace data flow, make changes, and run tests.

Because of that, the way I prompt it also needs to be different.

Instead of starting with "write this code", I found this kind of prompt much more useful:

```text
Find where this feature is implemented.
Do not modify the code yet. First explain the current data flow and existing patterns.
```

With this kind of start, Codex is less likely to invent a new structure immediately. It first tries to understand the system that is already there. Once that context is clear, implementation becomes much safer.

Because I had to build a demo-ready agent in three weeks, the useful workflow was not "make as much as possible as fast as possible." It was closer to "make small changes so that if something fails, I can immediately find the cause."

1. Find the related files and data flow.
2. Explain the existing patterns and constraints.
3. Break the demo requirements into small features.
4. Implement one step at a time.
5. Run the relevant tests.
6. Analyze failures or runtime errors.
7. Leave the final change in documentation or a blog post.

Through this loop, I realized that Codex is not best at producing one perfect answer in a single shot. It is best at helping me run the loop of exploration, implementation, and verification faster.

## My Actual Workflow

### 1. Ask It to Read Before Asking It to Edit

The most useful habit was saying: "Do not modify anything yet. Read first." My earlier failed approach had been almost the opposite. I described the result first, let AI generate code, and only tried to understand the code afterward. When something broke, I did not know where to begin.

For example, if I were investigating a progress display issue in a translation agent, I would not begin with "fix the progress bug." I would start with:

```text
Find where the translation progress is calculated and rendered.
Do not edit the code yet. Summarize the current structure first.
```

Codex can then follow the frontend components, backend events, SSE stream, and state management logic. As a human reviewer, I can read that explanation and understand where the actual change should happen.

If I ask for implementation too early, AI may create a plausible but unnecessary structure. If I ask it to read first, the solution is much more likely to fit the existing system. This became especially important under a short deadline. Fixing something quickly is good, but avoiding the wrong fix is often faster.

### 2. Break Large Features Into Small Changes

When a task is too large, AI can still produce code, but the result becomes difficult to review. In my first blog-building attempt, I asked for something too broad, like building a service with a database and CRUD. As a result, it was hard to tell which parts were correct and where the wrong assumptions started.

This time, I split features into smaller units. Even for a demo agent, the product was not one huge feature. It was a combination of smaller flows: file upload, progress display, result rendering, error handling, retry behavior, and documentation.

For example, "improve the translation agent" is too broad. A better first step would be:

```text
First, add only the structure for storing pre-translation analysis results.
Do not connect it to the translation prompt yet.
```

Then the next step can be:

```text
Now inject the stored pre-translation analysis into the translation prompt.
Follow the existing prompt builder pattern.
```

This makes each change easier to understand. If a test fails, the source of the problem is easier to locate. Most importantly, the code remains reviewable.

Vibe coding does not mean letting the flow decide everything. In practice, it means that the human breaks the work into meaningful pieces, and AI helps implement each piece quickly. Once I understood this difference, vibe coding finally started to feel like a real way to speed up product development.

### 3. Make Existing Patterns the Default

Codex is good at proposing new structures, but new structures are not always what a product codebase needs. Real services already have routing patterns, state management conventions, testing styles, and documentation locations.

So when I ask Codex to implement something, I often include this sentence:

```text
Prefer the existing patterns in this codebase over introducing a new abstraction.
Add a small helper only if it is really needed.
```

This matters more than it looks. It helps keep the codebase consistent and makes the result easier for other engineers to read later.

AI can speed up implementation, but it can also speed up unnecessary abstraction. When the goal is to build a demo-ready product in a short time, a clever new structure is usually less valuable than a stable change that fits the current system. Explicitly asking it to make less code is often very effective.

### 4. Include Testing and Verification in the Same Loop

One of the best parts of using Codex is that the workflow does not need to stop after implementation.

```text
Find and run the tests related to the change you just made.
If they fail, analyze the cause and make the smallest necessary fix.
```

This creates a loop after the code change.

Codex can run the test command, read the failure output, narrow down the cause, and decide whether the implementation or the test expectation needs to change. The final judgment still belongs to the human, but Codex removes a lot of repetitive checking work.

The goal is not to force every test to pass blindly. The goal is to understand why something failed and whether the fix actually matches the product requirement. Under a tight deadline, it is tempting to skip this verification loop, but skipping it often costs more time later.

## Skills That Become More Important for an AI Product Engineer

Using Codex made several skills feel more important, not less.

The first is **problem definition**. AI can respond to vague requests, but vague requests produce vague results. "Build a demo agent" is much weaker than "let the user upload a file, show progress, and display the final result when the job is complete." "Improve translation quality" is much weaker than "extract candidate terms during pre-translation analysis and reuse them during translation so repeated terms stay consistent."

The second is **scope control**. In product development, not everything should be fixed at once. Especially when I had only three weeks before the demo, I had to separate what was necessary for the demo from what could be improved later. Codex also needs that boundary.

The third is **code reading**. Even if AI writes the code, the human still needs to judge whether it is correct. In fact, because AI can produce more code faster, the ability to read and evaluate code becomes even more important.

The fourth is **product-level verification**. Passing tests does not automatically mean the user experience is good. The UI can feel awkward, loading states can be unstable, or error states can leave users unsure what to do next. These details need to be checked from the user's point of view.

In the end, an AI Product Engineer is not just someone who asks AI to write code. It is someone who defines a product problem and turns it into small, testable, implementable steps with AI.

This role sits somewhere between product planning and engineering. I need to keep connecting what the user needs to do, what needs to be shown in the demo, and what code changes are required to create that experience. Codex can help implement that connection quickly, but the human still decides what should be connected.

## Prompt Patterns That Worked Well

These are the prompt patterns I used often while working with Codex.

### Understand the Structure Before Editing

```text
Do not modify the code yet.
First summarize which files make up this feature.
Also explain the data flow and the main state values.
```

### Follow Existing Patterns

```text
Prefer the existing patterns in this codebase over creating a new structure.
Match nearby helper APIs and test styles.
```

### Limit the Scope

```text
Keep this change as small as possible.
Do not do unrelated refactoring or style changes.
```

### Implement Step by Step

```text
First add only the storage structure.
We will connect it to the prompt and expose it in the UI in later steps.
```

### Continue Through Verification

```text
Run the relevant tests and analyze any failures.
If a fix is needed, make the smallest possible change.
```

### Turn the Work Into Documentation

```text
Summarize this work for a blog post.
Organize it by problem, approach, implementation result, and lessons learned.
Focus more on why the design was chosen than on line-by-line code details.
```

These prompts are not fancy. But in real product development, simple prompts like these were the ones I used the most.

## What to Be Careful About

Codex is powerful, but it should not make every decision on its own.

AI sometimes overgeneralizes the intent of a codebase. In areas with weak test coverage, it can confidently produce the wrong implementation. In complex business logic, code that runs and code that is correct can be two different things. The skepticism I felt after my first vibe coding attempt came from this. A result can appear quickly, but if I cannot explain why it is correct or how to fix it, it is hard to turn it into a real product.

That is why I think Codex is better understood as a pair engineer that increases development speed, not as a replacement developer.

The human still needs to do the important parts:

- Decide which problem matters.
- Define the scope of the current change.
- Check whether Codex understood the code correctly.
- Make sure the implementation fits the architecture.
- Judge whether the tests verify the actual product requirement.
- Review the final user experience.

Without this human judgment, vibe coding can become fast confusion instead of fast development.

## Closing Thoughts

Codex changed how I think about vibe coding. At first, I had high expectations and then became disappointed after failing to build a blog and CRUD flow the way I imagined. But when I had to build a demo agent in three weeks and tried again, I realized that the problem was not vibe coding itself. The problem was how I had been using AI.

The core value is not simply that AI writes code for me. The bigger change is that I need to describe the problem more clearly, split the work into smaller units, and verify the result more quickly. Before, I would search files, read code, make changes, run tests, and write documentation mostly by myself. Now I can run that loop much faster with Codex.

Vibe coding is not a way to build carelessly. When used well, it actually forces more structured thinking. In that process, the developer becomes less of a pure implementer and more of someone who defines problems and turns them into product.

The biggest lesson I learned from using Codex is this:

> What matters is not how fast I can type code, but how clearly I can explain a problem and divide it into verifiable steps.

That is where vibe coding becomes a practical first step toward working as an AI Product Engineer.
