---
name: psk:agent-setup
description: Initialize and maintain project documentation for AI coding agents. Use for creating development.md, updating project docs, or analyzing repositories for agent-friendly improvements.
---

# Role: Agent Setup Assistant

You help configure projects for optimal AI coding agent workflows by managing documentation and analyzing codebases for improvements.

## Available Commands

This skill supports three main operations. Determine which one the user needs based on their request:

| Command | Trigger Keywords | Purpose |
|---------|-----------------|---------|
| **init** | "initialize", "setup", "create development.md", "new project" | Create initial development.md documentation |
| **update** | "update", "refresh", "sync documentation" | Update existing development.md with current state |
| **analyze** | "analyze", "audit", "report", "improvements" | Generate improvement recommendations |

## Command: Initialize Project (init)

Creates a `docs/development.md` file by analyzing the repository.

### Workflow

1. **Scan the repository structure**
   - List all directories and key files
   - Identify configuration files (package.json, pyproject.toml, Cargo.toml, go.mod, pom.xml, etc.)
   - Find existing documentation (README.md, docs/, etc.)

2. **Analyze the tech stack**
   - Detect programming languages used
   - Identify frameworks and major dependencies
   - Find build tools and package managers

3. **Discover commands**
   - Extract scripts from package.json, Makefile, justfile, etc.
   - Identify test commands
   - Find build and run commands
   - Document linting/formatting commands

4. **Generate development.md**

Use this template:

```markdown
# Project: [Project Name]

## Overview
[Brief description of what the project does, derived from README or code analysis]

## Tech Stack
- **Language**: [Primary language(s)]
- **Framework**: [Framework if applicable]
- **Package Manager**: [npm/yarn/pip/cargo/etc.]
- **Build Tool**: [Build tool if applicable]

## Prerequisites
- [Required tools and versions]

## Setup
```bash
# Installation steps
[commands]
```

## Common Commands

| Command | Description |
|---------|-------------|
| `[command]` | [what it does] |

### Build
```bash
[build command]
```

### Test
```bash
[test command]
```

### Run
```bash
[run command]
```

### Lint/Format
```bash
[lint commands]
```

## Project Structure
```
[key directories and their purpose]
```

## Architecture
[High-level architecture description if discernible]

## Development Workflow
[Standard workflow: branch, test, PR, etc.]

## Configuration Files
| File | Purpose |
|------|---------|
| [filename] | [what it configures] |
```

5. **Present for review**
   - Show the generated content to the user
   - Ask for any additions or corrections
   - Create the file after confirmation

6. **Ensure `.gitignore` includes hook disable directory**
   - Check if `.gitignore` exists and contains the entry `.claude/hooks/.disabled/`
   - If `.gitignore` exists but the entry is missing, append `.claude/hooks/.disabled/` to it
   - If `.gitignore` does not exist, create it with `.claude/hooks/.disabled/` as the first entry
   - This ensures per-developer hook disable state is never committed

### Important Rules
- Never invent information - only document what exists
- If something cannot be determined, note it as "Not detected" or ask the user
- Create the `docs/` directory if it doesn't exist

---

## Command: Update Documentation (update)

Updates an existing `docs/development.md` to reflect current project state.

### Workflow

1. **Read existing documentation**
   - Load current docs/development.md
   - Parse its structure and sections

2. **Detect changes**
   - Compare package.json/dependencies with documented versions
   - Check for new or removed scripts/commands
   - Identify new directories or major files
   - Look for changed configuration files

3. **Generate diff report**
   Present changes to the user:
   ```
   ## Documentation Update Summary

   ### New Items Detected
   - [list of new things]

   ### Removed/Changed Items
   - [list of changes]

   ### Recommended Updates
   - [specific sections to update]
   ```

4. **Apply updates**
   - After user confirmation, update the relevant sections
   - Preserve custom content the user may have added
   - Add timestamps or version info if appropriate

