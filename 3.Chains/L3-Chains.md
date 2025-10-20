# LangChain Chains - Quick Reference

## Overview
This notebook demonstrates how to chain multiple LLM operations together for complex workflows, including sequential processing and intelligent routing.

## Chain Types

### 1. LLMChain (Basic)
Single prompt-response chain.

```python
from langchain.chains import LLMChain

prompt = ChatPromptTemplate.from_template(
    "What is the best name for a company that makes {product}?"
)
chain = LLMChain(llm=llm, prompt=prompt)
chain.run("Queen Size Sheet Set")  # "Royal Bedding"
```

**Use Case:** Single-step tasks

---

### 2. SimpleSequentialChain
Linear pipeline where each output becomes the next input.

```python
from langchain.chains import SimpleSequentialChain

# Chain 1: Product → Company Name
chain_one = LLMChain(llm=llm, prompt=first_prompt)

# Chain 2: Company Name → Description
chain_two = LLMChain(llm=llm, prompt=second_prompt)

overall_chain = SimpleSequentialChain(
    chains=[chain_one, chain_two],
    verbose=True
)
overall_chain.run("Queen Size Sheet Set")
```

**Flow:**
```
Input → Chain 1 → Output1 → Chain 2 → Final Output
```

**Limitation:** Only one string passes between chains

---

### 3. SequentialChain
Multi-variable pipeline with named inputs/outputs.

```python
from langchain.chains import SequentialChain

# Define chains with specific output keys
chain_one = LLMChain(llm=llm, prompt=first_prompt, 
                     output_key="English_Review")
chain_two = LLMChain(llm=llm, prompt=second_prompt,
                     output_key="summary")

overall_chain = SequentialChain(
    chains=[chain_one, chain_two, chain_three, chain_four],
    input_variables=["Review"],
    output_variables=["English_Review", "summary", "followup_message"],
    verbose=True
)

result = overall_chain(review)
```

**Example Use Case (from notebook):**
1. Translate review to English → `English_Review`
2. Summarize English review → `summary`
3. Detect original language → `language`
4. Generate response in original language → `followup_message`

**Advantage:** Multiple named outputs accessible by any chain

---

### 4. Router Chain (MultiPromptChain)
Intelligently routes questions to specialized expert chains.

```python
from langchain.chains.router import MultiPromptChain
from langchain.chains.router.llm_router import LLMRouterChain

# Define expert prompts
prompt_infos = [
    {
        "name": "physics",
        "description": "Good for answering physics questions",
        "prompt_template": physics_template
    },
    {
        "name": "math",
        "description": "Good for answering math questions", 
        "prompt_template": math_template
    }
]

# Create destination chains
destination_chains = {}
for p_info in prompt_infos:
    chain = LLMChain(llm=llm, prompt=prompt)
    destination_chains[p_info["name"]] = chain

# Create router
router_chain = LLMRouterChain.from_llm(llm, router_prompt)

# Combine into MultiPromptChain
chain = MultiPromptChain(
    router_chain=router_chain,
    destination_chains=destination_chains,
    default_chain=default_chain,
    verbose=True
)
```

**Usage:**
```python
chain.run("What is black body radiation?")  
# Routes to → physics chain

chain.run("what is 2 + 2")  
# Routes to → math chain

chain.run("Why does DNA exist?")  
# No match → default chain
```

**How It Works:**
1. Router analyzes input
2. Compares against chain descriptions
3. Selects best matching expert
4. Routes to that chain (or DEFAULT)

---

## Key Concepts

### Output Keys
In SequentialChain, specify output keys to pass multiple variables:

```python
chain_one = LLMChain(llm=llm, prompt=prompt, output_key="English_Review")
chain_two = LLMChain(llm=llm, prompt=prompt, output_key="summary")
```

### Verbose Mode
Shows chain execution flow:

```python
chain = SimpleSequentialChain(chains=[...], verbose=True)
```

Output:
```
> Entering new SimpleSequentialChain chain...
Royal Comfort Bedding
Royal Comfort Bedding provides luxurious...
> Finished chain.
```

---

## Comparison Table

| Chain Type | Variables Passed | Use Case |
|------------|------------------|----------|
| **LLMChain** | N/A | Single operation |
| **SimpleSequentialChain** | One string | Linear pipeline, simple |
| **SequentialChain** | Multiple named | Complex workflows |
| **Router Chain** | Routes by topic | Different question types |

---

## Setup Requirements

```python
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.chains import (
    LLMChain,
    SimpleSequentialChain,
    SequentialChain
)
from langchain.chains.router import MultiPromptChain
from langchain.chains.router.llm_router import LLMRouterChain, RouterOutputParser
```

---

## Real-World Example

**Multi-language Review Processing:**
```python
# Process reviews in any language
# 1. Detect language
# 2. Translate to English  
# 3. Summarize
# 4. Generate follow-up in original language

overall_chain = SequentialChain(
    chains=[translate_chain, summary_chain, language_chain, response_chain],
    input_variables=["Review"],
    output_variables=["English_Review", "summary", "followup_message"]
)

result = overall_chain(french_review)
# All 4 steps execute automatically!
```