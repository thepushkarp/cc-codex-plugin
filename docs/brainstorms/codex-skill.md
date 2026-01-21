# Codex Skill

## Overview

This skill integrates OpenAI's Codex CLI into Claude Code, leveraging Codex's exceptional code review and planning capabilities. When users need deep code analysis, bug hunting, or detailed implementation planning, Claude can delegate to Codex via `codex exec` - getting a second expert perspective.

The skill is invoked explicitly with `/codex`. A dedicated subagent allows Claude to spawn Codex for delegated tasks autonomously.

**Core philosophy**: Codex is the "detail-oriented reviewer" - sharp-eyed, thorough, and complementary to Claude's broader capabilities. We use `gpt-5.2-codex` with "high" reasoning by default for consistent quality.

## Goals

- Invoke Codex CLI via `codex exec` for tasks requiring sharp, detail-oriented analysis
- Provide explicit `/codex` command as the only trigger (no auto-triggering)
- Support a subagent that Claude can spawn to delegate tasks to Codex
- Default to `gpt-5.2-codex` model with "high" reasoning effort
- Allow user override to any reasoning level (low, medium, high, xhigh)
- Allow user override to any model if explicitly requested
- Use `read-only` sandbox for safety, `on-request` approval mode
- Suppress thinking tokens by default for clean output
- Primary use cases: code review, planning - but flexible for other exploratory tasks
- Show helpful install instructions if Codex CLI isn't installed
- User explicitly specifies context (files, diffs) - no auto-inclusion

## Non-Goals

- No auto-triggering on phrases - explicit invocation only
- No interactive mode support (slash commands like `/review` don't work via `codex exec`)
- No session resume functionality - each invocation is fresh
- No `workspace-write` or `danger-full-access` sandbox by default (user must explicitly request)
- Not a replacement for Claude's capabilities - a complement for detail-oriented work
- No MCP server integration within this skill
- No JSON/structured output - human-readable only

## User Experience

### Explicit Invocation via `/codex`
```
/codex review the authentication middleware for security issues
/codex plan out how to refactor the database layer
/codex explore edge cases in the payment flow
```

### With Options
```
/codex --reasoning xhigh deeply analyze this algorithm's complexity
/codex --sandbox workspace-write generate a refactoring plan with code
/codex --model gpt-5.2 use a different model for this task
```

### Subagent Delegation
When Claude determines Codex would be valuable, it can spawn a Codex subagent:
```
"Let me have Codex take a detailed look at this code for potential bugs..."
[Spawns codex-agent subagent with the task]
[Returns Codex's findings to the user]
```

### Output Behavior
- Clean, focused output (thinking tokens suppressed)
- User can request `--verbose` to see full reasoning
- Results returned inline in the conversation

### Override Defaults
- `--reasoning <level>` - low, medium, high (default), xhigh
- `--model <model>` - gpt-5.2-codex (default), or any other model
- `--sandbox <mode>` - read-only (default), workspace-write, danger-full-access
- `--verbose` - show thinking tokens

## Technical Approach

### Skill Structure
```
.claude/
├── skills/
│   └── codex.md          # Main skill definition + trigger
├── agents/
│   └── codex-agent.md    # Subagent for delegated Codex tasks
└── commands/
    └── codex.md          # /codex slash command
```

### Core Command Pattern
```bash
echo "<user_prompt>" | codex exec \
  --model gpt-5.2-codex \
  --config reasoning_effort=high \
  --sandbox read-only \
  --ask-for-approval on-request \
  --skip-git-repo-check \
  2>/dev/null
```

### Key Implementation Details
- Use `codex exec` for non-interactive execution (pipes prompt via stdin)
- `--skip-git-repo-check` always included (avoids errors in non-git dirs)
- `2>/dev/null` suppresses thinking tokens unless `--verbose` specified
- Working directory inherited from current Claude session
- Model always `gpt-5.2-codex` unless user explicitly overrides
- Check if `codex` command exists; show install instructions if not

### Subagent Pattern
The `codex-agent` is a Task-spawnable agent that:
1. Receives a task description from Claude
2. Constructs and executes the `codex exec` command
3. Returns Codex's output back to Claude for synthesis

## Key Components

### 1. `/codex` Command (`.claude/commands/codex.md`)
- Parses arguments: `--reasoning`, `--sandbox`, `--verbose`, `--model`
- Builds the `codex exec` command with appropriate flags
- Executes and returns output to user
- Handles errors gracefully (Codex not installed, API issues)

### 2. Skill Definition (`.claude/skills/codex.md`)
- Describes when/how to use Codex
- Trigger phrases for the skill (explicit `/codex` only)
- Documents the capabilities and best use cases
- Guides Claude on when to suggest using Codex or spawn the subagent

### 3. Codex Subagent (`.claude/agents/codex-agent.md`)
- Autonomous agent Claude can spawn via Task tool
- Receives task prompt, constructs `codex exec` invocation
- Has access to Bash tool only (to run codex)
- Returns findings/analysis back to parent conversation

### Component Relationships
```
User types /codex ──► commands/codex.md ──► runs codex exec ──► output

Claude decides    ──► spawns codex-agent ──► runs codex exec ──►
Codex would help                                                 │
                  ◄── synthesizes results ◄─────────────────────┘
```

## Resolved Decisions

- **Error handling**: Show helpful install instructions if Codex CLI isn't installed
- **File context**: User explicitly specifies what to review (no auto-inclusion of files/diff)
- **Output format**: Human-readable only (no JSON support)
- **Model override**: Allow any model override for flexibility

## References
- https://developers.openai.com/codex/cli/reference
- https://developers.openai.com/codex/cli/features

---
*Generated via /brainstorm on 2026-01-21*
