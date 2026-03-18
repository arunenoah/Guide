# How to Use Agents + Skills Effectively

## Quick Answer

**Agents = Workers | Skills = Training Manuals**

When you invoke an agent, it should load its skill and follow that guidance.

```
/invoke tech-lead "Using tech-lead-gates-skill, [task]"
                  └─ Agent loads skill
                  └─ Agent follows skill's checklist
                  └─ Agent executes task with skill's guidance
```

---

## The Relationship

### What is an Agent?
- **Deployed worker** with specific role
- Located in `/claude_agents/agents/`
- Has access to specific MCPs (GitHub, MySQL, Playwright, etc.)
- Assigned to specific phase of pipeline
- Makes decisions and takes actions

### What is a Skill?
- **Reusable instruction set** teaching HOW to do the job
- Located in `/skills/[skill-name]/`
- YAML header + Markdown instructions
- NOT a worker, just guidance
- Can be used by multiple agents if needed

### Example Analogy

```
AGENT = Doctor in hospital
└─ Cardiologist (specialist)
└─ Has tools: stethoscope, EKG machine
└─ Makes decisions: approve treatment, order tests
└─ Reports to supervisor

SKILL = Medical textbook
└─ "Cardiology Procedures Manual"
└─ Step-by-step diagnosis process
└─ When to escalate
└─ Common errors to avoid
└─ Doctor reads manual WHILE working
```

---

## The 5-Phase Pipeline

### Phase 1: Design (Tech-Lead)

**What happens:**
```
You submit:
"Review this business requirement and create technical design"

Tech-Lead Agent:
1. Loads tech-lead-gates-skill
2. Reads the requirement
3. Follows skill's Gate A checklist:
   - Is requirement testable?
   - Is security explicit?
   - Are APIs specified?
   - Are edge cases documented?
4. Either approves or requests revision
5. Creates [feature-name].md design document
```

**How to invoke:**
```
Prompt: "Using tech-lead-gates-skill, review this business requirement
for Gate A approval and create technical design.

Business Requirement:
[Your requirement here]"
```

**What you get back:**
- ✅ Gate A approval with signature OR
- ❌ Requests revision with specific issues

---

### Phase 2: Specification (Tech-Writer)

**What happens:**
```
Tech-Lead hands off: [feature-name].md design

Tech-Writer Agent:
1. Loads tech-writer-spec-skill
2. Reads the design document
3. Creates [feature-name]-intended-docs.md using skill template:
   - Business purpose
   - Testable acceptance criteria (AC1, AC2, AC3)
   - Technical architecture
   - Security requirements (explicit, not vague)
   - Database schema
   - API endpoints with validation rules
   - FLKitOver integration
   - Performance targets with NUMBERS
   - Testing strategy (all 4 categories)
   - Migration plan
4. Skill prevents hallucination:
   - "< 200ms" not "fast"
   - Specific constraints, not implied
   - Edge cases documented
```

**How to invoke:**
```
Prompt: "Using tech-writer-spec-skill, create intended-docs.md for [feature].

Design document: [feature-name].md:
[paste design]

Requirements:
- Business purpose clear
- API endpoints with validation rules
- Database schema with UUID keys
- Security requirements explicit
- Performance targets with numbers
- Testing strategy for all 4 categories"
```

**What you get back:**
- ✅ [feature-name]-intended-docs.md with testable spec

---

### Phase 3: Implementation (Senior-Engineer)

**What happens:**
```
Tech-Lead hands off: [feature-name]-intended-docs.md

Senior-Engineer Agent:
1. Loads senior-engineer-laravel-skill
2. Reads the intended-docs.md spec
3. Implements following skill patterns:
   - Create migrations (UUID keys, relationships, indexes)
   - Create models (HasUuids, relationships, casts)
   - Create services (composition pattern, DI)
   - Create controllers (thin, Form Requests)
   - Create requests (validation rules)
   - Create tests (unit, feature, integration)
   - Add PHPDoc (comprehensive)
   - Follow strict types (declare(strict_types=1);)
4. Runs code quality checklist before finishing
5. Auto-invokes code-reviewer
```

**How to invoke:**
```
Prompt: "Using senior-engineer-laravel-skill, implement [feature-name].

Specification (intended-docs.md):
[paste complete spec]

Follow skill patterns for:
- Migrations with UUID keys and indexes
- Models with relationships and casts
- Services with composition (dependency injection)
- Controllers (thin - no business logic)
- Form Requests with validation
- Comprehensive tests
- PHPDoc documentation
- Strict typing on all functions

When complete:
1. Run code quality checklist
2. Auto-invoke code-reviewer"
```

