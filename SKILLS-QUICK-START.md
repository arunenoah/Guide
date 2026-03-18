# Claude Code Skills - Quick Start Guide

**Print this guide and keep at your desk!**

---

## Overview

Your 6 subagents now load specialized skills automatically. Each skill provides expert guidance for that role.

| Agent | Skill | Use When |
|-------|-------|----------|
| **tech-lead** | `tech-lead-gates-skill` | Approving designs (Gate A, B, C) |
| **tech-writer** | `tech-writer-spec-skill` | Creating specs and drift analysis |
| **senior-engineer** | `senior-engineer-laravel-skill` | Implementing features |
| **code-reviewer** | `code-reviewer-quality-skill` | Reviewing code quality |
| **qa-tester** | `qa-testing-skill` | Testing (4 categories) |
| **security-reviewer** | `security-audit-skill` | Security audits (OWASP Top 10) |

---

## 5-Phase Pipeline with Skills

```
Phase 1: DESIGN
  Agent: tech-lead (loads tech-lead-gates-skill)
  Gate A: Design-to-Spec Approval
  ├─ Reviews spec for clarity (no vague language)
  ├─ Checks security explicit (not implied)
  ├─ Verifies performance has numbers (< 500ms, not "fast")
  └─ Approves or rejects with signature

Phase 2: SPECIFICATION
  Agent: tech-writer (loads tech-writer-spec-skill)
  Creates: intended-docs.md
  ├─ Business purpose and overview
  ├─ Testable acceptance criteria (AC1, AC2, AC3)
  ├─ Technical architecture (models, services, API)
  ├─ Security requirements (explicit)
  ├─ FLKitOver integration workflow
  ├─ Performance requirements (with numbers)
  ├─ Testing strategy
  └─ Environment variables and dependencies

Phase 3: IMPLEMENTATION
  Agent: senior-engineer (loads senior-engineer-laravel-skill)
  Implements: Features following patterns
  ├─ Migrations with UUID keys and indexes
  ├─ Models with relationships and casts
  ├─ Services with composition pattern (DI)
  ├─ Controllers (thin - no business logic)
  ├─ Form Requests with validation
  ├─ Tests (unit, feature, integration)
  ├─ Comprehensive PHPDoc
  └─ Code quality checklist ✓

Phase 4: CODE REVIEW
  Agent: code-reviewer (loads code-reviewer-quality-skill)
  Checks: Critical/High/Medium issues
  ├─ 10 Critical Issues (blocking)
  │   └─ strict types, logic bugs, security, performance, errors
  ├─ 8 High Priority (should fix)
  │   └─ type hints, PHPDoc, conventions, relationships, validation
  ├─ 5 Medium Priority (suggestions)
  │   └─ duplication, large methods, tests, consistency, logging
  └─ Approves when all critical/high issues resolved

Phase 5: TESTING
  Agent: qa-tester (loads qa-testing-skill)
  4 Test Categories (ALL required):
  ├─ Positive: Happy path (user does everything right)
  ├─ Negative: Error cases (invalid input, edge cases, constraints)
  ├─ Non-Functional: Performance, security, load, accessibility
  ├─ Regression: Existing features still work
  └─ Report bugs with clear reproduction steps

GATE B: DRIFT & TEST AUDIT (tech-lead)
  Tech-lead: Spot-check for circular tests
  ├─ Read actual code (not just test reports)
  ├─ Verify constraints ENFORCED (not just "code exists")
  ├─ Review drift.md (differences from spec)
  └─ Approve or request fixes

SECURITY AUDIT (security-reviewer)
  Agent: security-reviewer (loads security-audit-skill)
  Checks: OWASP Top 10
  ├─ Authentication/Authorization
  ├─ Injection (SQL, command)
  ├─ Sensitive data exposure
  ├─ IDOR (Insecure Direct Object Reference)
  ├─ XSS (Cross-Site Scripting)
  ├─ FLKitOver security (credentials, webhooks)
  ├─ AWS service security (S3, SES)
  └─ Logging and monitoring

GATE C: STAGING-TO-PROD (tech-lead + SRE)
  Tech-lead: Pre-deployment checks
  ├─ All tests passed
  ├─ No new error logs
  ├─ Performance baseline met
  ├─ Rollback procedure tested
  ├─ Monitoring alerts configured
  └─ Dual signature (Tech-Lead + SRE)
```

