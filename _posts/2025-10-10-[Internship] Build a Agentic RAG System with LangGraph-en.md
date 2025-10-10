---
title: "[Internship] Build an Agentic RAG System with LangGraph: 1. State"
date: 2025-10-10 13:15:00 +09:00
categories: [Internship, AI Workflow Builder]
tags: [LangGraph, AI workflow builder, LLM, RAG, RAG Agent, Internship]    
description: A deep dive into State management, the cornerstone of building a robust Agentic RAG system in LangGraph
---

This post is based on the tutorial: [랭그래프 LangGraph 기초](https://wikidocs.net/261576).

### What is "State" in LangGraph?

In LangGraph, the **State** is a central object responsible for managing the entire flow of data through the graph. It acts as the "single source of truth" that is passed between nodes.

The State can be defined using either Python's standard `TypedDict` or Pydantic's `BaseModel`.

#### **Comparison: TypedDict vs. Pydantic BaseModel**

| Aspect | `TypedDict` | `Pydantic BaseModel` |
| :--- | :--- | :--- |
| **Primary Focus** | Type hinting for dictionary keys and values. | Strict data validation, parsing, and coercion. |
| **Runtime Validation**| No (type hints are for static analysis only). | Yes (raises errors for invalid data at runtime). |
| **Type Coercion** | No (e.g., the string `'1'` remains a string). | Yes (e.g., `'1'` can be auto-converted to the integer `1`). |
| **Dependencies** | Built into Python's standard `typing` library. | Requires installing the external `pydantic` library. |
| **Performance** | Faster (minimal overhead, it's a standard dict at runtime). | Slower due to the overhead of validation logic. |
| **Common Use Case** | Defining the shape of data for static type checkers. | Validating and parsing incoming data, like API requests. |

### Why is State Necessary in LangGraph?

* **Efficient Management and Tracking:** It allows you to manage complex data structures efficiently and makes it easy to track the graph's execution state at each step.
* **Indirect Communication:** It enables nodes to communicate indirectly. Each node operates independently, but they all interact by reading from and writing to the shared State.
* **Concurrency:** It supports parallel execution. Multiple nodes can be executed concurrently in a "super-step" if they don't have direct dependencies on each other's outputs for that step.
* **Consistency:** It ensures conversational consistency by maintaining the full context throughout long-running interactions, preventing information loss.

### Basic Structure of a State

Here is an example of a State defined with `TypedDict`.

```python
from typing_extensions import TypedDict
from typing import Annotated, List
from operator import add

class BasicState(TypedDict):
    # Values that are overwritten at each step
    current_step: str
    user_id: str

    # Values that are accumulated using a reducer function
    messages: Annotated[List[str], add]
    processing_log: Annotated[List[str], add]
```
- Annotated: This is a special type hint that allows you to add metadata.

    - The first argument is the actual type of the attribute (e.g., List[str]).

    - The second argument (add in this case) is a reducer function* that tells LangGraph how to update this value. Instead of overwriting the list, add appends new items to the existing list.

        > Reducer Function: A concept from functional programming. A reducer defines how a new value should be combined with an existing value (e.g., adding to a list, summing numbers, merging dictionaries).


## Type of State Schema

||Single Schema|Explicit Input/Output Schema|Schema|
|-|-|-|-|
|Concept|Use the same input and output|Define input and output separately|Inclue prviate schema for internal nodes communication|
|Description|Every node shares the same state schema|Often useage in API 설계 <br> Separate external data(output/input) and internal data to avoid internal changes affecting external interface|For huge system <br> Optimized schema for each step|
|Pros|Simple <br> Easy to debug <br> Clear data flow <br> Fast Programming|Capsulized Internal 구현 <br> version management <br> Increased security(Avoid exposing internal data)|Optimized data structure for each step <br> High Modulization  <br> Increasing capability of reusability  <br> Possible independent test for each step| 
|Cons|With system complexity, unnecessary fields can be created <br> Can be heavy <br> Security Problem|Relatively complicated setting <br> More code lines needed <br> State change logic needed|Sturcture Complexity <br> State change overhead  <br> Possibility of hard debugging|
|Use Case|Simple chatbot or QA system, Prototype| API (Explicit interface is needed) <br> Linking with the external system <br> Version management|RAG Pipeline  <br> Complicated System with multiple steps  <br> Different data structure for each step  <br> Microservice architecture|