**What you get back:**
- ✅ Implementation files (models, migrations, services, controllers, tests)
- ✅ Code quality checklist passed
- ✅ Code-reviewer auto-invoked

---

### Phase 4: Code Review (Code-Reviewer)

**What happens:**
```
Senior-Engineer hands off: [feature-name] PR with code

Code-Reviewer Agent:
1. Loads code-reviewer-quality-skill
2. Reviews code against skill checklist:
   - 10 Critical Issues (blocking merge)
   - 8 High Priority Issues (should fix)
   - 5 Medium Priority suggestions
3. Provides feedback with file:line references
4. Either approves OR requests changes
5. Senior-Engineer fixes → Code-Reviewer re-reviews
6. Iterate until approval
```

**How to invoke:**
```
Prompt: "Using code-reviewer-quality-skill, review this PR thoroughly.

[paste PR diff or code files]

Check against skill:
- 10 Critical Issues (blocking)
- 8 High Priority Issues (should fix)
- 5 Medium Priority suggestions

Provide specific feedback with file:line references.
Approve when all critical/high issues resolved."
```

**What you get back:**
- ✅ Code approved with feedback OR
- ❌ Detailed issues with file:line references

---

### Phase 5: Testing (QA-Tester)

**What happens:**
```
Code-Reviewer approves: Code is ready for testing

QA-Tester Agent:
1. Loads qa-testing-skill
2. Creates test plan with ALL 4 categories (mandatory):
   - Positive tests (happy path)
   - Negative tests (error cases, edge cases, constraints)
   - Non-functional tests (performance, security, load)
   - Regression tests (existing features still work)
3. Executes tests
4. Writes Playwright E2E tests for UI
5. Reports bugs with reproduction steps OR approves
```

**How to invoke:**
```
Prompt: "Using qa-testing-skill, create comprehensive test plan for [feature].

Specification: [paste intended-docs.md]

Create ALL 4 test categories:
1. Positive tests (happy path)
2. Negative tests (validation, edge cases, constraints)
3. Non-functional tests (performance, security, load)
4. Regression tests (existing features)

Also write Playwright tests for UI workflow.

Execute all tests and report results."
```

**What you get back:**
- ✅ Test results (passed) with coverage report OR
- ❌ Bug reports with reproduction steps

---

## The Three Gates (Tech-Lead)

### Gate A: Design-to-Spec (BEFORE coding)

**When:** Specification ready

**How to invoke:**
```
Prompt: "Using tech-lead-gates-skill, conduct Gate A approval.

Intended-Docs.md:
[paste spec]

Review against Gate A checklist:
1. Requirements testable? (not vague like 'should be fast')
2. Security explicit? (not implied)
3. Performance has numbers? (< 500ms, not 'quickly')
4. Database schema approved?
5. APIs specified?
6. Edge cases documented?
7. No AI hallucination triggers?

Approve with signature or request revision."
```

**What you get back:**
- ✅ Gate A Approved [Signature] OR
- ❌ Requests revision with specific issues

---

### Gate B: Drift & Test Audit (AFTER code review)

**When:** Code reviewed and tests passing

**How to invoke:**
```
Prompt: "Using tech-lead-gates-skill, conduct Gate B drift audit.

Design spec: [feature-name]-intended-docs.md:
[paste]

Actual code: [paste key files]

Tests: [paste test results]

1. Read actual code (not just test reports)
2. Spot-check for circular tests (tests mirror code flaw?)
3. Review drift.md differences from spec
4. Check for red flags:
   - 'Simplified implementation'?
   - 'Removed security check'?
   - Undocumented changes?
5. Verify circular tests caught and fixed

Approve with signature or request fixes."
```

**What you get back:**
- ✅ Gate B Approved [Signature] OR
- ❌ Issues to fix before Gate B approval

---

### Gate C: Production Deployment (BEFORE going live)

**When:** Staging tests all pass

**How to invoke:**
```
Prompt: "Using tech-lead-gates-skill, authorize Gate C production deployment.

Pre-deployment checklist:
- All tests passed? [Y/N]
- Error logs clean (last 24h)? [Y/N]
- Performance baseline met? [Y/N]
- Rollback procedure tested? [Y/N]
- Monitoring alerts configured? [Y/N]
- 30-min post-deploy ready? [Y/N]

Get SRE sign-off for dual authorization.

Approve with dual signature or hold deployment."
```

