# InteractiveAI CLI Reference

## Authentication

Two methods — session login is preferred for interactive use; API keys for SDK and automation.

**Session login** — access all organizations and projects the user can reach:

```bash
iai login --hostname $INTERACTIVEAI_HOST
iai organizations select <org> --hostname $INTERACTIVEAI_HOST
iai projects select <project> --hostname $INTERACTIVEAI_HOST
```

Selected org/project persist locally — subsequent commands don't need those flags.

**API key** — scoped to a single project, requires `--organization` and `--project` flags:

```bash
export INTERACTIVE_API_KEY=$INTERACTIVEAI_PUBLIC_KEY:$INTERACTIVEAI_SECRET_KEY
iai <command> --organization <org> --project <project> --hostname $INTERACTIVEAI_HOST
```

## Organizations and Projects

Organizations are top-level containers. Projects live inside organizations. Use `iai organizations list` and `iai projects list` to discover available ones.

## Discovery

The CLI is actively extended — always discover commands dynamically:

```bash
iai --help                             # list all commands
iai <command> --help                   # list actions for a command
iai <command> <subcommand> --help      # show args/options for an action
```

For non-default hosts, always pass `--hostname $INTERACTIVEAI_HOST`.
