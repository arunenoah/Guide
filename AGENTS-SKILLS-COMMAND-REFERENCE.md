# Agents + Skills - Command Reference

**Copy-paste ready prompts for each phase**

---

## PHASE 1: Design (Tech-Lead Gate A)

### Command Syntax
```
/invoke tech-lead "Using tech-lead-gates-skill, [task]"
```

### Copy-Paste Template

```
/invoke tech-lead "Using tech-lead-gates-skill, review this business
requirement for Gate A approval and create technical design.

Business Requirement:
[Paste your requirement here]

Tech-Lead should:
1. Check if requirement is testable (not 'should be fast')
2. Verify security is explicit (not implied)
3. Check API dependencies are specified
4. Document edge cases
5. Approve with signature OR request revision"
```

### Real Example

```
/invoke tech-lead "Using tech-lead-gates-skill, review this business
requirement for Gate A approval and create technical design.

Business Requirement:
Users need to reset forgotten passwords via email.
- Reset link should expire after 24 hours
- Rate limited to 5 attempts per hour per user
- Must integrate with FLKitOver for document signing
- Should work across web and mobile

Tech-Lead should:
1. Check if requirement is testable
2. Verify security is explicit
3. Check APIs specified
4. Document edge cases (expired links, rate limit edge case at hour boundary)
5. Create password-reset.md design
6. Approve Gate A with signature"
```

### Expected Output
```
✅ Gate A Approved

- Requirement is testable ✓
- Security explicit (FLKitOver integration, rate limiting) ✓
- APIs specified ✓
- Edge cases documented ✓

Approved by: Tech-Lead   Date: 2026-03-17

[feature-name].md design document created
Ready for Phase 2: Tech-Writer Specification
```

---

## PHASE 2: Specification (Tech-Writer)

### Command Syntax
```
/invoke tech-writer "Using tech-writer-spec-skill, create intended-docs"
```

### Copy-Paste Template

```
/invoke tech-writer "Using tech-writer-spec-skill, create [feature-name]-intended-docs.md
from this design document.

Design Document:
[Paste tech-lead's [feature-name].md]

Tech-Writer should create comprehensive specification with:
1. Business purpose (why users need this)
2. Testable acceptance criteria (AC1, AC2, AC3 format)
3. Technical architecture (models, services, controllers)
4. Security requirements (explicit constraints)
5. Database schema (UUID keys, relationships)
6. API endpoints (with validation rules)
7. FLKitOver integration (if applicable)
8. AWS service usage (S3, SES)
9. Performance requirements (with numbers: < 200ms, not 'fast')
10. Testing strategy (all 4 categories)
11. Migration plan
12. Environment variables needed

Prevent vague language:
- '< 500ms' not 'fast'
- 'max 5 per hour' not 'rate limited'
- 'bcrypt hashed' not 'secure'"
```

### Real Example

```
/invoke tech-writer "Using tech-writer-spec-skill, create password-reset-intended-docs.md
from this design document.

Design Document (password-reset.md):
# Password Reset Feature

## Overview
Users who forgot their password can reset via email link.

## Technical Approach
- Use Laravel password reset tokens
- Store hashed tokens in database
- Rate limit to 5 attempts per hour
- Integrate with FLKitOver for document signing
- Email delivery via SES
- 24-hour link expiry

## Database Changes
- password_resets table (user_id, token, expires_at)
- Add reset_requested_at to users table

Tech-Writer should create comprehensive password-reset-intended-docs.md with:
1. Testable acceptance criteria (AC1, AC2, AC3)
2. API: POST /api/password-reset with validation rules
3. Security: token hashing, rate limiting mechanism
4. Performance: < 500ms response time
5. All 4 test categories (positive, negative, non-functional, regression)"
```

### Expected Output
```
✅ [feature-name]-intended-docs.md Created

Includes:
- Business purpose ✓
- Testable AC (AC1, AC2, AC3) ✓
- Technical architecture ✓
- Security requirements ✓
- API endpoints with validation ✓
- Database schema (UUID) ✓
- Performance targets (with numbers) ✓
- Testing strategy (4 categories) ✓
- Environment variables ✓

Ready for Phase 3: Senior-Engineer Implementation
```