**What you get back:**
- ✅ Gate C Approved [Tech-Lead + SRE Signature] OR
- ❌ Issues blocking deployment

---

## Security Audit (Security-Reviewer)

**When:** Implementation complete

**How to invoke:**
```
Prompt: "Using security-audit-skill, conduct security audit of [feature].

Code files:
[paste implementation files]

Check:
- OWASP Top 10 vulnerabilities
- Hardcoded credentials?
- Input validation?
- Authorization checks?
- Sensitive data in responses?
- FLKitOver security (credentials, webhooks)?
- AWS service security (S3, SES, IAM)?
- Rate limiting?
- CSRF protection?
- SQL injection prevention?

Generate security report."
```

**What you get back:**
- ✅ Security approved OR
- ❌ Security issues to fix

---

## Real-World Example: Password Reset Feature

### Day 1: Business Requirement

```
You submit to tech-lead:
"Users should be able to reset forgotten passwords via email.
Reset link expires after 24 hours.
Rate limited to 5 attempts per hour.
Must work with FLKitOver document signing."

Tech-Lead responds:
"Using tech-lead-gates-skill, I'll design this..."
→ Creates: password-reset.md (technical design)
→ Gate A Approved ✓
```

### Day 1 Afternoon: Tech-Writer Creates Spec

```
You submit to tech-writer:
"Using tech-writer-spec-skill, create intended-docs.md for password reset.

Design: password-reset.md..."

Tech-Writer responds:
→ Creates: password-reset-intended-docs.md
→ Includes:
   - AC1: Valid reset link works
   - AC2: Expired link rejected
   - AC3: Rate limiting enforced (5/hour)
   - API: POST /api/password-reset with validation
   - Security: Hashed tokens, not plaintext
   - Performance: < 500ms response
   - Testing: 4 categories (positive, negative, non-functional, regression)
```

### Day 2: Senior-Engineer Implements

```
You submit to senior-engineer:
"Using senior-engineer-laravel-skill, implement password reset.

Spec: password-reset-intended-docs.md..."

Senior-Engineer responds:
→ Creates:
   - Migration: create_password_resets table (UUID, hashed_token, expires_at)
   - Model: PasswordReset (relationships, casts)
   - Service: PasswordResetService (composition)
   - Controller: PasswordResetController (thin)
   - Request: ResetPasswordRequest (validation)
   - Tests: Feature + Unit tests
   - PHPDoc: Comprehensive
→ Code quality checklist ✓
→ Auto-invokes code-reviewer
```

### Day 2 Afternoon: Code-Reviewer Reviews

```
Code-Reviewer loads code-reviewer-quality-skill:
→ Checks:
   - strict_types declared? ✓
   - Type hints on all functions? ✓
   - No SQL injection? ✓
   - Rate limiting middleware? ✓
   - Hashed tokens? ✓
   - PHPDoc complete? ✓
→ Approves or requests changes
```

### Day 3: QA-Tester Tests

```
You submit to qa-tester:
"Using qa-testing-skill, test password reset with all 4 categories."

QA-Tester responds:
→ Creates tests:
   1. Positive: Valid reset works
   2. Negative: Expired link rejected, rate limit enforced
   3. Non-functional: < 500ms response, no SQL injection
   4. Regression: Existing login still works
→ Executes Playwright E2E tests
→ Reports: All tests passed ✓
```

### Day 3 Evening: Tech-Lead Gates

```
Gate B: Tech-Lead audits code + tests
→ Reads actual code (not just test reports)
→ Spot-checks for circular tests
→ Reviews drift.md (no deviations)
→ Approves Gate B ✓

Gate C: After staging verification
→ Pre-deployment checklist ✓
→ 30-min post-deploy ready ✓
→ Gets SRE sign-off
→ Deploys to production ✓
```

### Day 4: Security Audit

```
You submit to security-reviewer:
"Using security-audit-skill, audit password reset security."

Security-Reviewer responds:
→ Checks:
   - Tokens hashed (not plaintext)? ✓
   - Rate limiting enforced? ✓
   - CSRF protected? ✓
   - No hardcoded credentials? ✓
   - FLKitOver integration secure? ✓
→ Security approved ✓
```

---

## Key Patterns

