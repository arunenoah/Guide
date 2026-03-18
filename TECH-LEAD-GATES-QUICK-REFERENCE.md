# Tech-Lead: Three Mandatory Gates - Quick Reference

**Print this card and keep at your desk**

---

## GATE A: Design-to-Spec (BEFORE Coding Starts)

**When:** Tech-writer completes intended-docs.md
**Time:** 15-30 minutes
**Your Role:** APPROVE or REJECT spec (no ambiguity allowed)

### Review Checklist
```
[ ] Requirements clear & testable (AC1, AC2, not vague)
[ ] Security explicitly listed (rate limiting, CSRF, encryption)
[ ] Performance has numbers (< 500ms, not "fast")
[ ] Database schema changes approved
[ ] Third-party APIs clearly specified (SES, S3, Twilio)
[ ] Edge cases documented (expired tokens, errors)
[ ] No AI "hallucination" triggers (vague language)
```

### Rejection Examples
❌ "User should be happy" → Too vague
❌ "Email should send quickly" → No number
❌ "Fix security issue" → Which issue?

### Approval Examples
✅ "User receives email within 2 seconds"
✅ "Rate limit: max 5 attempts per hour per user"
✅ "Password hashed with bcrypt, not plaintext"

### Signature
```
Gate A Approved: _________________ Date: _______
                 (Tech-Lead Name)

DO NOT PROCEED without signature.
```

---

## GATE B: Drift & Test Audit (After Code Review, BEFORE QA Approval)

**When:** Code-reviewer approves + qa-tester finishes tests
**Time:** 45-60 minutes
**Your Role:** AUDIT drift.md and verify tests are valid (not circular)

### Part 1: Read Actual Code (Not Just Test Results)

```bash
# Clone feature branch
git checkout feature/password-reset
git pull

# Read the actual implementation (not test output)
cat app/Services/PasswordResetService.php
cat app/Http/Controllers/PasswordResetController.php

# Verify against Gate A spec
# Does code match intended-docs.md?
```

### Part 2: Spot-Check Circular Tests

**Problem:** Tests pass because they mirror code flaws

**Your Test:** Make 6 password reset requests in 1 hour
- [ ] 5th request succeeds? ✅
- [ ] 6th request fails with 429 (Too Many Requests)? ✅
- [ ] Error message clear? ✅
- [ ] NOT just checking "test exists" but "constraint enforced"? ✅

**If tests only check "feature exists" (not "constraint enforced"):** BLOCK MERGE

### Part 3: Review drift.md

**drift.md = Differences between intended-docs.md and actual implementation**

```
Example:

## Intended Design
- Email sent synchronously (user waits for confirmation)

## Actual Implementation
- Email sent asynchronously via queue (user doesn't wait)

Reason: Synchronous caused 5-second delay in testing

## Your Review
[ ] Is this change acceptable? YES / NO
[ ] Does it violate security? NO
[ ] Does it improve stability? YES
[ ] Is it documented? YES
```

**Red Flags (BLOCK MERGE):**
- ❌ drift.md shows "simplified implementation"
- ❌ "removed security check for performance"
- ❌ "skipped error handling to save time"
- ❌ "database schema change not mentioned in Gate A"

### Part 4: Red Flag Checklist

**Block merge if ANY are true:**
```
[ ] Tests only cover happy path (no error cases)
[ ] Security tests don't actually verify constraint
  Example: Test claims "Rate limiting works"
           But manually testing shows you CAN make 11 requests
[ ] drift.md undocumented changes
[ ] Performance tests assume empty database
[ ] No rollback plan for database migrations
[ ] Passwords stored plaintext (should be bcrypt)
[ ] API changes not documented
[ ] Third-party integrations not tested
```

### Signature
```
Gate B Approved: _________________ Date: _______
                 (Tech-Lead Name)

I have personally verified:
  [ ] Code matches intended-docs.md
  [ ] Tests are valid (not circular)
  [ ] drift.md reviewed and approved
  [ ] No red flags detected

DO NOT MERGE to staging without signature.
```

