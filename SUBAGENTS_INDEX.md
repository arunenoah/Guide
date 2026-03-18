# Custom Subagents Documentation Index

Complete documentation for creating and managing custom subagents in Claude Code with practical examples and best practices.

---

## Documentation Files

### 1. SUBAGENT_QUICK_START.md (12 KB, ~580 lines)
**Start here for rapid implementation**

Quick reference for getting started in 5 minutes:
- 30-second overview of subagents
- Step-by-step guide to create your first subagent
- 10 essential agent patterns with templates
- Common issues and solutions
- Cheat sheet for quick reference

**Best for:**
- Getting your first agent working quickly
- Choosing the right template for your use case
- Quick troubleshooting
- Understanding tool permissions at a glance

**Read first if:** You want to create a subagent immediately

---

### 2. CUSTOM_SUBAGENTS_GUIDE.md (51 KB, ~1,958 lines)
**Comprehensive reference guide**

Complete documentation covering all aspects of subagent development:

**Part 1: Foundations (Sections 1-3)**
- Getting started - concepts and architecture
- File structure and location
- Anatomy of a subagent - YAML frontmatter breakdown
- System prompt best practices

**Part 2: Complete Examples (Section 4)**
Five fully-documented real-world examples:
1. Database Query Validator (read-only, restricted)
2. Bug Investigator (full tools for debugging)
3. Performance Optimizer (specialized for optimization)
4. Security Auditor (security-focused analysis)
5. Documentation Generator (writes docs efficiently)

**Part 3: Advanced Features (Section 5)**
- Using persistent memory for learning patterns
- Using hooks for conditional validation
- Chaining subagents (calling one from another)
- Permission modes explained with examples

**Part 4: Best Practices & Reference (Sections 6-8)**
- Naming conventions with examples
- Description writing for auto-delegation
- Tool restriction strategy matrix
- Testing your subagent checklist
- Version control checklist
- Troubleshooting guide
- Quick reference tables and cheat sheets

**Best for:**
- Deep understanding of subagent system
- Learning advanced features like memory and hooks
- Understanding permission models thoroughly
- Creating specialized agents for complex workflows
- Troubleshooting non-obvious issues

**Read for:** In-depth learning and advanced use cases

---

## Documentation Sections Map

### Quick Navigation by Topic

| Topic | Location |
|-------|----------|
| **Getting Started** | Quick Start: 30-Second Overview |
| **Your First Agent** | Quick Start: Create Your First Subagent |
| **Agent Templates** | Quick Start: 10 Essential Patterns; Full Guide: Examples |
| **Tool Permissions** | Quick Start: Tool Permissions; Full Guide: Advanced Features |
| **Naming Conventions** | Full Guide: Best Practices |
| **System Prompts** | Full Guide: Anatomy of a Subagent |
| **Workflows** | Quick Start: Connecting Agents; Full Guide: Chaining Subagents |
| **Troubleshooting** | Quick Start: Common Issues; Full Guide: Troubleshooting |
| **Testing** | Full Guide: Best Practices - Testing Your Subagent |
| **Memory & Learning** | Full Guide: Advanced Features - Persistent Memory |
| **Real Examples** | Quick Start: Real-World Examples; Full Guide: Complete Examples |

---

## Documentation Content Structure

### SUBAGENT_QUICK_START.md

```
30-Second Overview
Create Your First Subagent (5 Minutes)
├─ Step 1: Create the File
├─ Step 2: Invoke the Subagent
└─ Step 3: Iterate
Common Subagent Templates
├─ Template 1: Read-Only Validator
├─ Template 2: Analyzer with Testing
└─ Template 3: Full Implementation
10 Essential Agent Patterns
├─ Pattern 1-10: Different agent configurations
File Location & Naming
Tool Permissions Quick Reference
Testing Your Subagent
Real-World Examples
├─ Example 1: Quick Code Validator
├─ Example 2: Performance Investigator
└─ Example 3: Security Checker
Workflow: Connecting Multiple Agents
Tips & Tricks
Common Issues & Solutions
Next Steps
Resources
Summary Cheat Sheet
```