### Pattern 1: Agent Loads Skill

```
Your prompt: "Using [skill-name], [task]"
              └─ Agent sees this
              └─ Agent loads /skills/[skill-name]/SKILL.md
              └─ Agent reads entire skill file
              └─ Agent follows skill's guidance
```

### Pattern 2: Skill Provides Checklist

```
Agent loads skill → Skill has checklist
                  → Agent uses checklist to review
                  → Agent provides feedback based on checklist
                  → Agent approves/rejects with checklist reference
```

### Pattern 3: Hand-Off Between Agents

```
Tech-Lead (Gate A)
    ↓ Approves
Tech-Writer (Creates spec)
    ↓ Hands off: intended-docs.md
Senior-Engineer (Implements)
    ↓ Auto-invokes
Code-Reviewer (Reviews)
    ↓ Approves
QA-Tester (Tests)
    ↓ Reports
Tech-Lead (Gate B - Drift audit)
    ↓ Approves
Security-Reviewer (Security audit)
    ↓ Approves
Tech-Lead (Gate C - Deploy)
    ↓ Production deployment
```

---

## What NOT to Do

### ❌ Don't invoke agent without skill

```
❌ Wrong: "Review this code"
✅ Right: "Using code-reviewer-quality-skill, review this code"
```

Reason: Without skill reference, agent just reviews without checklist

### ❌ Don't skip gates

```
❌ Wrong: Tech-Lead approves code directly to production
✅ Right: Tech-Lead uses Gate A, Gate B, Gate C checkpoints
```

Reason: Gates catch issues early (spec → design → code → test → deploy)

### ❌ Don't skip test categories

```
❌ Wrong: QA-Tester only tests happy path
✅ Right: QA-Tester does all 4 categories (positive, negative, non-functional, regression)
```

Reason: Only happy path testing misses edge cases and security issues

### ❌ Don't assume agent follows all checklist

```
❌ Wrong: Assume agent checked everything
✅ Right: Explicitly ask agent to "check against [skill] checklist"
```

Reason: Makes it explicit what needs to be reviewed

---

## Summary

| Element | Role | Location | How Used |
|---------|------|----------|----------|
| **Agent** | Worker/Executor | `/claude_agents/agents/` | Invoke with `/invoke [agent-name]` |
| **Skill** | Training Manual | `/skills/[skill-name]/` | Reference in prompt: "Using [skill-name], ..." |
| **Gate** | Checkpoint | Tech-Lead Agent | Gate A, Gate B, Gate C |
| **Checklist** | Quality Standard | Inside Skill | Agent uses to review/approve |

---

## Your Next Steps

### Start with Phase 1: Design

```
Prompt to tech-lead:
"Using tech-lead-gates-skill, review this business requirement
for Gate A approval and create technical design.

Business Requirement:
[Your requirement here - be specific about what users need to do]"
```

The agent will:
1. Load tech-lead-gates-skill
2. Check if requirement is testable/specific
3. Ask questions if vague
4. Create technical design [feature-name].md
5. Approve for next phase

### Then Phase 2: Specification

```
Prompt to tech-writer:
"Using tech-writer-spec-skill, create intended-docs.md
from this design.

Design: [feature-name].md:
[paste from tech-lead approval]"
```

The agent will:
1. Load tech-writer-spec-skill
2. Create spec with testable AC, specific metrics
3. Prevent vague language
4. Document security and FLKitOver integration
5. Create [feature-name]-intended-docs.md

### Then Phase 3-5: Implementation → Review → Test → Gates → Deploy

---

## Questions?

**Q: Do I have to reference skill in prompt?**
A: Yes, explicit reference ensures agent loads it.

**Q: What if agent doesn't follow skill?**
A: Call agent out: "You didn't follow [skill] checklist. Please review against step #5..."

**Q: Can skills be updated?**
A: Yes! Update /skills/[skill-name]/SKILL.md and all agents get updated.

**Q: What if agent gets stuck?**
A: Skill provides guidance, but engineer must resolve. If stuck > 3 iterations, escalate to tech-lead.

**Q: Do I need all gates?**
A: Yes for production features. Gate A+B+C are mandatory for safety.

**Q: Can I skip QA testing?**
A: No, all 4 test categories required (positive, negative, non-functional, regression).

---

**Now you're ready to use Agents + Skills together effectively!**

Start with a business requirement → Tech-Lead Gate A → Follow the pipeline with skill guidance at each step.
