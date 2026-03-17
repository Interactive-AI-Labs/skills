---
name: interactiveai
description: Interact with InteractiveAI and access its documentation. Use when needing to (1) manage platform resources like services, secrets, traces, and vector stores via the iai CLI, (2) look up InteractiveAI documentation, concepts, integration guides, or SDK usage, or (3) understand how any InteractiveAI feature works. This skill covers CLI-based platform operations, SDK instrumentation, and documentation retrieval.
---

# InteractiveAI

This skill helps you use InteractiveAI effectively across all common workflows: instrumenting applications, migrating prompts, managing platform resources, and accessing documentation.

## Core Principles

Follow these principles for ALL InteractiveAI work:

1. **Documentation First**: NEVER implement based on memory. Always fetch current docs before writing code (InteractiveAI updates frequently). See the section below on how to access documentation.
2. **CLI for Platform Operations**: Use the `iai` CLI for managing services, secrets, traces, vector stores, and other platform resources. See the section below on how to use the CLI.
3. **Best Practices by Use Case**: Check the relevant reference file below for use-case-specific guidelines before implementing.
4. **Use latest InteractiveAI versions**: Unless the user specified otherwise or there's a good reason, always use the latest version of InteractiveAI SDKs/APIs.

## Use case specific references

- tips on using the iai CLI: references/cli.md
- instrumenting an existing function/application: references/instrumentation.md
- migrating prompts from a codebase into InteractiveAI: references/prompt-migration.md
- working with structured prompt types (routine, policy, variable, glossary, macro): references/structured-prompts.md

## 1. InteractiveAI CLI (iai)

Use the `iai` CLI to manage platform resources and access data. Unlike npx-based tools, `iai` requires installation.

### Quick Install Check

Before using the CLI, verify it's available. If not, install it:

```bash
# Check if installed
iai --help

# If not found — requires Go (https://go.dev/dl/)
go install github.com/Interactive-AI-Labs/interactive-cli/cmd/iai@latest
export PATH=$PATH:$(go env GOPATH)/bin
```

### Credentials

Set these environment variables once — they cover both the SDK and CLI:

```bash
# SDK credentials
export INTERACTIVEAI_PUBLIC_KEY=pk-lf-...
export INTERACTIVEAI_SECRET_KEY=sk-lf-...
export INTERACTIVEAI_HOST=https://app.interactive.ai # The server must always be specified in order to access InteractiveAI.

# CLI credential (public_key:secret_key joined by colon)
export INTERACTIVE_API_KEY=$INTERACTIVEAI_PUBLIC_KEY:$INTERACTIVEAI_SECRET_KEY
```

If not set, ask the user for their API keys (found in InteractiveAI UI → Settings → API Keys).

### Detailed CLI Reference

For full command list, authentication options, and usage patterns, see [references/cli.md](references/cli.md).

## 2. InteractiveAI Documentation

Two methods to access InteractiveAI docs, in order of preference. **Always prefer your application's native web fetch tools** (e.g., `WebFetch`, `WebSearch`, `mcp_fetch`, etc.) over `curl` when available. The URLs and patterns below work with any fetching method — the `curl` examples are just illustrative.

### 2a. Documentation Index (llms.txt)

Fetch the full index of all documentation pages:

```bash
curl -s https://docs.interactive.ai/llms.txt
```

Returns a structured list of every doc page with titles and relative paths. **Important**: Paths in llms.txt are relative (e.g., `/sdk/prompts.md`). Always prefix them with `https://docs.interactive.ai` to form the full URL (e.g., `https://docs.interactive.ai/sdk/prompts.md`). Use this to discover the right page for a topic, then fetch that page directly.

Alternatively, you can start on `https://docs.interactive.ai` and explore the site to find the page you need.

### 2b. Fetch Individual Pages as Markdown

Any page listed in llms.txt can be fetched as markdown by appending `.md` to its path. Use this when you know which page contains the information needed. Returns clean markdown with code examples and configuration details.

```bash
curl -s "https://docs.interactive.ai/sdk/tracing.md"
curl -s "https://docs.interactive.ai/sdk/prompts.md"
curl -s "https://docs.interactive.ai/cli/iai.md"
```

### Documentation Workflow

1. Start with **llms.txt** to orient — scan for relevant page titles
2. **Fetch specific pages** when you identify the right one
