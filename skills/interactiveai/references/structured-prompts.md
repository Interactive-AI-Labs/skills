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

## Key Rule

**Do NOT guess schemas or SDK methods.** Each type has dedicated SDK clients and specific required fields. Always fetch the doc page for the type before reading or creating structured prompts.
