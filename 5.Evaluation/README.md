# LangChain Evaluation - Quick Reference

## Overview
Demonstrates how to evaluate Q&A system accuracy using test examples and automated grading.

## Core Components

### 1. Test Example Generation

#### Manual Examples
```python
examples = [
    {"query": "Do the Cozy Comfort Pullover Set have side pockets?", "answer": "Yes"},
    {"query": "What collection is the Ultra-Lofty 850 Jacket from?", "answer": "The DownTek collection"}
]
```

#### LLM-Generated Examples
```python
from langchain.evaluation.qa import QAGenerateChain

example_gen_chain = QAGenerateChain.from_llm(ChatOpenAI())
new_examples = example_gen_chain.apply_and_parse([{"doc": t} for t in data[:5]])

# Extract qa_pairs (output is nested)
examples += [ex['qa_pairs'] for ex in new_examples]
```

**Output structure:**
```python
{'qa_pairs': {'query': '...', 'answer': '...'}}  # Must extract qa_pairs
```

---

### 2. Run Predictions

```python
# Batch process all test examples
predictions = qa.batch(examples)
```

**Returns:**
```python
[{'query': '...', 'result': 'System answer'}, ...]
```

---

### 3. Evaluation Methods

#### Debug Mode (Manual)
```python
import langchain
langchain.debug = True

qa.run(examples[0]["query"])
# Shows: retrieved docs, prompt, LLM output, token usage

langchain.debug = False  # Turn off
```

#### LLM-Assisted Grading (Automated)
```python
from langchain.evaluation.qa import QAEvalChain

eval_chain = QAEvalChain.from_llm(llm)
graded_outputs = eval_chain.evaluate(examples, predictions)
```

**Output:**
```python
[{'text': 'CORRECT'}, {'text': 'INCORRECT'}, ...]
```

---

## Complete Workflow

```python
# 1. Generate test examples
examples = [{"query": "...", "answer": "..."}]
generated = example_gen_chain.apply_and_parse([{"doc": d} for d in data[:5]])
examples += [ex['qa_pairs'] for ex in generated]

# 2. Run predictions
predictions = qa.batch(examples)

# 3. Grade results
grades = eval_chain.evaluate(examples, predictions)

# 4. Calculate accuracy
correct = sum(1 for g in grades if 'CORRECT' in g['text'])
accuracy = (correct / len(examples)) * 100
```

---

## Data Structures

### Test Example
```python
{"query": "Question", "answer": "Expected answer"}
```

### Prediction
```python
{"query": "Question", "result": "System's answer"}
```

### Evaluation Output
```python
{"text": "CORRECT" or "INCORRECT"}
```

---

## Comparison

| Method | Generation | Speed | Quality |
|--------|-----------|-------|---------|
| **Manual Examples** | Human-written | Slow | High |
| **LLM-Generated** | Automatic | Fast | Variable |

| Evaluation | Detail | Speed | Use Case |
|-----------|--------|-------|----------|
| **Debug Mode** | High | Slow | Understanding failures |
| **LLM Grading** | Medium | Fast | Batch evaluation |

---

## Common Issues

### Nested qa_pairs Structure
```python
# ❌ Wrong
examples += new_examples

# ✅ Correct  
examples += [ex['qa_pairs'] for ex in new_examples]
```

### Multiple Cell Executions
Avoid running `examples +=` multiple times - it accumulates duplicates. Restart kernel or use:
```python
examples = base_examples + [ex['qa_pairs'] for ex in new_examples]
```

---

## Setup Requirements

```python
from langchain.evaluation.qa import QAGenerateChain, QAEvalChain
import langchain  # For debug mode
```

---

## Key Metrics

```python
# Accuracy
correct = sum(1 for g in grades if 'CORRECT' in g['text'])
accuracy = (correct / len(examples)) * 100

# Failures
failures = [(ex, pred) for ex, pred, g in zip(examples, predictions, grades) 
            if 'INCORRECT' in g['text']]
```

---

## Summary

**L5 teaches:**
- Generate test examples (manual + LLM)
- Run batch predictions
- Evaluate with debug mode or LLM grading
- Calculate accuracy metrics

**Use Case:** Measure and improve Q&A system performance through automated testing.

