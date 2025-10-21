# LangChain Agents - Quick Reference

## Overview
Agents are LLMs that can decide which tools to use and take actions to accomplish tasks through iterative reasoning.

## Core Concept

**Agent = LLM + Tools + Decision Loop**

Unlike chains (fixed workflow), agents dynamically choose tools and strategies based on the task.

---

## How Agents Work (ReAct Pattern)

```
Thought: "I need to find information"
Action: Use Wikipedia tool
Observation: Returns search results
Thought: "Now I need to calculate"
Action: Use calculator tool
Observation: Returns calculation
Thought: "I have the answer"
Final Answer: Provides response
```

**Agents reason iteratively until the task is complete.**

---

## Agent Types

### 1. Built-in Tools (Pre-made)

#### Math Tool
```python
from langchain.agents import load_tools, initialize_agent, AgentType
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(temperature=0)
tools = load_tools(["llm-math"], llm=llm)

agent = initialize_agent(
    tools, 
    llm, 
    agent=AgentType.CHAT_ZERO_SHOT_REACT_DESCRIPTION,
    handle_parsing_errors=True,
    verbose=True
)

agent("What is 25% of 300?")
```

#### Wikipedia Tool
```python
tools = load_tools(["wikipedia"], llm=llm)
agent = initialize_agent(tools, llm, agent=AgentType.CHAT_ZERO_SHOT_REACT_DESCRIPTION)

agent("Who is Tom Mitchell and what book did he write?")
```

**Agent workflow:**
1. Decides Wikipedia search is needed
2. Searches for information
3. Returns synthesized answer

---

### 2. Python Agent (Code Execution)

Executes Python code dynamically to solve problems.

```python
from langchain_experimental.agents.agent_toolkits import create_python_agent
from langchain_experimental.tools import PythonREPLTool

agent = create_python_agent(
    llm,
    tool=PythonREPLTool(),
    verbose=True
)

customer_list = [["Harrison", "Chase"], ["Lang", "Chain"], ["Dolly", "Too"]]
agent.run(f"Sort these customers by last name: {customer_list}")
```

**What happens:**
- Agent writes Python code to sort the list
- Executes the code
- Returns the sorted result

**Use case:** Data manipulation, calculations, file processing

---

### 3. Custom Tools

Define your own tools using the `@tool` decorator.

```python
from langchain.agents import tool
from datetime import date

@tool
def time(text: str) -> str:
    """Returns todays date, use this for any 
    questions related to knowing todays date. 
    The input should always be an empty string."""
    return str(date.today())

# Add to agent
agent = initialize_agent(
    tools + [time],  # Combine built-in + custom
    llm,
    agent=AgentType.CHAT_ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

agent("What's the date today?")
```

**Key components:**
- `@tool` decorator: Converts function to agent tool
- **Docstring**: Describes when/how to use the tool (critical for agent decision-making)
- **Return value**: What the agent receives as observation

---

## Agent Parameters

### AgentType
```python
AgentType.CHAT_ZERO_SHOT_REACT_DESCRIPTION
```

- **CHAT**: Works with chat models
- **ZERO_SHOT**: No examples needed
- **REACT**: Reason + Act pattern
- **DESCRIPTION**: Chooses tools based on descriptions

### Verbose Mode
```python
verbose=True
```

Shows agent's internal reasoning:
- Thought process
- Tool selection
- Tool output
- Final reasoning

### Error Handling
```python
handle_parsing_errors=True
```

Retries if agent produces malformed output instead of crashing.

---

## Agent vs Chain vs Tool

| Component | Behavior | Example |
|-----------|----------|---------|
| **Tool** | Single action | Search Wikipedia |
| **Chain** | Fixed sequence | Always: Retrieve → Answer |
| **Agent** | Dynamic decisions | Decides: Search OR calculate OR both |

---

## Key Concepts

### Tool Selection
Agent reads tool descriptions and decides which to use:

```python
@tool
def inventory_check(product: str) -> str:
    """Checks if a product is in stock. 
    Use when user asks about availability."""
    return "In stock"
```

The docstring guides the agent's decision-making.

### Multi-Tool Reasoning
Agents can chain multiple tools:

```
Question: "What's the population of Tokyo and 10% of that?"
Agent: 
1. Use Wikipedia → Get population
2. Use calculator → Calculate 10%
3. Return combined answer
```

### Iterative Problem Solving
Agents loop until task completion:

```
Thought → Action → Observation → Thought → Action → ... → Final Answer
```

---

## Example: Combined Tools

```python
# Setup multiple tools
tools = load_tools(["wikipedia", "llm-math"], llm=llm)

@tool
def current_date(text: str) -> str:
    """Returns today's date"""
    return str(date.today())

# Initialize with all tools
agent = initialize_agent(
    tools + [current_date],
    llm,
    agent=AgentType.CHAT_ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

# Complex query requiring multiple tools
agent("What year was Python created? How many years ago from today?")
```

**Agent decides:**
1. Wikipedia tool → Find Python creation year
2. Current_date tool → Get today's date
3. Calculator tool → Compute difference
4. Synthesize answer

---

## Debug Mode

Like L5 evaluation, view detailed execution:

```python
import langchain
langchain.debug = True

agent.run("Your question")

langchain.debug = False
```

Shows complete chain execution with all intermediate steps.

---

## Comparison: Different Approaches

### Scenario: "Sort customer list by last name"

**Without Agent (Manual):**
```python
# Write code yourself
customers.sort(key=lambda x: x[1])
```

**With Python Agent:**
```python
# Agent writes and executes code
agent.run("Sort this list by last name")
```

**Advantage:** Agent adapts to different requests without code changes.

---

## Setup Requirements

```python
# Core imports
from langchain.agents import load_tools, initialize_agent, AgentType, tool
from langchain_openai import ChatOpenAI

# Python agent (experimental)
pip install langchain-experimental
from langchain_experimental.agents.agent_toolkits import create_python_agent
from langchain_experimental.tools import PythonREPLTool
```

---

## Use Cases

| Task | Tool Type | Example |
|------|-----------|---------|
| **Research** | Wikipedia | "Who invented X?" |
| **Math** | Calculator | "What's 25% of 300?" |
| **Data Processing** | Python REPL | "Sort this list by age" |
| **Custom Logic** | Custom tool | "Check inventory for Product X" |
| **Multi-step** | Combined | "Research X, calculate Y, summarize" |

---

## Important Notes

### Agent Limitations
- Can make mistakes (experimental)
- May choose wrong tool
- Can produce incorrect reasoning

**Mitigation:** 
- Use `verbose=True` to debug
- Set `handle_parsing_errors=True`
- Retry if needed

### Python Agent Security
Executes arbitrary code - use with caution.

**Best practice:** Only in controlled environments.

---

## Comparison Table

| Feature | Chains (L3) | Q&A (L4) | Agents (L6) |
|---------|-------------|----------|-------------|
| **Workflow** | Fixed | Fixed | Dynamic |
| **Tools** | No | Retriever only | Multiple tools |
| **Decisions** | None | Similarity search | Reasoning loop |
| **Adaptability** | Low | Medium | High |

---

## Summary

**L6 introduces:**
- **Agents**: LLMs that use tools and make decisions
- **Built-in tools**: Math, Wikipedia
- **Python agent**: Dynamic code execution
- **Custom tools**: Define with `@tool` decorator
- **ReAct pattern**: Iterative Thought → Action → Observation

**Key difference:** Agents adapt strategies dynamically vs chains with fixed workflows.

**Purpose:** Build autonomous systems that can reason about which tools to use and solve complex multi-step problems.