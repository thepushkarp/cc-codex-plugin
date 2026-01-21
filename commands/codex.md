---
description: Invoke OpenAI Codex CLI for detail-oriented code review, planning, and analysis
allowed-tools: Bash
argument-hint: "[--model name] [--sandbox mode] <task>"
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
| `--model` | any model name | gpt-5.2-codex |
| `--sandbox` | read-only, workspace-write, danger-full-access | read-only |

**Parsing rules:**
1. Scan for `--model <value>` - accept any string
2. Scan for `--sandbox <value>` - validate it's one of: read-only, workspace-write, danger-full-access
3. Everything remaining after extracting flags is the TASK

**Validation errors:**
- If `--sandbox` has invalid value: "Invalid sandbox mode: '<value>'. Valid: read-only, workspace-write, danger-full-access"

## Step 3: Validate Task

If no task remains after parsing flags, respond with:

```
No task specified.

Usage:
  /codex [options] <task>

Examples:
  /codex review the auth middleware for security issues
  /codex --model gpt-5.2 analyze this algorithm for edge cases
  /codex --sandbox workspace-write generate tests for this module

Options:
  --model <name>       Model to use (default: gpt-5.2-codex)
  --sandbox <mode>     Sandbox: read-only (default), workspace-write, danger-full-access
```

## Step 4: Execute Codex

Build and execute the command with parsed values (or defaults). Use inline prompt (not piped stdin):

```bash
codex exec \
  --model <MODEL> \
  --sandbox <SANDBOX> \
  "<TASK>" \
  <STDERR_REDIRECT>
```

Where:
- `<TASK>` = the task text from arguments (passed as inline prompt)
- `<MODEL>` = parsed --model or "gpt-5.2-codex"
- `<SANDBOX>` = parsed --sandbox or "read-only"
- `<STDERR_REDIRECT>` = `2>&1` always (to capture full output)

**Example with defaults:**
```bash
codex exec \
  --model gpt-5.2-codex \
  --sandbox read-only \
  "review auth.py for security issues" \
  2>&1
```

**Example with different model:**
```bash
codex exec \
  --model gpt-5.2 \
  --sandbox read-only \
  "deeply analyze sorting algorithm" \
  2>&1
```

## Step 5: Return Output

Return Codex's output to the user. If the command fails, show the error message.

## Tips

- Default model `gpt-5.2-codex` is optimized for code tasks
- Use `--sandbox workspace-write` only when Codex needs to create/modify files
- Be specific in your task description for better results
