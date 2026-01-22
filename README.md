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

Use `/codex` with scoped, specific prompts:

```
/codex Review auth middleware for security issues. ONLY report vulnerabilities, no style suggestions. Format: file:line - issue.

/codex Find bugs in payment flow. Look for edge cases and race conditions. Skip preambles. Lead with findings.

/codex Plan database layer refactor. Output: overview, files to modify, implementation steps. No code samples.
```

### With Options

```
/codex --model gpt-5.2 analyze this algorithm's complexity
/codex --sandbox workspace-write generate a test file
```

### Prompting Tips

For best results with gpt-5.2-codex:
- **Be specific** - "Review for SQL injection" > "Review code"
- **Set constraints** - "ONLY bugs, no style issues"
- **Skip preambles** - "Skip preambles. Lead with findings."
- **Request format** - "Format: `file:line` - description"

### Available Options

| Option | Values | Default | Description |
|--------|--------|---------|-------------|
| `--model` | any model name | gpt-5.2-codex | Model to use |
| `--sandbox` | read-only, workspace-write, danger-full-access | read-only | Sandbox mode |

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
codex exec \
  --model gpt-5.2-codex \
  --sandbox read-only \
  "<task>" \
  2>&1
```

The task is passed as an inline prompt argument.

## Troubleshooting

### "Codex CLI is not installed"

Install with npm:
```bash
brew install codex
```

### Authentication errors

Run `codex login` to authenticate with your OpenAI API key.

### Timeout on complex tasks

Complex analysis tasks may take time. Be patient or try a simpler task description.

## License

MIT

## Author

[thepushkarp](https://github.com/thepushkarp)
