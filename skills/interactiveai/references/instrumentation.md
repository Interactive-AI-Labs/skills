---
name: interactiveai-observability
description: Instrument LLM applications with InteractiveAI tracing. Use when setting up InteractiveAI, adding observability to LLM calls, or auditing existing instrumentation.
---

# InteractiveAI Observability

Instrument LLM applications with InteractiveAI tracing, following best practices and tailored to your use case.

## When to Use

- Setting up InteractiveAI in a new project
- Auditing existing InteractiveAI instrumentation
- Adding observability to LLM calls

## Workflow

### 1. Assess Current State

Check the project:

- Is the InteractiveAI SDK installed?
- What LLM frameworks are used? (OpenAI SDK, LangChain, LlamaIndex, CrewAI, Claude Agent SDK, etc.)
- Is there existing instrumentation?

**No integration yet:** Set up InteractiveAI using a framework integration if available. Integrations capture more context automatically and require less code than manual instrumentation.

**Integration exists:** Audit against baseline requirements below.

### 2. Verify Baseline Requirements

Every trace should have these fundamentals:

| Requirement               | Check                                                                                                                                           | Why                                                               |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| Model name                | Is the LLM model captured?                                                                                                                      | Enables model comparison and filtering                            |
| Token usage               | Are input/output tokens tracked?                                                                                                                | Enables automatic cost calculation                                |
| Good trace names          | Are names descriptive? (`chat-response`, not `trace-1`)                                                                                         | Makes traces findable and filterable                              |
| Span hierarchy            | Are multi-step operations nested properly?                                                                                                      | Shows which step is slow or failing                               |
| Correct observation types | Are observations using the right type? (see Observation Types below)                                                                            | Enables type-specific analytics                                   |
| Sensitive data masked     | Is PII/confidential data excluded or masked?                                                                                                    | Prevents data leakage                                             |
| Trace input/output        | Does the trace capture meaningful input/output? Is input explicitly set to show only relevant data (e.g., user message), not all function args? | Makes traces readable in the UI and avoids leaking sensitive args |

Framework integrations handle model name, tokens, and observation types automatically. Prefer integrations over manual instrumentation.

Docs: https://docs.interactive.ai/tracing

### 3. Explore Traces First

Once baseline instrumentation is working, encourage the user to explore their traces in the InteractiveAI UI before adding more context:

"Your traces are now appearing in InteractiveAI. Take a look at a few of them—see what data is being captured, what's useful, and what's missing. This will help us decide what additional context to add."

This helps the user:

- Understand what they're already getting
- Form opinions about what's missing
- Ask better questions about what they need

### 4. Discover Additional Context Needs

Determine what additional instrumentation would be valuable. **Infer from code when possible, only ask when unclear.**

**Infer from code:**

| If you see in code...                                | Infer             | Suggest                   |
| ---------------------------------------------------- | ----------------- | ------------------------- |
| Conversation history, chat endpoints, message arrays | Multi-turn app    | `session_id`              |
| User authentication, `user_id` variables             | User-aware app    | `user_id` on traces       |
| Multiple distinct endpoints/features                 | Multi-feature app | `feature` tag             |
| Customer/tenant identifiers                          | Multi-tenant app  | `customer_id` or tier tag |
| Feedback collection, ratings                         | Has user feedback | Capture as scores         |

**Only ask when not obvious from code:**

- "How do you know when a response is good vs bad?" → Determines scoring approach
- "What would you want to filter by in a dashboard?" → Surfaces non-obvious tags
- "Are there different user segments you'd want to compare?" → Customer tiers, plans, etc.

**Additions and their value:**

| Addition            | Why                                         | Docs                                         |
| ------------------- | ------------------------------------------- | -------------------------------------------- |
| `session_id`        | Groups conversations together               | https://docs.interactive.ai/tracing/sessions |
| `user_id`           | Enables user filtering and cost attribution | https://docs.interactive.ai/tracing/users    |
| User feedback score | Enables quality filtering and trends        | https://docs.interactive.ai/scoring          |
| `feature` tag       | Per-feature analytics                       | https://docs.interactive.ai/tracing/tags     |
| `customer_tier` tag | Cost/quality breakdown by segment           | https://docs.interactive.ai/tracing/tags     |

These are NOT baseline requirements—only add what's relevant based on inference or user input.

### 5. Guide to UI

After adding context, point users to relevant UI features:

- Traces view: See individual requests
- Sessions view: See grouped conversations (if session_id added)
- Dashboard: Build filtered views using tags
- Scores: Filter by quality metrics

## SDK Client

The InteractiveAI Python SDK uses the `Interactive` client:

```python
from interactiveai import Interactive

interactive = Interactive(
    public_key="pk-lf-...",    # or INTERACTIVEAI_PUBLIC_KEY env var
    secret_key="sk-lf-...",    # or INTERACTIVEAI_SECRET_KEY env var
    host="https://app.interactive.ai"  # or INTERACTIVEAI_HOST env var
)
```

