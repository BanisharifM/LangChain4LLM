# LangChain Evaluation - Quick Reference

## Overview
Evaluation measures Q&A system accuracy by testing with known correct answers and grading predictions automatically.

## Why Evaluation?

After building a Q&A system, you need to:
- Measure accuracy (e.g., "85% correct")
- Identify failure patterns
- Track improvements over time
- Validate before deployment

---

## Core Concepts

### Test Examples
Pairs of questions with known correct answers used to test the system.

```python
{"query": "Question to test", "answer": "Known correct answer"}
```

### Predictions
System's actual responses to test questions.

```python
{"query": "Question to test", "result": "System's answer"}
```

### Evaluation
Comparing predictions against expected answers to grade correctness.

---

## Two Ways to Generate Test Examples

### 1. Manual Examples
Human-written test cases with guaranteed quality.

```python
examples = [
    {"query": "Do the Cozy Comfort Pullover Set have side pockets?", "answer": "Yes"},
    {"query": "What collection is the jacket from?", "answer": "The DownTek collection"}
]
```

**Use for:** Critical scenarios, edge cases

### 2. LLM-Generated Examples
Automatically create Q&A pairs from documents.

```python
from langchain.evaluation.qa import QAGenerateChain

gen_chain = QAGenerateChain.from_llm(llm)
new_examples = gen_chain.apply_and_parse([{"doc": document} for document in data])
```

**How it works:** LLM reads documents and generates relevant questions with answers.

**Use for:** Bulk testing, broad coverage

---

## Two Ways to Evaluate

### 1. Manual Evaluation (Debug Mode)
Inspect the system's internal process to understand behavior.

```python
import langchain
langchain.debug = True

qa.run("Test question")
```

**Shows:**
- Which documents were retrieved
- Exact prompt sent to LLM
- Complete execution trace

**Use for:** Debugging specific failures, understanding system behavior

### 2. Automated Evaluation (LLM Grading)
LLM compares expected vs actual answers and grades them.

```python
from langchain.evaluation.qa import QAEvalChain

eval_chain = QAEvalChain.from_llm(llm)
grades = eval_chain.evaluate(examples, predictions)
```

**How it works:** Another LLM judges if the predicted answer matches the expected answer.

**Output:** `CORRECT` or `INCORRECT` for each test case

**Use for:** Batch grading, calculating overall accuracy

---

## Evaluation Workflow

```
1. Create/Generate Test Examples
   (Questions + Expected Answers)
        ↓
2. Run Q&A System
   (Get Predictions)
        ↓
3. Evaluate Predictions
   (Manual Debug or LLM Grading)
        ↓
4. Analyze Results
   (Calculate accuracy, find patterns)
        ↓
5. Improve System
   (Fix issues, iterate)
```

---

## Key Components

### QAGenerateChain
Automatically creates test Q&A pairs from documents.

```python
QAGenerateChain.from_llm(llm)
```

**Input:** Document text  
**Output:** Question-answer pairs based on document content

### QAEvalChain
Grades predictions by comparing to expected answers.

```python
QAEvalChain.from_llm(llm)
```

**Input:** Expected answers + Predictions  
**Output:** Correctness grades (CORRECT/INCORRECT)

### Debug Mode
Shows complete chain execution details.

```python
langchain.debug = True  # Enable
langchain.debug = False # Disable
```

---

## Metrics

### Accuracy
Percentage of correct predictions.

```python
correct = sum(1 for g in grades if 'CORRECT' in g['text'])
accuracy = (correct / len(examples)) * 100
```

### Error Analysis
Identify which questions failed.

```python
failures = [ex for ex, grade in zip(examples, grades) if 'INCORRECT' in grade['text']]
```

---

## Comparison

| Aspect | Manual Examples | LLM-Generated |
|--------|----------------|---------------|
| Quality | High | Variable |
| Speed | Slow | Fast |
| Coverage | Limited | Broad |
| Best For | Critical tests | Bulk coverage |

| Method | Detail | Speed | Best For |
|--------|--------|-------|----------|
| Debug Mode | High | Slow | Understanding failures |
| LLM Grading | Medium | Fast | Batch evaluation |

---

## Complete Example

```python
# 1. Generate test examples
examples = [{"query": "...", "answer": "..."}]  # Manual
generated = gen_chain.apply_and_parse([{"doc": d} for d in data])
examples += [ex['qa_pairs'] for ex in generated]  # Auto-generated

# 2. Run predictions
predictions = qa.batch(examples)

# 3. Grade automatically
grades = eval_chain.evaluate(examples, predictions)

# 4. Calculate accuracy
correct = sum(1 for g in grades if 'CORRECT' in g['text'])
print(f"Accuracy: {correct / len(examples) * 100}%")
```

---

## Setup

```python
from langchain.evaluation.qa import QAGenerateChain, QAEvalChain
from langchain.chains import RetrievalQA
import langchain  # For debug mode
```

---

## Summary

**Evaluation in LangChain:**
- **Test Generation:** Manual (high quality) or LLM (fast/scalable)
- **Prediction:** Run Q&A system on test questions
- **Grading:** Manual debug or automated LLM evaluation
- **Metrics:** Accuracy, error analysis, performance tracking

**Purpose:** Measure system quality, identify weaknesses, validate improvements.