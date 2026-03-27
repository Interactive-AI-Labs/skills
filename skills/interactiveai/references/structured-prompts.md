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
| `routine` | RoutinePromptClient | https://docs.interactive.ai/sdk/routines.md |
| `policy` | PolicyPromptClient | https://docs.interactive.ai/sdk/policies.md |
| `variable` | VariablePromptClient | https://docs.interactive.ai/sdk/variables.md |
| `glossary` | GlossaryPromptClient | https://docs.interactive.ai/sdk/glossary.md |
| `macro` | MacroPromptClient | https://docs.interactive.ai/sdk/macros.md |

General prompts SDK (get, list, create, delete, cache): https://docs.interactive.ai/sdk/prompts.md

## Key Rule

**Do NOT guess schemas or SDK methods.** Each type has dedicated SDK clients and specific required fields. Always fetch the doc page for the type before reading or creating structured prompts.

## Authoring Workflow

Follow this order every time:

1. Discover the CLI surface for the type with `iai <type> --help` plus `iai <type> create --help` or `iai <type> update --help`.
2. Fetch the InteractiveAI docs page for that type.
3. If editing an existing prompt, fetch the current content first and treat its shape as authoritative unless the docs/schema require otherwise.
4. If the CLI exposes `iai <type> schema`, use it as the source of truth for allowed fields and nesting.
5. Make the smallest possible change. Preserve keys, indentation style, block scalar style, and wrapper structure.

## Format Preservation Rules

- Never invent alternate field names because they sound equivalent.
- Never replace a compact schema field with explanatory prose.
- Never add comments unless comments are already part of the stored content and must be preserved.
- Never switch between YAML block styles such as `|` and `>` unless the existing content or docs explicitly require it.
- If the project already uses a specific wrapper format for a type, mirror that format exactly for new entries.
