---
title: "[AI Product Engineer] Building a Document Translation Agent"
date: 2026-05-17 14:50:00 +09:00
last_modified_at: 2026-05-17 14:50:00 +09:00
categories: [AI Product Engineer]
tags: [Codex, Vibe Coding, AI Product Engineer, Agent, Product Development, Translation, Doc2Doc, Doc2Doc Translation, B2C Service]
description: "Building a translation pipeline that understands context across long documents"
language: en
postid: 14
---

# From Legacy Translation Code to Document-Level Consistency: Adding Memory to a Translation Pipeline

At the company where I worked, we began developing a suite of task-specific agents for a B2C service. I was responsible for the translation agent, from UI planning to agent development and improvement.

My starting point was a translation agent already used by a client. I wanted to build on its existing workflow and make incremental improvements.

The client's agent already handled text extraction and inserting translations back into their original positions well. I therefore focused on consistency issues: the same term being translated differently, or the writing style shifting between the beginning and end of a document.

This post describes how I approached **maintaining consistent translation decisions throughout a document** by adding pre-translation analysis and memory to the pipeline.

## 1. Providing the Model with Relevant Context

Because this was a document-to-document (Doc2Doc) translation task, there was more to handle than simply calling a translation model. Compared with translating isolated passages, preserving context was essential. My first approach was to send the LLM the preceding and following pages along with the page being translated, giving it surrounding context to draw on.

However, nearby pages alone could not guarantee that a term on page 1 would be translated with the same meaning on page 140 of a roughly 200-page document.

## 2. Separating Terminology Consistency from Contextual Continuity

Providing nearby context in the prompt was not enough. I researched long-document translation and ultimately used the following two papers to guide improvements to the agent's design:

- [DelTA: An Online Document-Level Translation Agent Based on Multi-Level Memory](https://arxiv.org/abs/2410.08143): Research on document translation using multi-level memory and bilingual summaries.
- [Translating Step-by-Step: Decomposing the Translation Process for Improved Translation Quality of Long-Form Texts](https://aclanthology.org/2024.wmt-1.123/): Research that breaks translation into stages, from preliminary research to proofreading.

Drawing on these papers, I approached consistency from two perspectives.

- **Terminology consistency:** Terms used with the same meaning should follow the same translation conventions throughout the document. This also includes relationships between abbreviations and full names, as well as concepts that look similar but must remain distinct.
- **Continuity of context and style:** Later references should remain connected to earlier explanations. The translation should also preserve the progression of the discussion and a consistent style across headings and body text.

For example, if the same term is translated as “입금 처리” in one section and “예금 처리” in another—two different Korean renderings of deposit processing—that calls for terminology management. If a later sentence loses track of what “this procedure” refers to, it needs discourse context. These problems require different kinds of information.

## 3. Analyzing Terms and Meaning Before Translation

The first workflow I introduced for terminology consistency combined term scanning with pre-translation analysis.

```text
Collect translation units from the document
  → Collect candidate terms and supporting source passages
  → Run pre-translation analysis
  → Initialize document term memory
  → Inject relevant information into translation prompts
```

I assigned each component a distinct role.

| Component | Role |
| --- | --- |
| Temporary Glossary | Stores candidate terms and supporting evidence collected from the source |
| Pre-Translation Analysis | Analyzes the document's meaning and relationships between terms using the source text |
| Document Term Memory | Maintains the active terminology state used and updated during translation |

To build the glossary, I used techniques such as n-grams to collect frequently occurring words and phrases, excluding articles such as “the,” “a,” and “an” that were not useful as terminology candidates. The collected terms were first passed to the LLM together with sentences containing them. The LLM then used those sentences to determine how each term should be interpreted.

The pre-translation analysis was inspired by *Translating Step-by-Step*. My reasoning was that understanding the document's domain would help produce more natural translations.

> Related research: *Translating Step-by-Step* decomposes translation into preliminary research, drafting, revision, and proofreading. Although my implementation does not follow exactly the same process, the paper provides a methodological basis for treating pre-translation analysis as a separate stage. [Briakou et al., 2024](https://aclanthology.org/2024.wmt-1.123/)

## 4. Extending the Glossary into Updatable Document Term Memory

Document Term Memory manages more than a list of source terms and their translations. It also records their meanings within the document, preferred translations, translations to avoid, and source terms that should not be confused with one another.

The translation prompt instructs the model to use a preferred translation when one is provided. However, if the preliminary judgment is `source_meaning_only` or `blocked`, the model uses only the meaning information without applying the suggested translation. This distinction helps maintain consistency while reducing the risk of propagating an uncertain translation throughout the document.

After translation, a resolver reviews the results and supporting evidence and proposes memory updates. These proposals are applied through a defined set of allowed actions, such as adding a sense, adding a translation candidate, selecting a preferred translation, or marking a translation to avoid.

One issue to watch for is circular confirmation. If a translation stored in memory is injected into the prompt and the model repeats it, that output alone cannot establish that the translation is correct. The resolver therefore also includes a policy for checking the influence of translations injected into the prompt.

The key question for terminology memory was **what evidence should justify retaining or changing a translation decision**, rather than how much information to store.

## 5. Adding Memory to Preserve Context Across Long Documents

Even with shared terminology conventions, carrying explanations and writing style from the beginning of a document through to the end remained difficult. To address this, I added `Bilingual Summary Memory`.

Here, memory means external state that persists across multiple translation calls while processing a single long document. The implementation at the time tied memory to a specific job and document, and deleted it when the job ended. It did not provide permanent memory that automatically reused knowledge across documents or users.

The memory accumulates source–translation pairs from completed translation scopes. When compression is enabled, the pipeline checks the prompt memory budget. If that budget is exceeded, it summarizes the existing summary together with the newly accumulated content.

```text
Complete a translation scope
  → Accumulate source–translation pairs
  → Check the memory budget if compression is enabled
  → Update the cumulative summary if the budget is exceeded
  → Reference the memory in the next translation scope
```

The summary was designed to capture not only source content and the flow of the translation, but also writing style, the current state of the discussion, and unresolved references. The goal was to convey both what had been explained in the source and how it had been expressed in the translation so far.

> Related research: DelTA uses multi-level memory that incorporates both source and translated text. It distinguishes proper noun records, bilingual summaries, long-term memory, and short-term memory, retrieving and updating them during translation. My implementation does not reproduce the full architecture, but shares its focus on terminology management and accumulated context. [Wang et al., DelTA, ICLR 2025](https://arxiv.org/abs/2410.08143)

## 6. Preventing Summary Memory from Overriding Terminology Decisions

Using both memories required a clear division of responsibility.

Document Term Memory manages term meanings and translation conventions. Bilingual Summary Memory maintains continuity in content and writing style.

The summary prompt explicitly instructs the model not to create terminology rules, decide on or revise translations of terms, recommend terminology, or generate source-to-target term mappings. Otherwise, summarizing previous translations could introduce new terminology rules that conflict with the term memory.

For example, summary memory might record that “the previous section introduced a procedure, and the discussion is now moving on to exceptions.” Term memory determines which Korean term should be used for the name of that procedure.

## 7. Deciding When to Update Memory During Parallel Processing

Preserving context requires attention to order, but translating every scope sequentially can increase processing time. In the current local changes, multiple scopes are processed together in a wave, and the completed wave's results are then used to update summary memory.

```text
Translate multiple scopes in one wave
  → Collect the wave's results
  → Update summary memory
  → Translate the next wave
```

With this approach, scopes within the same wave cannot immediately reference one another's final translations. The next wave, however, can use the information accumulated through the preceding wave. This creates a way to balance throughput against how frequently context is passed forward.

I also avoided making a summarization call after every update. Even when compression is enabled, the pipeline keeps the source–translation buffer and skips summarization if there is still room in the budget. When compression is disabled, it simply accumulates raw records. Enabling memory and enabling summary compression are therefore separate settings.

## 8. Verifying Improvements Through Comparable Results

Adding memory does not, by itself, establish that translation quality has improved. An incorrect term might simply be repeated more consistently, and summarization can lose important information.

At the time, I added an evaluation-only path that returned source text, translations, context scopes, and insertion locations, along with functionality for saving memory and prompt snapshots. This made it possible to compare the same document under the following conditions.

| Condition | Question to Investigate |
| --- | --- |
| Baseline | What problems occur before adding memory? |
| Term memory enabled | Are terms used with the same meaning translated more consistently? |
| Context memory added | Are references and writing style better preserved? |
| Compression enabled | How do context retention and the cost of additional calls change? |

Beyond overall translation quality, evaluation should consider terminology accuracy, consistency within each sense of a term, content omissions, stylistic continuity, processing time, and token usage.

## 9. What I Learned

I learned two main lessons while developing this Doc2Doc translation agent.

First, solving the task required more than straightforward LLM calls. I needed a structure that could carry earlier decisions into later translations and revise those decisions when new evidence appeared.

Pre-translation analysis established an initial understanding of the document. Document Term Memory managed terminology decisions, while Bilingual Summary Memory helped preserve content and style across long documents. Adding controls for update timing and cost led me to treat translation as a process in which multiple stages share state.

Next, I plan to run controlled comparisons on the same documents, enabling and disabling individual components to identify which types of errors each memory component reduces.

Second, the task could be handled without a complex multi-agent architecture.

At first, features such as memory updates naturally led me to consider multiple agents. But after implementing rule-based orchestration, I found that processing time was lower than expected while maintaining sufficient quality for users.

That experience taught me that a task requiring several roles does not automatically require a multi-agent design.
