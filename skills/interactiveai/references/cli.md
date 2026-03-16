# InteractiveAI CLI Reference

## Install

```bash
# Requires Go — verify first
go version

# Install
go install github.com/Interactive-AI-Labs/interactive-cli/cmd/iai@latest

# Ensure Go bin is in PATH (add to shell profile for persistence)
export PATH=$PATH:$(go env GOPATH)/bin

# Verify installation
iai --help
```

## Authentication

**Preferred: env var (set once, works for all commands)**

```bash
# public_key:secret_key joined by colon
export INTERACTIVE_API_KEY=$INTERACTIVEAI_PUBLIC_KEY:$INTERACTIVEAI_SECRET_KEY
```

Once set, all `iai` commands authenticate automatically — no flags needed.

For non-default hosts, also pass `--hostname`:

```bash
iai traces list --hostname $INTERACTIVEAI_HOST
```

**Alternatives:**

```bash
# Interactive login — stores session credentials locally
iai login

# Per-command flag
iai --api-key "$INTERACTIVEAI_PUBLIC_KEY:$INTERACTIVEAI_SECRET_KEY" <command>

# Remove stored credentials
iai logout
```

## Discovery

```bash
# List all available commands
iai --help

# Show help for any subcommand
iai <command> --help

# Show help for nested subcommands
iai <command> <subcommand> --help
```

## Global Flags

These flags work with any command:

- `--api-key string` — API key for authentication
- `--hostname string` — Platform hostname (default: `https://app.interactive.ai`)
- `--deployment-hostname string` — Deployment hostname (default: `https://deployment.interactive.ai`)
- `--cfg-file string` — Path to YAML config file
- `-h, --help` — Help for any command

## Commands

### Platform Management

```bash
iai organizations    # Manage organizations
iai projects         # Manage projects
iai services         # Manage services (deploy, configure, scale)
iai replicas         # Manage service replicas
iai secrets          # Manage secrets
iai vector-stores    # Manage vector stores
iai images           # Manage container images
```

### Data Access

```bash
# List traces with optional filters
iai traces list
iai traces list --limit 20 --page 2
iai traces list --name my-trace --user-id user123
iai traces list --from-timestamp 2025-01-01T00:00:00Z
iai traces list --order-by timestamp.desc
iai traces list --tags tag1 --tags tag2
iai traces list --columns id,name,timestamp,user_id,session_id,latency,cost,tags

# Get a specific trace
iai traces get <trace-id>
```

### Shell Completion

```bash
# Generate completion for your shell (bash, zsh, fish, powershell)
iai completion <shell>
```

## Tips

- Use `iai <command> --help` to discover available subcommands and flags for any resource
- For self-hosted instances, override the hostname: `iai --hostname https://your-instance.example.com <command>`
- The CLI is being actively extended — run `iai --help` to see the latest available commands
- Full documentation: https://docs.interactive.ai/cli
