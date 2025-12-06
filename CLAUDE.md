# CLAUDE.md - AI Assistant Guide for Recipe Chatbot

**Last Updated**: 2025-12-06
**Repository**: Recipe Chatbot - AI Evaluations Course

## Table of Contents
1. [Overview](#overview)
2. [Repository Structure](#repository-structure)
3. [Development Workflows](#development-workflows)
4. [Key Conventions](#key-conventions)
5. [Common Tasks](#common-tasks)
6. [Important Files Reference](#important-files-reference)
7. [LLM Integration Patterns](#llm-integration-patterns)
8. [Testing & Evaluation](#testing--evaluation)
9. [Gotchas & Best Practices](#gotchas--best-practices)

---

## Overview

### Purpose
This repository is an **educational AI evaluations course** built around a Recipe Chatbot. It teaches practical techniques for evaluating and improving AI systems through 5 progressive homework assignments.

### Core Technologies
- **Backend**: FastAPI + LiteLLM (multi-provider LLM support)
- **Frontend**: Vanilla HTML/CSS/JS chat interface
- **Retrieval**: BM25-based recipe search
- **Evaluation**: `judgy` library for bias-corrected metrics
- **Annotation**: FastHTML-based manual labeling tools

### Key Features
- Multi-model LLM support (OpenAI, Anthropic, etc.)
- Query rewriting with LLM agents
- Manual and automated evaluation pipelines
- Conversation trace analysis
- Progressive learning through homeworks (HW1-HW5)

---

## Repository Structure

### High-Level Layout
```
recipe-chatbot/
├── backend/              # Core FastAPI application
├── frontend/             # Chat UI (single HTML file)
├── homeworks/            # 5 progressive assignments (hw1-hw5)
├── annotation/           # Manual annotation web interface
├── scripts/              # Utility scripts (bulk testing)
├── lesson-*/             # Supplementary lesson materials
├── data/                 # Sample queries
├── requirements.txt      # Python dependencies
└── env.example           # Environment template
```

### Critical Directories

#### `/backend/` - Core Application
| File | Purpose | Key Components |
|------|---------|----------------|
| `main.py` | FastAPI app entry point | `/chat` endpoint, request/response models |
| `utils.py` | LLM integration & system prompt | `get_agent_response()`, `SYSTEM_PROMPT`, `MODEL_NAME` |
| `retrieval.py` | BM25 recipe search | `RecipeRetriever` class, index caching |
| `query_rewrite_agent.py` | Query optimization | `QueryRewriteAgent` with 3 strategies |
| `evaluation_utils.py` | Retrieval metrics | `BaseRetrievalEvaluator`, Recall@k, MRR |

#### `/homeworks/` - Progressive Course Content

**HW1: Basic Prompt Engineering**
- Focus: System prompts and test query expansion
- See HW2 walkthrough for HW1 content

**HW2: Error Analysis & Failure Taxonomy** (`homeworks/hw2/`)
- Focus: Systematic error analysis, failure mode identification
- Key files: `generate_synthetic_queries.py`, `hw2_solution_walkthrough.ipynb`
- Output: `failure_mode_taxonomy.md`, `error_analysis_template.csv`

**HW3: LLM-as-Judge Evaluation** (`homeworks/hw3/`)
- Focus: Automated evaluation using dietary adherence checks
- Pipeline: `generate_traces.py` → `label_data.py` → `split_data.py` → `develop_judge.py` → `evaluate_judge.py`
- Data: ~2400 raw traces → 150 labeled → train/dev/test splits
- Output: `judge_prompt.txt`, `judge_performance.json`, TPR/TNR metrics

**HW4: RAG/Retrieval Evaluation** (`homeworks/hw4/`)
- Focus: BM25 retrieval with synthetic query generation
- Pipeline: `process_recipes.py` → `generate_queries.py` → `evaluate_retrieval.py`
- Data: 200 longest recipes, synthetic queries
- Output: `retrieval_evaluation.json` with Recall@k, MRR metrics

**HW5: Agent Failure Analysis** (`homeworks/hw5/`)
- Focus: Conversation trace analysis and failure transitions
- Key: `transition_heatmaps.py` for state transition visualization
- Data: 100 labeled traces
- Output: `failure_transition_heatmap.png`

#### `/annotation/` - Manual Labeling Tools
- `annotation.py`: FastHTML web UI for trace annotation
- `traces/`: Saved conversation traces (JSON format)
- Supports open coding and axial coding workflows

---

## Development Workflows

### Running the Application

#### 1. Initial Setup
```bash
# Clone and setup virtual environment
git clone <repo-url>
cd recipe-chatbot
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Configure environment
cp env.example .env
# Edit .env to add MODEL_NAME, MODEL_NAME_JUDGE, API keys
```

#### 2. Start the Chatbot
```bash
uvicorn backend.main:app --reload
# Open http://127.0.0.1:8000
```

#### 3. Run Annotation Tool
```bash
python annotation/annotation.py
# Access annotation interface in browser
```

### Homework Execution Patterns

#### HW3 Full Pipeline
```bash
cd homeworks/hw3
python scripts/generate_traces.py      # Generate ~2400 traces
python scripts/label_data.py           # Create ground truth (150)
python scripts/split_data.py           # Train/dev/test split
python scripts/develop_judge.py        # Develop judge prompt
python scripts/evaluate_judge.py       # Calculate TPR/TNR
python scripts/run_full_evaluation.py  # Bias-corrected metrics
```

#### HW4 Full Pipeline
```bash
cd homeworks/hw4
python scripts/process_recipes.py                    # Clean recipe data
python scripts/generate_queries.py                   # Generate test queries
python scripts/evaluate_retrieval.py                 # Baseline BM25
python scripts/evaluate_retrieval_with_agent.py      # Enhanced retrieval
```

### Bulk Testing
```bash
python scripts/bulk_test.py
# Concurrent query testing with rich console output
```

---

## Key Conventions

### Code Style

#### Type Hints & Annotations
- **Required**: All functions have full type hints
- **Pattern**: `from __future__ import annotations` at file top
- **Example**:
  ```python
  from __future__ import annotations
  from typing import List, Dict, Any, Optional, Final

  def get_agent_response(messages: List[Dict[str, str]]) -> str:
      """Get LLM response for chat messages."""
      ...
  ```

#### Naming Conventions
| Type | Convention | Example |
|------|-----------|---------|
| Functions | `snake_case` | `get_agent_response`, `retrieve_bm25` |
| Classes | `PascalCase` | `RecipeRetriever`, `QueryRewriteAgent` |
| Constants | `UPPER_CASE` | `SYSTEM_PROMPT`, `MODEL_NAME` |
| Private helpers | `_prefixed` | `_process_query_with_retry` |
| CSV columns | `Title_Case` or `lowercase_with_underscores` | Context-dependent |

#### Import Organization
Standard order:
1. Future imports (`from __future__ import annotations`)
2. Standard library (`import os, json, datetime`)
3. Third-party packages (`import pandas, litellm, fastapi`)
4. Local modules (`from backend.utils import ...`)

### File Organization

#### Module Structure Pattern
```python
"""
Module docstring: 1-2 sentences describing purpose.

Key classes/functions:
- ClassName: Description
- function_name: Description
"""
from __future__ import annotations

# Imports
from typing import List, Dict, Any, Final
import litellm

# Constants
SYSTEM_PROMPT: Final[str] = "..."

# Classes
class MyClass:
    """Class docstring."""
    pass

# Functions
def my_function(param: str) -> str:
    """
    Function docstring.

    Parameters:
        param: Description

    Returns:
        Description of return value
    """
    pass
```

#### Data File Organization
- **Input data**: `data/` subdirectories (CSV/JSON)
- **Processing scripts**: `scripts/` subdirectories
- **Output results**: `results/` subdirectories
- **Walkthroughs**: `.ipynb` (Jupyter) or `.py` (Marimo) in homework roots

### Error Handling

#### Graceful Degradation Pattern
```python
try:
    # Attempt enhanced operation
    result = complex_llm_operation()
except Exception as e:
    # Fallback to simpler operation
    print(f"⚠️  Warning: {e}")
    result = simple_fallback()
```

#### No Exceptions for Expected Failures
- Query rewrite failures → return original query
- Missing data → skip with warning, continue processing
- LLM timeouts → retry with exponential backoff

---

## Common Tasks

### Adding New LLM Functionality

#### 1. Basic LLM Call Pattern
```python
import litellm
from dotenv import load_dotenv
import os

load_dotenv()

response = litellm.completion(
    model=os.environ.get("MODEL_NAME"),
    messages=[{"role": "user", "content": prompt}],
    temperature=0.3,  # Lower = more deterministic
    max_tokens=200
)
answer = response.choices[0].message.content.strip()
```

#### 2. Structured Output with Pydantic
```python
from pydantic import BaseModel, Field
from typing import List

class RecipeQuery(BaseModel):
    query: str = Field(..., description="Search query")
    dietary_restrictions: List[str] = Field(
        default_factory=list,
        description="Dietary restrictions"
    )

# In LLM call, add response_format
response = litellm.completion(
    model=MODEL_NAME,
    messages=[{"role": "user", "content": prompt}],
    response_format=RecipeQuery
)
```

#### 3. Parallel LLM Calls Pattern
```python
from concurrent.futures import ThreadPoolExecutor, as_completed
from tqdm import tqdm

def process_item(item):
    # LLM processing
    return litellm.completion(...)

items = [...]  # Your data
results = []

with ThreadPoolExecutor(max_workers=10) as executor:
    futures = {executor.submit(process_item, item): item for item in items}
    for future in tqdm(as_completed(futures), total=len(items)):
        try:
            result = future.result()
            results.append(result)
        except Exception as e:
            print(f"Error: {e}")
```

### Adding New Evaluation Metrics

#### 1. Extend BaseRetrievalEvaluator
```python
from backend.evaluation_utils import BaseRetrievalEvaluator

class MyCustomEvaluator(BaseRetrievalEvaluator):
    def evaluate_query(self, query: str, relevant_ids: List[int]) -> Dict:
        # Retrieve results
        results = self.retriever.retrieve(query, k=self.k)

        # Calculate custom metric
        my_metric = calculate_my_metric(results, relevant_ids)

        return {
            "query": query,
            "my_metric": my_metric,
            "results": results
        }
```

#### 2. Save Evaluation Results
```python
import json
from datetime import datetime

results = evaluator.evaluate_all(queries)

# Save with timestamp
timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
output_path = f"results/evaluation_{timestamp}.json"

with open(output_path, "w") as f:
    json.dump(results, f, indent=2)

print(f"✅ Results saved to {output_path}")
```

### Working with Conversation Traces

#### Trace Format
```python
{
    "request": {
        "messages": [
            {"role": "system", "content": "..."},
            {"role": "user", "content": "..."}
        ]
    },
    "response": {
        "messages": [
            {"role": "assistant", "content": "..."}
        ]
    },
    "metadata": {
        "timestamp": "2025-12-06T10:00:00",
        "model": "gpt-4-mini"
    },
    "open_coding": "Optional annotation",
    "axial_coding_code": "failure_category"
}
```

#### Reading Traces
```python
import json

with open("annotation/traces/trace_123.json") as f:
    trace = json.load(f)

user_message = trace["request"]["messages"][-1]["content"]
assistant_message = trace["response"]["messages"][0]["content"]
```

#### Saving Traces
```python
from datetime import datetime
import json

def save_trace(request, response, metadata=None):
    trace = {
        "request": request,
        "response": response,
        "metadata": metadata or {},
        "timestamp": datetime.now().isoformat()
    }

    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S_%f")
    filename = f"annotation/traces/trace_{timestamp}.json"

    with open(filename, "w") as f:
        json.dump(trace, f, indent=2)

    return filename
```

### Modifying the System Prompt

**Location**: `backend/utils.py`

```python
SYSTEM_PROMPT: Final[str] = """
You are a helpful recipe chatbot assistant.

Your capabilities:
1. Search for recipes based on user queries
2. Provide cooking instructions and ingredient lists
3. Answer questions about dietary restrictions

Guidelines:
- Be concise and helpful
- Ask clarifying questions when needed
- Only recommend recipes from the database
"""
```

**Testing Changes**:
1. Modify `SYSTEM_PROMPT` in `backend/utils.py`
2. Restart server: `uvicorn backend.main:app --reload`
3. Test with `scripts/bulk_test.py` or web interface

---

## Important Files Reference

### Must-Read Files for Common Tasks

#### Understanding the Application
1. `README.md` - Project overview and quick start
2. `backend/main.py` - FastAPI application entry point
3. `backend/utils.py` - LLM configuration and system prompt
4. `frontend/index.html` - Chat interface implementation

#### Working with Retrieval
1. `backend/retrieval.py` - BM25 implementation
2. `backend/query_rewrite_agent.py` - Query optimization
3. `backend/evaluation_utils.py` - Retrieval metrics
4. `homeworks/hw4/README.md` - Retrieval evaluation guide

#### Developing Judges
1. `homeworks/hw3/scripts/develop_judge.py` - Judge development pattern
2. `homeworks/hw3/scripts/evaluate_judge.py` - TPR/TNR calculation
3. `lesson-4/judge_substantiation.py` - Real-world judge example

#### Manual Annotation
1. `annotation/annotation.py` - Annotation tool implementation
2. `lesson-7/labeling-tool/main.py` - Alternative labeling interface

### Configuration Files

#### Environment Variables (`env.example`)
```bash
# Chatbot LLM (general-purpose, capable model)
MODEL_NAME=openai/gpt-4.1-mini

# Judge LLM (cheaper, smaller model for evaluations)
MODEL_NAME_JUDGE=openai/gpt-4.1-nano

# API Keys
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
TOGETHER_API_KEY=...
```

**Supported Providers** (via LiteLLM):
- OpenAI: `openai/gpt-4`, `openai/gpt-4-mini`
- Anthropic: `anthropic/claude-3-sonnet-20240229`
- Together: `together_ai/meta-llama/Llama-3-70b-chat-hf`
- See [LiteLLM docs](https://docs.litellm.ai/docs/providers) for full list

#### Dependencies (`requirements.txt`)
Core dependencies:
- `fastapi`, `uvicorn` - Web framework
- `litellm` - Multi-provider LLM interface
- `python-dotenv` - Environment management
- `judgy` - Bias-corrected evaluation metrics
- `rank-bm25` - BM25 retrieval
- `pydantic` - Data validation
- `rich` - Terminal formatting
- `pandas`, `numpy` - Data processing
- `matplotlib`, `seaborn`, `plotly` - Visualization

---

## LLM Integration Patterns

### Multi-Model Strategy

#### Two-Model Architecture
```python
import os
from dotenv import load_dotenv

load_dotenv()

# Chatbot: General-purpose, capable model
CHATBOT_MODEL = os.environ.get("MODEL_NAME", "openai/gpt-4-mini")

# Judge: Cheaper, smaller model for evaluations
JUDGE_MODEL = os.environ.get("MODEL_NAME_JUDGE", "openai/gpt-4-nano")
```

**Rationale**:
- Chatbot needs reasoning capability → use capable model
- Judge performs simple classification → use cheaper model
- Cost optimization: Judges run 1000s of times in evaluation pipelines

### Temperature Settings

```python
# Deterministic outputs (evaluation, structured data)
temperature = 0.3

# Creative outputs (query generation, diverse responses)
temperature = 0.7

# Maximum randomness (brainstorming, exploration)
temperature = 1.0
```

### Retry Pattern with Exponential Backoff

```python
import time

def llm_call_with_retry(prompt: str, max_retries: int = 3) -> str:
    for attempt in range(max_retries):
        try:
            response = litellm.completion(
                model=MODEL_NAME,
                messages=[{"role": "user", "content": prompt}],
                timeout=30
            )
            return response.choices[0].message.content.strip()
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            wait_time = 2 ** attempt  # 1s, 2s, 4s
            print(f"⚠️  Retry {attempt + 1}/{max_retries} after {wait_time}s")
            time.sleep(wait_time)
```

### Cost Monitoring

```python
def estimate_cost(num_calls: int, avg_tokens: int = 500):
    """
    Rough cost estimation for planning.

    Example pricing (2025):
    - gpt-4-mini: $0.15/1M input tokens
    - gpt-4-nano: $0.075/1M input tokens
    """
    cost_per_token = 0.15 / 1_000_000  # gpt-4-mini
    total_cost = num_calls * avg_tokens * cost_per_token
    print(f"💰 Estimated cost: ${total_cost:.2f}")
    return total_cost
```

---

## Testing & Evaluation

### No Traditional Unit Tests
This repository uses **evaluation-driven testing** instead of pytest/unittest:
- HW3: Judge TPR/TNR on test sets
- HW4: Retrieval Recall@k, MRR metrics
- HW5: Transition matrix analysis
- Manual: Bulk testing with `scripts/bulk_test.py`

### Running Evaluations

#### HW3: Judge Evaluation
```bash
cd homeworks/hw3
python scripts/evaluate_judge.py

# Output: judge_performance.json
{
    "true_positive_rate": 0.85,
    "true_negative_rate": 0.92,
    "accuracy": 0.89,
    "confusion_matrix": [[tn, fp], [fn, tp]]
}
```

#### HW4: Retrieval Evaluation
```bash
cd homeworks/hw4
python scripts/evaluate_retrieval.py

# Output: retrieval_evaluation.json
{
    "recall@5": 0.75,
    "recall@10": 0.88,
    "mrr": 0.62,
    "per_query_results": [...]
}
```

#### Bulk Testing
```bash
python scripts/bulk_test.py

# Interactive: Prompts for queries from data/sample_queries.csv
# Parallel execution with rich console output
# Shows: Query, Response, Latency, Status
```

### Evaluation Metrics Reference

#### Retrieval Metrics
- **Recall@k**: Fraction of relevant documents in top-k results
- **MRR (Mean Reciprocal Rank)**: Average of 1/rank for first relevant doc
- **Precision@k**: Fraction of relevant docs among retrieved docs

#### Classification Metrics (Judges)
- **TPR (True Positive Rate)**: Sensitivity, recall for positive class
- **TNR (True Negative Rate)**: Specificity, recall for negative class
- **Accuracy**: Overall correctness
- **Bias-Corrected Metrics**: Using `judgy` library to adjust for label imbalance

### Data Validation

#### CSV Parsing with Error Handling
```python
import pandas as pd

try:
    df = pd.read_csv("data/recipes.csv", encoding="utf-8")
except UnicodeDecodeError:
    df = pd.read_csv("data/recipes.csv", encoding="latin-1")

# Validate required columns
required_cols = ["name", "ingredients", "steps"]
missing = set(required_cols) - set(df.columns)
if missing:
    raise ValueError(f"Missing columns: {missing}")
```

#### Pydantic Validation
```python
from pydantic import BaseModel, validator

class Recipe(BaseModel):
    name: str
    ingredients: List[str]
    steps: List[str]

    @validator("ingredients", "steps")
    def validate_non_empty(cls, v):
        if not v:
            raise ValueError("Cannot be empty")
        return v

# Use in data processing
recipes = [Recipe(**row) for row in data]
```

---

## Gotchas & Best Practices

### Common Pitfalls

#### 1. Environment Variables Not Loaded
**Problem**: `MODEL_NAME` is None, LLM calls fail

**Solution**:
```python
from dotenv import load_dotenv
import os

load_dotenv()  # Must call before accessing os.environ

MODEL_NAME = os.environ.get("MODEL_NAME")
if not MODEL_NAME:
    raise ValueError("MODEL_NAME not set in .env")
```

#### 2. BM25 Index Not Cached
**Problem**: Slow retrieval performance, re-indexing on every query

**Solution**: `RecipeRetriever` automatically caches index in `backend/retrieval.py`
```python
# Index is cached in memory after first build
retriever = RecipeRetriever(recipes)  # First call: builds index
retriever.retrieve("pasta")           # Subsequent calls: uses cache
```

#### 3. Trace File Naming Conflicts
**Problem**: Overwriting traces with same timestamp

**Solution**: Use nanosecond precision
```python
from datetime import datetime

# Good: Includes microseconds
timestamp = datetime.now().strftime("%Y%m%d_%H%M%S_%f")

# Bad: Only second precision
timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
```

#### 4. CSV Encoding Issues
**Problem**: `UnicodeDecodeError` when reading recipe data

**Solution**: Try multiple encodings
```python
for encoding in ["utf-8", "latin-1", "cp1252"]:
    try:
        df = pd.read_csv(filepath, encoding=encoding)
        break
    except UnicodeDecodeError:
        continue
```

#### 5. Parallel LLM Calls Exceeding Rate Limits
**Problem**: 429 errors with high concurrency

**Solution**: Limit max_workers
```python
# Too aggressive
with ThreadPoolExecutor(max_workers=50) as executor:
    ...

# Better: Respect rate limits
with ThreadPoolExecutor(max_workers=10) as executor:
    ...
```

### Best Practices

#### 1. Always Use Type Hints
```python
# Good
def get_response(messages: List[Dict[str, str]]) -> str:
    ...

# Bad
def get_response(messages):
    ...
```

#### 2. Save Intermediate Results
```python
# Good: Save after each pipeline step
traces = generate_traces()
save_json(traces, "results/raw_traces.json")

labeled = label_traces(traces)
save_json(labeled, "results/labeled_traces.json")

# Bad: Only save final result (lose intermediate data if crash)
final = full_pipeline()
save_json(final, "results/final.json")
```

#### 3. Use Rich for User Feedback
```python
from rich.console import Console
from rich.progress import track

console = Console()

console.print("✅ Processing complete", style="bold green")
console.print("❌ Error occurred", style="bold red")

for item in track(items, description="Processing..."):
    process(item)
```

#### 4. Log Evaluation Results, Not Just Metrics
```python
# Good: Save full results for debugging
results = {
    "overall_metrics": {"recall@5": 0.75},
    "per_query": [
        {"query": "pasta", "recall@5": 0.8, "retrieved_ids": [1, 2, 3]},
        ...
    ]
}

# Bad: Only save aggregate metrics
results = {"recall@5": 0.75}
```

#### 5. Document Prompt Changes
```python
# Good: Version control for prompts
JUDGE_PROMPT_V1 = "..."
JUDGE_PROMPT_V2 = "..."  # Added dietary context

# Save prompts with results
with open("results/judge_prompt.txt", "w") as f:
    f.write(JUDGE_PROMPT_V2)
```

### Performance Optimization

#### 1. BM25 Retrieval Caching
Already implemented in `backend/retrieval.py`:
- Index built once, reused for all queries
- Pre-tokenization of recipes
- Efficient top-k retrieval

#### 2. Parallel Processing
Use for independent operations:
```python
# Good: Parallel trace generation
with ThreadPoolExecutor(max_workers=10) as executor:
    futures = [executor.submit(generate_trace, q) for q in queries]
    traces = [f.result() for f in as_completed(futures)]

# Bad: Sequential processing
traces = [generate_trace(q) for q in queries]
```

#### 3. Batch LLM Calls
Group related queries:
```python
# Instead of 100 individual calls
for query in queries:
    response = litellm.completion(...)

# Batch into fewer calls with multiple items
batches = chunk(queries, size=10)
for batch in batches:
    prompt = "\n".join([f"{i}. {q}" for i, q in enumerate(batch)])
    response = litellm.completion(...)
```

### Security Considerations

#### 1. Never Commit API Keys
- Use `.env` for secrets
- `.gitignore` includes `.env`
- Provide `env.example` template

#### 2. Validate User Input
```python
from fastapi import HTTPException

@app.post("/chat")
async def chat(request: ChatRequest):
    if len(request.message) > 1000:
        raise HTTPException(400, "Message too long")

    if not request.message.strip():
        raise HTTPException(400, "Empty message")

    return process_chat(request)
```

#### 3. Rate Limiting (Not Implemented)
Consider adding for production:
```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/chat")
@limiter.limit("10/minute")
async def chat(request: ChatRequest):
    ...
```

---

## Quick Reference Card

### Most Common Commands
```bash
# Start application
uvicorn backend.main:app --reload

# Run annotation tool
python annotation/annotation.py

# Bulk testing
python scripts/bulk_test.py

# HW3 pipeline
cd homeworks/hw3 && python scripts/run_full_evaluation.py

# HW4 pipeline
cd homeworks/hw4 && python scripts/evaluate_retrieval.py
```

### Key File Paths
| Purpose | Path |
|---------|------|
| System prompt | `backend/utils.py` → `SYSTEM_PROMPT` |
| Model config | `.env` → `MODEL_NAME`, `MODEL_NAME_JUDGE` |
| Chat endpoint | `backend/main.py` → `/chat` POST |
| Retrieval logic | `backend/retrieval.py` → `RecipeRetriever` |
| Sample queries | `data/sample_queries.csv` |
| Saved traces | `annotation/traces/*.json` |

### Important Constants
```python
# backend/utils.py
MODEL_NAME = os.environ.get("MODEL_NAME", "openai/gpt-4-mini")
SYSTEM_PROMPT = "You are a helpful recipe chatbot..."

# Common temperature settings
TEMPERATURE_DETERMINISTIC = 0.3
TEMPERATURE_CREATIVE = 0.7

# Retrieval defaults
DEFAULT_K = 5  # Top-k results
DEFAULT_BM25_K1 = 1.5
DEFAULT_BM25_B = 0.75
```

### Useful Patterns
```python
# LLM call
response = litellm.completion(
    model=os.environ.get("MODEL_NAME"),
    messages=[{"role": "user", "content": prompt}],
    temperature=0.3
)

# Parallel processing
with ThreadPoolExecutor(max_workers=10) as executor:
    futures = {executor.submit(fn, x): x for x in items}
    results = [f.result() for f in as_completed(futures)]

# Save JSON
with open(f"results/{name}.json", "w") as f:
    json.dump(data, f, indent=2)
```

---

## Contributing Guidelines for AI Assistants

When modifying this codebase:

### ✅ DO
- Read existing files before proposing changes
- Follow established naming conventions (`snake_case`, `PascalCase`, etc.)
- Add type hints to all new functions
- Save intermediate results in evaluation pipelines
- Use Rich library for console output
- Test changes with `bulk_test.py` before committing
- Document prompt changes with version comments
- Preserve existing error handling patterns

### ❌ DON'T
- Modify `SYSTEM_PROMPT` without testing impact
- Change existing evaluation metrics (breaks comparisons)
- Add dependencies without updating `requirements.txt`
- Commit API keys or `.env` files
- Remove existing homework solutions
- Break backward compatibility with saved traces
- Add complex abstractions for one-time operations

### 🔍 When Making Changes
1. **Search first**: Use grep to find similar patterns
2. **Read context**: Understand surrounding code
3. **Test locally**: Run affected pipelines
4. **Document**: Update comments and docstrings
5. **Verify**: Check no regressions in evaluations

---

## Additional Resources

### Documentation
- **Main README**: `/README.md`
- **HW2 Walkthrough**: `homeworks/hw2/hw2_solution_walkthrough.ipynb`
- **HW3 Walkthrough**: `homeworks/hw3/hw3_walkthrough.ipynb`
- **HW4 Walkthrough**: `homeworks/hw4/hw4_walkthrough.py` (Marimo)
- **HW5 Walkthrough**: `homeworks/hw5/hw5_walkthrough.py` (Marimo)

### External Links
- [LiteLLM Provider Docs](https://docs.litellm.ai/docs/providers)
- [Judgy Library](https://github.com/psobot/judgy) (bias-corrected metrics)
- [FastAPI Docs](https://fastapi.tiangolo.com/)
- [Rank-BM25](https://github.com/dorianbrown/rank_bm25)

### Video Walkthroughs
- HW2: [Code walkthrough](https://youtu.be/h9oAAAYnGx4), [Coding walkthrough](https://youtu.be/AKg27L4E0M8)
- HW3: [Solution walkthrough](https://youtu.be/1d5aNfslwHg)
- HW4: [Solution walkthrough](https://youtu.be/GMShL5iC8aY)
- HW5: [Solution walkthrough](https://youtu.be/z1oISsDUKLA)

---

**End of CLAUDE.md** - This guide should help AI assistants navigate and contribute to the Recipe Chatbot repository effectively.
