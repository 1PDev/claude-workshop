# Claude Code Workshop — Day 2

**Sub-agents & Multi-Agent Pipelines**

*Duration: 4 hours | Focus: Sub-agents, Parallelization, Pipelines, and CI/CD Integration*

---

## Day 2 Overview

Today you'll learn to scale your Claude Code workflows through delegation and parallelization. By the end of Day 2, you'll have:

- Custom sub-agents for specialized healthcare tasks
- Multi-agent pipelines for complex workflows
- Parallel development setups using git worktrees
- CI/CD integration for automated AI-powered checks

### What You'll Build

A **multi-agent code review and documentation pipeline** with:
- HIPAA Auditor sub-agent
- Test Generator sub-agent
- Code Reviewer sub-agent
- Documentation Writer sub-agent

### Prerequisites

- Completed Day 1 (CLAUDE.md, Skills, MCP, Hooks configured)
- Git repository with at least one feature branch
- GitHub CLI (gh) installed and authenticated

---

## Module 6: Understanding Sub-agents (60 minutes)

Sub-agents are specialized AI assistants that Claude Code can delegate tasks to. Each sub-agent has its own context window, custom system prompt, and tool permissions — preventing context pollution and enabling parallel execution.

### Why Sub-agents?

| Benefit | Description |
|---------|-------------|
| **Context Isolation** | Each sub-agent has its own context, preventing pollution of the main conversation |
| **Parallelization** | Multiple sub-agents can run concurrently |
| **Specialization** | Tailored prompts with specific expertise |
| **Tool Control** | Limit each sub-agent to specific tools, reducing risk |
| **Fresh Perspective** | No bias from earlier conversation context |

### Built-in Sub-agents

Claude Code includes several built-in sub-agents:

| Sub-agent | Purpose | Model |
|-----------|---------|-------|
| **General-purpose** | Complex, multi-step tasks requiring exploration and modification | Sonnet |
| **Explore** | Read-only codebase exploration | Haiku (efficient) |
| **Plan** | Research before creating plans (used in Plan Mode) | Sonnet |

### Sub-agent Locations: Project vs. Personal

Like Skills, sub-agents can be project-specific or personal:

| Location | Scope | Best For |
|----------|-------|----------|
| `.claude/agents/` | Project-specific (commit to git) | Team reviewers, project validators |
| `~/.claude/agents/` | Personal/Global (your machine only) | Personal productivity agents |

### Creating Custom Sub-agents

Sub-agents are defined as markdown files with YAML frontmatter:

```markdown
# .claude/agents/code-reviewer.md

---
name: code-reviewer
description: Expert code review specialist. Use for quality,
  security, and maintainability reviews.
tools: Read, Grep, Glob
model: sonnet
---

You are a code review specialist with expertise in:
- Security vulnerabilities
- Performance issues
- Code quality and maintainability
- HIPAA compliance for healthcare code
- FastAPI best practices

When reviewing code:
1. Identify security vulnerabilities first
2. Check for performance issues
3. Verify coding standards adherence
4. Suggest specific improvements
```

### Sub-agent Configuration Options

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier for the sub-agent |
| `description` | Yes | When to use this sub-agent (trigger keywords) |
| `tools` | No | Allowed tools (default: all) |
| `model` | No | Model to use: `sonnet`, `haiku`, `opus` |
| `skills` | No | Skills to include in sub-agent context |

### Healthcare-Specific Sub-agents Suite

Build a team of specialized sub-agents:

```
┌─────────────────────────────────────────────────────────┐
│                    Main Claude Session                   │
│                                                          │
│   "Review the patient-service module for production"     │
└────────────────────────┬────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┬────────────────┐
         ▼               ▼               ▼                ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   HIPAA     │  │    Test     │  │    Code     │  │    Doc      │
│   Auditor   │  │  Generator  │  │  Reviewer   │  │   Writer    │
├─────────────┤  ├─────────────┤  ├─────────────┤  ├─────────────┤
│ PHI scans   │  │ pytest      │  │ Security    │  │ API docs    │
│ Encryption  │  │ Async tests │  │ Performance │  │ FHIR notes  │
│ Audit logs  │  │ Edge cases  │  │ Standards   │  │ Examples    │
└─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘
```

### Hands-On Exercise 2A: Create Healthcare Sub-agents

**Time:** 30 minutes

Create the following sub-agents in `.claude/agents/`:

1. **hipaa-auditor.md** — Scans for PHI exposure, encryption issues
2. **test-generator.md** — Creates pytest tests with anonymized data
3. **code-reviewer.md** — Reviews for quality and security
4. **doc-writer.md** — Generates API documentation

For each sub-agent:
- Define appropriate tool restrictions
- Include the `hipaa-validator` skill where relevant
- Add healthcare-specific instructions

**Test your sub-agents:**
```bash
# View available agents
> /agents

# Invoke a specific agent
> Use the hipaa-auditor to scan app/services/patient_service.py
```

**Success Criteria:**
- [ ] All four sub-agents created in `.claude/agents/`
- [ ] `/agents` shows all four agents
- [ ] Each agent has appropriate tool restrictions
- [ ] Agents load the hipaa-validator skill when needed
- [ ] Agents committed to git for team use

---

## Module 7: Building Multi-Agent Pipelines (60 minutes)

Chain sub-agents together to create sophisticated development pipelines. This enables reproducible, governed workflows that handle complex multi-step processes.

### Pipeline Design Patterns

**Pattern 1: Sequential Pipeline**
```
Input → Agent A → Agent B → Agent C → Output
```
Use when each step depends on the previous step's output.

**Pattern 2: Parallel Pipeline**
```
         ┌→ Agent A ─┐
Input ───┼→ Agent B ─┼→ Aggregator → Output
         └→ Agent C ─┘
```
Use when tasks are independent and can run concurrently.

**Pattern 3: Conditional Pipeline**
```
Input → Classifier → (if type A) → Agent A
                   → (if type B) → Agent B
```
Use when different inputs need different processing.

### Example: Code Review Pipeline

```markdown
# .claude/commands/review-pipeline.md

Run the complete code review pipeline on $ARGUMENTS:

## Pipeline Stages

### Stage 1: HIPAA Audit (Parallel)
Use the hipaa-auditor agent to:
- Scan for PHI in logs and outputs
- Check encryption configuration
- Verify audit logging

### Stage 2: Security Review (Parallel)
Use the code-reviewer agent to:
- Check for OWASP Top 10 vulnerabilities
- Review authentication/authorization
- Identify injection risks

### Stage 3: Test Coverage (Sequential, after Stage 1-2)
Use the test-generator agent to:
- Generate tests for uncovered code paths
- Create tests for identified edge cases
- Add regression tests for any found issues

### Stage 4: Documentation (Sequential, after Stage 3)
Use the doc-writer agent to:
- Update API documentation
- Add compliance notes
- Document any breaking changes

## Output
Compile a final report with:
- [ ] HIPAA compliance status
- [ ] Security issues found
- [ ] Test coverage delta
- [ ] Documentation updates made
```

### Invoking Pipelines

```bash
# Run the full pipeline
> /project:review-pipeline app/services/patient_service.py

# Or invoke stages manually
> First, use hipaa-auditor to scan app/services/
> Then, use code-reviewer to check for security issues
> Finally, use test-generator to create tests for the issues found
```

### Hands-On Exercise 2B: Build a Code Review Pipeline

**Time:** 25 minutes

1. Create `.claude/commands/review-pipeline.md` with:
   - HIPAA audit stage
   - Security review stage
   - Test generation stage
   - Documentation stage

2. Test the pipeline on a sample module:
   ```bash
   > /project:review-pipeline app/routers/appointments.py
   ```

3. Verify each stage runs and produces output

4. Refine the pipeline based on results

**Success Criteria:**
- [ ] Pipeline command created
- [ ] All four stages execute
- [ ] Sub-agents are invoked correctly
- [ ] Final report is generated
- [ ] Pipeline committed to git

---

## Module 8: Parallelizing Work with Git Worktrees (45 minutes)

Run multiple Claude Code sessions simultaneously with complete code isolation. Git worktrees allow you to check out multiple branches at once, each in its own directory.

### Why Git Worktrees?

| Approach | Isolation | Speed | Use Case |
|----------|-----------|-------|----------|
| Single branch | None | Fast switch | Simple features |
| `git stash` | Temporary | Medium | Quick context switch |
| Multiple clones | Full | Slow | Long-running parallel work |
| **Git Worktrees** | Full | Fast | **Parallel Claude sessions** |

### Creating Worktrees

