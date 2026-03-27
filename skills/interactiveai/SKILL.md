---
name: interactiveai
description: Interact with InteractiveAI and access its documentation. Use when needing to (1) manage platform resources such as services, secrets, traces, vector stores, and prompts via the `iai` CLI, (2) look up InteractiveAI documentation, concepts, integration guides, or SDK usage, or (3) understand how any InteractiveAI feature works. This skill covers CLI-based platform operations, SDK instrumentation, and documentation retrieval.
---

# InteractiveAI

This skill helps you use InteractiveAI effectively across common workflows: instrumenting applications, migrating prompts, managing platform resources, and accessing documentation.

## Core Principles

Follow these principles for ALL InteractiveAI work:

1. **Documentation First**: NEVER implement from memory. Always fetch current docs before writing code. See the documentation section below.
2. **CLI and Schema Discovery First**: Before changing InteractiveAI resources, discover the exact command shape with `iai --help`, `iai <command> --help`, and `iai <command> <subcommand> --help`. For structured prompts, use `iai <type> schema` when available.
3. **Best Practices by Use Case**: Check the relevant reference file below before implementing.
4. **Default to Latest Prompt Fetches**: When reading prompts or wiring prompt fetches, use the `latest` version unless the user explicitly asks for a different label or a fixed version.
5. **Explicit Confirmation for Live Changes**: Scaling services or replicas, or assigning the `production` label to a prompt version, changes live behavior. Do not do either unless the user explicitly confirms.
6. **Use Latest InteractiveAI Versions**: Unless the user specified otherwise or there is a good reason not to, use the latest InteractiveAI SDK and API versions.

## Use case specific references

- instrumenting an existing function/application: references/instrumentation.md
- migrating prompts from a codebase into InteractiveAI: references/prompt-migration.md
- working with structured prompt types (routine, policy, variable, glossary, macro): references/structured-prompts.md
- further tips on using the `iai` CLI: references/cli.md

## 1. InteractiveAI CLI (`iai`)

Use the `iai` CLI to manage platform resources and inspect data. Unlike `npx`-based tools, `iai` must be installed locally.

Start by verifying availability and discovering the command shape:

```bash
iai --help
iai <command> --help
iai <command> <subcommand> --help
```

For structured prompt work, also inspect the type schema when available:

```bash
iai <type> schema
```

If `iai` is not installed, install the latest version:

```bash
go install github.com/Interactive-AI-Labs/interactive-cli/cmd/iai@latest
export PATH=$PATH:$(go env GOPATH)/bin
```

### Credentials

Set these environment variables before using the SDK or CLI:

```bash
export INTERACTIVEAI_PUBLIC_KEY=pk-lf-...
export INTERACTIVEAI_SECRET_KEY=sk-lf-...
export INTERACTIVEAI_HOST=https://app.interactive.ai
export INTERACTIVE_API_KEY=$INTERACTIVEAI_PUBLIC_KEY:$INTERACTIVEAI_SECRET_KEY
```

`INTERACTIVEAI_HOST` must always be set. If the credentials are not available, ask the user for their API keys from InteractiveAI Settings.

### Guardrails

- Ask for confirmation before running `iai services` or `iai replicas` commands that change capacity.
- Prompt versions receive `latest` automatically. Do not add `--labels production` unless the user explicitly confirms they want the new version live now.
- If the task would replace the version currently used in production, stop and ask for direct confirmation before proceeding.

### Detailed CLI Reference

For common workflows, authentication details, and usage patterns, see [references/cli.md](references/cli.md).

## 2. InteractiveAI Documentation

Two methods to access InteractiveAI docs, in order of preference. Always prefer native web fetch/search tools over `curl` when available. The URL patterns below work with any fetch method; the `curl` examples are illustrative.

### 2a. Documentation Index (`llms.txt`)

Fetch the full documentation index:

```bash
curl -s https://docs.interactive.ai/llms.txt
```

This returns doc titles and relative paths. Prefix each path with `https://docs.interactive.ai` to form the full URL.

Alternatively, start from `https://docs.interactive.ai` and navigate to the relevant page.

### 2b. Fetch Individual Pages as Markdown

Any page listed in `llms.txt` can be fetched as markdown by appending `.md` to its path:

```bash
curl -s "https://docs.interactive.ai/sdk/tracing.md"
curl -s "https://docs.interactive.ai/sdk/prompts.md"
curl -s "https://docs.interactive.ai/cli/iai.md"
```

### Documentation Workflow

1. Start with `llms.txt` to find the right page.
2. Fetch the specific markdown page once you know where the relevant guidance lives.
