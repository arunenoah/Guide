# Custom Subagents Quick Start

Get started creating your first custom subagent in 5 minutes.

---

## 30-Second Overview

A custom subagent is a specialized AI persona for specific tasks:

- **Read-only validators** → Review code without changing it
- **Focused specialists** → Optimize, audit, or analyze specific domains
- **Workflow agents** → Handle multi-step processes with handoffs
- **Tool-restricted roles** → Prevent accidental changes

---

## Create Your First Subagent (5 Minutes)

### Step 1: Create the File

Create `/Users/arunkumar/Documents/Application/expenseSettle/.claude/agents/my-validator.md`:

```markdown
---
name: my-validator
description: Validates code for specific issues
model: inherit
color: blue
tools:
  allow:
    - Read
    - Grep
    - Glob
---

name: my-validator
description: |
  Specialized validator for reviewing code and identifying issues.

## Responsibilities
- Review code for specific issues
- Provide clear feedback
- Never modify code (read-only)

## How I Work
1. You provide code or ask me to review files
2. I analyze systematically
3. I provide structured feedback with file references
4. I explain issues and suggest solutions

## Output Format
I structure findings as:

### Issues Found
1. **Issue Name** - File:Line
   - Problem: [Description]
   - Solution: [How to fix]

instructions: |
  You are a specialized validator. When reviewing code, analyze systematically
  and provide structured feedback with specific file locations and line numbers.
  Be thorough but concise. Focus on actionable issues.
```

### Step 2: Invoke the Subagent

```bash
# List available agents
/list-agents

# Invoke your agent
/invoke my-validator

# Ask it to do something
"Please review app/Models/User.php for security issues"
```

### Step 3: Iterate

Improve the agent by editing the description and instructions based on output.

---

## Common Subagent Templates

### Template 1: Read-Only Validator

```yaml
---
name: code-validator
description: Reviews code without making changes
model: claude-haiku-4-5
color: green
tools:
  allow:
    - Read
    - Grep
    - Glob
---
```

**Use for**: Code review, security audits, quality checks

### Template 2: Analyzer with Testing

```yaml
---
name: performance-analyzer
description: Analyzes performance bottlenecks
model: claude-opus-4-6
color: purple
tools:
  allow:
    - Read
    - Grep
    - Glob
    - Bash
---
```

**Use for**: Performance analysis, testing, investigation

### Template 3: Full Implementation

```yaml
---
name: documentation-writer
description: Generates documentation from code
model: inherit
color: purple
tools:
  deny:
    - WebSearch
---
```

**Use for**: Documentation generation, writing features

---

## 10 Essential Agent Patterns

### 1. Read-Only Reviewer
Focus: Quality gates, no changes
```yaml
tools:
  allow: [Read, Glob, Grep]
```

### 2. Fast Validator
Focus: Quick checks with light model
```yaml
model: claude-haiku-4-5
tools:
  allow: [Read, Grep, Glob]
```

### 3. Complex Analyzer
Focus: Deep analysis with powerful model
```yaml
model: claude-opus-4-6
tools:
  allow: [Read, Grep, Glob, Bash]
```

### 4. Implementation Agent
Focus: Full capability for development
```yaml
model: inherit
tools:
  deny: [WebSearch]  # Stay focused on codebase
```

### 5. Documentation Agent
Focus: Write technical docs efficiently
```yaml
model: claude-haiku-4-5
tools:
  allow: [Read, Write, Edit, Glob, Grep]
```

### 6. Security Specialist
Focus: Security-focused analysis
```yaml
model: claude-opus-4-6
tools:
  allow: [Read, Grep, Glob, Bash]
```

### 7. Test Coordinator
Focus: Testing and verification
```yaml
model: inherit
tools:
  allow: [Read, Bash, Glob, Grep]
```

### 8. Database Optimizer
Focus: Query analysis without modifications
```yaml
model: claude-haiku-4-5
tools:
  allow: [Read, Grep, Glob, Bash]
```

### 9. Error Investigator
Focus: Debug production issues
```yaml
model: claude-opus-4-6
tools:
  allow: [Read, Glob, Grep, Bash]
```