```bash
# Create a worktree for a new feature branch
git worktree add ../patient-portal-auth -b feature/auth

# Create a worktree for an existing branch
git worktree add ../patient-portal-api feature/api

# List all worktrees
git worktree list

# Output:
# /home/dev/patient-portal        abc1234 [main]
# /home/dev/patient-portal-auth   def5678 [feature/auth]
# /home/dev/patient-portal-api    ghi9012 [feature/api]
```

### Running Parallel Claude Sessions

**Terminal 1: Auth Feature**
```bash
cd ../patient-portal-auth
claude
> Implement OAuth2 authentication for the patient portal
```

**Terminal 2: API Feature**
```bash
cd ../patient-portal-api
claude
> Build the appointment scheduling API endpoints
```

Both sessions run independently with full code isolation!

### Worktree Management

```bash
# Remove a worktree (keeps the branch)
git worktree remove ../patient-portal-auth

# Prune stale worktree references
git worktree prune

# Move a worktree
git worktree move ../patient-portal-auth ../auth-feature
```

### Hands-On Exercise 2C: Parallel Development with Worktrees

**Time:** 20 minutes

1. Create two worktrees for parallel features:
   ```bash
   git worktree add ../portal-feature-a -b feature/a
   git worktree add ../portal-feature-b -b feature/b
   ```

2. Open two terminal windows and start Claude in each:
   ```bash
   # Terminal 1
   cd ../portal-feature-a && claude
   
   # Terminal 2
   cd ../portal-feature-b && claude
   ```

3. Give each Claude a different task:
   - Terminal 1: "Add input validation to the patient form"
   - Terminal 2: "Add error handling to the appointment service"

4. Verify changes are isolated to each worktree

5. Clean up:
   ```bash
   git worktree remove ../portal-feature-a
   git worktree remove ../portal-feature-b
   ```

**Success Criteria:**
- [ ] Both worktrees created successfully
- [ ] Claude sessions run independently
- [ ] Changes in one worktree don't affect the other
- [ ] Worktrees cleaned up properly

---

## Module 9: CI/CD Integration (45 minutes)

Integrate Claude Code into your CI/CD pipeline for automated code review, compliance checking, and documentation generation.

### Headless Mode

Run Claude Code without interactive input using the `-p` (print) flag:

```bash
# Run a single prompt and exit
claude -p "Check app/services/ for HIPAA violations" --output-format json

# Pipe input to Claude
cat error.log | claude -p "Explain this error and suggest fixes"

# Use in scripts
result=$(claude -p "List all TODO comments in the codebase")
```

### Output Formats

| Format | Flag | Use Case |
|--------|------|----------|
| Text | (default) | Human-readable output |
| JSON | `--output-format json` | Parsing in scripts |
| Stream | `--output-format stream-json` | Real-time processing |

### GitHub Actions Integration

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review

on:
  pull_request:
    branches: [main]

jobs:
  hipaa-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code
      
      - name: Run HIPAA Compliance Check
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          # Get changed files
          CHANGED_FILES=$(git diff --name-only origin/main...HEAD | grep '\.py$' || true)
          
          if [ -n "$CHANGED_FILES" ]; then
            claude -p "Check these files for HIPAA violations: $CHANGED_FILES" \
              --output-format json > hipaa-report.json
            
            # Check for critical violations
            if grep -q '"severity": "CRITICAL"' hipaa-report.json; then
              echo "❌ Critical HIPAA violations found!"
              cat hipaa-report.json
              exit 1
            fi
          fi
      
      - name: Upload Report
        uses: actions/upload-artifact@v4
        with:
          name: hipaa-report
          path: hipaa-report.json
```

### Resume and Continue Conversations

```bash
# Continue the most recent conversation
claude --continue

# Show conversation picker to select which to resume
claude --resume

