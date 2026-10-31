# Local University Assistant with LangGraph

A practical project exploring AI workflows using LangGraph, LangChain, and a local Qwen3.5:4b model served through Ollama.

The notebook progresses from a basic graph to a university assistant with conditional routing, specialized prompts, tools, conversation memory, and error handling.

## Features

- Local LLM inference without an OpenAI API key
- Multi-step question analysis and answer generation
- Question classification and conditional routing
- Specialized agents for courses, programming, mathematics, administration, and general questions
- Arithmetic calculator and current date/time tools
- Conversation memory separated by thread ID
- Execution traces and routing evaluation

## Setup

### 1. Install dependencies

In your Python environment:

```bash
python -m pip install -U langgraph langchain langchain-ollama langchain-community pandas jupyterlab
```

The recorded execution used Python 3.10 with:
- LangGraph 1.2.19
- LangChain 1.4.7
- langchain-ollama 1.1.0
- pandas 2.3.3

These are the versions shown in the notebook output, not a complete dependency lock file.

### 2. Download the model

Install Ollama from https://ollama.com, then run:

```bash
ollama pull qwen3.5:4b
```

If Ollama is not already running, start it in a separate terminal:

```bash
ollama serve
```

### 3. Configure the notebook

The model configuration is:

```python
from langchain_ollama import ChatOllama

llm = ChatOllama(
    model="qwen3.5:4b",
    base_url="http://localhost:11434",
    temperature=0,
    reasoning=False,
    num_ctx=8192,
)
```

The notebook must be able to reach Ollama at the configured address.

## Running the Lab

```bash
jupyter lab
```

1. Open the notebook and select the kernel associated with your Python environment.
2. Run the cells from top to bottom.
3. Inspect the responses, categories, tool calls, and memory tests.
4. Save the executed notebook with its outputs as `langgraph_lab.ipynb`.

Some state classes and functions are redefined between exercises, so execution order matters.

## Workflow

The final assistant:

1. Classifies the latest question.
2. Routes it to a specialized agent.
3. Generates a direct answer or requests a tool.
4. Executes any requested tool.
5. Returns to the same agent to process the result.
6. Preserves conversation state using a checkpointer and thread ID.

See `architecture.md` for the routing and tool-execution diagrams.

## Design Choices

### Local model

Qwen3.5:4b runs through Ollama. Model inference does not require an OpenAI API key. Model downloads and the default PNG graph renderer require network access.

### Shared state

Nodes exchange information through state. The final assistant stores the message history and the selected category.

### Specialized agents

The five agents use the same model with different system prompts. They are not separately trained models.

### Tools

`bind_tools()` allows the model to request the calculator or date/time tool. `ToolNode` executes those requests and returns tool messages.

The active calculator uses an AST parser that permits arithmetic operations. The unsafe `eval()` example is included only for comparison and is not bound to the assistant.

### Memory

`MemorySaver` preserves conversation state between calls with the same `thread_id`. Separate thread IDs maintain separate histories.

This memory is temporary and disappears when the kernel restarts.

### Error handling

Empty questions are routed to a fallback response. The code also includes handling for model-call failures and tool errors, although those failure paths were not all tested in the recorded execution.

## Recorded Results

### Three-category router

| Question | Predicted category | Correct | Workflow time |
|---|---|---|---|
| Derivative of x² | mathematics | Yes | 5.29 s |
| Solve 2x + 3 = 11 | mathematics | Yes | 3.18 s |
| Reverse a list in Python | programming | Yes | 10.25 s |
| Write a palindrome function | programming | Yes | 7.48 s |
| Explain a convolutional neural network | artificial_intelligence | Yes | 17.00 s |
| Explain overfitting | artificial_intelligence | Yes | 23.61 s |

Routing accuracy was **6/6 (100%)** on these examples.

The recorded times include classification and answer generation. They are not classifier-only timings. This small evaluation does not establish general accuracy or answer correctness.

### Functional demonstrations

| Test | Observed result |
|---|---|
| `25 * 4 + 10` | Calculator returned `110` |
| Unsupported expression `__import__('os')` | Returned `Invalid expression` |
| `125 * 38` | Trace showed a calculator call and result `4750` |
| Capital of Morocco | Answered Rabat without a tool call |
| Name recall without checkpointing | Did not recall the name |
| Name recall within the same thread | Recalled Ahmed |
| Name recall in a separate thread | Did not recall Ahmed |
| Empty question | Returned the fallback message |
| `(15 + 27) * 3` | Final assistant trace showed calculator result `126` |

The final demonstration also selected the intended categories for the five sample university questions.

## Known Issues and Limitations

- PNG graph rendering failed because the Mermaid rendering service could not be reached. Mermaid source was produced successfully.
- The date/time tool generated a Pydantic warning because its argument is named `_`.
- No official university database is connected.
- Generated answers and code have not undergone a comprehensive correctness evaluation.
- The classifier uses text matching rather than validated structured output.
- The calculator has no explicit expression-size or computation limits.
- Memory is stored in RAM.
- The optional Wikipedia integration is not demonstrated as part of the working assistant.

## Deliverables

- Executed notebook with outputs
- README
- Source code contained in the notebook, with a separate export if required
- Architecture diagrams
- Execution screenshots
- Short report