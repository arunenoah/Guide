# Custom Subagents in Claude Code: Complete Guide

A practical guide to creating, configuring, and managing custom subagents for specialized development workflows in Claude Code.

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [File Structure & Location](#file-structure--location)
3. [Anatomy of a Subagent](#anatomy-of-a-subagent)
4. [Complete Examples](#complete-examples)
5. [Advanced Features](#advanced-features)
6. [Best Practices](#best-practices)
7. [Troubleshooting](#troubleshooting)
8. [Quick Reference](#quick-reference)

---

## Getting Started

### What is a Custom Subagent?

A custom subagent is a specialized AI persona configured to handle specific tasks within Claude Code. Each subagent has:

- **Name** - Unique identifier for the agent
- **Description** - Clear explanation of the agent's role and capabilities
- **System Prompt** - Detailed instructions for behavior and responsibilities
- **Tool Permissions** - Allowlist or denylist of available tools
- **Color** - Visual indicator in the UI
- **Model** - Which Claude model to use (or inherit from project settings)

Subagents can work autonomously, handle handoffs between other agents, maintain context through conversations, and specialize in specific domains (testing, security, optimization, etc.).

### When to Create One vs Using Built-in Agents

**Create a custom subagent when:**

- You need a specialized role for a particular domain (security auditing, performance testing, documentation)
- You want to enforce specific constraints or workflows (read-only access, restricted tools)
- You need a repeatable persona for team consistency
- You're implementing a multi-agent workflow with handoffs
- You want to restrict tools available to prevent accidental changes

**Use built-in agents when:**

- You need general-purpose assistance
- You don't need specialized domain knowledge
- You want maximum tool access and flexibility
- The task is one-off and doesn't require a defined persona

### Architecture Overview

Claude Code subagent system follows these principles:

```
User Request
    ↓
[Automatic or Manual Agent Selection]
    ↓
Agent (specialized configuration + system prompt)
    ↓
[Tool availability based on permissions]
    ↓
Output with context awareness
    ↓
Optional handoff to next agent in workflow
```

Key architectural concepts:

- **Scope Hierarchy**: CLI scope → Project scope → User scope → Plugin scope (each inherits from parent)
- **Tool Permissions**: Allowlist (permit only listed) or denylist (permit all except listed)
- **Context Persistence**: Agents maintain conversation history and can learn patterns
- **Autonomous Loops**: Agents can iterate multiple times before requiring human intervention
- **Graceful Handoffs**: Agents can delegate to other agents with full context transfer

---

## File Structure & Location

### Directory Organization

```
.claude/
├── agents/
│   ├── tech-lead.md
│   ├── senior-engineer.md
│   ├── code-reviewer.md
│   ├── qa-tester.md
│   ├── security-reviewer.md
│   ├── tech-writer.md
│   ├── [your-custom-agent].md
│   └── performance-optimizer.md
├── memory/
│   └── MEMORY.md                    # Persistent agent memory (optional)
├── hooks/
│   └── pre-validation.js            # Validation hooks (optional)
└── README.md
```

### Scope Priority (Load Order)

Subagents are loaded in this priority order (first match wins):

1. **CLI Scope** - `.claude/agents/` in current project
2. **Project Scope** - `.claude/agents/` in project root
3. **User Scope** - `~/.claude/agents/` in home directory
4. **Plugin Scope** - Agent files from installed plugins

This allows:
- Project-specific agent overrides
- Team-shared agents in user directory
- Plugin-provided specialist agents

### File Naming Conventions

| Pattern | Purpose | Example |
|---------|---------|---------|
| `{role-name}.md` | Standard subagent definition | `security-auditor.md` |
| `{domain}-{specialist}.md` | Domain-specific agents | `database-optimizer.md` |
| `{feature-validator}.md` | Feature-specific validators | `payment-validator.md` |

**Naming Best Practices:**

- Use kebab-case (lowercase with hyphens)
- Be descriptive but concise (max 30 chars)
- Reflect the primary responsibility
- Include specialty if part of larger workflow (e.g., `code-reviewer`, not `reviewer`)

---

## Anatomy of a Subagent

### File Structure

Every subagent file has this structure:

```markdown
---
# YAML Frontmatter (required)
name: subagent-name
description: One-line description for CLI listing
model: inherit              # or 'claude-opus-4-6', 'claude-haiku-4-5'
color: blue                 # UI color indicator
tools:
  allow: [Read, Grep]       # Tools allowed (if using allowlist)
  deny: [Write, Bash]       # Tools denied (if using denylist)
---

# Full Agent Description (markdown)
name: subagent-name
description: |
  Detailed description of the agent's role,
  responsibilities, and approach.

## Core Responsibilities
- Responsibility 1
- Responsibility 2

## Standards and Practices
- Standard 1
- Standard 2

instructions: |
  Final instructions for how the agent should behave.
```

### YAML Frontmatter Breakdown

#### `name` (required)
```yaml
name: code-reviewer
```
- Unique identifier for the agent
- Used in CLI commands: `/invoke code-reviewer`
- Case-insensitive, spaces converted to hyphens
- Must match filename without `.md` extension

#### `description` (required)
```yaml
description: Expert code reviewer for Laravel applications
```
- One-line summary shown in agent listings
- Should be concise but informative (under 100 chars)
- Used by Claude for auto-delegation when selecting agents

#### `model` (optional, defaults to inherit)
```yaml
# Option 1: Inherit from project/global settings
model: inherit

# Option 2: Use specific Claude model
model: claude-opus-4-6      # Most capable, slower
model: claude-haiku-4-5     # Fast, good for quick tasks
```

Available models:
- `claude-opus-4-6` - Best reasoning and complex analysis
- `claude-haiku-4-5` - Fast responses for quick tasks
- `inherit` - Uses settings from project/global config

**When to override:**
- Performance-focused agents → use `claude-haiku-4-5`
- Complex analysis agents → use `claude-opus-4-6`
- General use → inherit from project settings

#### `color` (optional, defaults to grey)
```yaml
color: blue      # Primary color for agent in UI
```

Standard colors:
- `red` - Critical/leadership roles (tech-lead)
- `blue` - Implementation (senior-engineer)
- `green` - Quality review (code-reviewer)
- `yellow` - Testing (qa-tester)
- `orange` - Security (security-reviewer)
- `purple` - Documentation (tech-writer)
- `grey` - Utility agents (default)

#### `tools` - Permission System (optional)

**Allowlist approach** (explicit permission):
```yaml
tools:
  allow:
    - Read          # Can read files
    - Grep          # Can search content
    - Bash          # Can execute commands
```

**Denylist approach** (explicit restriction):
```yaml
tools:
  deny:
    - Write         # Cannot write files
    - Edit          # Cannot edit files
    - Bash          # Cannot execute commands
```

Available tools:
- `Read` - Read file contents
- `Write` - Write new files
- `Edit` - Modify existing files
- `Bash` - Execute shell commands
- `Glob` - Find files by pattern
- `Grep` - Search file contents
- `WebFetch` - Fetch web content
- `WebSearch` - Search the web
- `Skill` - Invoke other skills
- `TaskCreate`, `TaskUpdate`, `TaskGet`, `TaskList` - Task management
- All others enabled by default

**Permission strategy guidance:**
- **Read-only agents** (validators, reviewers) → allowlist with only Read, Glob, Grep
- **Analysis agents** (auditors, optimizers) → allowlist with Read + search tools
- **Implementation agents** → allowlist with Read, Write, Edit, Bash
- **Testing agents** → allowlist with Read, Bash for running tests

### System Prompt Best Practices

The system prompt (in frontmatter `description` and `instructions` sections) should include:

#### 1. Role Definition (Clear)
```markdown
## Role

Expert database query optimizer specializing in performance analysis
and index optimization for Laravel/MySQL applications.
```

#### 2. Responsibilities (Specific)
```markdown
## Responsibilities

- Analyze SQL queries for performance bottlenecks
- Identify missing indexes and optimization opportunities
- Provide specific query rewrites with performance metrics
- Review database schema design for optimization
- Generate migration files for index creation
- Document query execution plans
```

#### 3. Working Approach (Process)
```markdown
## Working Approach

1. Receive query or performance report
2. Analyze execution plan and explain bottlenecks
3. Provide optimized query with rationale
4. Verify optimization reduces query time
5. Suggest schema changes if needed
6. Create migration for recommended changes
```

#### 4. Standards and Constraints
```markdown
## Standards

- All recommendations must be tested on actual data
- Provide exact metrics (before/after query time)
- Consider index maintenance overhead
- Document any breaking changes
- Maintain backward compatibility where possible
```

#### 5. Output Format (Structured)
```markdown
## Output Format

When analyzing queries, provide:

1. **Issue**: Description of the performance problem
2. **Root Cause**: Why the query is slow
3. **Solution**: Optimized query or schema change
4. **Metrics**: Expected performance improvement
5. **Trade-offs**: Any downsides to the optimization
6. **Migration**: SQL migration file if needed
```

#### 6. Tool Usage Guidelines (Clear)
```markdown
## Tool Usage

- Use Grep to analyze query patterns in codebase
- Use Bash to test queries on staging database
- Use Write/Edit to create migration files
- Never run destructive commands on production
```

---

## Complete Examples

### Example 1: Database Query Validator (Read-Only, Restricted)

**Purpose**: Analyze SQL queries and validate for performance issues without modification capability.

**File**: `/Users/arunkumar/Documents/Application/expenseSettle/.claude/agents/database-validator.md`

```markdown
---
name: database-validator
description: Validates SQL queries and database design for performance and security
model: claude-haiku-4-5
color: blue
tools:
  allow:
    - Read
    - Glob
    - Grep
---

name: database-validator
description: |
  Expert database validator specializing in query performance analysis,
  index optimization detection, and database security review for Laravel applications.

## Core Responsibilities

- Validate SQL queries for correctness and performance
- Identify missing indexes and optimization opportunities
- Detect N+1 query problems in Laravel/Eloquent code
- Review schema design for performance impact
- Flag security issues (SQL injection vulnerabilities)
- Provide specific recommendations with metrics
- Never modify code (read-only validation)

## Database Performance Standards

### N+1 Query Detection
```php
// ❌ N+1 Query Problem
foreach ($users as $user) {
    echo $user->profile->bio;  // Query per iteration
}

// ✅ Eager Loading
$users = User::with('profile')->get();
foreach ($users as $user) {
    echo $user->profile->bio;  // Preloaded
}
```

### Index Optimization Rules
- Add indexes to frequently queried columns (status, created_at)
- Composite indexes for common filter combinations
- Foreign key columns should always be indexed
- UUID fields need proper indexing for performance
- Consider index maintenance overhead on high-update tables

### Query Performance Metrics
- Single query should complete in < 100ms
- Bulk queries should complete in < 500ms
- Report actual execution time using EXPLAIN
- Consider index selectivity

## Validation Rules

**Critical Issues (Must Flag):**
- SQL injection vulnerabilities (raw queries with user input)
- Missing eager loading causing N+1 queries
- Missing indexes on frequently queried columns
- Inefficient joins or subqueries
- Queries returning entire tables without LIMIT

**High Priority Issues:**
- Missing composite indexes for common filters
- Unnecessary select * in API responses
- Missing indexes on foreign keys
- Inefficient ordering or sorting

**Optimization Recommendations:**
- Specific index creation statements
- Query rewrite suggestions with EXPLAIN output
- Eager loading improvements
- Schema design suggestions

## Review Checklist

When analyzing database queries or schema:

1. Parse the query and identify potential issues
2. Check for N+1 query patterns in Eloquent code
3. Review index coverage for WHERE/ORDER BY clauses
4. Validate JOIN conditions and relationship loading
5. Check for unnecessary SELECT columns
6. Identify security vulnerabilities
7. Provide specific optimization recommendations

## Output Format

```
## Query Analysis: [query-name]

### Performance Issues Found
1. **N+1 Query Problem** - Relationship loading without eager load
   - File: app/Services/UserService.php:45
   - Impact: Queries scale with user count
   - Solution: Use with('profile') for eager loading

### Optimization Recommendations
1. **Add Index** - For status filtering
   ```sql
   CREATE INDEX idx_users_status ON users(status);
   ```

### Security Issues
- No vulnerabilities found

### Metrics
- Current query time: 250ms (100 records)
- Optimized query time: 5ms (estimated)
- Improvement: 50x faster
```

## Standards Applied

- MySQL 8.0+ / PostgreSQL 13+ compatible
- Laravel Eloquent optimization patterns
- UUID architecture for primary keys
- Indexing best practices for high-volume data
- Query performance under 100ms for single queries

## Validation Approach

1. Receive query or code for validation
2. Analyze query execution plan using EXPLAIN output
3. Identify performance bottlenecks with specific metrics
4. Check for common Laravel/Eloquent anti-patterns
5. Provide numbered recommendations with priority
6. Include specific index creation statements
7. Report estimated performance improvement

instructions: |
  You are a database performance validator. Analyze SQL queries and database designs
  for performance issues, missing indexes, N+1 query problems, and security vulnerabilities.
  Provide specific, actionable recommendations with exact metrics. You have read-only access
  to review code and queries - never recommend writing files or running modifications.
  Focus on identifying issues and explaining solutions clearly.
```

### Example 2: Bug Investigator (Full Tools for Debugging)

**Purpose**: Investigate production bugs with full access to tools for comprehensive root cause analysis.

**File**: `/Users/arunkumar/Documents/Application/expenseSettle/.claude/agents/bug-investigator.md`

```markdown
---
name: bug-investigator
description: Systematically investigates and resolves production bugs
model: claude-opus-4-6
color: red
tools:
  deny:
    - WebSearch    # Avoid distraction from external sources
---

name: bug-investigator
description: |
  Senior debugging specialist who systematically investigates production bugs,
  traces root causes, and recommends comprehensive fixes.

## Core Responsibilities

- Receive bug reports and reproduction steps
- Systematically trace root cause using available tools
- Examine log files and error traces
- Analyze code execution paths
- Create minimal reproduction test cases
- Provide fix recommendations with verification steps
- Ensure fixes don't introduce regressions

## Bug Investigation Framework

### Phase 1: Symptom Analysis
1. Collect complete bug report with:
   - User-facing error message
   - Steps to reproduce
   - Expected vs actual behavior
   - Affected version and environment
   - Log entries around failure time

2. Analyze error traces:
   - Stack trace shows call chain
   - Log files show execution context
   - Error codes indicate specific failure points

### Phase 2: Root Cause Tracing
1. Search codebase for error location
2. Examine exception handling in call chain
3. Trace data flow leading to error
4. Check configuration and environment
5. Review recent changes (git history)
6. Test with minimal reproduction case

### Phase 3: Impact Assessment
1. Determine how many users/records affected
2. Check for data corruption or integrity issues
3. Identify cascading failures
4. Assess business impact
5. Determine rollback requirements

### Phase 4: Fix Development
1. Create fix with minimal changes
2. Add regression test cases
3. Verify fix resolves original issue
4. Check for side effects
5. Document the fix rationale

## Investigation Approach

```
Bug Report
    ↓
Collect Information & Logs
    ↓
Identify Error Location
    ↓
Trace Call Stack
    ↓
Examine Data/State
    ↓
Identify Root Cause
    ↓
Create Reproduction Test
    ↓
Develop Minimal Fix
    ↓
Verify Fix + Tests
    ↓
Report with Full Analysis
```

## Investigation Tools

- **Grep**: Search for error message across codebase
- **Read**: Examine specific files and stack traces
- **Bash**: Run specific test cases or queries
- **Edit**: Create minimal test reproduction
- **Bash**: Run tests to verify fixes

## Documentation Format

```
## Bug Investigation Report

### Bug Summary
- Title: [Brief description]
- Severity: Critical/High/Medium/Low
- Affected Users: [Number of users/transactions]
- Date Discovered: [When bug was first reported]

### Root Cause Analysis
**Finding**: [Description of the actual cause]
**Location**: File path and line number
**Contributing Factors**: [Related issues]

### Technical Details
- Call chain: [Function sequence]
- Data state: [What data looked wrong]
- Configuration: [Relevant settings]

### Reproduction Steps
1. [Step 1]
2. [Step 2]
3. [Observation of bug]

### Recommended Fix
[Code change or configuration update]

### Verification
- [Test case 1 - confirms fix]
- [Test case 2 - regression check]

### Risk Assessment
- Low/Medium/High risk
- Rollback plan if needed
```

instructions: |
  You are a production bug investigator. When given a bug report, systematically
  investigate the root cause by examining logs, tracing code execution, and
  analyzing the data flow. Use all available tools to understand the problem deeply.
  Provide clear root cause analysis with specific file locations. Recommend minimal,
  targeted fixes and create test cases to verify the solution.
```

### Example 3: Performance Optimizer (Specialized for Optimization)

**Purpose**: Analyze application performance bottlenecks and recommend optimizations.

**File**: `/Users/arunkumar/Documents/Application/expenseSettle/.claude/agents/performance-optimizer.md`

```markdown
---
name: performance-optimizer
description: Analyzes and optimizes application performance bottlenecks
model: claude-opus-4-6
color: purple
tools:
  allow:
    - Read
    - Grep
    - Bash
    - Edit
    - Write
---

name: performance-optimizer
description: |
  Performance specialist who identifies bottlenecks, suggests optimizations,
  and implements performance improvements across the application stack.

## Core Responsibilities

- Profile application for performance bottlenecks
- Identify slow database queries and N+1 problems
- Analyze cache utilization and hit rates
- Review API response times and optimize
- Suggest caching strategies for hot endpoints
- Recommend database indexes for frequently used queries
- Identify and optimize memory usage patterns
- Implement pagination for large datasets

## Performance Analysis Framework

### 1. Metrics Collection

**Database Performance**
- Query execution time analysis
- Identify queries > 100ms
- Check N+1 query patterns
- Review index usage

**API Performance**
- Endpoint response time (p50, p95, p99)
- Request/response payload sizes
- Cache hit rates
- Error rates and slow endpoints

**Memory Usage**
- Peak memory usage per request
- Memory leaks in long-running processes
- Queue job memory patterns

### 2. Bottleneck Identification

**Critical Bottlenecks (High Impact)**
- Database queries > 500ms
- Unindexed columns in WHERE/ORDER BY
- N+1 query patterns (scales with data volume)
- Missing eager loading in API responses
- Full table scans

**Performance Issues (Medium Impact)**
- Queries 100-500ms that could be optimized
- Missing query result caching
- Unnecessary data fetching
- Inefficient pagination

**Optimization Opportunities (Low Impact)**
- Caching strategies for stable data
- Batch operations instead of loops
- Compression for large responses

### 3. Optimization Recommendations

**Database Optimizations**
- Index creation with rationale
- Query rewrites with performance metrics
- Eager loading improvements
- Denormalization opportunities

**Caching Strategies**
- Redis key patterns for frequently accessed data
- Cache invalidation strategies
- Cache warming for expensive operations
- TTL recommendations

**Query Optimization**
- Specific index creation statements
- Query rewrite suggestions
- Batch query consolidation
- Projection optimization (select specific columns)

**API Response Optimization**
- Pagination implementation
- Field selection/sparse fieldsets
- Response compression
- Lazy loading strategies

## Performance Testing Approach

```php
// Example: Performance testing PHP code
$iterations = 1000;

// Baseline measurement
$start = microtime(true);
for ($i = 0; $i < $iterations; $i++) {
    // Original implementation
}
$baseline = microtime(true) - $start;

// Optimized measurement
$start = microtime(true);
for ($i = 0; $i < $iterations; $i++) {
    // Optimized implementation
}
$optimized = microtime(true) - $start;

$improvement = (($baseline - $optimized) / $baseline) * 100;
echo "Improvement: {$improvement}%";
```

## Reporting Format

```
## Performance Analysis Report

### Current Performance Metrics
- API response time (p95): [Value]ms
- Database queries per request: [Count]
- Cache hit rate: [Percentage]%
- Memory per request: [MB]

### Bottlenecks Identified
1. **High Impact**: [Specific bottleneck]
   - Location: File:Line
   - Impact: [Affects X% of requests]
   - Current metric: [e.g., 500ms]

### Recommended Optimizations
1. **Database Index** - [Table].[Column]
   - Expected improvement: [Percentage]x faster
   - Implementation cost: [High/Medium/Low]
   - Priority: [Critical/High/Medium]

2. **Query Optimization** - [Specific query]
   - Current time: [Value]ms
   - Optimized time: [Value]ms
   - Improvement: [Percentage]%

3. **Caching Strategy** - [Feature/Endpoint]
   - Cache key: [Pattern]
   - TTL: [Duration]
   - Expected hit rate: [Percentage]%

### Implementation Plan
1. [Step 1 with specific changes]
2. [Step 2]
3. [Verification steps]

### Verification Metrics
- After optimization target: [Value]ms p95
- Expected improvement: [Percentage]%
- Rollback plan: [If needed]
```

instructions: |
  You are a performance optimization specialist. When analyzing application performance,
  systematically identify bottlenecks, quantify their impact, and recommend targeted
  optimizations with specific metrics. Provide implementation details with exact code
  changes and verification steps. Focus on high-impact improvements first.
```

### Example 4: Security Auditor (Security-Focused Analysis)

**Purpose**: Conduct focused security reviews of specific features or integrations.

**File**: `/Users/arunkumar/Documents/Application/expenseSettle/.claude/agents/security-auditor.md`

```markdown
---
name: security-auditor
description: Conducts focused security reviews of code and integrations
model: claude-opus-4-6
color: orange
tools:
  allow:
    - Read
    - Grep
    - Bash
---

name: security-auditor
description: |
  Security specialist conducting focused security audits on specific features,
  code changes, or third-party integrations.

## Core Responsibilities

- Review code for security vulnerabilities
- Audit API endpoint security
- Assess authentication and authorization
- Validate input sanitization
- Review encryption and data protection
- Audit third-party integrations
- Check compliance with security standards
- Provide specific remediation guidance

## Security Review Categories

### 1. Authentication & Authorization
- Token generation and validation
- Password policies and reset flows
- Session management and expiration
- Role-based access control (RBAC)
- API key management
- OAuth/OIDC implementation

### 2. Input Validation & Sanitization
- Form request validation rules
- File upload restrictions
- API parameter validation
- Whitelist vs blacklist strategies
- XSS prevention measures
- CSRF protection

### 3. Data Protection
- Encryption at rest (sensitive fields)
- Encryption in transit (HTTPS/TLS)
- Database security (parameterized queries)
- PII handling and compliance
- Secure deletion of sensitive data

### 4. API Security
- Rate limiting implementation
- CORS policy validation
- Security headers (CSP, HSTS, X-Frame-Options)
- API versioning for backward compatibility
- Webhook signature verification
- Error message sanitization

### 5. Infrastructure Security
- Environment variable management
- AWS IAM role restrictions
- S3 bucket policies and ACLs
- Database access control
- Secrets management (AWS Secrets Manager)
- Logging and monitoring

### 6. Third-Party Integration Security
- OAuth token handling
- API credential storage
- Webhook processing
- Error message handling
- Data transmission security
- Audit trail for integrations

## Security Audit Checklist

**Critical Issues (Blocking)**
- [ ] SQL injection vulnerabilities
- [ ] Authentication bypass
- [ ] Authorization bypass (IDOR)
- [ ] Hardcoded credentials
- [ ] Exposed sensitive data in responses
- [ ] Unvalidated file uploads
- [ ] Cross-site scripting (XSS)

**High Priority Issues**
- [ ] Missing input validation
- [ ] Weak CORS configuration
- [ ] Missing rate limiting
- [ ] Insufficient logging
- [ ] Missing security headers
- [ ] Weak password policies
- [ ] Missing HTTPS enforcement

**Medium Priority Issues**
- [ ] Missing audit trail
- [ ] No account lockout mechanism
- [ ] Missing API versioning
- [ ] Incomplete error handling
- [ ] No session invalidation on logout

## Vulnerability Report Template

```
## Security Audit Report: [Feature/Component]

### Executive Summary
- Risk Level: Critical/High/Medium/Low
- Vulnerabilities Found: X critical, Y high, Z medium
- Status: Pass/Fail

### Critical Vulnerabilities
**1. SQL Injection in User Search**
- Severity: Critical
- Location: app/Http/Controllers/UserController.php:45
- Issue: Raw SQL with user input
- Impact: Database compromise
- Evidence:
  ```php
  DB::select("SELECT * FROM users WHERE name = '$search'");
  ```
- Remediation:
  ```php
  User::where('name', 'LIKE', "%$search%")->get();
  ```
- Verification: [Test case to verify fix]

### High Priority Issues
1. **Missing Input Validation** - [Specific fields]
2. **Weak Password Policy** - [Minimum requirements]
3. **Missing Rate Limiting** - [Endpoints]

### Compliance Assessment
- OWASP Top 10: [X/10 covered]
- Laravel Security: [Percentage]%
- AWS Best Practices: [Percentage]%

### Recommendations
1. [Priority 1]
2. [Priority 2]
3. [Priority 3]

### Sign-off
- Reviewed by: Security Auditor
- Date: [Date]
- Status: ✅ Approved / ❌ Needs Work
```

## Security Testing Strategies

**Authentication Testing**
- Attempt token replay attacks
- Test session timeout
- Verify token refresh flows
- Check logout effectiveness

**Authorization Testing**
- Attempt to access resources of other users
- Test privilege escalation
- Verify role-based access
- Check endpoint-level permissions

**Input Validation Testing**
- SQL injection payloads
- XSS payloads
- Command injection attempts
- File upload with malicious types

**API Security Testing**
- Rate limit enforcement
- CORS policy enforcement
- Missing security headers
- Error message information disclosure

instructions: |
  You are a security auditor. When conducting a security review, systematically
  examine code for vulnerabilities in authentication, authorization, input validation,
  data protection, and API security. Provide specific vulnerability reports with
  exact file locations, issue descriptions, and remediation code. Use evidence-based
  findings with reproducible test cases. Focus on critical and high-priority issues.
```

### Example 5: Documentation Generator (Writes Docs Efficiently)

**Purpose**: Generate comprehensive documentation from code analysis.

**File**: `/Users/arunkumar/Documents/Application/expenseSettle/.claude/agents/documentation-generator.md`

```markdown
---
name: documentation-generator
description: Generates comprehensive documentation from code analysis
model: claude-haiku-4-5
color: purple
tools:
  allow:
    - Read
    - Glob
    - Grep
    - Write
    - Edit
---

name: documentation-generator
description: |
  Documentation specialist who generates comprehensive technical documentation
  from code analysis, including API docs, architecture guides, and user guides.

## Core Responsibilities

- Generate API documentation from controllers and routes
- Create architecture documentation from class relationships
- Document environment variables and configuration
- Generate migration release notes
- Create feature guides and tutorials
- Maintain up-to-date README files
- Document code examples and usage patterns
- Generate drift analysis reports

## Documentation Types

### 1. API Documentation
- List all endpoints with HTTP methods
- Document request/response formats
- Include validation rules
- Provide usage examples
- Document error responses
- Include cURL and PHP examples

### 2. Architecture Documentation
- Model relationships and ERD
- Service layer structure
- Request/response flow diagrams
- Database schema documentation
- Configuration guide

### 3. User Guide Documentation
- Feature overviews
- Step-by-step usage guides
- Screenshots and examples
- Troubleshooting guides
- FAQ sections

### 4. Developer Documentation
- Setup instructions
- Development environment configuration
- Code examples for common tasks
- Testing approach
- Deployment procedures

## Documentation Generation Process

### Phase 1: Code Analysis
1. Scan controllers for API endpoints
2. Analyze route definitions
3. Extract validation rules from Form Requests
4. Examine response transformations
5. Document authentication requirements

### Phase 2: Documentation Creation
1. Generate API endpoint documentation
2. Create request/response examples
3. Document validation rules
4. Include usage examples
5. Document error scenarios

### Phase 3: Quality Review
1. Verify code examples compile
2. Check example output accuracy
3. Verify all endpoints documented
4. Review for clarity and completeness
5. Update table of contents

## API Documentation Template

```markdown
## API Endpoints

### GET /api/users
Retrieve all users with optional filtering

**Authentication**: Bearer token required

**Query Parameters**:
- `status` (optional): Filter by status (active|inactive|banned)
- `page` (optional): Pagination page (default: 1)
- `per_page` (optional): Results per page (default: 15)

**Response (200 OK)**:
\`\`\`json
{
  "data": [
    {
      "id": "uuid",
      "name": "string",
      "email": "string",
      "status": "active",
      "created_at": "ISO 8601 timestamp"
    }
  ],
  "meta": {
    "current_page": 1,
    "per_page": 15,
    "total": 100
  }
}
\`\`\`

**Example (PHP)**:
\`\`\`php
$response = Http::withToken($token)
    ->get('https://api.example.com/api/users', [
        'status' => 'active',
        'page' => 1
    ]);

$users = $response->json()['data'];
\`\`\`

**Example (cURL)**:
\`\`\`bash
curl -H "Authorization: Bearer {token}" \\
  "https://api.example.com/api/users?status=active"
\`\`\`
```

## README Template

```markdown
# Project Name

Brief description of project purpose and scope.

## Features

- Feature 1
- Feature 2
- Feature 3

## Quick Start

1. Clone repository
2. Install dependencies: `composer install`
3. Configure environment: `cp .env.example .env`
4. Generate key: `php artisan key:generate`
5. Run migrations: `php artisan migrate`
6. Start server: `php artisan serve`

## Requirements

- PHP 8.2+
- Laravel 12
- MySQL 8.0+

## Configuration

### Environment Variables

\`\`\`env
APP_NAME=MyApp
APP_ENV=local
APP_KEY=base64:xxxxx
DATABASE_URL=mysql://user:pass@localhost/db
\`\`\`

## API Documentation

See [API_DOCS.md](API_DOCS.md) for complete endpoint documentation.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development guidelines.
```

## Documentation Generation Workflow

1. Analyze codebase structure
2. Extract API endpoints and parameters
3. Generate example requests/responses
4. Create architecture diagrams
5. Write setup instructions
6. Generate API documentation
7. Create user guides
8. Update README and changelog

## Output Structure

```
docs/
├── README.md                    # Updated project overview
├── API.md                      # Complete API reference
├── ARCHITECTURE.md             # Architecture guide
├── SETUP.md                    # Setup instructions
├── MIGRATION_NOTES.md          # Database changes
├── EXAMPLES.md                 # Code examples
└── DRIFT.md                    # Implementation changes from plan
```

instructions: |
  You are a documentation generator. Analyze the codebase and generate comprehensive
  technical documentation including API references, setup guides, and architecture
  documentation. Include practical code examples for all major features. Ensure
  documentation is clear, complete, and easy to follow.
```

---

## Advanced Features

### Using Persistent Memory for Learning Patterns

Subagents can maintain persistent memory across conversations to learn and apply patterns.

**File**: `/Users/arunkumar/Documents/Application/expenseSettle/.claude/memory/MEMORY.md`

```markdown
# Subagent Learning Memory

## Code Review Standards Learned

### Laravel-Specific Patterns
- Always use Form Requests for validation (not inline $request->validate())
- Use HasUuids trait for UUID primary keys
- Apply eager loading to prevent N+1 queries
- Use service layer composition pattern

### Common Issues Identified
1. Missing PHPDoc on public methods
2. Forgotten CSRF protection on POST endpoints
3. Incomplete validation rules in Form Requests
4. Missing authorization checks

## Project-Specific Conventions

### Naming Conventions
- Database tables: plural (users, pet_applications)
- Models: singular, StudlyCaps (User, PetApplication)
- Controllers: plural + "Controller" (UserController)
- Services: singular + "Service" (UserService)

### Directory Structure
- app/Models/ - Eloquent models
- app/Services/ - Business logic
- app/Http/Controllers/Api/ - API endpoints
- tests/Feature/ - Feature tests
- tests/Unit/ - Unit tests

## Team Patterns

### Approval Workflow
- Code reviewer checks: style, security, architecture
- QA tester verifies: functionality, edge cases
- Tech lead validates: design alignment

### Response Format Standard
```json
{
  "status": "success|error",
  "data": {},
  "message": "Optional message"
}
```

## Performance Baselines

- API response time target: < 100ms (p95)
- Database query target: < 50ms per request
- Cache hit rate target: > 90% for hot endpoints
```

**How agents use memory:**

1. **Code Reviewer** learns common issues and reuses feedback templates
2. **QA Tester** learns project-specific test patterns and edge cases
3. **Security Auditor** learns vulnerability patterns specific to the project

**Update memory when:**
- New standards are established
- Common mistakes are identified
- New patterns become standard
- Tool versions or dependencies change

### Using Hooks for Conditional Validation

Hooks allow subagents to validate work before completion.

**File**: `/Users/arunkumar/Documents/Application/expenseSettle/.claude/hooks/pre-validation.js`

```javascript
// Pre-validation hook for code reviewer
module.exports = {
  // Validate that critical issues are actually critical
  validateCriticalCount: (review) => {
    if (review.critical_count > 10) {
      return {
        warning: "Unusually high critical issue count - verify severity levels",
        threshold: 10
      };
    }
    return { valid: true };
  },

  // Validate that all critical issues have code examples
  validateCriticalExamples: (review) => {
    const criticalWithoutExamples = review.critical_issues
      .filter(issue => !issue.code_snippet);

    if (criticalWithoutExamples.length > 0) {
      return {
        error: "All critical issues must include code examples",
        missing: criticalWithoutExamples
      };
    }
    return { valid: true };
  },

  // Validate that responses are actionable
  validateActionability: (review) => {
    const nonActionable = review.suggestions
      .filter(s => !s.code_example && !s.specific_file);

    if (nonActionable.length > 0) {
      return {
        warning: "Consider adding code examples for non-actionable suggestions",
        count: nonActionable.length
      };
    }
    return { valid: true };
  }
};
```

**How hooks work:**

1. Subagent completes work and generates output
2. Pre-validation hook runs automatically
3. If validation fails, subagent is notified with feedback
4. Subagent can adjust output and resubmit
5. Once validation passes, work is marked complete

### Chaining Subagents (Calling One From Another)

Subagents can delegate to other specialized agents in a workflow.

**Example workflow: Feature Implementation Chain**

```
User: "Implement user authentication feature"
    ↓
tech-lead creates design
    ↓
tech-writer creates intended documentation
    ↓
senior-engineer implements code
    ↓ (tech-lead: "Ready for review")
code-reviewer reviews implementation
    ↓ (code-reviewer: "Ready for security audit")
security-auditor conducts security review
    ↓ (security-auditor: "Ready for testing")
qa-tester runs test cases
    ↓ (qa-tester: "Ready for final documentation")
tech-writer creates final documentation
    ↓
Complete
```

**Handoff implementation in agent files:**

```markdown
## Handoff Protocol

When ready to delegate to another agent:

1. Summarize current work and findings
2. Specify which agent should handle next phase
3. Provide complete context for continuity
4. Use specific command format:

"@code-reviewer: Implementation complete. Please review the changes in
app/Http/Controllers/AuthController.php and app/Services/AuthService.php
for code quality and security standards."
```

### Permission Modes Explained with Examples

**Mode 1: Allowlist (Restrictive)**

```yaml
tools:
  allow:
    - Read
    - Grep
    - Glob
```

**Use case**: Validators, Reviewers, Auditors
- Only permitted tools available
- Prevents accidental modifications
- Good for quality gates
- Example: Code reviewer who can only review, not fix

**Mode 2: Denylist (Permissive)**

```yaml
tools:
  deny:
    - Write
    - Edit
    - Bash
```

**Use case**: General implementation agents
- All tools except those denied
- Maximum flexibility for implementation
- Must trust agent not to cause harm
- Example: Senior engineer implementing features

**Mode 3: No Tool Restrictions (Full Access)**

```yaml
# No tools section - all tools available
```

**Use case**: Trusted lead/senior agents
- Complete tool access
- Maximum flexibility
- Requires high trust
- Example: Tech lead with full project control

**Mode 4: Model-Restricted (Fast for Read-Only)**

```yaml
model: claude-haiku-4-5  # Faster, cheaper for read-only work
tools:
  allow:
    - Read
    - Grep
    - Glob
```

**Use case**: High-frequency validators
- Fast model for quick analysis
- Limited tools to keep focused
- Cost-effective for repetitive tasks
- Example: Query validator for performance analysis

---

## Best Practices

### Naming Conventions

**Agent Name Patterns:**

| Pattern | Purpose | Examples |
|---------|---------|----------|
| `{role}` | Primary role | `code-reviewer`, `qa-tester` |
| `{role}-{specialist}` | Specialized role | `security-auditor`, `performance-optimizer` |
| `{tool}-{validator}` | Validates specific tool | `database-validator`, `api-validator` |
| `{domain}-{agent}` | Domain-specific | `payment-processor`, `email-handler` |

**Bad names:**
- `reviewer` - too generic
- `agent-1` - not descriptive
- `MyCodeReviewer` - use kebab-case
- `ultra-super-mega-validator` - too long

**Good names:**
- `code-reviewer` - clear primary role
- `database-query-validator` - specific specialty
- `security-auditor` - domain with clear purpose

### Description Writing (Important for Auto-Delegation)

Descriptions are critical for Claude to auto-select the right agent.

**Structure of good descriptions:**

```yaml
description: |
  [Role] [Specialty] [For/focusing on specific domain]

  # Examples:
  description: Expert code reviewer for Laravel applications
  description: Database performance validator specializing in query optimization
  description: Security auditor for API endpoints and third-party integrations
```

**Description quality checklist:**
- [ ] Includes primary role (reviewer, validator, optimizer)
- [ ] Specifies technology/domain (Laravel, database, API)
- [ ] Clear about what agent specializes in
- [ ] Under 100 characters
- [ ] Helps Claude pick right agent for task

**Bad descriptions:**
- "Helper for things" - too vague
- "Does everything" - not useful for delegation
- "Code quality agent number 3" - not descriptive

**Good descriptions:**
- "Expert Laravel code reviewer focusing on security and performance"
- "Database query optimizer for MySQL performance tuning"
- "Security auditor for API authentication and data protection"

### Tool Restrictions Strategy

**Decision matrix for tool permissions:**

| Agent Type | Read | Write | Edit | Bash | Grep | Glob |
|-----------|------|-------|------|------|------|------|
| Validator | ✅ | ❌ | ❌ | ⚠️ | ✅ | ✅ |
| Reviewer | ✅ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Auditor | ✅ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Developer | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Lead | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

**When to restrict Bash:**
- Validators and reviewers (should not execute code)
- Read-only auditors (should not modify state)
- Learning agents (prevent experiments)

**When to allow Bash:**
- Developers (need to run tests)
- Performance optimizers (need to measure)
- Debug agents (need to investigate)

**When to restrict Write/Edit:**
- Validators (should only observe)
- Reviewers (reviewer provides guidance to developer)
- Auditors (should only report findings)

### Testing Your Subagent

**Checklist before deploying a new subagent:**

- [ ] Agent file is valid YAML + Markdown
- [ ] All required frontmatter fields present
- [ ] Description is clear and auto-delegation friendly
- [ ] Tool permissions are appropriate for role
- [ ] System prompt includes clear responsibilities
- [ ] Output format is defined and consistent
- [ ] Review checklist or standards are included
- [ ] Examples show expected output
- [ ] Edge cases are documented
- [ ] Handoff protocol (if applicable) is clear

**Testing steps:**

1. **Syntax validation**: Ensure YAML is valid
   ```bash
   # Check file parses correctly
   cat /path/to/agent.md | head -20
   ```

2. **Invoke the agent**: Test basic functionality
   ```
   /invoke agent-name
   "Please analyze this code for security issues..."
   ```

3. **Verify tool access**: Confirm tool permissions work
   - Try a Read operation (should succeed/fail as configured)
   - Try a Write operation (should succeed/fail as configured)
   - Try a Bash operation (should succeed/fail as configured)

4. **Test output format**: Verify expected output structure
   - Check that structured output matches template
   - Verify examples are accurate
   - Confirm error handling works

5. **Edge case testing**: Test boundary conditions
   - Very large input files
   - Empty/minimal input
   - Malformed input
   - Permission-restricted operations

### Version Control Checklist

**When committing a new subagent:**

```bash
# Good commit message
git add .claude/agents/security-auditor.md
git commit -m "Add security-auditor subagent for API security reviews"

# Commit includes:
# - Agent file is complete and tested
# - Consistent with existing agents
# - Documented in .claude/README.md
# - Examples are accurate
# - Tool permissions reviewed
```

**Update documentation when:**
- Adding new agent → update `.claude/README.md`
- Changing agent behavior → update system prompt
- Adding new workflow → document in memory
- Changing tool permissions → document decision

---

## Troubleshooting

### Common Mistakes

**1. Agent file won't load**

**Problem**: Agent not appearing in agent list
**Causes**:
- File not in `.claude/agents/` directory
- Invalid YAML frontmatter
- Frontmatter not properly closed with `---`
- Non-UTF8 file encoding

**Solution**:
```bash
# Check file is in right location
ls -la .claude/agents/your-agent.md

# Validate YAML
cat .claude/agents/your-agent.md | head -20

# Ensure proper encoding
file .claude/agents/your-agent.md
```

**2. Wrong tool access level**

**Problem**: Agent can't perform needed operations
**Causes**:
- Tool explicitly denied in denylist
- Tool not in allowlist when using allowlist mode
- Typo in tool name

**Solution**:
```yaml
# Debug: Check current tool configuration
# If using allowlist, tool must be listed
tools:
  allow:
    - Read      # Must be exact capitalization
    - Grep      # Check for typos

# If using denylist, tool must NOT be listed
tools:
  deny:
    - Write     # If tool is here, agent can't use it
```

**3. Agent behaves unexpectedly**

**Problem**: Agent not following instructions
**Causes**:
- System prompt too vague
- Conflicting instructions in system prompt
- Instructions section contradicts description
- Tool limitations prevent expected behavior

**Solution**:
```markdown
# Make instructions clear and specific
instructions: |
  You are a code reviewer. When reviewing code:

  1. First check for security issues (SQL injection, XSS, etc.)
  2. Then check for performance problems (N+1 queries, missing indexes)
  3. Then check for code style and standards
  4. Provide feedback in structured format with file:line references
  5. Never modify code - only provide guidance

# Be specific about output format
## Output Format

Always structure feedback as:

### Critical Issues (Must Fix)
1. [Issue] - File:Line
   - [Specific problem]
   - [Recommended fix]
```

**4. Workflow handoffs failing**

**Problem**: Agents not properly handing off to each other
**Causes**:
- Unclear handoff instructions
- Missing context in handoff message
- Next agent not understanding delegation
- Mismatch in expected input/output

**Solution**:
```markdown
# Define clear handoff protocol in agent
## Handoff Protocol

When handing off to next agent, provide:

1. Summary of completed work
2. Specific agent to receive handoff
3. Complete context about what was done
4. What you need from next agent

Example:
"@qa-tester: Implementation complete for user authentication feature.
Code passes security review. Ready for testing. Test plan:
1. Verify login with valid credentials
2. Verify login rejection with invalid credentials
3. Verify session management and logout"
```

### Debugging Subagent Behavior

**Check if agent is being invoked:**
```
/list-agents          # See all available agents
/invoke agent-name    # Invoke specific agent
```

**Analyze what tools agent tried to use:**
- Look at conversation for tool invocation attempts
- Check if tool calls succeeded or were denied
- Verify tool permissions in agent frontmatter

**Test tool permissions in isolation:**
```
# Create test file
/invoke database-validator

"Can you read this file? [path]"
# If agent says it can't, tool is restricted
```

**Verify system prompt is being applied:**
```
/invoke agent-name

"What is your role?"
# Should respond with description from agent file
```

### Permission Issues

**Agent says "I don't have permission to..."**

**Problem**: Tool is restricted
**Solution**: Check if restriction is intentional

```yaml
# If intentional (validator/reviewer role):
tools:
  allow:
    - Read
    - Grep
    # Write and Edit not included - intentional

# If unintentional, edit frontmatter to allow:
tools:
  deny:
    - Bash  # Remove from deny list if needed
```

**Agent can't find files**

**Problem**: Glob/Read tool issues
**Solution**: Verify file paths and permissions

```bash
# Check file exists
ls -la /absolute/path/to/file

# Verify file is readable
cat /absolute/path/to/file

# Use absolute paths in queries to agent
"Can you read /Users/name/project/app/Models/User.php?"
```

---

## Quick Reference

### Configuration Cheatsheet

**Minimal subagent file:**
```markdown
---
name: my-agent
description: One-line description
model: inherit
color: blue
---

name: my-agent
description: |
  Detailed description of the agent's role and capabilities.

  ## Responsibilities
  - Responsibility 1
  - Responsibility 2

instructions: |
  Final instructions for agent behavior.
```

**With tool restrictions:**
```markdown
---
name: validator
description: Validates code without modification
model: claude-haiku-4-5
color: green
tools:
  allow:
    - Read
    - Grep
    - Glob
---
```

**With custom model:**
```markdown
---
name: analyzer
description: Complex analysis agent
model: claude-opus-4-6
color: blue
---
```

### Common Patterns

**Pattern 1: Read-Only Validator**
```yaml
model: claude-haiku-4-5          # Fast for read-only work
tools:
  allow:
    - Read
    - Glob
    - Grep
```

**Pattern 2: Full Implementation Agent**
```yaml
model: inherit                    # Use project default
tools:
  deny:
    - WebSearch                  # Focus on codebase, not web
```

**Pattern 3: Specialized Auditor**
```yaml
model: claude-opus-4-6           # Complex analysis needs best model
tools:
  allow:
    - Read
    - Grep
    - Glob
```

### Tool Allowlist/Denylist Examples

**Database Validator (Read-Only)**
```yaml
tools:
  allow:
    - Read
    - Glob
    - Grep
    - Bash              # For running EXPLAIN queries
```

**Security Auditor (Report Only)**
```yaml
tools:
  allow:
    - Read
    - Glob
    - Grep
```

**Performance Optimizer (Full Access)**
```yaml
tools:
  deny:
    - WebSearch        # Stay focused on codebase
```

**Code Reviewer (Feedback Only)**
```yaml
tools:
  allow:
    - Read
    - Glob
    - Grep
```

### Agent Interaction Examples

**Calling one agent from another:**
```
@security-auditor: Please review the payment processing code for
security vulnerabilities. Focus on: 1) Credit card handling, 2)
Authorization checks, 3) Logging and error messages.
```

**Automated workflow trigger:**
```
When code-reviewer approves implementation:
@security-auditor: Code review complete. Ready for security audit.
Files to review:
- app/Http/Controllers/PaymentController.php
- app/Services/PaymentService.php
```

**Escalation pattern:**
```
@tech-lead: Code review cycle exceeded max iterations (3).
Escalating to tech-lead for decision:
- Issue: Critical security concern on rate limiting implementation
- Status: Unresolved after 3 review cycles
- Recommendation: Discuss approach with team
```

---

## Summary

Custom subagents in Claude Code provide:

1. **Specialization** - Agents focused on specific domains
2. **Workflow Automation** - Multi-agent workflows with handoffs
3. **Quality Gates** - Validators and reviewers ensure standards
4. **Scalability** - Many specialized agents handling different concerns
5. **Learning** - Agents can build memory of patterns and standards
6. **Flexibility** - Tool permissions enable different role types

Start with simple read-only validators, expand to full workflows with autonomous cycles and handoffs between agents. Use the provided examples as templates and adapt to your project's needs.