### 10. Lead Oversight
Focus: Full access for strategic decisions
```yaml
model: inherit
# No tool restrictions
```

---

## File Location & Naming

```
.claude/agents/
├── code-reviewer.md          # Existing project agent
├── security-auditor.md       # Existing project agent
├── my-validator.md           # Your new agent
├── database-optimizer.md     # Another new agent
└── documentation-writer.md   # Another new agent
```

**Naming convention**: `{role}-{specialty}.md`

| Pattern | Example |
|---------|---------|
| `{role}.md` | `reviewer.md` |
| `{role}-{specialty}.md` | `code-reviewer.md` |
| `{domain}-{specialist}.md` | `database-optimizer.md` |

---

## Tool Permissions Quick Reference

### Read-Only Analysis (Validators/Reviewers)
```yaml
tools:
  allow:
    - Read
    - Glob
    - Grep
```

### Analysis + Testing
```yaml
tools:
  allow:
    - Read
    - Glob
    - Grep
    - Bash
```

### Full Implementation
```yaml
tools:
  deny:
    - WebSearch  # Optional: keep focused on codebase
```

### No Restrictions
```yaml
# Omit tools section entirely for full access
```

---

## Testing Your Subagent

**Checklist:**

- [ ] File is in `.claude/agents/` directory
- [ ] File ends with `.md` extension
- [ ] YAML frontmatter is valid (check syntax)
- [ ] Name and description fields are present
- [ ] Agent appears in `/list-agents`
- [ ] `/invoke agent-name` successfully loads agent
- [ ] Agent can read files (if Read tool included)
- [ ] Output matches expected format

**Debug command:**
```bash
# Check if file is readable
cat /Users/arunkumar/Documents/Application/expenseSettle/.claude/agents/my-agent.md | head -20

# Verify YAML syntax
grep "^---" /Users/arunkumar/Documents/Application/expenseSettle/.claude/agents/my-agent.md
```

---

## Real-World Examples

### Example 1: Quick Code Validator

```markdown
---
name: quick-validator
description: Fast code validation for common issues
model: claude-haiku-4-5
color: green
tools:
  allow:
    - Read
    - Grep
    - Glob
---

name: quick-validator
description: |
  Fast validator focusing on common code quality issues.

## What I Check
- Missing error handling
- Unvalidated user input
- Code duplication
- Missing tests

## My Approach
1. Scan for patterns that indicate issues
2. Find specific file locations
3. Suggest concrete fixes

instructions: |
  You are a fast code validator. Focus on high-impact issues.
  Report file:line with specific problems and solutions.
```

### Example 2: Performance Investigator

```markdown
---
name: performance-investigator
description: Investigates performance bottlenecks
model: claude-opus-4-6
color: purple
tools:
  allow:
    - Read
    - Grep
    - Glob
    - Bash
---

name: performance-investigator
description: |
  Investigates performance issues and bottlenecks.

## Investigation Process
1. Identify slow operations
2. Analyze database queries
3. Check for N+1 problems
4. Suggest optimizations

## Output Format
- Issue: [What's slow]
- Location: [File:Line]
- Cause: [Why it's slow]
- Solution: [How to fix]
- Expected improvement: [Before/after metrics]

instructions: |
  You are a performance investigator. When analyzing performance issues,
  provide specific metrics and improvements. Include before/after timing.
```

### Example 3: Security Checker

```markdown
---
name: security-checker
description: Checks code for security vulnerabilities
model: claude-opus-4-6
color: orange
tools:
  allow:
    - Read
    - Grep
    - Glob
---

name: security-checker
description: |
  Security-focused code analysis.

## What I Check
- Input validation missing
- SQL injection risks
- Authorization checks
- Hardcoded credentials
- Insecure data handling

## Severity Levels
- Critical: Immediate risk
- High: Important to fix
- Medium: Should fix soon
- Low: Nice to address

instructions: |
  You are a security checker. Identify vulnerabilities with severity levels.
  Provide specific fixes with code examples.
```

