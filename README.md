# Claude Code Workshop

A comprehensive 3-day workshop for mastering Claude Code in enterprise environments, with a focus on healthcare/HIPAA compliance scenarios.

## Overview

This workshop teaches developers how to effectively use Claude Code for production software development. Through hands-on exercises, participants build a HIPAA-compliant patient data validation system while learning Claude Code's core features.

## Workshop Structure

| Day | Focus | Topics |
|-----|-------|--------|
| **Day 1** | Foundations & Core Configuration | CLAUDE.md, Agent Skills, MCP Integration, Hooks, Essential Workflows |
| **Day 2** | Sub-agents & Multi-Agent Pipelines | Custom Sub-agents, Pipeline Design, Git Worktrees, CI/CD Integration |
| **Day 3** | AI Evaluations & Production Readiness | Golden Datasets, Evaluator Types, CI/CD Gates, Production Monitoring |

## Contents

```
├── workshop/
│   ├── Workshop-Day1.md    # Foundations & Core Configuration
│   ├── Workshop-Day2.md    # Sub-agents & Multi-Agent Pipelines
│   └── Workshop-Day3.md    # AI Evaluations & Production Readiness
└── Claude-Code-Workshop.pptx  # Presentation slides
```

## Prerequisites

- Claude Code installed and configured (Pro, Max subscription, or API key)
- Python 3.11+
- Git and GitHub CLI (gh) configured
- VS Code or preferred IDE with terminal access

## What You'll Learn

### Day 1: Foundations
- Creating effective CLAUDE.md configuration files
- Building custom Agent Skills for domain-specific tasks
- Integrating external tools via Model Context Protocol (MCP)
- Setting up Hooks for deterministic code quality controls
- Mastering Plan Mode, Extended Thinking, and file references

### Day 2: Multi-Agent Systems
- Designing and deploying custom sub-agents
- Building multi-agent pipelines for complex workflows
- Parallel development with Git worktrees
- CI/CD integration with headless mode

### Day 3: Production Readiness
- Understanding the Three Gulfs of AI development
- Building golden datasets for evaluation
- Implementing code-based and LLM-as-judge evaluators
- Setting up CI/CD evaluation gates
- Production monitoring and observability

## What You'll Build

A **HIPAA-compliant patient data validation system** featuring:
- Project configuration (CLAUDE.md)
- Custom compliance skill (hipaa-validator)
- GitHub integration for issue tracking (MCP)
- Auto-formatting and PHI scanning hooks
- Multi-agent code review pipeline
- Production evaluation framework

## License

This work is licensed under the [Creative Commons Attribution 4.0 International License](LICENSE).

You are free to use, share, and adapt this material for any purpose, including commercial use, as long as you provide appropriate attribution to **1P Solutions** (https://1p.solutions/).

## Author

Created by **1P Solutions**
https://1p.solutions/