---

## PHASE 3: Implementation (Senior-Engineer)

### Command Syntax
```
/invoke senior-engineer "Using senior-engineer-laravel-skill, implement"
```

### Copy-Paste Template

```
/invoke senior-engineer "Using senior-engineer-laravel-skill, implement [feature-name].

Specification (intended-docs.md):
[Paste the intended-docs.md from tech-writer]

Implementation should follow skill patterns:
1. Migrations (UUID keys, relationships, indexes)
2. Models (HasUuids, relationships, casts)
3. Services (composition pattern, dependency injection)
4. Controllers (thin - no business logic)
5. Form Requests (validation rules)
6. Tests (unit, feature, integration)
7. PHPDoc documentation (all classes/methods)
8. Strict typing (declare(strict_types=1);)

Code Quality Checklist (before finishing):
- [ ] All files have declare(strict_types=1);
- [ ] All functions have type hints (parameters + return)
- [ ] All methods have PHPDoc with @param/@return/@throws
- [ ] No debug functions (die, var_dump, print_r)
- [ ] All exceptions caught with specific types
- [ ] Eloquent used (no raw SQL)
- [ ] UUID primary keys (not numeric)
- [ ] Services use constructor DI
- [ ] Controllers are thin
- [ ] Form Requests validate input
- [ ] Tests cover happy path + error cases
- [ ] No N+1 queries

When complete:
1. Run code quality checklist
2. Auto-invoke code-reviewer"
```

### Real Example

```
/invoke senior-engineer "Using senior-engineer-laravel-skill, implement password-reset.

Specification (password-reset-intended-docs.md):
# Password Reset Feature - Intended Design

## AC1: User can request password reset via email
Given: Registered user
When: POST /api/password-reset with valid email
Then: Email sent within 2 seconds with reset link

## AC2: Reset link expires after 24 hours
Given: Email with reset link sent
When: User clicks link after 24+ hours
Then: Return 410 Gone with 'Link expired' message

## AC3: Rate limited to 5 attempts per hour
Given: User attempts password reset 6 times in 60 minutes
When: 6th request submitted
Then: Return 429 Too Many Requests

[paste full intended-docs.md]

Implementation should follow skill patterns:
1. Create migration: create_password_resets table
2. Create PasswordReset model (relationships)
3. Create PasswordResetService (composition, DI)
4. Create PasswordResetController (thin)
5. Create ResetPasswordRequest (validation)
6. Create tests (feature, unit)

When complete:
1. Run code quality checklist
2. Auto-invoke code-reviewer"
```

### Expected Output
```
✅ Implementation Complete

Files created:
- database/migrations/XXXX_create_password_resets_table.php
- app/Models/PasswordReset.php
- app/Services/PasswordResetService.php
- app/Http/Controllers/PasswordResetController.php
- app/Http/Requests/ResetPasswordRequest.php
- tests/Feature/PasswordResetTest.php
- tests/Unit/PasswordResetServiceTest.php

Code Quality Checklist: ✓ PASSED
- All files: declare(strict_types=1) ✓
- All functions: type hints ✓
- All methods: PHPDoc ✓
- No debug code ✓
- Exceptions handled ✓
- Eloquent used ✓
- UUID keys ✓
- DI in services ✓
- Thin controllers ✓
- Tests coverage ✓

Code-Reviewer auto-invoked for Phase 4
```

---

## PHASE 4: Code Review (Code-Reviewer)

### Command Syntax
```
/invoke code-reviewer "Using code-reviewer-quality-skill, review"
```

### Copy-Paste Template