---

## Workflow: Connecting Multiple Agents

Create a multi-agent workflow for feature implementation:

```
User: "Implement user authentication"
    ↓
/invoke tech-lead
"Design user authentication feature"
    ↓
/invoke code-reviewer
"Review the implementation from tech-lead"
    ↓
/invoke security-checker
"Check authentication code for security issues"
    ↓
/invoke qa-tester
"Test the authentication feature"
    ↓
/invoke tech-writer
"Create documentation for authentication feature"
```

Each agent:
- Receives output from previous agent
- Processes based on its specialty
- Hands off to next agent in chain

---

## Tips & Tricks

### Tip 1: Use Fast Model for Read-Only Tasks

```yaml
model: claude-haiku-4-5    # Faster, cheaper
tools:
  allow:
    - Read
    - Grep
    - Glob
```

### Tip 2: Use Powerful Model for Analysis

```yaml
model: claude-opus-4-6     # Slower, better reasoning
tools:
  allow:
    - Read
    - Grep
    - Glob
    - Bash
```

### Tip 3: Create Agent Memory for Patterns

In `.claude/memory/MEMORY.md`:
```markdown
# Common Issues Found
- Missing Form Request validation
- Forgotten eager loading in queries
- No PHPDoc on public methods

# Project Standards
- Use UUIDs for primary keys
- Service layer for business logic
- Factories for testing
```

### Tip 4: Clear Tool Restrictions

Be explicit about what tools agent can/cannot use:

```yaml
# Good: Clear allowlist
tools:
  allow:
    - Read
    - Grep
    - Glob

# Also good: Clear denylist
tools:
  deny:
    - Write
    - Bash
```

### Tip 5: Specific Descriptions

Descriptions help Claude pick the right agent:

```yaml
# Bad
description: Code helper agent

# Good
description: Expert Laravel code reviewer focusing on security and performance
```

---

## Common Issues & Solutions

| Problem | Cause | Solution |
|---------|-------|----------|
| Agent doesn't appear in list | File not in `.claude/agents/` | Move file to correct directory |
| YAML error | Invalid syntax | Check frontmatter closes with `---` |
| Can't read files | Read tool not in allowlist | Add Read to tools.allow |
| Agent too slow | Using powerful model unnecessarily | Use `claude-haiku-4-5` for simple tasks |
| Wrong tool access | Tool not permitted | Check tools.allow or tools.deny |

---

## Next Steps

1. **Read full guide**: Check `CUSTOM_SUBAGENTS_GUIDE.md` for comprehensive documentation
2. **Create your first agent**: Use one of the templates above
3. **Test thoroughly**: Use checklist from "Testing Your Subagent" section
4. **Build workflows**: Connect multiple agents for complex tasks
5. **Iterate**: Improve agents based on results

---

## Resources

- **Full Guide**: `/Users/arunkumar/Documents/Application/expenseSettle/CUSTOM_SUBAGENTS_GUIDE.md`
- **Existing Agents**: `/Users/arunkumar/Documents/Application/expenseSettle/.claude/agents/`
- **Project Memory**: `/Users/arunkumar/Documents/Application/expenseSettle/.claude/memory/MEMORY.md`
- **Agent README**: `/Users/arunkumar/Documents/Application/expenseSettle/.claude/README.md`

---

## Summary Cheat Sheet

```markdown
# Create Agent File
Location: .claude/agents/{name}.md
Content:
  - YAML frontmatter (name, description, model, color, tools)
  - Markdown description (role, responsibilities, approach)
  - Instructions (how agent should behave)

# Key Fields
- name: agent-name (kebab-case)
- description: One-line summary
- model: inherit (or specific model)
- color: blue (for UI)
- tools.allow or tools.deny: [List of tools]

# Common Tool Combinations
- Read-only: [Read, Grep, Glob]
- Analysis: [Read, Grep, Glob, Bash]
- Implementation: All tools
- Leading: All tools (no restrictions)

# Test
/list-agents          # See your agent
/invoke agent-name    # Invoke it
                      # Ask it to do something
```