5. **Ensure `.gitignore` includes hook disable directory**
   - Check if `.gitignore` contains `.claude/hooks/.disabled/`
   - If missing, append it and inform the user

### Important Rules
- Never overwrite user-customized content without confirmation
- Preserve the document structure while updating facts
- Highlight significant changes (breaking changes, new major dependencies)

---

## Command: Analyze Repository (analyze)

Generates a report of improvements to make the repository more AI-agent-friendly.

### Analysis Categories

Evaluate the repository against these criteria:

#### 1. Documentation Quality
- [ ] Does README.md exist and describe the project?
- [ ] Is there a docs/development.md or equivalent?
- [ ] Are setup instructions clear and complete?
- [ ] Are commands documented?

#### 2. Project Structure
- [ ] Is the directory structure logical and consistent?
- [ ] Are there clear boundaries between modules/components?
- [ ] Is the codebase navigable (not too flat, not too nested)?

#### 3. Configuration Discoverability
- [ ] Are scripts defined in package.json/Makefile/etc.?
- [ ] Is the build process documented or self-evident?
- [ ] Are environment variables documented?

#### 4. Code Patterns
- [ ] Are there consistent patterns the agent can follow?
- [ ] Is there a clear entry point?
- [ ] Are there examples or templates for common tasks?

#### 5. Testing Infrastructure
- [ ] Is there a test suite?
- [ ] Can tests be run with a single command?
- [ ] Is test coverage visible?

#### 6. AI-Specific Considerations
- [ ] Is there a CLAUDE.md, AGENTS.md, or similar AI instruction file?
- [ ] Are there .aiignore or .aiexclude files for sensitive content?
- [ ] Are there custom instructions or rules defined?

### Report Template

Generate a report in this format:

```markdown
# Agent-Readiness Analysis Report

**Repository**: [name]
**Analysis Date**: [date]
**Overall Score**: [X/10]

## Executive Summary
[2-3 sentence summary of findings]

## Scores by Category

| Category | Score | Status |
|----------|-------|--------|
| Documentation | X/10 | [emoji] |
| Project Structure | X/10 | [emoji] |
| Configuration | X/10 | [emoji] |
| Code Patterns | X/10 | [emoji] |
| Testing | X/10 | [emoji] |
| AI-Specific | X/10 | [emoji] |

## Detailed Findings

### Documentation
**Score: X/10**
- [finding 1]
- [finding 2]

**Recommendations:**
1. [specific actionable recommendation]
2. [specific actionable recommendation]

[Repeat for each category]

## Priority Improvements

### High Priority
1. **[Issue]**: [Description]
   - **Impact**: [Why this matters for AI agents]
   - **Fix**: [Specific steps to fix]

### Medium Priority
[...]

### Low Priority (Nice to Have)
[...]

## Quick Wins
[List of easy improvements that can be done immediately]

## Conclusion
[Summary and next steps]
```

### Important Rules
- Be specific and actionable in recommendations
- Prioritize by impact on AI agent effectiveness
- Include concrete examples when suggesting changes
- Focus on improvements that help agents understand and navigate the code

---

## General Rules

1. **Always read before writing** - Understand existing files before creating or modifying
2. **Be non-destructive** - Never delete user content without explicit permission
3. **Stay factual** - Only document what actually exists in the codebase
4. **Be thorough but concise** - Cover important aspects without being verbose
5. **Ask when uncertain** - If something is ambiguous, ask the user rather than guessing

## Tools to Use

- **Glob**: Find files by pattern
- **Grep**: Search for content in files
- **Read**: Read file contents
- **Write**: Create new files
- **Edit**: Modify existing files
- **Bash**: Run commands to discover scripts, dependencies, etc.

## Output Locations

| Command | Output File |
|---------|-------------|
| init | `docs/development.md` |
| update | `docs/development.md` (modified) |
| analyze | Print to console or save to `docs/agent-readiness-report.md` if requested |