---

## GATE C: Staging-to-Production (BEFORE Deployment)

**When:** All staging tests pass + Playwright evidence generated
**Time:** 30-45 minutes
**Your Role:** AUTHORIZE production deployment (dual sign-off with SRE)

### Pre-Deployment Checklist
```
[ ] All tests passed in staging (screenshots attached)
[ ] No new error logs in staging (last 24 hours)
[ ] Performance baseline met (response times OK)
[ ] Rollback procedure tested and documented
[ ] On-call engineer notified and ready
[ ] Database migrations successful
[ ] Feature flag working (if used)
[ ] Third-party services confirmed working (SES, S3)
[ ] Monitoring alerts configured
```

### Playwright Evidence
```
[ ] Screenshots of key steps attached?
[ ] Video recording of flow working?
[ ] Browser console clean (no errors)?
[ ] Performance metrics captured?
```

### Post-Deploy Monitoring (You + SRE)
```
Immediate (30 minutes after deploy):
  [ ] Monitor error logs - any spikes?
  [ ] Check key performance metrics
  [ ] Manually test end-user workflow
  [ ] If issues found → ROLLBACK immediately
  [ ] If 30-min clean → Consider stable
```

### Signature
```
Gate C Approved: _________________ Date: _______
                 (Tech-Lead Name)

Gate C Authorized: ________________ Date: ______
                   (SRE/DevOps Name)

BOTH signatures required before production deployment.

Ready to monitor for 30 minutes post-deploy: YES / NO
```

---

## EMERGENCY: Rollback Authority

**You can trigger rollback immediately (no approval needed)**

### Auto-Rollback Triggers
```
ERROR RATE > 5% for 5 minutes → ROLLBACK
Database response time > 2 seconds → ROLLBACK
Third-party service failures (SES, S3) blocking feature → ROLLBACK
Security incident detected → ROLLBACK
Data corruption reported → ROLLBACK
```

### Rollback Procedure
```
1. Declare incident immediately
   "INCIDENT: Feature X rollback in progress"

2. Execute rollback script
   scripts/rollback.sh 2026-03-17-14-30

3. Monitor logs 10 minutes
   tail -f logs/production.log | grep ERROR

4. Confirm rollback succeeded
   If errors still present → CALL DEVOPS

5. Brief team
   "Feature X rolled back due to [reason]"
   "Root cause investigation in progress"
   "Re-deployment blocked until fixed"

6. DO NOT re-deploy same code
   Must fix in staging first
   Tech-Lead approval required before re-deploy
```

---

## Quick Decision Tree

```
Feature is ready for testing?
├─ YES → Run Gate B (Drift & Test Audit)
└─ NO → Send back to engineer

Tests pass in staging?
├─ YES → Run Gate C (Production Approval)
└─ NO → Run Gate B again (re-audit)

Ready for production?
├─ YES → Get dual signature, deploy
└─ NO → Send back with specific feedback

Production issues detected?
├─ CRITICAL → ROLLBACK immediately
├─ MINOR → Continue monitoring
└─ NONE → 30-min window clear, consider stable
```

---

## Time Budget

- **Gate A:** 15-30 min (new feature)
- **Gate B:** 45-60 min (after code review)
- **Gate C:** 30-45 min (before production)
- **Total per feature:** ~3 hours of tech-lead time

**Fast-Track (low-risk only):** 5 minutes (one review, no gates)

---

## Key Reminders

⚠️ **You are the final human authority** - AI agents work for you
⚠️ **Do not approve based on agent reports** - Verify yourself
⚠️ **If you're unsure, reject** - Better safe than compromised
⚠️ **Circular tests fool everyone** - Always spot-check manually
⚠️ **drift.md changes must be justified** - Demand explanations

---

**Print & Post This Card**
Keep at your desk for quick reference during code review
