# Claude Code Workshop — Day 1

**Foundations & Core Configuration**

*Duration: 4 hours | Focus: CLAUDE.md, Agent Skills, MCP, Hooks, and Essential Workflows*

---

## Day 1 Overview

Today you'll establish the foundational configuration that powers effective Claude Code usage. By the end of Day 1, you'll have:

- A comprehensive CLAUDE.md tailored to your healthcare project
- Custom Agent Skills for HIPAA compliance validation
- MCP integrations for GitHub and database access
- Automated Hooks for deterministic code quality
- Mastery of essential workflows (Plan Mode, Extended Thinking, @ References)

### What You'll Build

A **HIPAA-compliant patient data validation system** with:
- Project configuration (CLAUDE.md)
- Custom compliance skill (hipaa-validator)
- GitHub integration for issue tracking (MCP)
- Auto-formatting and PHI scanning hooks

### Prerequisites

- Claude Code installed and configured (Pro, Max subscription, or API key)
- Python 3.11+ installed
- Git and GitHub CLI (gh) configured
- VS Code or preferred IDE with terminal access

---

## Module 1: CLAUDE.md Configuration (45 minutes)

CLAUDE.md is a special configuration file that provides Claude with persistent project context. It's like onboarding documentation that teaches Claude about your specific codebase, conventions, and constraints.

### Understanding the CLAUDE.md Hierarchy

Claude Code reads CLAUDE.md files from multiple locations, with more specific files taking precedence:

1. **Personal:** `~/.claude/CLAUDE.md` — Your preferences across all projects
2. **Project:** `./CLAUDE.md` — Team rules for the project (committed to git)
3. **Local:** `./CLAUDE.local.md` — Personal overrides (gitignored)
4. **Folder-specific:** `./src/auth/CLAUDE.md` — Module-specific rules

### What Makes a Good CLAUDE.md?

| Section | Purpose | Example |
|---------|---------|---------|
| **Tech Stack** | Tell Claude exact versions and tools | "Python 3.11+ with FastAPI" |
| **Code Standards** | Enforce consistency | "Use functional components with hooks" |
| **Key Commands** | Enable Claude to run tests/checks | "npm run test — Run test suite" |
| **Critical Rules** | Non-negotiable requirements | "NEVER log PHI" |

### Healthcare-Specific CLAUDE.md Template

```markdown
# Project: Patient Portal API

## Tech Stack
- Backend: Python 3.11+ with FastAPI
- Database: PostgreSQL with SQLAlchemy ORM
- Authentication: OAuth2 with SMART on FHIR
- Validation: Pydantic v2
- Testing: pytest with httpx

## HIPAA Compliance Rules (CRITICAL)
- NEVER log PHI (Protected Health Information)
- All patient data must be encrypted at rest and in transit
- Audit logging required for all data access
- Use anonymized/synthetic data in tests

## Evaluation Requirements
- All AI-generated code must pass HIPAA compliance eval
- Minimum 80% test coverage on generated code
- LLM outputs must be validated against golden dataset

## Common Commands
- uvicorn app.main:app --reload — Start development server
- pytest — Run test suite
- ruff check . — Run linter
- ruff format . — Format code
- alembic upgrade head — Run database migrations
- python -m pytest tests/evals/ — Run AI evaluation suite
```

### Hands-On Exercise 1A: Create Your Team's CLAUDE.md

**Time:** 15 minutes

1. Navigate to your project directory and run: `claude`
2. Run the initialization command: `/init`
3. Review the generated CLAUDE.md and customize it:
   - Add your specific tech stack
   - Include HIPAA compliance rules
   - Add your team's common commands
4. Test by asking Claude: "Explain the project structure and our coding standards"

**Success Criteria:**
- [ ] CLAUDE.md exists in project root
- [ ] Tech stack section is complete
- [ ] HIPAA rules are documented
- [ ] Common commands are listed
- [ ] Claude can accurately describe your project

---

## Module 2: Agent Skills — Extending Claude's Capabilities (60 minutes)

Agent Skills are modular capabilities that transform Claude from a general-purpose assistant into a domain specialist. They package instructions, metadata, and optional resources that Claude uses automatically when relevant.

### How Skills Work: Progressive Disclosure

Skills use a three-level loading system to optimize context usage:

| Level | When Loaded | Content | Token Cost |
|-------|-------------|---------|------------|
| 1. Metadata | Always (at startup) | name and description from YAML | ~100 tokens |
| 2. Instructions | When Skill is triggered | SKILL.md body | <5k tokens |
| 3. Resources | As needed during execution | Scripts, templates, references | Unlimited |

### Trigger Keywords