### CUSTOM_SUBAGENTS_GUIDE.md

```
1. Getting Started
├─ What is a Custom Subagent?
├─ When to Create One vs Using Built-in Agents
└─ Architecture Overview

2. File Structure & Location
├─ Directory Organization
├─ Scope Priority (Load Order)
└─ File Naming Conventions

3. Anatomy of a Subagent
├─ File Structure
├─ YAML Frontmatter Breakdown
│  ├─ name
│  ├─ description
│  ├─ model
│  ├─ color
│  └─ tools
└─ System Prompt Best Practices

4. Complete Examples
├─ Example 1: Database Query Validator
├─ Example 2: Bug Investigator
├─ Example 3: Performance Optimizer
├─ Example 4: Security Auditor
└─ Example 5: Documentation Generator

5. Advanced Features
├─ Using Persistent Memory for Learning Patterns
├─ Using Hooks for Conditional Validation
├─ Chaining Subagents (Calling One From Another)
└─ Permission Modes Explained with Examples

6. Best Practices
├─ Naming Conventions
├─ Description Writing (Important for Auto-Delegation)
├─ Tool Restrictions Strategy
├─ Testing Your Subagent
└─ Version Control Checklist

7. Troubleshooting
├─ Common Mistakes
├─ Debugging Subagent Behavior
└─ Permission Issues

8. Quick Reference
├─ Configuration Cheatsheet
├─ Common Patterns
└─ Tool Allowlist/Denylist Examples
```

---

## Learning Paths

### Path 1: Quick Implementation (15 minutes)
**Goal:** Create your first working subagent

1. Read: Quick Start - 30-Second Overview
2. Read: Quick Start - Create Your First Subagent
3. Read: Quick Start - Common Subagent Templates
4. Do: Create one subagent using a template
5. Read: Quick Start - Testing Your Subagent
6. Test and iterate

**Result:** Working subagent for your specific use case

---

### Path 2: Understanding Best Practices (45 minutes)
**Goal:** Learn how to create quality subagents

1. Complete Path 1 above
2. Read: Full Guide - Getting Started
3. Read: Full Guide - Anatomy of a Subagent
4. Read: Full Guide - Best Practices (all sections)
5. Review: Full Guide - Complete Examples (at least 2)
6. Do: Refactor your first agent based on best practices

**Result:** Well-designed, maintainable subagent

---

### Path 3: Advanced Workflows (2 hours)
**Goal:** Build multi-agent workflows with advanced features

1. Complete Path 2 above
2. Read: Full Guide - Advanced Features (all sections)
3. Read: Full Guide - Complete Examples (all 5 examples)
4. Read: Full Guide - Troubleshooting
5. Do: Create 2-3 specialized agents for different domains
6. Do: Design a workflow connecting 3+ agents

**Result:** Advanced multi-agent system for complex processes

---

### Path 4: Full Mastery (4 hours)
**Goal:** Complete understanding of subagent system

1. Complete Path 3 above
2. Read: Full Guide - File Structure & Location (all details)
3. Read: Full Guide - Troubleshooting (all sections)
4. Read: Full Guide - Quick Reference (all sections)
5. Do: Create memory file for agent learning patterns
6. Do: Set up validation hooks for quality gates
7. Do: Document your agent system

**Result:** Expert-level understanding and custom agent ecosystem

---

## Quick Reference Tables

### File Locations

| Component | Path |
|-----------|------|
| Agent files | `.claude/agents/` |
| Agent README | `.claude/README.md` |
| Memory file | `.claude/memory/MEMORY.md` |
| Hooks (optional) | `.claude/hooks/` |
| Config | `.claude/settings.local.json` |

### Agent Scope Priority

