---
description: Invoke OpenAI Codex CLI for detail-oriented code review, planning, and analysis
allowed-tools: Bash
argument-hint: "[--reasoning level] [--model name] [--sandbox mode] [--verbose] <task>"
---

# Codex Command

Invoke OpenAI's Codex CLI for detail-oriented code review, planning, and exploratory analysis.

## Arguments

$ARGUMENTS

## Step 1: Check if Codex is Installed

First, verify the Codex CLI is available:

```bash
command -v codex >/dev/null 2>&1
```

If not installed, respond with:

```
Codex CLI is not installed.

To install:
  brew install codex

Then authenticate:
  codex login

For more info: https://developers.openai.com/codex/cli
```

Do not proceed if Codex is not installed.

## Step 2: Parse Arguments

Extract optional flags from $ARGUMENTS:

| Flag | Valid Values | Default |
|------|--------------|---------|
| `--reasoning` | low, medium, high, xhigh | high |
| `--model` | any model name | gpt-5.2-codex |
| `--sandbox` | read-only, workspace-write, danger-full-access | read-only |
| `--verbose` | (flag, no value needed) | false |

**Parsing rules:**
1. Scan for `--reasoning <value>` - validate it's one of: low, medium, high, xhigh
2. Scan for `--model <value>` - accept any string
3. Scan for `--sandbox <value>` - validate it's one of: read-only, workspace-write, danger-full-access
4. Scan for `--verbose` flag (no value)
5. Everything remaining after extracting flags is the TASK

**Validation errors:**
- If `--reasoning` has invalid value: "Invalid reasoning level: '<value>'. Valid: low, medium, high, xhigh"
- If `--sandbox` has invalid value: "Invalid sandbox mode: '<value>'. Valid: read-only, workspace-write, danger-full-access"

## Step 3: Validate Task

If no task remains after parsing flags, respond with:

```
No task specified.

Usage:
  /codex [options] <task>

Examples:
  /codex review the auth middleware for security issues
  /codex --reasoning xhigh analyze this algorithm for edge cases
  /codex --verbose plan out the database refactoring

Options:
  --reasoning <level>  Reasoning effort: low, medium, high (default), xhigh
  --model <name>       Model to use (default: gpt-5.2-codex)
  --sandbox <mode>     Sandbox: read-only (default), workspace-write, danger-full-access
  --verbose            Show thinking tokens (default: suppressed)
```

## Step 4: Execute Codex

Build and execute the command with parsed values (or defaults):

```bash
echo "<TASK>" | codex exec \
  --model <MODEL> \
  --config reasoning_effort=<REASONING> \
  --sandbox <SANDBOX> \
  --ask-for-approval on-request \
  --skip-git-repo-check \
  <STDERR_REDIRECT>
```

Where:
- `<TASK>` = the task text from arguments
- `<MODEL>` = parsed --model or "gpt-5.2-codex"
- `<REASONING>` = parsed --reasoning or "high"
- `<SANDBOX>` = parsed --sandbox or "read-only"
- `<STDERR_REDIRECT>` = `2>/dev/null` if NOT verbose, empty string if --verbose

**Example with defaults:**
```bash
echo "review auth.py for security issues" | codex exec \
  --model gpt-5.2-codex \
  --config reasoning_effort=high \
  --sandbox read-only \
  --ask-for-approval on-request \
  --skip-git-repo-check \
  2>/dev/null
```

**Example with verbose and xhigh:**
```bash
echo "deeply analyze sorting algorithm" | codex exec \
  --model gpt-5.2-codex \
  --config reasoning_effort=xhigh \
  --sandbox read-only \
  --ask-for-approval on-request \
  --skip-git-repo-check
```

## Step 5: Return Output

Return Codex's output to the user. If the command fails, show the error message.

## Tips

- For complex code review, consider using `--reasoning xhigh` for more thorough analysis
- Use `--sandbox workspace-write` only when Codex needs to create/modify files
- Use `--verbose` to see Codex's reasoning process (helpful for debugging)
