# LangChain Models, Prompts & Output Parsers - Quick Reference

## Overview
This notebook demonstrates three core LangChain components: Models, Prompt Templates, and Output Parsers for structured LLM outputs.

## Core Components

### 1. Models
Initialize the LLM with temperature control.

```python
from langchain_openai import ChatOpenAI

chat = ChatOpenAI(temperature=0.0, model="gpt-3.5-turbo")
```

**Temperature:**
- `0.0` = Deterministic, consistent outputs
- `1.0` = Creative, varied outputs

### 2. Prompt Templates
Reusable prompts with variable placeholders.

```python
from langchain.prompts import ChatPromptTemplate

template_string = """Translate the text \
that is delimited by triple backticks \
into a style that is {style}. \
text: ```{text}```
"""

prompt_template = ChatPromptTemplate.from_template(template_string)

# Format with variables
messages = prompt_template.format_messages(
    style="American English in a calm tone",
    text=customer_email
)
```

**Benefits:**
- Reusable across multiple inputs
- Consistent formatting
- Easy variable substitution

### 3. Output Parsers
Convert LLM string responses into structured Python objects.

#### Problem
```python
response = chat(messages)
type(response.content)  # str (not dict!)
response.content.get('gift')  # ❌ AttributeError
```

#### Solution: StructuredOutputParser
```python
from langchain.output_parsers import ResponseSchema, StructuredOutputParser

# Define schema
gift_schema = ResponseSchema(
    name="gift",
    description="Was the item purchased as a gift? Answer True or False."
)
delivery_days_schema = ResponseSchema(
    name="delivery_days",
    description="Days to arrive. Output -1 if not found."
)

response_schemas = [gift_schema, delivery_days_schema]
output_parser = StructuredOutputParser.from_response_schemas(response_schemas)

# Get format instructions
format_instructions = output_parser.get_format_instructions()

# Include in prompt
prompt = ChatPromptTemplate.from_template(
    "Extract info from: {text}\n{format_instructions}"
)

# Parse response
output_dict = output_parser.parse(response.content)
type(output_dict)  # dict ✅
output_dict.get('gift')  # Works! ✅
```

## Workflow Example

```python
# 1. Create template
template = "Translate {text} to {style}"
prompt = ChatPromptTemplate.from_template(template)

# 2. Format message
messages = prompt.format_messages(text="Hello", style="Spanish")

# 3. Get response
response = chat(messages)

# 4. Parse output (if structured data needed)
output_dict = output_parser.parse(response.content)
```

## Key Concepts

### Response Schema
Defines the structure of expected output:
- **name**: Field name in output
- **description**: Instructions for LLM on what to extract

### Format Instructions
Auto-generated JSON schema that tells the LLM how to format its response.

```python
format_instructions = output_parser.get_format_instructions()
# Returns markdown JSON schema with field descriptions
```

## Common Use Cases

| Component | Use Case |
|-----------|----------|
| **Models** | Control LLM behavior (temperature, model selection) |
| **Prompt Templates** | Reusable prompts with variables |
| **Output Parsers** | Extract structured data from LLM responses |

## Setup Requirements

```python
# Install packages
pip install langchain-openai langchain-community

# Import core components
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.output_parsers import ResponseSchema, StructuredOutputParser
```

## Before vs After Output Parsers

**Before:**
```python
response.content  # '{"gift": true, "days": 2}'  (string)
```

**After:**
```python
output_dict  # {'gift': True, 'days': 2}  (dict)
output_dict.get('gift')  # True (accessible!)
```