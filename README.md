# LangChain for LLM Applications

A comprehensive guide to building LLM applications using LangChain, covering foundational concepts to advanced agent systems.

## Overview

This repository contains 6 progressive lessons demonstrating key LangChain concepts:
- Building blocks (models, prompts, parsers)
- Conversational memory
- Workflow orchestration (chains)
- Document Q&A systems
- Evaluation and testing
- Autonomous agents with tools

---

## Repository Structure

```
LangChain4LLM/
├── 1.Models,Prompts,Parsers/    # Foundation components
├── 2.Memory/                    # Conversation state management
├── 3.Chains/                    # Workflow orchestration
├── 4.Question&Answer/           # RAG systems
├── 5.Evaluation/                # Testing and metrics
└── 6.Agents/                    # Autonomous tool-using systems
```

---

## Lessons

### [L1: Models, Prompts & Output Parsers](./1.Models,Prompts,Parsers/)

**Foundation components for LLM applications.**

- **Models**: LLM initialization and configuration
- **Prompt Templates**: Reusable prompts with variables
- **Output Parsers**: Convert LLM strings to structured data (JSON, dicts)

**Key Concept:** Structure LLM interactions with templates and parse responses into usable formats.

---

### [L2: Memory](./2.Memory/)

**Give LLMs conversational memory.**

Four memory types:
- **ConversationBufferMemory**: Stores complete history
- **ConversationBufferWindowMemory**: Keeps last K turns
- **ConversationTokenBufferMemory**: Limits by token count
- **ConversationSummaryBufferMemory**: Stores recent + summaries

**Key Concept:** Manage conversation context to control costs and maintain coherent multi-turn dialogs.

---

### [L3: Chains](./3.Chains/)

**Link multiple LLM operations into workflows.**

Four chain types:
- **LLMChain**: Single prompt-response
- **SimpleSequentialChain**: Linear pipeline (output → next input)
- **SequentialChain**: Multi-variable pipeline with named inputs/outputs
- **Router Chain**: Intelligent routing to specialized chains

**Key Concept:** Build complex workflows by chaining operations, either sequentially or conditionally.

---

### [L4: Question & Answer](./4.Question&Answer/)

**Query your own documents using RAG (Retrieval-Augmented Generation).**

**Process:**
1. Load documents (CSV, PDF, etc.)
2. Create embeddings (vector representations)
3. Store in vector database
4. Retrieve relevant docs for queries
5. LLM answers using retrieved context

**Chain Types:**
- **Stuff**: All docs in one prompt
- **Map-Reduce**: Process chunks in parallel, then combine
- **Refine**: Iteratively improve answer
- **Map-Rerank**: Score answers, pick best

**Key Concept:** Enable LLMs to answer questions about custom data through semantic search and retrieval.

---

### [L5: Evaluation](./5.Evaluation/)

**Measure and improve Q&A system accuracy.**

**Test Generation:**
- Manual examples (high quality, controlled)
- LLM-generated examples (fast, broad coverage)

**Evaluation Methods:**
- Debug mode (manual inspection of chain execution)
- LLM-assisted grading (automated CORRECT/INCORRECT scoring)

**Key Concept:** Systematic testing with known correct answers to quantify accuracy and identify failures.

---

### [L6: Agents](./6.Agents/)

**LLMs that use tools and make decisions autonomously.**

**Agent = LLM + Tools + Decision Loop**

**Tool Types:**
- **Built-in**: Math calculator, Wikipedia search
- **Python Agent**: Dynamic code execution
- **Custom**: User-defined with `@tool` decorator

**ReAct Pattern:**
```
Thought → Action → Observation → Thought → ... → Final Answer
```

**Key Concept:** Agents adaptively choose tools and strategies, unlike chains with fixed workflows.

---

## Key Differences

| Component | Decision Making | Tools | Workflow | Use Case |
|-----------|----------------|-------|----------|----------|
| **Prompt** | None | None | Single call | Basic Q&A |
| **Chain** | None | Limited | Fixed sequence | Multi-step tasks |
| **Agent** | Dynamic | Multiple | Adaptive | Complex reasoning |

---

## Progressive Complexity

```
L1: Basic LLM interactions
  ↓
L2: + Conversation memory
  ↓
L3: + Multi-step workflows
  ↓
L4: + Document retrieval (RAG)
  ↓
L5: + Testing & evaluation
  ↓
L6: + Autonomous tool use
```

---

## Prerequisites

### Core Dependencies
```bash
pip install langchain langchain-openai langchain-community
pip install python-dotenv openai
pip install docarray tiktoken
```

### Lesson-Specific
```bash
# L4: Vector stores
pip install faiss-cpu  # or chromadb

# L6: Agents (experimental)
pip install langchain-experimental
```

### Environment Setup
Create `.env` file:
```bash
OPENAI_API_KEY=your_key_here
```

---

## Quick Start

Each lesson folder contains:
- **Jupyter Notebook**: Hands-on implementation
- **README.md**: Concept explanation and reference
- **Data files**: Sample datasets (where applicable)

**Recommended path:** Follow lessons sequentially (L1 → L6) as each builds on previous concepts.

---

## Core Concepts Summary

### Embeddings
Vector representations of text enabling semantic similarity search.

### Vector Stores
Databases optimized for similarity search (DocArrayInMemorySearch, Chroma, Pinecone).

### Retrieval
Finding relevant documents based on query similarity.

### RAG (Retrieval-Augmented Generation)
Pattern combining retrieval + LLM generation for answering questions about custom data.

### ReAct (Reason + Act)
Agent pattern alternating between reasoning and tool use.

---

## Comparison: Traditional vs LangChain Approach

### Traditional
```python
# Manual prompt construction
prompt = f"Translate: {text}"
response = openai.ChatCompletion.create(...)
# Manual parsing
result = json.loads(response)
```

### LangChain
```python
# Structured approach
prompt = ChatPromptTemplate.from_template("Translate: {text}")
chain = LLMChain(llm=llm, prompt=prompt)
parser = StructuredOutputParser(...)
result = parser.parse(chain.run(text))
```

**Advantages:**
- Reusable components
- Built-in memory management
- Tool integration
- Standardized patterns

---

## Use Cases by Lesson

| Lesson | Use Case Examples |
|--------|------------------|
| **L1** | Chatbots, text transformation, data extraction |
| **L2** | Customer support, tutoring systems, personal assistants |
| **L3** | Data pipelines, multi-language workflows, routing systems |
| **L4** | Document Q&A, knowledge bases, semantic search |
| **L5** | System validation, accuracy tracking, CI/CD testing |
| **L6** | Research assistants, data analysis, task automation |

---

## When to Use What

### Use Chains When:
- Workflow is predictable
- Steps are always the same
- No dynamic decision-making needed

### Use Agents When:
- Task requires multiple tools
- Solution path is unclear
- Need adaptive problem-solving

### Use RAG When:
- Answering questions about custom documents
- LLM needs access to proprietary data
- Information changes frequently

---

## Notes

- Examples use OpenAI models but can be adapted to other providers
- Agent functionality is experimental and may produce errors
- Always use `verbose=True` when debugging
- Memory management is critical for production systems

---

## Additional Resources

- [LangChain Documentation](https://python.langchain.com/)
- [OpenAI API Reference](https://platform.openai.com/docs/)
- [Vector Database Comparison](https://www.sicara.fr/blog-technique/vector-databases-comparison)

---

**Built with:** LangChain, OpenAI, Python 3.9+