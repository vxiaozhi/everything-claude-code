# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code plugin** repository containing production-ready configurations for Claude Code including agents, skills, hooks, commands, and rules. The plugin has cross-platform support (Windows, macOS, Linux) with all hooks and scripts written in Node.js for maximum compatibility.

**Key Architecture:**
- **Agents** (`agents/`): Specialized subagents with limited scope for delegation (planner, architect, code-reviewer, security-reviewer, tdd-guide, etc.)
- **Skills** (`skills/`): Workflow definitions and domain knowledge that agents/commands invoke
- **Commands** (`commands/`): Slash commands for quick execution (/plan, /tdd, /code-review, /build-fix, /e2e, etc.)
- **Hooks** (`hooks/`): Event-driven automations configured in `hooks.json`
- **Rules** (`rules/`): Always-follow guidelines (copy to user's `~/.claude/rules/` when installed)
- **Scripts** (`scripts/`): Cross-platform Node.js utilities for hooks and package management

## Common Development Commands

### Running Tests
```bash
# Run all tests
node tests/run-all.js

# Run individual test files
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

### Package Manager Setup
```bash
# Interactive setup
node scripts/setup-package-manager.js

# Set global package manager
node scripts/setup-package-manager.js --global pnpm

# Set project-specific package manager
node scripts/setup-package-manager.js --project bun

# Detect current setting
node scripts/setup-package-manager.js --detect
```

### Package Manager Detection
The plugin automatically detects preferred package manager with this priority:
1. Environment variable `CLAUDE_PACKAGE_MANAGER`
2. Project config `.claude/package-manager.json`
3. `package.json` `packageManager` field
4. Lock file detection (pnpm-lock.yaml, bun.lockb, yarn.lock, package-lock.json)
5. Global config `~/.claude/package-manager.json`
6. Fallback to first available (pnpm > bun > yarn > npm)

## Architecture Deep Dive

### Hook System
Hooks are configured in `hooks/hooks.json` and fire on specific events:

**PreToolUse**: Before tool execution
- Blocks dev servers outside tmux
- Warns about running long commands in tmux
- Blocks creation of unnecessary .md files
- Suggests manual compaction at intervals

**PostToolUse**: After tool execution
- Logs PR URLs after `gh pr create`
- Auto-formats JS/TS with Prettier after edits
- Runs TypeScript checks after .ts/.tsx edits
- Warns about console.log statements

**SessionStart**: On new session
- Loads previous context from session files
- Detects and reports package manager
- Shows available learned skills

**SessionEnd**: On session end
- Persists session state to `~/.claude/sessions/`
- Evaluates session for extractable patterns

**PreCompact**: Before context compaction
- Saves state to maintain context across compaction

**Stop**: After each response
- Checks for console.log in modified files

### Script Utilities
Located in `scripts/lib/`:

**utils.js**: Cross-platform utilities
- File operations (readFile, writeFile, findFiles)
- Directory management (ensureDir, getClaudeDir, getSessionsDir)
- Git operations (isGitRepo, getGitModifiedFiles)
- Date/time helpers (getDateTimeString, getDateString)
- Security-hardened command execution (commandExists, runCommand)

**package-manager.js**: Package manager abstraction
- Auto-detection from lock files, package.json, or config
- Command generation for npm/pnpm/yarn/bun
- Pattern matching for hook matchers (e.g., `getCommandPattern('dev')`)

### Agent-Skill-Command Relationship
- **Commands** invoke specific **skills** or **agents**
- **Agents** use **skills** for domain knowledge
- **Skills** contain reusable workflows and patterns

Example flow:
1. User runs `/plan` → invokes `plan.md` command
2. Command delegates to `planner.md` agent
3. Agent uses `tdd-workflow/SKILL.md` for methodology

### Memory Persistence System
The Longform Guide (see README) implements:
- **Session files**: Stored in `~/.claude/sessions/*.tmp`
- **Learned skills**: Auto-extracted patterns to `~/.claude/skills/learned/`
- **Compaction suggestions**: Strategic context reduction
- **Verification loops**: Continuous evaluation checkpoints

### Plugin Manifest
`.claude-plugin/plugin.json` defines:
- Metadata (name, description, author)
- Component paths (commands, skills, agents, rules, hooks)
- Marketplace configuration for `/plugin marketplace add`

## File Organization Conventions

### Agents (`agents/*.md`)
Each agent file must have a frontmatter section:
```markdown
---
name: agent-name
description: What this agent does
tools: Read, Grep, Glob, Bash
model: opus
---

You are a specialized agent...
```

### Skills (`skills/*/SKILL.md`)
Skills are organized by domain:
- `coding-standards/`: Language best practices
- `backend-patterns/`: API, database, caching patterns
- `frontend-patterns/`: React, Next.js patterns
- `tdd-workflow/`: Test-driven development methodology
- `continuous-learning/`: Auto-extract patterns from sessions
- `security-review/`: Security checklist
- `verification-loop/`: Continuous verification patterns

### Commands (`commands/*.md`)
Slash commands follow naming convention: `command-name.md` → `/command-name`

### Hooks (`hooks/hooks.json`)
Hook matchers use expression syntax:
```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "node \"${CLAUDE_PLUGIN_ROOT}/scripts/hook.js\""
  }]
}
```

## Cross-Platform Compatibility

All scripts use Node.js with:
- `path.join()` for file paths (no hardcoded `/`)
- `fs.existsSync()` before file operations
- `spawnSync`/`execSync` with proper error handling
- Platform detection (`isWindows`, `isMacOS`, `isLinux`)

## Important Constraints

- **No native binaries**: Use Node.js for all hooks/scripts
- **Security**: Never execute user-controlled shell commands directly
- **Context window**: Don't enable all MCPs at once (recommend <10 active, <80 tools)
- **Hook blocking**: Hooks that exit(1) will block the tool execution
- **Package manager**: Always use `getPackageManager()` for command generation

## Installation as Plugin

```bash
# Add marketplace
/plugin marketplace add affaan-m/everything-claude-code

# Install plugin
/plugin install everything-claude-code@everything-claude-code
```

Or manually add to `~/.claude/settings.json`:
```json
{
  "extraKnownMarketplaces": {
    "everything-claude-code": {
      "source": {
        "source": "github",
        "repo": "affaan-m/everything-claude-code"
      }
    }
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": true
  }
}
```

## Key File Locations

- Plugin manifest: `.claude-plugin/plugin.json`
- Hook configuration: `hooks/hooks.json`
- Cross-platform utilities: `scripts/lib/utils.js`
- Package manager abstraction: `scripts/lib/package-manager.js`
- Hook implementations: `scripts/hooks/*.js`
- Test suite: `tests/run-all.js`
- Marketplace config: `.claude-plugin/marketplace.json`