Claude reads the `description` field in your Skill's YAML frontmatter. When your prompt matches keywords in the description, Claude automatically loads the full Skill instructions.

**Example triggers for a HIPAA validator:**
- "accessibility", "a11y", "508", "WCAG" → Accessibility Skill
- "HIPAA", "PHI", "compliance", "audit" → HIPAA Validator Skill
- "test", "pytest", "coverage" → Test Generator Skill

### Skill Locations: Project vs. Personal

| Location | Scope | Best For |
|----------|-------|----------|
| `.claude/skills/` | Project-specific (commit to git) | Team standards, compliance rules |
| `~/.claude/skills/` | Personal/Global (your machine only) | General patterns you use everywhere |

### Creating a Custom Healthcare Skill

Skills are markdown files with YAML frontmatter:

```markdown
# File: .claude/skills/hipaa-validator/SKILL.md

---
name: hipaa-validator
description: Validates code for HIPAA compliance. Use when
  reviewing healthcare code, checking for PHI exposure,
  or auditing data handling practices.
---

# HIPAA Compliance Validator

## Validation Checklist
1. Check for PHI in logs (names, SSN, DOB, MRN)
2. Verify encryption on data at rest
3. Confirm audit logging is enabled
4. Check access control implementation

## Common Violations
- logger.info() or print() with patient data
- Unencrypted database connections
- Missing authentication dependencies (Depends)
- Raw SQL without parameterization
```

### Hands-On Exercise 1B: Build a HIPAA Validator Skill

**Time:** 20 minutes

1. Create the skill directory:
   ```bash
   mkdir -p .claude/skills/hipaa-validator
   ```

2. Create `.claude/skills/hipaa-validator/SKILL.md` with:
   - YAML frontmatter with name and description
   - Validation checklist
   - Common violations to detect
   - Example code patterns (good vs. bad)

3. Add a `scripts/check_phi.py` helper script (optional)

4. Test by asking Claude: "Check app/services/patient_service.py for HIPAA compliance"

5. Commit to git so your team can use it:
   ```bash
   git add .claude/skills/
   git commit -m "feat: add HIPAA validator skill"
   ```

**Success Criteria:**
- [ ] Skill loads when you mention "HIPAA" or "PHI"
- [ ] Claude follows the validation checklist
- [ ] Claude identifies common violations
- [ ] Skill is committed to git for team use

---

## Module 3: Model Context Protocol (MCP) Integration (45 minutes)

MCP is an open protocol that enables Claude to connect to external tools, databases, and APIs. Think of it as "USB-C for AI" — a universal connector that standardizes how Claude accesses external resources.

### MCP Architecture Overview

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Claude Code   │────▶│   MCP Client    │────▶│   MCP Server    │
│   (Host)        │     │   (Protocol)    │     │   (GitHub, DB)  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

- **MCP Hosts:** Applications like Claude Code that initiate connections
- **MCP Clients:** Protocol clients that maintain connections with servers
- **MCP Servers:** Lightweight programs exposing specific capabilities

### Adding MCP Servers

```bash
# Add GitHub MCP server (user-level)
claude mcp add github --scope user -- npx -y @modelcontextprotocol/server-github

# Add PostgreSQL database server
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres

# List configured servers
claude mcp list

# Test a server
claude mcp get github
```

### Project-Level MCP Configuration (.mcp.json)

Create an `.mcp.json` in your project root for team-wide MCP configuration:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

### Natural Language Queries with MCP

Once configured, you can ask Claude questions like:

| Query | What Happens |
|-------|--------------|
| "What's my next ticket?" | GitHub MCP finds issues assigned to you |
| "Show issues labeled 'hipaa'" | GitHub MCP filters by label |
| "List patients with appointments tomorrow" | PostgreSQL MCP queries database |
| "Create an issue for the login bug" | GitHub MCP creates new issue |

### Hands-On Exercise 1C: Configure MCP for Healthcare Workflow

**Time:** 15 minutes

1. Set up your GitHub token:
   ```bash
   export GITHUB_TOKEN="your_personal_access_token"
   ```

2. Create `.mcp.json` in your project root with GitHub configuration

3. Verify the server is working:
   ```bash
   claude mcp list
   ```

4. Test by asking Claude: "List open issues for this repository"

5. Commit `.mcp.json` to git (tokens come from environment variables)

**Success Criteria:**
- [ ] `.mcp.json` exists in project root
- [ ] `claude mcp list` shows github server
- [ ] Claude can list repository issues
- [ ] Configuration is committed to git

---

## Module 4: Hooks — Deterministic Control (45 minutes)