1. **CLI Scope** - `.claude/agents/` in current directory (highest priority)
2. **Project Scope** - `.claude/agents/` in project root
3. **User Scope** - `~/.claude/agents/` in home directory
4. **Plugin Scope** - From installed plugins (lowest priority)

### Tool Combinations by Agent Type

| Agent Type | Read | Write | Edit | Bash | Grep | Glob | Notes |
|-----------|------|-------|------|------|------|------|-------|
| Validator | ✅ | ❌ | ❌ | ⚠️ | ✅ | ✅ | Read-only review |
| Reviewer | ✅ | ❌ | ❌ | ❌ | ✅ | ✅ | No code changes |
| Auditor | ✅ | ❌ | ❌ | ❌ | ✅ | ✅ | Security focused |
| Developer | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | Full access |
| Lead | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | No restrictions |

### Model Selection Guide

| Model | Best For | Speed | Cost | Thinking |
|-------|----------|-------|------|----------|
| `claude-haiku-4-5` | Read-only validation, quick checks | Very Fast | $ | Good |
| `inherit` | General purpose (uses project default) | Medium | $$ | Good |
| `claude-opus-4-6` | Complex analysis, reasoning | Slower | $$$ | Excellent |

### Color Codes (Suggested)

| Color | Typical Use | Examples |
|-------|------------|----------|
| Red | Critical/Leadership | tech-lead |
| Blue | Implementation | senior-engineer |
| Green | Quality review | code-reviewer |
| Yellow | Testing | qa-tester |
| Orange | Security | security-reviewer |
| Purple | Documentation | tech-writer |
| Grey | Utility | Default |

---

## Key Concepts Reference

### YAML Frontmatter Fields

```yaml
---
name: agent-name              # Required: kebab-case identifier
description: Short line       # Required: one-line summary
model: inherit                # Optional: inherit/claude-opus/claude-haiku
color: blue                   # Optional: UI indicator color
tools:                        # Optional: permission configuration
  allow: [Read, Grep]         # Allowlist: only these tools
  # OR
  deny: [Write, Bash]         # Denylist: all except these
---
```

### Scope Order

Agents load in priority order (first match wins):
1. CLI scope (highest priority)
2. Project scope
3. User scope
4. Plugin scope (lowest priority)

This allows:
- Override agents per project
- Share agents across projects
- Use plugin agents

### Permission Strategies

| Strategy | Use Case | Example |
|----------|----------|---------|
| Allowlist | Restrict to specific tools | Read-only validator |
| Denylist | Full access except specific | Block Write but allow Read |
| No restrictions | Full tool access | Senior developer agent |

### Output Formats

Recommended agent outputs:
- **Structured tables** for comparisons
- **Numbered lists** for steps/priorities
- **Code blocks** for examples
- **Headers** for organization
- **Bold/italic** for emphasis

---

## Common Patterns Quick Reference

### Pattern: Read-Only Validator
```yaml
model: claude-haiku-4-5  # Fast
tools:
  allow: [Read, Grep, Glob]
```

### Pattern: Full Implementation Agent
```yaml
model: inherit
tools:
  deny: [WebSearch]  # Stay focused
```

### Pattern: Complex Analyzer
```yaml
model: claude-opus-4-6  # Powerful
tools:
  allow: [Read, Grep, Glob, Bash]
```

### Pattern: Documentation Writer
```yaml
model: claude-haiku-4-5  # Fast
tools:
  allow: [Read, Write, Edit, Glob, Grep]
```

---

## Checklist: Before Creating Your Agent

- [ ] Understand your agent's primary responsibility
- [ ] Choose appropriate model (fast vs powerful)
- [ ] Decide on tool permissions
- [ ] Draft description for auto-delegation
- [ ] Plan output format for consistency
- [ ] Consider how agent fits in workflow
- [ ] Review similar agents for patterns
- [ ] Choose appropriate color indicator
- [ ] Plan testing strategy

---

## Checklist: After Creating Your Agent

