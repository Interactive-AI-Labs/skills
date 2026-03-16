---
name: interactiveai-structured-prompts
description: Work with InteractiveAI structured prompt types (routine, policy, variable, glossary, macro). Use when creating, reading, or migrating structured prompts.
---

# Structured Prompts

InteractiveAI extends standard text/chat prompts with five structured types. Each has a specific schema and typed SDK accessors.

## Types and Documentation

**Always fetch the relevant doc page before creating or modifying a structured prompt** — schemas have required fields that vary by type.

| Type | SDK Client | Doc Page |
|------|-----------|----------|
| `routine` | RoutinePromptClient | https://docs.interactive.ai/sdk/policies.md |
| `policy` | PolicyPromptClient | https://docs.interactive.ai/sdk/policies.md |
| `variable` | VariablePromptClient | https://docs.interactive.ai/sdk/variables.md |
| `glossary` | GlossaryPromptClient | https://docs.interactive.ai/sdk/glossary.md |
| `macro` | MacroPromptClient | https://docs.interactive.ai/sdk/macros.md |

General prompts SDK (get, list, create, delete, cache): https://docs.interactive.ai/sdk/prompts.md

## Quick Reference

All structured types are created via `interactive.create_prompt()` with `type=` set to the type name. Content is passed as a YAML or JSON string in the `prompt=` parameter.

All structured types share: `.name`, `.version`, `.labels`, `.tags`, `.config`, `.is_fallback`, `.raw_content`, `.compile()`.

### Reading

```python
prompt = interactive.get_prompt("name", type="policy", label="production")

# Structured types expose typed accessors
for entry in prompt.entries:
    print(entry)  # PolicyEntry, VariableEntry, GlossaryEntry, or MacroEntry

# Routines have specific properties
routine = interactive.get_prompt("name", type="routine", label="production")
print(routine.title, routine.steps, routine.conditions)
```

### Creating

Always fetch the doc page for the type first to get the exact schema. Then:

```python
interactive.create_prompt(
    name="my-prompt",
    prompt=content_string,  # YAML or JSON matching the type's schema
    type="policy",          # or routine, variable, glossary, macro
    labels=["production"],
    commit_message="description of change"
)
```

### Key Rule

**Do NOT guess schemas.** Each type has specific required fields (e.g., policies need `id`, `criticality`, `action`; variables need `name`, `type`, `default_value`). Fetch the doc page for the type before creating content.