## Observation Types

InteractiveAI supports these observation types via `start_observation(as_type='...')`:

| Type         | Use for                                      |
| ------------ | -------------------------------------------- |
| `span`       | Generic operations, function calls           |
| `generation` | LLM calls (model name, tokens, cost tracked) |
| `embedding`  | Embedding operations                         |
| `agent`      | Agent orchestration steps                    |
| `tool`       | Tool/function calls within agents            |
| `chain`      | Chain/pipeline steps                         |
| `retriever`  | RAG retrieval operations                     |
| `evaluator`  | Evaluation/scoring operations                |
| `guardrail`  | Safety/guardrail checks                      |

### Creating Observations

```python
# As context manager (recommended — auto-ends the span)
with interactive.start_as_current_observation(name="my-step", as_type="agent") as obs:
    # ... your code ...
    pass

# Manual (requires explicit .end())
obs = interactive.start_observation(name="my-step", as_type="tool")
# ... your code ...
obs.end()

# Update the current span with additional data
interactive.update_current_span(input="user query", output="response")

# Update trace metadata
interactive.update_current_trace(user_id="user-123", session_id="session-456", tags=["feature-x"])
```

### Trace Utilities

```python
# Generate trace IDs
trace_id = interactive.create_trace_id()            # random
trace_id = interactive.create_trace_id(seed="key")  # deterministic

# Get current IDs (useful for linking feedback)
trace_id = interactive.get_current_trace_id()
obs_id = interactive.get_current_observation_id()

# Get UI link to a trace
url = interactive.get_trace_url()
```

### Linking Prompts to Traces

When using InteractiveAI prompt management, link prompts to observations for version tracking:

```python
prompt = interactive.get_prompt("my-prompt", label="production")
compiled = prompt.compile(variable="value")

with interactive.start_as_current_observation(name="llm-call", as_type="generation") as obs:
    # Pass the prompt object to link it
    interactive.update_current_span(prompt=prompt)
    # ... make LLM call with compiled prompt ...
```

### Scoring

Attach scores to traces or spans for quality tracking:

```python
# Score by ID
interactive.create_score(
    trace_id="trace-id",
    name="user-thumbs",
    value=1,
    data_type="BOOLEAN",
    comment="User liked the response"
)

# Score the current span (within a traced context)
interactive.score_current_span(name="relevance", value=0.95, data_type="NUMERIC")

# Score the entire current trace
interactive.score_current_trace(name="user-thumbs", value=1, data_type="BOOLEAN")
```

Supported data types: `NUMERIC` (float), `BOOLEAN` (0/1), `CATEGORICAL` (string).

## Framework Integrations

Prefer these over manual instrumentation:

| Framework        | Integration          | Docs                                                   |
| ---------------- | -------------------- | ------------------------------------------------------ |
| OpenAI SDK       | Drop-in replacement  | https://docs.interactive.ai/integrations/openai        |
| LangChain        | Callback handler     | https://docs.interactive.ai/integrations/langchain     |
| CrewAI           | Built-in integration | https://docs.interactive.ai/integrations/crewai        |
| Claude Agent SDK | Built-in integration | https://docs.interactive.ai/integrations/claude-agent  |
| OpenTelemetry    | Exporter             | https://docs.interactive.ai/integrations/opentelemetry |

Full list: https://docs.interactive.ai/integrations

## Always Explain Why

When suggesting additions, explain the user benefit:

```
"I recommend adding session_id to your traces.

Why: This groups messages from the same conversation together.
You'll be able to see full conversation flows in the Sessions view,
making it much easier to debug multi-turn interactions.

Learn more: https://docs.interactive.ai/tracing/sessions"
```

## Common Mistakes

| Mistake                                        | Problem                                                            | Fix                                                                                    |
| ---------------------------------------------- | ------------------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| No `flush()` in scripts                        | Traces never sent                                                  | Call `interactive.flush()` before exit                                                 |
| Flat traces                                    | Can't see which step failed                                        | Use nested spans for distinct steps                                                    |
| Generic trace names                            | Hard to filter                                                     | Use descriptive names: `chat-response`, `doc-summary`                                  |
| Logging sensitive data                         | Data leakage risk                                                  | Mask PII before tracing                                                                |
| Not explicitly setting input                   | All function args become trace input (including API keys, configs) | Use `interactive.update_current_span(input=...)` — set only the relevant input         |
| Manual instrumentation when integration exists | More code, less context                                            | Use framework integration                                                              |
| InteractiveAI import before env vars loaded    | SDK initializes with missing/wrong credentials                     | Import InteractiveAI AFTER loading environment variables (e.g., after `load_dotenv()`) |
| Wrong import order with OpenAI                 | SDK can't patch the OpenAI client                                  | Import InteractiveAI and call its setup BEFORE importing OpenAI client                 |
| Using `start_generation()` (deprecated)        | Will be removed in future versions                                 | Use `start_observation(as_type='generation')` instead                                  |