- [ ] Agent appears in `/list-agents`
- [ ] Agent loads with `/invoke agent-name`
- [ ] File is in correct location (`.claude/agents/`)
- [ ] YAML frontmatter is valid
- [ ] Tool permissions work as expected
- [ ] Output format is consistent
- [ ] Agent works with sample input
- [ ] Documentation is complete
- [ ] Committed to version control

---

## Workflow Examples

### Single Agent Usage
```
User Query
    ↓
/invoke agent-name
    ↓
Agent Response
```

### Multi-Agent Workflow
```
Request
    ↓
/invoke tech-lead (design)
    ↓
/invoke senior-engineer (implement)
    ↓
/invoke code-reviewer (review)
    ↓
/invoke security-auditor (audit)
    ↓
/invoke qa-tester (test)
    ↓
/invoke tech-writer (document)
    ↓
Complete
```

### Handoff Pattern
```
@next-agent: Work from previous agent complete.
[Context summary]
Please proceed with [next phase]
```

---

## File Size Reference

| Document | Size | Lines | Reading Time |
|----------|------|-------|--------------|
| Quick Start | 12 KB | ~580 | 10-15 min |
| Full Guide | 51 KB | ~1,958 | 45-60 min |
| This Index | 8 KB | ~250 | 5-10 min |
| **Total** | **71 KB** | **~2,800** | **60-85 min** |

---

## How to Use These Documents

### Scenario 1: "I need to create a subagent RIGHT NOW"
1. Read: Quick Start (5-10 min)
2. Pick a template
3. Create and test
4. Done

### Scenario 2: "I want to understand the system deeply"
1. Read: Quick Start (10-15 min)
2. Read: Full Guide (45-60 min)
3. Review examples (20-30 min)
4. Create multiple test agents
5. Done

### Scenario 3: "I need to troubleshoot an issue"
1. Check: Quick Start - Common Issues & Solutions
2. If not resolved, check: Full Guide - Troubleshooting
3. Review: Full Guide - Examples for similar patterns
4. Modify and test

### Scenario 4: "I want to learn specific topics"
Use the Quick Navigation table above to find exact sections.

---

## Resources

### Primary Documentation
- **Quick Start Guide**: `/Users/arunkumar/Documents/Application/expenseSettle/SUBAGENT_QUICK_START.md`
- **Full Guide**: `/Users/arunkumar/Documents/Application/expenseSettle/CUSTOM_SUBAGENTS_GUIDE.md`
- **This Index**: `/Users/arunkumar/Documents/Application/expenseSettle/SUBAGENTS_INDEX.md`

### Project Agents
- **Tech Lead**: `.claude/agents/tech-lead.md`
- **Senior Engineer**: `.claude/agents/senior-engineer.md`
- **Code Reviewer**: `.claude/agents/code-reviewer.md`
- **QA Tester**: `.claude/agents/qa-tester.md`
- **Security Reviewer**: `.claude/agents/security-reviewer.md`
- **Tech Writer**: `.claude/agents/tech-writer.md`

### Related Documentation
- **Agent README**: `.claude/README.md`
- **Memory File**: `.claude/memory/MEMORY.md`
- **Project README**: `README.md`

---

## Summary

This documentation package provides:

1. **Quick Start Guide** (12 KB)
   - Fastest path to your first working agent
   - Essential patterns and templates
   - Quick troubleshooting

2. **Comprehensive Guide** (51 KB)
   - Complete system understanding
   - 5 full real-world examples
   - Advanced features and best practices
   - Detailed troubleshooting

3. **This Index** (8 KB)
   - Navigation and cross-referencing
   - Learning paths for different goals
   - Quick reference tables
   - Quick navigation by topic

**Total: 71 KB of documentation with 5 complete working examples and comprehensive reference material.**

Start with the Quick Start guide and progress to the Full Guide as needed. All examples are production-ready and can be adapted to your specific use cases.

