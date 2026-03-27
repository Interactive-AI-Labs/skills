---
name: interactiveai-prompt-migration
description: Migrate hardcoded prompts to InteractiveAI for version control and deployment-free iteration. Use when user wants to externalize prompts, move prompts to InteractiveAI, or set up prompt management.
---

# InteractiveAI Prompt Migration

Migrate hardcoded prompts to InteractiveAI for version control, A/B testing, and deployment-free iteration.

## Prerequisites

Verify credentials before starting:

```bash
echo $INTERACTIVEAI_PUBLIC_KEY   # pk-lf-...
echo $INTERACTIVEAI_SECRET_KEY   # sk-lf-...
echo $INTERACTIVEAI_HOST         # https://app.interactive.ai
```

If not set, ask user to configure them first.

## Migration Flow

```
1. Scan codebase for prompts
2. Analyze templating compatibility
3. Propose structure (names, types, subprompts, variables)
4. User approves
5. Create prompts in InteractiveAI
6. Refactor code to use get_prompt()
7. Link prompts to traces (if tracing enabled)
8. Verify application works
```

## Step 1: Find Prompts

Search for these patterns:

| Framework | Look for |
|-----------|----------|
| OpenAI | `messages=[{"role": "system", "content": "..."}]` |
| Anthropic | `system="..."` |
| LangChain | `ChatPromptTemplate`, `SystemMessage` |
| Vercel AI | `system: "..."`, `prompt: "..."` |
| Raw | Multi-line strings near LLM calls |

**Also scan for structured content** that maps to InteractiveAI's extended prompt types:

| Content pattern | Potential type |
|----------------|----------------|
| Step-by-step workflows, procedures, runbooks | `routine` |
| Rules, guidelines, constraints, policies | `policy` |
| Key-value configuration, settings, parameters | `variable` |
| Term definitions, domain vocabulary | `glossary` |
| Reusable text blocks, snippets, templates | `macro` |

## Step 2: Check Templating Compatibility

**CRITICAL:** InteractiveAI only supports simple `{{variable}}` substitution for text and chat prompts. No conditionals, loops, or filters.

| Template Feature | Supported | Action |
|------------------|-----------|--------|
| `{{variable}}` | Yes | Direct migration |
| `{var}` / `${var}` | No | Convert to `{{var}}` |
| `{% if %}` / `{% for %}` | No | Move logic to code |
| `{{ var \| filter }}` | No | Apply filter in code |

### Decision Tree

```
Contains {% if %}, {% for %}, or filters?
├─ No → Direct migration
└─ Yes → Choose:
    ├─ Option A (RECOMMENDED): Move logic to code, pass pre-computed values
    └─ Option B: Store raw template, compile client-side with Jinja2
        └─ Warning: Loses Playground preview and UI experiments
```

### Simplifying Complex Templates

**Conditionals** → Pre-compute in code:
```python
# Instead of {% if user.is_premium %}...{% endif %} in prompt
# Use {{tier_message}} and compute value in code before compile()
```

**Loops** → Pre-format in code:
```python
# Instead of {% for tool in tools %}...{% endfor %} in prompt
# Use {{tools_list}} and format the list in code before compile()
```

## Step 3: Propose Structure

### Naming Conventions

| Rule | Example | Bad |
|------|---------|-----|
| Lowercase, hyphenated | `chat-assistant` | `ChatAssistant_v2` |
| Feature-based | `document-summarizer` | `prompt1` |
| Hierarchical for related | `support/triage` | `supportTriage` |
| Prefix subprompts with `_` | `_base-personality` | `shared-personality` |

### Choose Prompt Type

InteractiveAI supports seven prompt types. Use the best fit:

| Type | Client Class | Key Properties | Best for |
|------|-------------|----------------|----------|
| `text` | TextPromptClient | `.prompt`, `.compile()`, `.variables` | Simple text prompts |
| `chat` | ChatPromptClient | `.prompt`, `.compile()`, `.variables` | Chat/conversation prompts |
| `routine` | RoutinePromptClient | `.title`, `.description`, `.conditions`, `.steps`, `.raw_content` | Step-by-step workflows |
| `policy` | PolicyPromptClient | `.entries`, `.raw_content` | Rules and guidelines |
| `variable` | VariablePromptClient | `.entries`, `.raw_content` | Key-value configurations |
| `glossary` | GlossaryPromptClient | `.entries`, `.raw_content` | Term definitions |
| `macro` | MacroPromptClient | `.entries`, `.raw_content` | Reusable text blocks |

All types share: `.name`, `.version`, `.labels`, `.tags`, `.config`, `.is_fallback`, `.compile()`, `.get_langchain_prompt()`.