Hooks are shell commands that execute automatically at specific lifecycle points in Claude Code. Unlike Skills (which Claude may or may not use), Hooks are **guaranteed** to run — they provide deterministic control over Claude's behavior.

### Why Hooks Matter

| Approach | Behavior | Use Case |
|----------|----------|----------|
| **Instructions** | Claude *might* follow them | General guidance |
| **Skills** | Claude loads when *relevant* | Domain expertise |
| **Hooks** | *Always* execute at lifecycle point | Guaranteed actions |

### Lifecycle Events

| Event | When It Fires | Common Uses |
|-------|---------------|-------------|
| `PreToolUse` | Before a tool runs | Block dangerous commands |
| `PostToolUse` | After a tool completes | Auto-format, run linters |
| `UserPromptSubmit` | When user sends prompt | Inject context |
| `PermissionRequest` | On permission dialog | Auto-approve safe operations |
| `Stop` | When Claude finishes | Run final checks |
| `SubagentStop` | When subagent finishes | Aggregate results |
| `SessionStart` | On session start/resume | Load environment |
| `PreCompact` | Before context compaction | Save important state |

### Hook Configuration

Configure hooks in `.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "ruff format $CLAUDE_FILE_PATHS && ruff check --fix $CLAUDE_FILE_PATHS"
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/block-dangerous-commands.sh"
          }
        ]
      }
    ]
  }
}
```

### Exit Codes

| Exit Code | Behavior |
|-----------|----------|
| 0 | Success — continue normally |
| 2 | Block — stop the operation, feed stderr to Claude |

### Healthcare Hooks Examples

**1. Auto-PHI Scanner (PostToolUse)**
```json
{
  "matcher": "Write|Edit",
  "hooks": [{
    "type": "command",
    "command": "python .claude/skills/hipaa-validator/scripts/check_phi.py $CLAUDE_FILE_PATHS"
  }]
}
```

**2. Block Production Database (PreToolUse)**
```json
{
  "matcher": "Bash",
  "hooks": [{
    "type": "command",
    "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -q 'PROD_DATABASE'; then echo 'Blocked: Production database access' >&2; exit 2; fi"
  }]
}
```

**3. Auto-Format Python Files (PostToolUse)**
```json
{
  "matcher": "Write(*.py)",
  "hooks": [{
    "type": "command",
    "command": "ruff format $CLAUDE_FILE_PATHS && ruff check --fix $CLAUDE_FILE_PATHS"
  }]
}
```

### Hands-On Exercise 1D: Configure Healthcare Hooks

**Time:** 20 minutes

1. Create `.claude/settings.json` with:
   - PostToolUse hook for auto-formatting Python files
   - PostToolUse hook for PHI scanning on .py files
   - PreToolUse hook to block production database commands

2. Test the auto-format hook:
   - Ask Claude to create a poorly formatted Python file
   - Verify it gets auto-formatted after creation

3. Test the PHI scanner:
   - Ask Claude to create a file with a logging statement containing "patient.name"
   - Verify the hook catches the violation

4. Review hooks with `/hooks` command

**Success Criteria:**
- [ ] `.claude/settings.json` exists with hook configuration
- [ ] Auto-format runs after Python file creation
- [ ] PHI scanner runs after file writes
- [ ] `/hooks` shows configured hooks

---

## Module 5: Essential Workflows (45 minutes)

### Workflow 1: Plan Mode for Safe Code Analysis

Plan Mode instructs Claude to analyze your codebase with read-only operations before making any changes.

```bash
# Start Claude Code in Plan Mode
claude --permission-mode plan

# Or toggle during session with Shift+Tab
# Look for: ⏸ plan mode on
```

**When to use Plan Mode:**
- Multi-step implementations requiring many file changes
- Security audits and compliance reviews
- Exploring unfamiliar codebases
- Creating implementation plans before executing

### Workflow 2: Extended Thinking for Complex Problems

Extended thinking enables Claude to engage in deeper reasoning for complex decisions.

```bash
# Enable extended thinking with prompts:
> think deeply about the best approach for implementing OAuth2

# Intensify thinking depth:
> think hard about potential security vulnerabilities
> think longer about edge cases we should handle

# Toggle with Tab key during session
```

### Workflow 3: Referencing Files and Directories

Use the `@` symbol to quickly include file or directory contents:

```bash
# Reference a single file
> Explain the authentication logic in @app/auth/middleware.py

# Reference a directory
> What's the structure of @app/routers?

# Reference multiple files
> Compare @app/api/v1/endpoints.py and @app/api/v2/endpoints.py
```

### Workflow 4: Custom Slash Commands

Create reusable commands for your team's common workflows:

