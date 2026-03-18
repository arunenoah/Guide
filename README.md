# Claude Code Agents + Skills Framework

A comprehensive, enterprise-grade framework for AI-assisted software development using Claude Code with 6 specialized agents, reusable skills, and 3 mandatory quality gates.

## 🎯 Overview

This repository contains a complete system for managing AI-assisted development workflows with strict quality controls, security audits, and production deployment procedures. It combines:

- **6 Specialized Subagents** - Each with a specific role in the development pipeline
- **6 Reusable Skills** - Training manuals (3,084 lines) that guide each agent's behavior
- **3 Mandatory Gates** - Quality checkpoints for design, code, and production
- **4 Test Categories** - Positive, Negative, Non-Functional, Regression tests
- **OWASP Top 10** - Complete security audit coverage
- **Generic Patterns** - Works with ANY project and ANY third-party API

## 📋 Table of Contents

1. [Quick Start](#quick-start)
2. [The 5-Phase Pipeline](#the-5-phase-pipeline)
3. [The 3 Mandatory Gates](#the-3-mandatory-gates)
4. [Agents & Skills](#agents--skills)
5. [Repository Structure](#repository-structure)
6. [Key Files](#key-files)
7. [How to Use This Framework](#how-to-use-this-framework)
8. [Security & Compliance](#security--compliance)
9. [Common Workflows](#common-workflows)
10. [Troubleshooting](#troubleshooting)

---

## Quick Start

### Installation

1. Clone this repository:
```bash
git clone https://github.com/arunenoah/Guide.git
cd Guide
```

2. Review the framework:
```bash
# Start here
cat README.md

# Then read the quick-start guide
cat SKILLS-QUICK-START.md

# Then understand the pipeline
cat AGENTS-SKILLS-VISUAL-GUIDE.txt

# Then see real examples
cat HOW-TO-USE-AGENTS-WITH-SKILLS.md
```

### First Use

When you have a business requirement, invoke the tech-lead agent:

```
/invoke tech-lead "Using tech-lead-gates-skill, design this requirement:
[Your business requirement here]"
```

The framework will guide you through Gate A → Specification → Implementation → Code Review → Testing → Gate B → Security Audit → Gate C → Production.

---

## The 5-Phase Pipeline

```
PHASE 1: DESIGN (Tech-Lead)
        ↓
PHASE 2: SPECIFICATION (Tech-Writer)
        ↓
PHASE 3: IMPLEMENTATION (Senior-Engineer)
        ↓
PHASE 4: CODE REVIEW (Code-Reviewer)
        ↓
PHASE 5: TESTING (QA-Tester)
        ↓
    GATE B: DRIFT & TEST AUDIT (Tech-Lead)
        ↓
    SECURITY AUDIT (Security-Reviewer)
        ↓
    GATE C: PRODUCTION (Tech-Lead + SRE)
```

### Phase 1: Design (Tech-Lead)
- **Goal**: Approve or reject business requirement
- **Deliverable**: `[feature-name].md` design document
- **Gate**: Gate A - Design-to-Spec approval

### Phase 2: Specification (Tech-Writer)
- **Goal**: Create testable, unambiguous specification
- **Deliverable**: `[feature-name]-intended-docs.md` with acceptance criteria, APIs, security requirements
- **Mandatory**: Avoid vague language ("fast" → "< 200ms response time")

### Phase 3: Implementation (Senior-Engineer)
- **Goal**: Write code following Laravel patterns
- **Deliverable**: Feature branch with code, tests, migrations
- **Standards**: PHP 8.2+ strict types, UUID keys, dependency injection, 4 test categories

### Phase 4: Code Review (Code-Reviewer)
- **Goal**: Check for critical issues, security vulnerabilities, code quality
- **Checklist**: 10 critical issues (blocking), 8 high priority, 5 medium priority
- **Decision**: Approve or request changes

### Phase 5: Testing (QA-Tester)
- **Goal**: Test ALL 4 categories - Positive, Negative, Non-Functional, Regression
- **Deliverable**: Test results with Playwright evidence
- **Mandatory**: All 4 categories must pass

### Gate B: Drift & Test Audit (Tech-Lead)
- **Goal**: Verify code matches specification
- **Checks**: Circular test detection, specification drift, security constraints
- **Red Flags**: Cut corners, skipped error handling, unauthorized scope changes

### Security Audit (Security-Reviewer)
- **Goal**: OWASP Top 10 verification
- **Coverage**: Authentication, injection prevention, sensitive data, rate limiting, third-party APIs
- **Decision**: Security approved or issues found

### Gate C: Production Deployment (Tech-Lead + SRE)
- **Goal**: Authorize production deployment
- **Checks**: All tests pass, no errors, performance OK, rollback plan tested
- **Dual Sign-Off**: Tech-Lead AND SRE must approve
- **Post-Deploy**: 30-minute monitoring window

---

## The 3 Mandatory Gates

### Gate A: Design-to-Spec Approval
**When**: Tech-writer creates intended-docs.md
**Who**: Tech-Lead
**Purpose**: Ensure spec is testable and unambiguous

**REJECT if**:
- Requirements are vague ("should be fast" - no number)
- Security not explicitly listed
- Third-party APIs not specified (name, authentication, error handling)
- No acceptance criteria or edge cases documented

**APPROVE if**: Spec has numbers, security details, and testable criteria

### Gate B: Drift & Test Audit
**When**: Code-reviewer approves + qa-tester finishes tests
**Who**: Tech-Lead
**Purpose**: Spot-check code and detect circular tests

**Circular Test Detection**: Tests pass because they mirror code flaws exactly
- Example Bad: Test only checks "feature exists" (not "constraint enforced")
- Example Good: Rate limit test makes 6 requests, 5 succeed, 6th fails with 429

**REJECT if**:
- Code doesn't match intended-docs.md
- Tests are circular (mock the constraint being tested)
- drift.md shows "cut corners" or "skipped error handling"
- Security constraints not actually enforced

### Gate C: Staging-to-Production
**When**: All tests pass + Playwright evidence + 30min staging run
**Who**: Tech-Lead + SRE
**Purpose**: Authorize production deployment

**Checks**:
- All tests passed in staging
- No new error logs
- Performance baselines met
- Database migrations successful and reversible
- Rollback procedure tested
- Third-party services confirmed working

**Auto-Rollback Triggers** (execute immediately):
- Error rate > 5% for 5 minutes
- Database response time > 2 seconds
- Third-party service failures blocking feature
- Security incident detected

---

## Agents & Skills

### The Relationship
```
Agent = Worker (Executes Task)
Skill = Training Manual (Guidance)

When you invoke an agent, it loads its skill and follows that skill's checklist
```

### 6 Specialized Agents

| Agent | Skill | Role | Deliverable |
|-------|-------|------|-------------|
| **tech-lead** | tech-lead-gates-skill | Design approval, code audit, production auth | Gate A/B/C decisions |
| **tech-writer** | tech-writer-spec-skill | Create specification | intended-docs.md |
| **senior-engineer** | senior-engineer-laravel-skill | Implement feature | Code, tests, migrations |
| **code-reviewer** | code-reviewer-quality-skill | Review code quality | Approved/rejected PR |
| **qa-tester** | qa-testing-skill | Test all 4 categories | Test results, Playwright evidence |
| **security-reviewer** | security-audit-skill | OWASP Top 10 audit | Security report |

### 6 Reusable Skills (3,084 Lines)

#### 1. tech-lead-gates-skill (280 lines)
**Purpose**: Gate A, B, C checklists and approval decisions

**Contains**:
- Gate A: Design-to-Spec review checklist
- Gate B: Code audit + circular test detection
- Gate C: Production deployment authorization
- Rollback procedures and auto-rollback triggers
- Red flag checklist for blocking issues

**Usage**: `/invoke tech-lead "Using tech-lead-gates-skill, [task]"`

#### 2. tech-writer-spec-skill (320 lines)
**Purpose**: Specification writing standards

**Contains**:
- Anti-pattern language to avoid (vague words)
- Pattern language to use (testable criteria)
- intended-docs.md template structure
- drift.md template for spec deviations
- Security, performance, API, database schema requirements

**Usage**: `/invoke tech-writer "Using tech-writer-spec-skill, create specification for [feature]"`

#### 3. senior-engineer-laravel-skill (450 lines)
**Purpose**: Laravel implementation patterns

**Contains**:
- PHP 8.2+ strict typing, constructor promotion
- Database migrations with UUID primary keys
- Eloquent models with relationships and casts
- Service layer with dependency injection
- Form Request validation
- API Controllers and Resources
- Unit, feature, and integration test patterns
- Code quality checklist with 50+ items
- Exception handling and error responses
- PHPDoc documentation standards

**Usage**: `/invoke senior-engineer "Using senior-engineer-laravel-skill, implement [feature]"`

#### 4. code-reviewer-quality-skill (380 lines)
**Purpose**: Code quality and security checklist

**Contains**:
- 10 Critical Issues (blocking merge)
- 8 High Priority Issues (should fix)
- 5 Medium Priority suggestions
- SQL injection, N+1 queries, authorization checks
- Security vulnerability detection
- Hardcoded credentials and secrets
- Error handling and logging
- Performance anti-patterns
- File:line specific feedback format

**Usage**: `/invoke code-reviewer "Using code-reviewer-quality-skill, review this PR"`

#### 5. qa-testing-skill (420 lines)
**Purpose**: 4 mandatory test categories

**Contains**:
- Positive tests (happy path)
- Negative tests (error cases, edge cases, validation failures)
- Non-functional tests (performance, security, load testing)
- Regression tests (existing features still work)
- Playwright E2E testing patterns
- Mocking and unit test examples
- Bug reporting format with reproduction steps
- Performance benchmarking
- Security testing scenarios

**Usage**: `/invoke qa-tester "Using qa-testing-skill, test [feature] with all 4 categories"`

#### 6. security-audit-skill (390 lines)
**Purpose**: OWASP Top 10 security verification

**Contains**:
- A01: Broken Access Control
- A02: Cryptographic Failures
- A03: Injection
- A04: Insecure Design
- A05: Security Misconfiguration
- A06: Sensitive Data Exposure
- A07: Identification and Authentication Failures
- A08: Software and Data Integrity Failures
- A09: Logging and Monitoring Failures
- A10: SSRF
- Third-party API integration security
- AWS service security (S3, SES, DynamoDB)
- Laravel security configuration
- Webhook signature verification
- Credential management

**Usage**: `/invoke security-reviewer "Using security-audit-skill, audit [feature] for OWASP Top 10"`

---

## Repository Structure

```
.
├── README.md                              ← START HERE
├── SKILLS-QUICK-START.md                  ← Print-friendly quick reference
├── AGENTS-SKILLS-VISUAL-GUIDE.txt         ← ASCII diagrams of pipeline
│
├── claude_agents/                         ← 6 AGENT INSTRUCTIONS
│   └── agents/
│       ├── tech-lead.md
│       ├── tech-writer.md
│       ├── senior-engineer.md
│       ├── code-reviewer.md
│       ├── qa-tester.md
│       └── security-reviewer.md
│
├── skills/                                ← 6 REUSABLE SKILLS (3,084 lines)
│   ├── tech-lead-gates-skill/
│   │   └── SKILL.md                      ← Gate A, B, C checklists
│   ├── tech-writer-spec-skill/
│   │   └── SKILL.md                      ← Specification standards
│   ├── senior-engineer-laravel-skill/
│   │   └── SKILL.md                      ← Laravel patterns
│   ├── code-reviewer-quality-skill/
│   │   └── SKILL.md                      ← Code review checklist
│   ├── qa-testing-skill/
│   │   └── SKILL.md                      ← 4 test categories
│   └── security-audit-skill/
│       └── SKILL.md                      ← OWASP Top 10
│
├── HOW-TO-USE-AGENTS-WITH-SKILLS.md      ← Real-world example (password reset)
├── AGENTS-SKILLS-COMMAND-REFERENCE.md    ← Copy-paste commands for each phase
├── SKILLS-INTEGRATION-SUMMARY.md         ← What each skill contains with examples
├── TECH-LEAD-GATES-QUICK-REFERENCE.md    ← Printable Gate A/B/C card
│
├── SKILLS-UPDATE-GENERIC.md              ← Generic patterns (not project-specific)
├── SKILLS-GENERIC-CHECKLIST.md           ← Checklist to update skills
├── SKILLS-GENERIC-SUMMARY.txt            ← Overview of generic updates
│
├── ARCHITECTURAL-REVIEW-SUMMARY.txt      ← 10 critical gaps addressed
├── ARCHITECTURAL-REVIEW-IMPLEMENTATION.md ← How gaps were resolved
│
└── DEVELOPER-COMPLETE-POLICY.html        ← Complete policy document
    DEVELOPER-COMPLETE-POLICY-CRISP.html  ← Concise version (350 lines)
    TRAINING-QUIZ-COMPREHENSIVE.html      ← 50-question quiz
    SECURITY-HARDENED-STRATEGY.html       ← Security best practices
    ISO27001-SOC2-COMPLIANCE-MAPPING.html ← Compliance mapping
```

---

## Key Files

### 📖 Must-Read Documentation

1. **README.md** (you are here)
   - Overview and quick start
   - Phase pipeline and gates
   - Agent-skill relationship

2. **SKILLS-QUICK-START.md**
   - Print-friendly quick reference
   - One-line commands for each phase
   - What each skill checks
   - Troubleshooting guide

3. **AGENTS-SKILLS-VISUAL-GUIDE.txt**
   - ASCII diagrams of 5-phase pipeline
   - Relationship diagram
   - Real-world workflow timeline
   - Key principles checklist

4. **HOW-TO-USE-AGENTS-WITH-SKILLS.md**
   - Real-world password reset example
   - Phase-by-phase walkthrough
   - Gate explanations with examples
   - Common mistakes to avoid

### 🎯 Reference Guides

5. **AGENTS-SKILLS-COMMAND-REFERENCE.md**
   - Copy-paste ready prompts
   - Expected outputs for each phase
   - Gate signature templates

6. **SKILLS-INTEGRATION-SUMMARY.md**
   - Complete skill details
   - Code examples
   - Benefits of each skill
   - Validation checklist

7. **TECH-LEAD-GATES-QUICK-REFERENCE.md**
   - Printable 1-page card
   - Gate A/B/C checklists
   - Decision trees
   - Emergency rollback procedures

### 🔄 Generic Skills Updates

8. **SKILLS-UPDATE-GENERIC.md**
   - Detailed before/after for each skill
   - Generic patterns for ANY API
   - Cloud service security checklist
   - How to apply updates

9. **SKILLS-GENERIC-CHECKLIST.md**
   - Line-by-line update checklist
   - Specific examples to replace
   - Validation tests

10. **SKILLS-GENERIC-SUMMARY.txt**
    - Quick overview
    - Before/after examples
    - Benefits after update
    - Placeholder mapping

### 📋 Architecture & Policy

11. **ARCHITECTURAL-REVIEW-SUMMARY.txt**
    - 10 critical gaps identified
    - How each gap was addressed
    - Circular test examples

12. **DEVELOPER-COMPLETE-POLICY.html**
    - 12 comprehensive sections
    - Security hardening strategy
    - Development workflow
    - Code review checklist

13. **TRAINING-QUIZ-COMPREHENSIVE.html**
    - 50-question interactive quiz
    - 85% passing requirement
    - Covers critical policies

---

## How to Use This Framework

### Step 1: Business Requirement
You have a feature to build: "Add password reset functionality"

### Step 2: Invoke Tech-Lead (Gate A)
```
/invoke tech-lead "Using tech-lead-gates-skill, design this requirement:
Users should be able to reset their password via email.
They click 'forgot password', enter email, receive reset link,
click link, enter new password."
```
→ **Deliverable**: Design document with specification
→ **Gate A**: Tech-Lead approves design

### Step 3: Invoke Tech-Writer (Phase 2)
```
/invoke tech-writer "Using tech-writer-spec-skill, create intended-docs.md
for password reset feature from this design:
[paste design document]

Include:
- Testable acceptance criteria (AC1, AC2, AC3)
- API endpoints with validation
- Security requirements (CSRF, password strength, rate limiting)
- Performance targets with numbers
- Database schema
- Third-party services if needed
- Error handling scenarios"
```
→ **Deliverable**: intended-docs.md with specification
→ **Your Review**: Verify spec is clear and testable

### Step 4: Invoke Senior-Engineer (Phase 3)
```
/invoke senior-engineer "Using senior-engineer-laravel-skill, implement
password reset feature following this specification:
[paste intended-docs.md]

This should include:
- Database migrations with UUID keys
- Models, services, controllers
- Form request validation
- Unit, feature, integration tests
- Error handling and logging"
```
→ **Deliverable**: Feature branch with complete code
→ **Auto-Invokes**: Code-Reviewer

### Step 5: Invoke Code-Reviewer (Phase 4)
```
/invoke code-reviewer "Using code-reviewer-quality-skill, review this PR:
[paste code]

Check for:
- 10 critical issues (blocking)
- 8 high priority issues
- 5 medium priority suggestions
- Security vulnerabilities
- Code quality"
```
→ **Deliverable**: Review with feedback
→ **Decision**: Approve or request changes

### Step 6: Invoke QA-Tester (Phase 5)
```
/invoke qa-tester "Using qa-testing-skill, test password reset feature
with ALL 4 test categories:

1. Positive tests (happy path - user resets password successfully)
2. Negative tests (invalid email, expired link, weak password, CSRF)
3. Non-functional tests (performance < 200ms, load test 100 concurrent)
4. Regression tests (existing auth still works)

Include Playwright E2E tests and evidence."
```
→ **Deliverable**: Test results with Playwright screenshots
→ **All 4 Categories**: Must pass

### Step 7: Tech-Lead Gate B (Drift & Test Audit)
```
/invoke tech-lead "Using tech-lead-gates-skill, conduct Gate B audit:

1. Read actual code (not test reports)
2. Spot-check for circular tests
3. Review drift.md (any spec deviations?)
4. Verify security constraints actually enforced

Approve or request fixes."
```
→ **Gate B**: Tech-Lead approves implementation

### Step 8: Invoke Security-Reviewer
```
/invoke security-reviewer "Using security-audit-skill, audit password reset
for OWASP Top 10:
[paste code]

Check:
- Authentication and authorization
- Injection prevention
- Sensitive data protection
- Rate limiting on password reset endpoint
- CSRF token verification
- Hardcoded credentials"
```
→ **Deliverable**: Security audit report
→ **Decision**: Approved or issues found

### Step 9: Tech-Lead Gate C (Production)
```
/invoke tech-lead "Using tech-lead-gates-skill, authorize Gate C deployment:

1. All tests passed in staging
2. No new errors
3. Performance OK
4. Rollback tested
5. Monitoring alerts configured

Dual sign-off required (Tech-Lead + SRE)."
```
→ **Gate C**: Tech-Lead + SRE authorize production

### Step 10: Monitor & Rollback
- Deploy to production
- Monitor for 30 minutes
- Auto-rollback if error rate > 5%

---

## Security & Compliance

### 🔒 Security Standards

This framework implements:
- **OWASP Top 10** - Complete checklist in security-audit-skill
- **Secure Coding** - Input validation, output encoding, authorization checks
- **Credential Management** - Environment variables, never hardcoded
- **Third-Party APIs** - Authentication, webhook signature verification, error handling
- **Circular Test Detection** - Prevents false positives in security tests

### 📋 Compliance

- **ISO 27001** - Security control mapping available
- **SOC 2** - Audit trail and logging standards
- **GDPR** - Sensitive data protection and retention
- **Zero-Trust** - Never trust external services by default

### ✅ Mandatory Policies

1. **Never commit secrets** - Use environment variables
2. **Never skip tests** - All 4 categories required
3. **Never bypass gates** - Gate A, B, C are mandatory
4. **Never assume safety** - Verify constraints are enforced, not just "code exists"

---

## Common Workflows

### Starting a New Feature
1. Read requirement
2. `/invoke tech-lead` → Design
3. `/invoke tech-writer` → Specification
4. `/invoke senior-engineer` → Implementation
5. Continue through gates to production

### Hotfixing a Production Bug
1. Create fix in separate branch
2. Minimal `/invoke code-reviewer` review
3. Comprehensive `/invoke qa-tester` testing
4. `/invoke tech-lead` Gate C approval
5. Deploy with monitoring

### Reviewing Someone Else's Code
1. `/invoke code-reviewer` → Quality checklist
2. Check for critical issues
3. Request changes or approve
4. Pass to QA testing phase

### Updating a Third-Party API Integration
1. Read API documentation
2. Review current implementation
3. Update code following senior-engineer-laravel-skill patterns
4. Update tests for new API behavior
5. `/invoke security-reviewer` → Verify auth, rate limits, error handling
6. Gate C approval before production

---

## Troubleshooting

### "My design was rejected at Gate A"
- Requirement is too vague
- Security not explicitly listed
- Third-party API not specified (name, endpoints, auth method)
- Add numbers to performance requirements ("fast" → "< 200ms")
- Revise and resubmit

### "My code was rejected at code review"
- Critical issue found (blocking merge)
- Ask code-reviewer for specific feedback
- Address issues in new commit
- Request re-review

### "Tests are failing"
- Run individual test categories
- Check error messages
- Verify test setup (mocks, fixtures)
- `qa-testing-skill` has troubleshooting section

### "Gate B audit blocked implementation"
- Code doesn't match specification
- Tests are circular (mock the constraint)
- Security not actually enforced
- Revise in staging and resubmit

### "I need to rollback production"
- **Auto-rollback**: > 5% error rate for 5 min
- **Manual rollback**: Run `scripts/rollback.sh`
- **Post-rollback**: Investigate root cause in staging
- **Re-deploy**: Fix and get Gate C re-approval

### "Can I use this framework with different tech stacks?"
- Skills are generic patterns
- Replace `[FeatureName]`, `[Third-Party API]` with your values
- Adapt Laravel patterns to your framework (Node, Python, Go, etc.)
- See `SKILLS-UPDATE-GENERIC.md` for mapping

---

## Next Steps

### 1. Read the Quick Start Guides
```bash
cat SKILLS-QUICK-START.md
cat AGENTS-SKILLS-VISUAL-GUIDE.txt
```

### 2. Review a Real Example
```bash
cat HOW-TO-USE-AGENTS-WITH-SKILLS.md
```

### 3. Check the Skill Details
```bash
cat SKILLS-INTEGRATION-SUMMARY.md
```

### 4. Study Your First Skill
```bash
cat skills/tech-lead-gates-skill/SKILL.md
```

### 5. Run Your First Gate A Approval
Identify a business requirement and invoke tech-lead with Gate A checklist

### 6. Join the Flow
Take your feature through all 5 phases and 3 gates to production

---

## License

Internal use only. Proprietary framework for enterprise-grade AI-assisted development.

---

## Support

For questions about using this framework:

1. Check **SKILLS-QUICK-START.md** for quick reference
2. Read **HOW-TO-USE-AGENTS-WITH-SKILLS.md** for detailed explanations
3. Review relevant skill in **skills/** directory
4. Check **TROUBLESHOOTING** section above

---

## Summary

This framework enables enterprise-grade software development with:

✅ 6 specialized agents ensuring quality at each phase
✅ 6 reusable skills (3,084 lines) guiding correct behavior
✅ 3 mandatory gates preventing defects from reaching production
✅ 4 test categories ensuring comprehensive coverage
✅ OWASP Top 10 security verification
✅ Generic patterns working with ANY project and ANY API
✅ Circular test detection preventing false security claims
✅ Clear rollback procedures and auto-rollback triggers

**Start with a business requirement. Follow the 5-phase pipeline. Execute the 3 gates. Ship with confidence.**

Happy shipping! 🚀