```
/invoke code-reviewer "Using code-reviewer-quality-skill, review this PR thoroughly.

[Paste code files OR PR diff]

Code-Reviewer should check:

CRITICAL ISSUES (Blocking merge if found):
1. Missing declare(strict_types=1);
2. Logic errors or bugs
3. Security vulnerabilities (injection, auth bypass, hardcoded secrets)
4. Performance bottlenecks (N+1 queries, missing indexes)
5. Missing error handling
6. Architecture deviation
7. Breaking changes without migration
8. Debug functions left (die, var_dump, print_r)
9. Missing/improper UUID usage
10. Sensitive data exposed in responses

HIGH PRIORITY ISSUES (Should fix):
1. Missing type hints
2. Incomplete PHPDoc
3. Not following Laravel conventions
4. Missing database relationships
5. No input validation
6. Missing exception handling
7. Improper dependency injection
8. Missing database transactions

MEDIUM PRIORITY (Suggestions):
1. Code duplication
2. Large methods (>50 lines)
3. Missing test coverage
4. Inconsistent error responses
5. Missing audit logging

Provide feedback with file:line references.
Approve when all critical/high issues resolved."
```

### Real Example

```
/invoke code-reviewer "Using code-reviewer-quality-skill, review password-reset PR.

[Paste password-reset files]

Code-Reviewer should check against checklist:

CRITICAL ISSUES to block:
- declare(strict_types=1); in all files?
- Type hints on all functions?
- Security: rate limiting implemented?
- Security: tokens hashed (not plaintext)?
- FLKitOver integration error handling?
- No SQL injection?

HIGH PRIORITY:
- PHPDoc complete?
- Form Request validation rules?
- Service DI in constructor?
- Exception handling specific (not generic)?

Provide specific feedback with file:line.
Approve when critical/high issues resolved."
```

### Expected Output
```
✅ Code Review Complete

Critical Issues: 0 ❌
High Priority Issues: 0 ❌
Medium Priority Suggestions: 2 (optional fixes)

✅ CODE APPROVED

Code-Reviewer signature: [Agent Name]
Date: 2026-03-17

Ready for Phase 5: QA Testing
```

---

## PHASE 5: Testing (QA-Tester)

### Command Syntax
```
/invoke qa-tester "Using qa-testing-skill, test [feature]"
```

### Copy-Paste Template

```
/invoke qa-tester "Using qa-testing-skill, create comprehensive test plan for [feature-name].

Specification (intended-docs.md):
[Paste the intended-docs.md]

QA-Tester should create tests for ALL 4 CATEGORIES (mandatory):

1. POSITIVE TESTS (Happy Path)
   - User completes workflow correctly
   - Example: User requests password reset → receives email → resets password

2. NEGATIVE TESTS (Error Cases)
   - Invalid email validation
   - Expired link (> 24 hours)
   - Rate limiting (6th attempt in 1 hour)
   - SQL injection attempts
   - CSRF protection
   - Unicode/special characters
   - Whitespace handling

3. NON-FUNCTIONAL TESTS
   - Performance: Response < 500ms
   - Security: Tokens hashed (not plaintext)
   - Load: 100 concurrent requests
   - Accessibility: Keyboard navigation

4. REGRESSION TESTS
   - Existing login still works
   - Existing email delivery still works
   - Backward compatibility

Also write:
- Playwright E2E tests for UI workflow
- Test execution commands
- Coverage report

Execute all tests.
Report: Passed ✓ OR list bugs with reproduction steps"
```

### Real Example

```
/invoke qa-tester "Using qa-testing-skill, test password-reset feature.

Specification (password-reset-intended-docs.md):
[Paste full spec]

QA-Tester should test:

POSITIVE:
- POST /api/password-reset with valid email → 200 OK + email sent ✓
- Click email link → password reset form displays ✓
- Reset password → login works with new password ✓

NEGATIVE:
- POST /api/password-reset with invalid email → 422 validation error ✓
- Click reset link after 24 hours → 410 Gone ✓
- 6th reset request in 1 hour → 429 Too Many Requests ✓
- SQL injection attempt in email field → 422 validation error ✓

NON-FUNCTIONAL:
- Response time < 500ms ✓
- Tokens are hashed (check DB, not plaintext) ✓
- Load: 100 concurrent requests complete in < 10s ✓

REGRESSION:
- Existing login flow still works ✓
- Existing email templates still work ✓

Write Playwright E2E tests:
- User flows through password reset form
- Error messages display correctly
- Responsive on mobile

Execute all tests."
```