```markdown
# File: .claude/commands/hipaa-check.md

Review the file at $ARGUMENTS for HIPAA compliance:
1. Check for PHI in logs
2. Verify data encryption
3. Confirm audit logging
4. Report any violations found
```

Usage:
```bash
> /project:hipaa-check app/services/patient_service.py
```

### Hands-On Exercise 1E: Master Essential Workflows

**Time:** 15 minutes

1. **Plan Mode Practice:**
   - Enter Plan Mode with `Shift+Tab`
   - Ask Claude to analyze your authentication module
   - Review the plan without executing changes

2. **Extended Thinking:**
   - Ask Claude to "think deeply about potential security vulnerabilities in the patient data API"
   - Compare the depth of response with and without extended thinking

3. **File References:**
   - Use `@` to reference a specific file
   - Ask Claude to explain it

4. **Create a Slash Command:**
   - Create `.claude/commands/hipaa-check.md`
   - Test with `/project:hipaa-check <filename>`

**Success Criteria:**
- [ ] Can toggle Plan Mode on/off
- [ ] Extended thinking produces deeper analysis
- [ ] `@` references load file contents correctly
- [ ] Custom slash command works

---

## Day 1 Capstone: Build a HIPAA Validation System

**Time:** 30 minutes

Combine everything from Day 1 to build a complete validation system:

### Requirements

1. **CLAUDE.md** with healthcare-specific rules
2. **HIPAA Validator Skill** in `.claude/skills/`
3. **MCP Configuration** with GitHub for issue tracking
4. **Hooks** for auto-format and PHI scanning
5. **Slash Command** for manual compliance checks

### Validation Checklist

Test your complete system:

- [ ] Create a new Python file with intentional PHI logging
- [ ] Verify the PHI hook catches the violation
- [ ] Verify auto-formatting runs
- [ ] Use `/project:hipaa-check` to run manual check
- [ ] Ask Claude to create a GitHub issue for any violations found
- [ ] Commit all configuration to git

### Expected File Structure

```
your-project/
├── CLAUDE.md                          # Project configuration
├── .mcp.json                          # MCP server configuration
├── .claude/
│   ├── settings.json                  # Hooks configuration
│   ├── skills/
│   │   └── hipaa-validator/
│   │       ├── SKILL.md               # Skill definition
│   │       └── scripts/
│   │           └── check_phi.py       # PHI detection script
│   └── commands/
│       └── hipaa-check.md             # Slash command
```

---

## Day 1 Summary

### What You Learned

| Topic | Key Takeaway |
|-------|--------------|
| **CLAUDE.md** | Persistent project context — like onboarding docs for AI |
| **Agent Skills** | Domain specialization with progressive disclosure |
| **MCP** | External tool integration via standardized protocol |
| **Hooks** | Deterministic control — guaranteed to run |
| **Workflows** | Plan Mode, Extended Thinking, @ References, Slash Commands |

### Files Created Today

| File | Purpose |
|------|---------|
| `./CLAUDE.md` | Project configuration |
| `.claude/skills/hipaa-validator/SKILL.md` | HIPAA compliance skill |
| `.mcp.json` | MCP server configuration |
| `.claude/settings.json` | Hooks configuration |
| `.claude/commands/hipaa-check.md` | Compliance check command |

### Preparing for Day 2

Tomorrow you'll learn about:
- **Sub-agents** for task delegation and context isolation
- **Multi-agent pipelines** for complex workflows
- **Git worktrees** for parallel development
- **CI/CD integration** with headless mode

**Homework (Optional):**
1. Add a personal Skill to `~/.claude/skills/` for patterns you use across projects
2. Explore additional MCP servers at [modelcontextprotocol.io](https://modelcontextprotocol.io)
3. Create additional slash commands for your team's workflows

---

## Quick Reference

### File Locations

| Item | Project (Team) | Personal (Global) |
|------|----------------|-------------------|
| CLAUDE.md | `./CLAUDE.md` | `~/.claude/CLAUDE.md` |
| Skills | `.claude/skills/` | `~/.claude/skills/` |
| Commands | `.claude/commands/` | `~/.claude/commands/` |
| MCP Config | `.mcp.json` | `~/.claude/settings.json` |
| Hooks | `.claude/settings.json` | `~/.claude/settings.json` |

### Essential Commands

| Command | Purpose |
|---------|---------|
| `/init` | Initialize CLAUDE.md |
| `/hooks` | View configured hooks |
| `/mcp` | View MCP server status |
| `Shift+Tab` | Toggle Plan Mode |
| `Tab` | Toggle Extended Thinking |
| `@filename` | Reference file contents |

---

*End of Day 1 — See you tomorrow for Sub-agents and Multi-Agent Pipelines!*
