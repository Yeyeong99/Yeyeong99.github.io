---
title: "[Internship] Build a Agentic RAG System with Flowsie"
date: 2025-08-12 13:15:00 +09:00
last_modified_at: 2025-08-13 10:48:00, +09:00
categories: [Internship, AI Workflow Builder]
tags: [Flowise, Low-code LLM Platform, AI workflow builder, LLM, RAG, RAG Agent, Internship]     # TAG names should always be lowercase
description: An Exploration of a Key Concept in Flowise Agentflow, with a Comparison to LangChain and LangGraph.
---

## Introduction to Flowise
![Showing Flowise workflow UI](https://docs.flowiseai.com/~gitbook/image?url=https%3A%2F%2F823733684-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F00tYLwhz5RyR7fJEhrWy%252Fuploads%252FK5NWsHkLAelZq9sBlY8x%252FFlowiseIntro.gif%3Falt%3Dmedia%26token%3Dea75ba7b-32fa-447d-8872-41fe5578fe1f&width=400&dpr=3&quality=100&sign=35a65986&sv=2)_GIF From Flowise Official Docs_

At my current internship, we use Flowise, a low-code LLM platform that powers our company's AI workflow builder.

Flowise offers two main paradigms: Chatflows and Agentflows, making it essential to understand both. My previous experience involved building a RAG chatbot with Chatflow (v2.3.3). In this post, I will revisit some key concepts from that project while demonstrating how to build an Agentflow.

This post will serve as a guide to understanding the basic nodes and steps required to build an Agentflow.

## Key ChatFlow Nodes for Flowise Workflows
### Structured Output Parser
I wanted to see if I could control the output format of a Flowise node. Programmatically, this is straightforward; for example, with the OpenAI API, you can simply add the `response_format={'type': 'json_object'}` parameter to the `client.chat.completions.create` method.

While this wasn't covered in our internal documentation, I discovered the solution: the Structured Output Parser node. Using this node achieves a similar result to setting the `response_format` parameter.

For example, if you want to ensure your prompt's output is in a specific JSON format, you can define the schema in the parser like this:

```json
{
    "your_key": "your_value"
}
```
## A Key Concept in Agentflow v2: Flow State
**Flow State** is a shared, key-value context that exists only for the duration of a single execution run.
It enables:
    - Explicit data sharing between nodes.
    - Passing data which is conditionally generated.
    - Accessing data between non-adjacent steps in the flow.

#### How it works
- **Initialization**: The state's schema (its structure and keys) is declared in the `Start Node`. All possible keys for the flow state must be defined at this stage.
- **Read / Update**: Any node in the flow can read from or update the values of the keys defined in the flow state.
- **Immutability of Keys**: New keys cannot be created during the execution; only the values of pre-defined keys can be updated.
- **Maintaining State**: The updated state is available until the end of the execution run.

#### Comparison with Other State Management Approaches
1. Langchain
    - Separates state management into two concepts: `State` and `Memory`.
        - `State`: Manages the current conversational turn or immediately relevant information.
        - `Memory`: Stores long-term information about the user, such as their profile and past conversations.
    - Common Memory Modules: Includes `ConversationBufferMemory`, `ConversationSummaryMemory`, `Redis`
        - e.g., `BufferMemory` stores the entire conversation history, while `SummaryMemory` stores a condensed, summarized version.
    - Code Example:
        ```python
            memory = ConversationBufferMemory()
            chain = ConversationChain(llm=llm, memory=memory)
        ```

2. LangGraph (and similar Agent Framework)
    - Use a central state object, often defined as a dictionary or a typed class (like Python's `TypedDict`)
        ```python
            class State(TypedDict):
                messages: list  # chatting message list
                name: str       # user info
                birthday: str   # additional information
        ```
    - Each node in the graph receives the current state and can return an updated version, managing the entire flow of data.
    - This approach also naturally supports features like Human-in-the-loop, where execution can be paused for user input.

## Agentflow in Flowise
### Key Nodes
#### Condition Node
- This node is conceptually imilar to LangGraph's `add_conditional_edges` method.
- It routes the workflow based on pre-defined, fixed critria

#### Condition Agent Node
This powerful node routes the workflow by categorizing user input into different scenarios. What makes it special is that this routing logic is controlled by natural language instructions, eliminating the need for custom code.

For comparison, here is how you could implement similar routing logic using LangGraph:
```python
from langgraph.graph import StateGraph
from langgraph.pregel import ToolNode
from typing import TypedDict, Literal

# Define the state object that will be passed between nodes
class State(TypedDict):
    user_input: str
    # The category will be determined by the classifier node
    category: Literal["weather", "general", "other"]

# This node uses an LLM to classify the user's input
def classify(state: State):
    prompt = f"Classify the following query: {state['user_input']} into one of the following categories: weather, general, or other."
    result = llm.invoke(prompt)
    # The parsed result updates the 'category' in our state
    state["category"] = parse_result(result) 
    return state

# Define the state machine graph
graph = StateGraph(State)

# Add all the nodes to the graph
graph.add_node("classify", classify)
graph.add_node("weather_handler", handle_weather)
graph.add_node("general_handler", handle_general)
graph.add_node("other_handler", handle_other)

# The conditional edge directs the flow based on the 'category' key in the state
graph.add_conditional_edges(
    "classify", # The source node
    lambda state: state["category"], # The function that determines the route
    { # A mapping from the function's output to the next node
        "weather": "weather_handler",
        "general": "general_handler",
        "other": "other_handler",
    }
)

# Set the entry point and compile the graph into a runnable workflow
graph.set_entry_point("classify")
workflow = graph.compile()
```
## Building a Simple Agentic RAG System
![Agentic RAG Flow. Start node - Condition Agent to validate query - LLM node for query generation | LLM node for general response - Vector DB Retriever node - Document relevance check node - LLM node for general response | LLM node for final response - Loop node](https://docs.flowiseai.com/~gitbook/image?url=https%3A%2F%2F823733684-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F00tYLwhz5RyR7fJEhrWy%252Fuploads%252FvqsOL3lc2hKQ2rq5IZpW%252Fimage.png%3Falt%3Dmedia%26token%3D8b3121f8-836a-494c-a881-5f5c9ee96102&width=400&dpr=3&quality=100&sign=ce56d133&sv=2)_Image from Official Flowise Docs_

[Practice this tutorial via the official docs](https://docs.flowiseai.com/tutorials/agentic-rag#step-2-adding-query-validation)

### Difference between Agentic RAG and Standard RAG
An Agentic RAG system enhances a standard RAG system with several key capabilities:

1. Query Validation & Categorization: It first analyzes the user's input to understand its intent.

2. Optimized Query Generation: It creates optimized queries specifically for searching the Vector Database (VDB).

3. **Relevance Evaluation**: It assesses whether the retrieved documents are actually relevant to the user's query.

4. Self-Correction: It can identify irrelevant results and correct its course to find better ones.

### Key Differentiators
- Conditional Agent Node(Decision Points): Agentic RAG systems use agents to create decision points that dynamically determine the next step in the workflow. This allows the agent to make flexible, context-aware decisions rather than following a fixed path.

- Loop-Back Mechanism: If the generated results are unsatisfactory or irrelevant, a Loop Node can re-execute a specific part of the flow to try again, improving the final output.