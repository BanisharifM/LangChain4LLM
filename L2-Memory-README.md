# LangChain Memory Types - Quick Reference

## Overview
This notebook demonstrates how to give LangChain conversational agents memory to retain context across interactions.

## Memory Types

### 1. ConversationBufferMemory
- **Stores:** Complete conversation history
- **Use Case:** Short conversations, demos
- **Limitation:** Can become very long and expensive

```python
memory = ConversationBufferMemory()
```

### 2. ConversationBufferWindowMemory
- **Stores:** Last `k` conversation turns
- **Use Case:** Fixed-size context window
- **Parameter:** `k=1` (keeps only 1 exchange)

```python
memory = ConversationBufferWindowMemory(k=1)
```

### 3. ConversationTokenBufferMemory
- **Stores:** Up to specified token limit
- **Use Case:** Budget control, API cost management
- **Parameter:** `max_token_limit=30`

```python
memory = ConversationTokenBufferMemory(llm=llm, max_token_limit=30)
```

### 4. ConversationSummaryBufferMemory
- **Stores:** Recent messages + summaries of older ones
- **Use Case:** Long conversations with context preservation
- **Parameter:** `max_token_limit=100`

```python
memory = ConversationSummaryBufferMemory(llm=llm, max_token_limit=100)
```

## Key Concepts

### Verbose Mode
```python
conversation = ConversationChain(llm=llm, memory=memory, verbose=True)
```
- `verbose=True`: Shows full prompts and memory content
- `verbose=False`: Silent execution

### Memory Operations
```python
# Save to memory
memory.save_context({"input": "Hi"}, {"output": "Hello"})

# Load from memory
memory.load_memory_variables({})
```

## Quick Comparison

| Type | Remembers | Pros | Cons |
|------|-----------|------|------|
| Buffer | Everything | Perfect recall | Expensive |
| BufferWindow | Last k turns | Size control | Forgets old data |
| TokenBuffer | Up to n tokens | Cost control | Unpredictable cutoff |
| SummaryBuffer | Recent + summary | Balanced | Extra LLM calls |

## Setup Requirements
```python
from langchain_openai import ChatOpenAI
from langchain.chains import ConversationChain
from langchain.memory import (
    ConversationBufferMemory,
    ConversationBufferWindowMemory,
    ConversationTokenBufferMemory,
    ConversationSummaryBufferMemory
)
```