### Identify Subprompts

Extract when:
- Same text in 2+ prompts
- Represents distinct component (personality, safety rules, format)
- Would need to change together

### Variable Extraction

| Make Variable | Keep Hardcoded |
|---------------|----------------|
| User-specific (`{{user_name}}`) | Output format instructions |
| Dynamic content (`{{context}}`) | Safety guardrails |
| Per-request (`{{query}}`) | Persona/personality |
| Environment-specific (`{{company_name}}`) | Static examples |

## Step 4: Present Plan to User

Format:
```
Found N prompts across M files:

src/chat.py:
  - System prompt (47 lines) → 'chat-assistant' (type: text)

src/support/triage.py:
  - Triage prompt (34 lines) → 'support/triage' (type: text)
    Warning: Contains {% if %} - will simplify

src/policies/guidelines.py:
  - Company guidelines (20 rules) → 'company-guidelines' (type: policy)

src/workflows/onboarding.py:
  - Onboarding steps (8 steps) → 'onboarding-flow' (type: routine)

Subprompts to extract:
  - '_base-personality' - used by: chat-assistant, support/triage

Variables to add:
  - {{user_name}} - hardcoded in 2 prompts

Proceed?
```

## Step 5: Create Prompts in InteractiveAI

Use `interactive.create_prompt()` with:
- `name`: Your chosen name
- `prompt`: Template text (or message array for chat, or YAML/JSON for structured types)
- `type`: `"text"`, `"chat"`, `"routine"`, `"policy"`, `"variable"`, `"glossary"`, or `"macro"`
- `labels`: only include `["production"]` if the user explicitly confirms the new version should go live immediately
- `config`: Optional model settings

**For structured types:** fetch the doc page for the type first to get the required schema. See [references/structured-prompts.md](structured-prompts.md).

**Labeling strategy:**
- `production` → Use only after explicit approval to make the new version live
- `staging` → Preferred for pre-production verification when a non-live label is needed
- `latest` → Auto-applied by InteractiveAI (system-reserved)

For full API: fetch https://docs.interactive.ai/sdk/prompts

## Step 6: Refactor Code

### Text and Chat Prompts

Replace hardcoded prompts with:

```python
prompt = interactive.get_prompt("name", type="text", label="latest")
compiled = prompt.compile(var1=value1, var2=value2)
```

**Key points:**
- Use `label="latest"` by default unless the user explicitly asks for a different label or a fixed version
- Pass `type=` to get the correct client class
- Call `.compile()` to substitute variables
- For chat prompts, result is a message array ready for the API

### Structured Prompts

For structured types (routine, policy, variable, glossary, macro), **fetch the doc page for the specific type before creating content** — each has a distinct schema with required fields.

See [references/structured-prompts.md](structured-prompts.md) for the type-to-doc mapping, schema workflow, and exact-format preservation rules.

## Step 7: Link Prompts to Traces

If codebase uses InteractiveAI tracing, link prompts so you can see which version produced each response.

### Detect Existing Tracing

Look for:
- `start_as_current_observation()` calls
- `Interactive()` client initialization
- `from interactiveai import Interactive`

### Link Method

Pass the prompt object when updating the current span:

```python
prompt = interactive.get_prompt("chat-assistant", type="text", label="latest")
compiled = prompt.compile(user_name="Alice")

with interactive.start_as_current_observation(name="llm-call", as_type="generation"):
    interactive.update_current_span(prompt=prompt)
    # ... make LLM call with compiled prompt ...
```

### Verify in UI

1. Go to **Traces** → select a trace
2. Click on the **Generation** observation
3. Check **Prompt** field shows name and version

## Step 8: Verify Migration

### Checklist

- [ ] `production` label only applied after explicit user approval
- [ ] Code fetches with `label="latest"` by default, unless the user asked for a different label or version
- [ ] Variables compile without errors
- [ ] Subprompts resolve correctly
- [ ] Structured types return expected properties (`.steps`, `.entries`, etc.)
- [ ] Application behavior unchanged
- [ ] Generations show linked prompt in UI (if tracing)

### Common Issues

| Issue | Solution |
|-------|----------|
| `PromptNotFoundError` | Check name spelling and `type=` parameter |
| Variables not replaced | Use `{{var}}` not `{var}`, call `.compile()` |
| Wrong client class returned | Ensure `type=` matches how the prompt was created |
| Old prompt cached | Restart app or call `interactive.clear_prompt_cache()` |

## Out of Scope

- Prompt engineering (writing better prompts)
- Evaluation setup
- A/B testing workflow
- Non-LLM string templates