---

## Using Skills in Prompts

### Tech-Lead Gate A

```
"Using tech-lead-gates-skill, review this spec for Gate A approval.

[paste intended-docs.md]

Questions:
1. Are all requirements testable (not vague)?
2. Is security explicitly listed?
3. Does performance have numbers?
4. Are third-party APIs specified?
5. Are edge cases documented?

Ready to approve or needs revision?"
```

### Tech-Writer Specification

```
"Using tech-writer-spec-skill, create intended-docs.md for password reset feature.

Business requirement:
- Users forgot password should be able to reset via email
- Reset link expires in 24 hours
- Rate limited to 5 attempts per hour
- Must work with FLKitOver document signing

Create:
1. Testable acceptance criteria
2. API endpoint design with validation
3. Database schema (UUID keys)
4. Security implementation details
5. Testing strategy (all 4 categories)
6. Performance requirements with numbers"
```

### Senior-Engineer Implementation

```
"Using senior-engineer-laravel-skill, implement password reset from password-reset-intended-docs.md.

Spec: [paste intended-docs.md]

Follow the skill patterns for:
- Migrations with proper indexes
- Models with relationships
- Services with composition (DI)
- Controllers (thin)
- Form Requests with validation
- Comprehensive tests
- PHPDoc documentation

When complete, run code quality checklist and auto-invoke code-reviewer."
```

### Code-Reviewer Quality Check

```
"Using code-reviewer-quality-skill, review this PR thoroughly.

[paste PR diff/code]

Check against skill:
- 10 Critical Issues (blocking)
- 8 High Priority Issues (should fix)
- 5 Medium Priority suggestions

Provide specific feedback with file:line references.
Approve or request changes."
```

### QA Testing

```
"Using qa-testing-skill, create comprehensive test plan for password reset.

Spec: [paste intended-docs.md]

Create ALL 4 categories:
1. Positive tests (happy path)
2. Negative tests (error cases, edge cases)
3. Non-functional tests (performance, security)
4. Regression tests (existing features still work)

Also write Playwright E2E tests for UI workflow.

Execute tests and report bugs with reproduction steps."
```

### Security Audit

```
"Using security-audit-skill, conduct security audit of password reset feature.

Code: [paste implementation files]

Check:
- OWASP Top 10 vulnerabilities
- FLKitOver security (webhook verification, credentials)
- AWS service security (S3, SES, IAM)
- Input validation and authorization
- Sensitive data exposure
- Rate limiting enforcement
- Error handling (no details leaked)

Generate security report with severity levels and remediation."
```

---

## Gate Signatures

### Gate A (Design Approval)

```
✅ APPROVED FOR CODING

Requirements testable: YES
Security explicit: YES
Performance numbered: YES (< 500ms)
APIs specified: YES
Edge cases documented: YES
No vague language: YES

Approved by: _________________ Date: _______
             Tech-Lead Name
```

### Gate B (Code & Test Audit)

```
✅ APPROVED FOR STAGING

Code matches spec: YES
Tests valid (not circular): YES
drift.md reviewed: YES
Red flags: NONE
Circular tests caught: NONE

Approved by: _________________ Date: _______
             Tech-Lead Name
```

### Gate C (Production Deploy)

```
✅ APPROVED FOR PRODUCTION

Tests passed: YES
Error logs clean: YES
Performance OK: YES
Rollback tested: YES
Monitoring configured: YES
30-min post-deploy ready: YES

Tech-Lead Approval: _________________ Date: _______
SRE/DevOps Sign-off: ________________ Date: _______

BOTH signatures required before deploy.
```

---

## What Each Skill Checks

### tech-lead-gates-skill