### Expected Output
```
✅ All Tests Passed

Test Results:
- Positive tests: 5/5 passed ✓
- Negative tests: 8/8 passed ✓
- Non-functional: 4/4 passed ✓
- Regression tests: 3/3 passed ✓
- Playwright E2E: 4/4 passed ✓

Coverage: 87% (exceeds 80% requirement)

All 4 test categories completed: ✓

QA-Tester signature: [Agent Name]
Date: 2026-03-17

Ready for Gate B: Tech-Lead Drift Audit
```

---

## GATE B: Drift & Test Audit (Tech-Lead)

### Command Syntax
```
/invoke tech-lead "Using tech-lead-gates-skill, conduct Gate B"
```

### Copy-Paste Template

```
/invoke tech-lead "Using tech-lead-gates-skill, conduct Gate B drift audit.

Intended Specification:
[Paste password-reset-intended-docs.md]

Actual Implementation:
[Paste key files or code summary]

Test Results:
[Paste test results summary]

Tech-Lead should:
1. READ ACTUAL CODE (not just test reports)
2. SPOT-CHECK for circular tests
   - Tests that pass because they mirror code flaw?
   - Example: If code has bug, test also has bug?
3. REVIEW drift.md
   - What changed from intended spec?
   - Is change justified?
   - Any 'simplified implementation'?
   - Any 'removed security check'?
4. CHECK red flags
   - Are security constraints ENFORCED (not just coded)?
   - Is rate limiting actually working?
   - Is token hashing actually in DB?
   - Can you manually reproduce test scenarios?
5. APPROVE with signature OR request fixes"
```

### Real Example

```
/invoke tech-lead "Using tech-lead-gates-skill, conduct Gate B drift audit.

Intended-docs.md:
[Paste]

Actual code (password-reset files):
[Paste]

Test results:
[Paste test results]

Tech-Lead review:
1. I read the actual code in PasswordResetService.php
2. I spot-checked: Do tests actually verify rate limiting?
   - Manual test: Can I make 6 requests in 60 minutes?
   - Check: Is request #6 actually rejected with 429?
3. I review drift.md (any differences from spec?)
4. Red flags check:
   - Are tokens hashed in database? (CHECK: don't just trust code)
   - Does rate limiting actually work? (TEST: manually)
   - Are errors not exposing details? (CHECK: error messages)
5. APPROVE or REQUEST FIXES with signature"
```

### Expected Output
```
✅ Gate B Approved

Code matches intended-docs.md: ✓
Circular tests: None detected ✓
drift.md: Reviewed and acceptable ✓
Red flags: None found ✓

Tech-Lead Gate B Approval:
Signed: Tech-Lead Name
Date: 2026-03-17

Ready for Security Audit
```

---

## Security Audit (Security-Reviewer)

### Command Syntax
```
/invoke security-reviewer "Using security-audit-skill, audit"
```

### Copy-Paste Template

```
/invoke security-reviewer "Using security-audit-skill, conduct security audit of [feature-name].

Code Files:
[Paste implementation files]

Security-Reviewer should check:

OWASP TOP 10:
1. Broken Auth: Passwords hashed? Sessions secure? MFA?
2. Broken Access Control: Authorization checks? IDOR?
3. Injection: Eloquent used? No raw SQL?
4. Insecure Design: Rate limiting? Security explicit?
5. Sensitive Data: No passwords in API responses?
6. IDOR: Object-level authorization? Can't access others' data?
7. XSS: Blade auto-escapes? Dangerous characters escaped?
8. Insecure Deserialization: JSON, not unserialize()?
9. Vulnerable Components: composer audit passed?
10. Insufficient Logging: Security events tracked?

FLKitOver Security:
- Credentials in env vars (not hardcoded)?
- Webhook signatures verified?
- Errors don't expose API details?

AWS Security:
- S3 files private by default?
- SES credentials in env vars?

Generate security report."
```

### Real Example

