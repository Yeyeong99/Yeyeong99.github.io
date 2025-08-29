---
title: "[Internship] Build an Agentic RAG Chatbot"
date: 2025-08-29 16:15:00 +09:00
categories: [Internship]
tags: [Internship, Agentic RAG, RAG, Flowise, LangGraph]
description: Build an Agentic RAG Chatbot with Flowise
language: en
postid: 5
---

When Flowise released its new Agentflow (v3.0.0), I was assigned the task of rebuilding our existing RAG Chatbot, which was originally based on the older Chatflow (v2.2.3). This post documents the process and some of the challenges I encountered.

---
# Node Structure

Here is the basic node structure for the agent:

* **Start**: Receives the user's question and initializes the flow state.
* **Tool**: Gets the current date and time.
* **Function**: Formats the date and time string from the Tool node.
* **Conditional Agent**: Determines if the user's question is domain-related or a general query.
* **LLM (General)**: Generates an answer for general, non-domain questions.
* **Agent (Domain)**: Generates an answer for domain-specific questions, using retrieved documents.


# Challenges Encountered
## Undefined Message Error

I encountered an unexpected error when I connected the `Function` node to the `Conditional Agent` node.

```
Error in Condition Agent node: Cannot read properties of undefined (reading 'message')

```

My initial hypothesis was that the output from the `Function` node was not in the format expected by the `Conditional Agent` node. This was confusing because the agent's input was supposed to be the original user question, not the output of the function.

After some troubleshooting, the solution was surprisingly simple: **I changed the LLM model used in the agent.** For reasons that are still unclear, this resolved the error completely.

## The Agent Ignores the System Prompt

I attempted to write an advanced system prompt to improve the agent's performance, but it had the opposite effect. The agent performed much better with a simpler prompt that followed the basic guidelines.

### **Agent Misunderstands the Prompt**

A key issue was that the agent misinterpreted formatting requests as retrieval instructions. For example, if the user asked for a summary **in a table format**, the agent would try to **find a document containing a table** rather than creating a table from the retrieved information.

Here are the solutions that worked for me:

#### **Solution 1: Query Transformation**

* I added another LLM node upstream to transform the user's raw question into a clear, optimized query for the vector database search.
* Including **few-shot examples** in the transformation prompt was crucial for ensuring the LLM produced consistent and precise results.

#### **Solution 2: Revise the System Prompt**

* I made the instructions in the main agent's system prompt more direct and unambiguous.
* Using markdown formatting (like lists and bolding) helped to structure the instructions explicitly, making them easier for the agent to follow.