**Gate A (Design):**
- [ ] Requirements testable (AC1, AC2, not "should be fast")
- [ ] Security explicit (rate limiting, CSRF, encryption)
- [ ] Performance has numbers (< 500ms, not "quickly")
- [ ] Database schema changes approved
- [ ] Third-party APIs specified (SES, S3, Twilio, FLKitOver)
- [ ] Edge cases documented (expired tokens, failures)
- [ ] No AI hallucination triggers (vague language)

**Gate B (Code Review & Tests):**
- [ ] Read actual code (not just test reports)
- [ ] Spot-check for circular tests (tests mirror code flaw)
- [ ] Review drift.md (differences from spec)
- [ ] No red flags (simplified implementation, removed security)

**Gate C (Production):**
- [ ] All tests passed
- [ ] Error logs clean (last 24h)
- [ ] Performance meets targets
- [ ] Rollback procedure tested
- [ ] Monitoring alerts configured
- [ ] 30-minute post-deploy window ready

---

### tech-writer-spec-skill

**Specification Standards:**
- Use numbers, not adjectives ("< 200ms" not "fast")
- Testable acceptance criteria (AC1, AC2, AC3 format)
- Technical architecture (models, services, API)
- Security explicit (not implied)
- Database schema defined (UUID keys, relationships)
- FLKitOver integration workflow
- AWS service usage (S3, SES, etc.)
- Performance requirements (with metrics)
- Testing strategy (4 categories)

**Drift Analysis:**
- Side-by-side intended vs actual
- Reason for each deviation
- Impact assessment (positive/negative/neutral)
- Tech-lead approval signature

---

### senior-engineer-laravel-skill

**Code Quality Checklist:**
- [ ] `declare(strict_types=1);` in every PHP file
- [ ] Type hints on all functions (parameters + return)
- [ ] PHPDoc with @param/@return/@throws
- [ ] No debug functions (no die(), var_dump(), print_r())
- [ ] Specific exception handling (not generic Exception)
- [ ] Eloquent queries (not raw SQL)
- [ ] UUID primary keys (not numeric IDs)
- [ ] Services with dependency injection
- [ ] Thin controllers (no business logic)
- [ ] Form Requests for validation
- [ ] Tests for happy path + error cases
- [ ] Database indexes on filtered/joined columns
- [ ] No N+1 queries (use eager loading)
- [ ] FLKitOver integration error handling

---

### code-reviewer-quality-skill

**Critical Issues (Blocking):**
- Missing `declare(strict_types=1);`
- Logic bugs or errors
- Security vulnerabilities (injection, auth bypass, hardcoded secrets)
- Performance bottlenecks (N+1 queries, missing indexes)
- Missing error handling
- Architecture deviation
- Breaking changes without migration
- Debug functions left in code (die, var_dump)
- Missing or improper UUID usage
- Sensitive data exposed in responses

**High Priority Issues (Should Fix):**
- Missing type hints
- Incomplete PHPDoc
- Not following Laravel conventions
- Missing database relationships
- No input validation
- Missing exception handling
- Improper dependency injection
- Missing database transactions

---

### qa-testing-skill

**4 Test Categories (ALL Required):**
1. **Positive** - Happy path (user does everything right)
2. **Negative** - Error cases (invalid input, constraints violated)
3. **Non-Functional** - Performance, security, load
4. **Regression** - Existing features still work

**Test Coverage Required:**
- All functional requirements verified
- Error cases handled
- Security constraints enforced
- Performance meets targets (< 200ms for list, < 500ms for create)
- Rate limiting works
- CSRF protection active
- Unicode/special characters handled
- Edge cases covered

---

### security-audit-skill

**OWASP Top 10 Check:**
1. Broken Authentication (passwords hashed, sessions secure)
2. Broken Access Control (authorization checks, IDOR prevention)
3. Injection (Eloquent used, no raw SQL with user input)
4. Insecure Design (rate limiting, security explicit)
5. Sensitive Data Exposure (no passwords in responses)
6. IDOR (object-level authorization)
7. XSS (Blade auto-escapes)
8. Insecure Deserialization (JSON, not unserialize)
9. Vulnerable Components (dependency audit)
10. Insufficient Logging (security events tracked)