# Continue with a specific prompt
claude --continue --print "Continue implementing the patient API"
```

### Hands-On Exercise 2D: Create a CI/CD Integration

**Time:** 20 minutes

1. Create `.github/workflows/ai-review.yml` with:
   - Trigger on pull requests to main
   - HIPAA compliance check step
   - Security review step (optional)
   - Report upload

2. Create a test pull request to verify the workflow

3. Review the generated reports

**Success Criteria:**
- [ ] Workflow file created
- [ ] Workflow triggers on PR
- [ ] HIPAA check runs successfully
- [ ] Report is uploaded as artifact
- [ ] Critical violations fail the build

---

## Day 2 Capstone: Complete Healthcare Development Pipeline

**Time:** 45 minutes

Combine everything from Days 1 and 2 to build a production-ready development pipeline.

### Pipeline Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                    Developer Workflow                           │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Start Feature                                               │
│     └─▶ git worktree add ../feature -b feature/name            │
│         └─▶ cd ../feature && claude                            │
│                                                                 │
│  2. Implement (with hooks)                                      │
│     └─▶ Auto-format on save                                    │
│     └─▶ PHI scanner on file write                              │
│     └─▶ Block dangerous commands                               │
│                                                                 │
│  3. Review Pipeline                                             │
│     └─▶ /project:review-pipeline                               │
│         ├─▶ hipaa-auditor (parallel)                           │
│         ├─▶ code-reviewer (parallel)                           │
│         ├─▶ test-generator (sequential)                        │
│         └─▶ doc-writer (sequential)                            │
│                                                                 │
│  4. Create PR                                                   │
│     └─▶ /project:create-pr                                     │
│         └─▶ CI/CD runs automated checks                        │
│                                                                 │
│  5. Merge                                                       │
│     └─▶ Clean up worktree                                      │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### Final Exercise: End-to-End Feature Implementation

Implement a new patient appointment feature using your complete pipeline:

1. **Create worktree:**
   ```bash
   git worktree add ../portal-appointments -b feature/appointments
   cd ../portal-appointments
   claude
   ```

2. **Plan the implementation:**
   ```bash
   > Enter Plan Mode and design the appointment scheduling API
   ```

3. **Implement with hooks active:**
   ```bash
   > Implement the appointment model and router
   # Hooks auto-format and scan for PHI
   ```

4. **Run review pipeline:**
   ```bash
   > /project:review-pipeline app/routers/appointments.py
   ```

5. **Generate tests:**
   ```bash
   > Use test-generator to create tests for the appointments module
   ```

6. **Create PR:**
   ```bash
   > /project:create-pr
   ```

7. **Clean up:**
   ```bash
   cd ../patient-portal
   git worktree remove ../portal-appointments
   ```

### Validation Checklist

- [ ] Feature implemented with all hooks running
- [ ] HIPAA audit passed
- [ ] Security review passed
- [ ] Tests generated with >80% coverage
- [ ] Documentation updated
- [ ] PR created and CI/CD passed
- [ ] Worktree cleaned up

---

## Day 2 Summary

### What You Learned

| Topic | Key Takeaway |
|-------|--------------|
| **Sub-agents** | Isolated contexts for specialized tasks |
| **Multi-agent Pipelines** | Chain agents for complex workflows |
| **Git Worktrees** | Parallel development with full isolation |
| **CI/CD Integration** | Automated AI-powered checks in your pipeline |

### Files Created Today

| File | Purpose |
|------|---------|
| `.claude/agents/hipaa-auditor.md` | HIPAA compliance sub-agent |
| `.claude/agents/test-generator.md` | Test generation sub-agent |
| `.claude/agents/code-reviewer.md` | Code review sub-agent |
| `.claude/agents/doc-writer.md` | Documentation sub-agent |
| `.claude/commands/review-pipeline.md` | Multi-agent pipeline |
| `.github/workflows/ai-review.yml` | CI/CD integration |

### Preparing for Day 3

Tomorrow you'll learn about:
- **AI Evaluations** — The Three Gulfs and why evals matter
- **Evaluation Types** — Code-based, LLM-as-Judge, Human-in-Loop
- **Golden Datasets** — Building your source of truth
- **Production Monitoring** — Observability and continuous improvement

**Homework (Optional):**
1. Create additional sub-agents for your team's specific needs
2. Expand the CI/CD workflow with more checks
3. Document your pipeline for team onboarding

---

## Quick Reference

### Sub-agent Commands

| Command | Purpose |
|---------|---------|
| `/agents` | List available sub-agents |
| `Use [agent-name] to...` | Invoke a specific agent |

### Git Worktree Commands

| Command | Purpose |
|---------|---------|
| `git worktree add <path> -b <branch>` | Create new worktree with new branch |
| `git worktree add <path> <branch>` | Create worktree for existing branch |
| `git worktree list` | List all worktrees |
| `git worktree remove <path>` | Remove a worktree |

### Headless Mode

| Command | Purpose |
|---------|---------|
| `claude -p "prompt"` | Run single prompt |
| `claude --output-format json` | JSON output for parsing |
| `claude --continue` | Resume last conversation |
| `claude --resume` | Pick conversation to resume |

---

*End of Day 2 — See you tomorrow for AI Evaluations and Production Readiness!*
