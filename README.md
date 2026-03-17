# InteractiveAI Skills

[Agent Skills](https://github.com/anthropics/skills) that teach AI coding assistants (Claude Code, Cursor, etc.) how to work with [InteractiveAI](https://interactive.ai).

## Skills

| Skill                                   | Description                                                                                                                                                         |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [interactiveai](./skills/interactiveai) | Main skill to work with InteractiveAI. Manage platform resources via the iai CLI, instrument applications with tracing, migrate prompts, and look up documentation. |

## Installation

### Cursor Plugin

Install as a [Cursor plugin](https://cursor.com/docs/plugins):

```
/add-plugin interactiveai
```

### skills CLI

Install via the [skills CLI](https://github.com/anthropics/skills):

```bash
npx skills add Interactive-AI-Labs/skills --skill "interactiveai"
```

### Manual symlink

Clone this repo and symlink the skill into your agent's skills directory:

```bash
git clone https://github.com/Interactive-AI-Labs/skills.git /path/to/interactiveai-skills
ln -s /path/to/interactiveai-skills/skills/interactiveai /path/to/skills-directory/interactiveai
```

## Prerequisites

You need an InteractiveAI account ([cloud](https://app.interactive.ai) or self-hosted) and API keys:

```bash
# SDK credentials
export INTERACTIVEAI_PUBLIC_KEY=pk-lf-...
export INTERACTIVEAI_SECRET_KEY=sk-lf-...
export INTERACTIVEAI_HOST=https://app.interactive.ai  # or self-hosted URL

# CLI credential (public_key:secret_key joined by colon)
export INTERACTIVE_API_KEY=$INTERACTIVEAI_PUBLIC_KEY:$INTERACTIVEAI_SECRET_KEY
```

API keys are found in your InteractiveAI project under **Settings > API Keys**.

## Usage

Once installed, the agent will automatically use these skills when relevant — for example:

- Managing platform resources (services, secrets, vector stores) via the iai CLI
- Setting up InteractiveAI tracing in a project
- Auditing existing instrumentation
- Migrating prompts to InteractiveAI prompt management
- Looking up InteractiveAI docs, SDK usage, or integration guides

## Feedback & Requests

Something not working as expected, or want a new skill? [Start a discussion](https://github.com/Interactive-AI-Labs/skills/discussions/new?category=ideas-improvements).
