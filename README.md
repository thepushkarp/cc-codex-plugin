# cc-codex-plugin

A Claude Code plugin that integrates OpenAI's Codex CLI for detail-oriented code review, planning, and analysis.

## What is this?

This plugin gives Claude Code access to OpenAI's Codex CLI, enabling a powerful "second opinion" workflow. Codex excels at thorough, meticulous code analysis - complementing Claude's broader capabilities with sharp-eyed detail work.

**Use cases:**
- Deep security review of authentication code
- Bug hunting in complex algorithms
- Detailed implementation planning
- Edge case analysis
- Getting a second perspective on critical code

## Prerequisites

1. **Codex CLI** - Install globally:
   ```bash
   brew install codex
   ```

2. **OpenAI API Key** - Authenticate:
   ```bash
   codex login
   ```

## Installation

Install via Claude Code's plugin marketplace:

```
/plugin marketplace add thepushkarp/cc-codex-plugin
```

Or install directly from the repository:

```
/plugin install https://github.com/thepushkarp/cc-codex-plugin
```

## Usage

### Explicit Command

Use `/codex` to invoke Codex directly:

```
/codex review the authentication middleware for security issues
/codex plan out how to refactor the database layer
/codex explore edge cases in the payment flow
```

### With Options

```
/codex --reasoning xhigh deeply analyze this algorithm's complexity
/codex --verbose --reasoning high explain this code in detail
/codex --sandbox workspace-write generate a test file
```

### Available Options

| Option | Values | Default | Description |
|--------|--------|---------|-------------|
| `--reasoning` | low, medium, high, xhigh | high | Reasoning effort level |
| `--model` | any model name | gpt-5.2-codex | Model to use |
| `--sandbox` | read-only, workspace-write, danger-full-access | read-only | Sandbox mode |
| `--verbose` | (flag) | false | Show thinking tokens |

### Automatic Delegation

Claude can also spawn Codex autonomously when it determines detailed analysis would be valuable. You'll see messages like:

> "Let me have Codex take a detailed look at this code for potential bugs..."

## Components

This plugin includes:

- **`/codex` command** - Direct invocation with configurable options
- **`codex-agent`** - Subagent Claude can spawn for delegated analysis
- **Codex skill** - Guidance on when and how to use Codex effectively

## How it Works

The plugin uses Codex's non-interactive `codex exec` mode to run analysis:

```bash
echo "<task>" | codex exec \
  --model gpt-5.2-codex \
  --config reasoning_effort=high \
  --sandbox read-only \
  --ask-for-approval on-request \
  --skip-git-repo-check
```

Thinking tokens are suppressed by default for clean output (use `--verbose` to see them).

## Troubleshooting

### "Codex CLI is not installed"

Install with npm:
```bash
brew install codex
```

### Authentication errors

Run `codex login` to authenticate with your OpenAI API key.

### Timeout on complex tasks

Try `--reasoning high` instead of `xhigh` for faster responses.

## License

MIT

## Author

[thepushkarp](https://github.com/thepushkarp)