```
/invoke security-reviewer "Using security-audit-skill, audit password-reset security.

Code:
[Paste password-reset files]

Security-Reviewer should check:

1. OWASP:
   - Passwords hashed with bcrypt? (not plaintext, not MD5)
   - Reset tokens hashed? (not plaintext in DB)
   - Rate limiting enforced? (5/hour working)
   - CSRF protected? (token in forms)
   - Input validated? (Form Request rules)
   - SQL injection prevented? (Eloquent, not raw)

2. Logging:
   - Failed reset attempts logged?
   - No sensitive data in logs?

3. Error handling:
   - Generic error messages? (not exposing internal details)

Generate security report with approval."
```

### Expected Output
```
✅ Security Audit Complete

OWASP Top 10: ✓ COMPLIANT
- No critical vulnerabilities
- All security controls in place

FLKitOver: ✓ SECURE
- Credentials in env vars
- Webhooks verified
- Errors sanitized

AWS: ✓ COMPLIANT
- S3 private by default
- SES credentials secure

Security Audit Approved: [Agent Name]
Date: 2026-03-17

Ready for Gate C: Production Deployment
```

---

## GATE C: Production Deployment (Tech-Lead + SRE)

### Command Syntax
```
/invoke tech-lead "Using tech-lead-gates-skill, authorize Gate C deployment"
```

### Copy-Paste Template

```
/invoke tech-lead "Using tech-lead-gates-skill, authorize Gate C production deployment.

Deployment Checklist:
- [ ] All tests passed in staging
- [ ] Error logs clean (last 24 hours)
- [ ] Performance baseline met
- [ ] Rollback procedure tested
- [ ] Monitoring alerts configured
- [ ] Security audit passed
- [ ] SRE ready to monitor

Tech-Lead and SRE:
1. Verify pre-deployment checklist ✓
2. Tech-Lead Gate C signature
3. SRE/DevOps signature
4. Deploy to production
5. Monitor 30-minute window
6. Auto-rollback if:
   - Error rate > 5%
   - Response time > 2s
   - Third-party service failure
7. Post-deploy verification"
```

### Real Example

```
/invoke tech-lead "Using tech-lead-gates-skill, authorize Gate C deployment.

Deployment Status:
✓ All tests passed
✓ Error logs clean
✓ Performance OK (avg 120ms)
✓ Rollback tested (scripts/rollback.sh password-reset)
✓ Monitoring: Error rate, response time, email delivery
✓ Security audit passed
✓ SRE confirmed ready

Tech-Lead Gate C Approval:
Signed: Tech-Lead Name
Date: 2026-03-17

SRE/DevOps Authorization:
Signed: SRE Name
Date: 2026-03-17

✅ Authorized for production deployment

Deploying to production...
Monitoring 30-minute window...
✓ No errors
✓ Performance stable
✓ All systems healthy

✅ Deployment successful and stable"
```

---

## Summary: Copy-Paste Commands

### Phase 1: Tech-Lead Design
```
/invoke tech-lead "Using tech-lead-gates-skill, review this business requirement for Gate A approval. [requirement]"
```

### Phase 2: Tech-Writer Specification
```
/invoke tech-writer "Using tech-writer-spec-skill, create intended-docs.md from this design. [design doc]"
```

### Phase 3: Senior-Engineer Implementation
```
/invoke senior-engineer "Using senior-engineer-laravel-skill, implement [feature] from spec. [spec]"
```

### Phase 4: Code-Reviewer Quality
```
/invoke code-reviewer "Using code-reviewer-quality-skill, review this PR thoroughly. [code]"
```

### Phase 5: QA-Tester Testing
```
/invoke qa-tester "Using qa-testing-skill, test [feature] with all 4 categories. [spec]"
```

### Gate B: Tech-Lead Drift Audit
```
/invoke tech-lead "Using tech-lead-gates-skill, conduct Gate B drift audit. [spec + code]"
```

### Security Audit
```
/invoke security-reviewer "Using security-audit-skill, audit [feature] security. [code]"
```

### Gate C: Tech-Lead Production Authorization
```
/invoke tech-lead "Using tech-lead-gates-skill, authorize Gate C deployment. [checklist]"
```

---

**That's it! Use these commands to build production-ready features with quality gates and comprehensive testing.**
