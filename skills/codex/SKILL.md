---
name: codex-integration
description: This skill provides guidance on using OpenAI's Codex CLI for detail-oriented code analysis. Use when the user explicitly invokes "/codex", asks to "use codex", "run codex", "have codex review", or when Claude determines that Codex's sharp-eyed analysis would complement its work on code review, bug hunting, or planning tasks.
allowed-tools: Bash
---

# Codex Integration

OpenAI's Codex CLI is a complementary tool for detail-oriented code analysis. Codex excels at thorough, meticulous review work - think of it as a "sharp-eyed second opinion" for code.

## When to Use Codex

**Ideal use cases:**
- Deep code review for security vulnerabilities or correctness issues
- Thorough bug hunting in complex code paths
- Detailed implementation planning with attention to edge cases
- Getting a second perspective on critical code
- Exploratory analysis requiring meticulous attention

**Not ideal for:**
- Simple, straightforward tasks Claude handles well alone
- Interactive debugging sessions (Codex exec is non-interactive)
- Tasks where speed matters more than depth
- Trivial code changes or obvious fixes

## Invocation Methods

### Method 1: Explicit /codex Command

Users invoke directly with optional flags:

```
/codex review the authentication middleware for security issues
/codex --reasoning xhigh deeply analyze this algorithm
/codex --verbose plan the database refactoring
```

**Available flags:**
| Flag | Values | Default |
|------|--------|---------|
| `--reasoning` | low, medium, high, xhigh | high |
| `--model` | any model name | gpt-5.2-codex |
| `--sandbox` | read-only, workspace-write, danger-full-access | read-only |
| `--verbose` | (flag) | false |

### Method 2: Spawning codex-agent

When Claude determines Codex would add value, spawn the `codex-agent` subagent:

```
"Let me have Codex take a detailed look at this code for potential bugs..."
[Spawn codex-agent with task description]
[Synthesize Codex's findings with own analysis]
```

The agent always uses safe defaults (gpt-5.2-codex, high reasoning, read-only sandbox).

## CLI Reference

Core command pattern for non-interactive execution:

```bash
echo "<prompt>" | codex exec \
  --model gpt-5.2-codex \
  --config reasoning_effort=high \
  --sandbox read-only \
  --ask-for-approval on-request \
  --skip-git-repo-check \
  2>/dev/null
```

**Key flags:**
- `--model, -m` - Model to use (gpt-5.2-codex recommended for code tasks)
- `--config reasoning_effort=<level>` - Reasoning depth (low/medium/high/xhigh)
- `--sandbox, -s` - Execution permissions (read-only safest)
- `--ask-for-approval, -a` - When to pause for human approval
- `--skip-git-repo-check` - Always include (works in non-git directories)
- `2>/dev/null` - Suppresses thinking tokens for clean output

**Note:** Interactive slash commands like `/review` only work in Codex's interactive mode, not via `codex exec`.

## Error Handling

### Codex Not Installed

If `codex` command not found, show:

```
Codex CLI is not installed.

To install:
  brew install codex

Then authenticate:
  codex login

For more info: https://developers.openai.com/codex/cli
```

### Common Issues

- **API key issues**: User needs to run `codex login` to authenticate
- **Timeout**: Complex tasks may take time - consider `--reasoning high` instead of `xhigh`
- **Sandbox errors**: If Codex needs to write files, user must specify `--sandbox workspace-write`

## Best Practices

1. **Start with defaults** - gpt-5.2-codex with high reasoning works well for most tasks
2. **Use xhigh sparingly** - Only for truly complex analysis where depth matters
3. **Keep read-only sandbox** - Unless task explicitly requires file modifications
4. **Be specific in prompts** - Tell Codex exactly what to look for
5. **Synthesize results** - When using codex-agent, combine Codex's findings with Claude's analysis