**Additional Checks:**
- FLKitOver security (credentials, webhooks)
- AWS service security (S3 private, SES config)
- Laravel security config (CSRF, headers, sessions)
- Hardcoded credentials (none allowed)
- Sensitive data in logs (not allowed)

---

## Example: Complete Feature Flow with Skills

```
Day 1: TECH-LEAD Gate A
  Prompt: "Using tech-lead-gates-skill, approve this password reset spec"
  → Tech-lead reviews with Gate A checklist
  → Approves with signature or requests revision

Day 1 (afternoon): TECH-WRITER
  Prompt: "Using tech-writer-spec-skill, create intended-docs.md for password reset"
  → Creates detailed spec with testable AC, API design, security requirements
  → Tech-lead reviews for clarity

Day 2: SENIOR-ENGINEER
  Prompt: "Using senior-engineer-laravel-skill, implement password reset from spec"
  → Implements with all patterns: migrations, models, services, controllers, tests
  → Runs code quality checklist
  → Auto-invokes code-reviewer

Day 2 (afternoon): CODE-REVIEWER
  Prompt: "Using code-reviewer-quality-skill, review password reset PR"
  → Checks 10 critical issues, 8 high priority, 5 medium priority suggestions
  → Provides feedback with file:line references
  → Requests changes or approves

Day 3: QA-TESTER
  Prompt: "Using qa-testing-skill, test password reset with all 4 categories"
  → Creates and executes: positive, negative, non-functional, regression tests
  → Writes Playwright E2E tests
  → Reports bugs or approves

Day 3 (afternoon): TECH-LEAD Gate B
  Prompt: "Using tech-lead-gates-skill, conduct Gate B drift audit"
  → Reads actual code
  → Spot-checks for circular tests
  → Reviews drift.md
  → Approves for staging

Day 4: SECURITY-REVIEWER
  Prompt: "Using security-audit-skill, audit password reset security"
  → Checks OWASP Top 10
  → Verifies FLKitOver security
  → Checks AWS service security
  → Approves or requests security fixes

Day 4 (evening): TECH-LEAD Gate C
  Prompt: "Using tech-lead-gates-skill, authorize Gate C production deployment"
  → Verifies pre-deployment checklist
  → Gets dual sign-off (Tech-Lead + SRE)
  → Deploys to production
  → Monitors 30-minute window

RESULT: Feature in production with:
✅ Thorough design review
✅ Comprehensive specification
✅ Enterprise code quality
✅ Thorough testing (4 categories)
✅ Security audit
✅ Production approval gates
```

---

## Troubleshooting

### "Agent didn't load the skill"
→ Make sure skill reference is in prompt
→ Try: "Using [skill-name], [task]"

### "Skill doesn't have what I need"
→ Check if all relevant sections are in the skill
→ Skills are versioned - may need update
→ File an issue to improve the skill

### "Gate approval taking too long"
→ Gates are thorough by design
→ If spec needs clarification, Gates will request it
→ Better to fix at Gate A than in production

### "Skill seems incomplete"
→ Skills can be updated to address gaps
→ Document the gap and file an improvement
→ Skills evolve with team standards

---

## Quick Links

**Skills Location:**
`/Users/arunkumar/Documents/Application/Agents/skills/`

**Subagents Location:**
`/Users/arunkumar/Documents/Application/Agents/claude_agents/agents/`

**Full Documentation:**
- `SKILLS-INTEGRATION-SUMMARY.md` - Complete details
- `DEVELOPER-COMPLETE-POLICY-ARCHIVE.html` - Full policy (includes Gate details, skills section)

---

## Remember

✅ **All 6 agents use skills** - Consistent quality across all roles
✅ **Skills are version-controlled** - Updates propagate to all agents
✅ **Gates are mandatory** - Design, code review, and deployment gates
✅ **4 test categories required** - Don't skip regression or non-functional
✅ **OWASP Top 10 covered** - Security is not optional

**Start using skills with your next feature!**

Print this guide and keep at your desk for quick reference.
