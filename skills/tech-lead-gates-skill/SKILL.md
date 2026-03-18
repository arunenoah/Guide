---
name: tech-lead-gates-skill
description: Mandatory gate checkpoints for tech-lead approval (Gate A design, Gate B drift audit, Gate C production authorization) - generic for any project/API
---

# Tech-Lead Gates Skill

You are the technical lead making final approval decisions at three critical checkpoints.

## GATE A: Design-to-Spec Approval (BEFORE Coding Starts)

**When:** Tech-writer completes intended-docs.md
**Your Role:** APPROVE or REJECT spec (no ambiguity allowed)

### Review Checklist
- [ ] Requirements clear & testable (AC1, AC2, not vague like "should be fast")
- [ ] Security explicitly listed (rate limiting, CSRF, encryption, OAuth, MFA, etc.)
- [ ] Performance has numbers (< 500ms response time, not "fast")
- [ ] Database schema changes approved and documented
- [ ] Third-party APIs clearly specified (name, endpoints, authentication method)
- [ ] Edge cases documented (timeouts, network failures, rate limits, retries)
- [ ] No AI "hallucination" triggers (vague language, ambiguity, subjective terms)
- [ ] Migration strategy defined (backward compatibility, data migration)
- [ ] Third-party integration requirements clear (credentials, webhooks, error handling)
- [ ] External service requirements specified (authentication, error scenarios)

### Rejection Triggers - BLOCK if ANY are true:
- ❌ "User should be happy" → Too vague, reject for specificity
- ❌ "API should respond quickly" → No number, reject for metric
- ❌ "Integrate with third-party service" → Missing details (which? how? errors?)
- ❌ "Call external API" → No authentication, retry logic, or timeout specified
- ❌ Spec has contradictions between sections
- ❌ Third-party dependencies unspecified (name, endpoints, auth method, error handling)
- ❌ No clear acceptance criteria or error scenarios

### Approval Decision:
```
Gate A Approved: _________________ Date: _______
                 (Tech-Lead Name)

DO NOT PROCEED without signature.
Cannot approve ambiguous specs - demand revision.
```

---

## GATE B: Drift & Test Audit (After Code Review, BEFORE QA Approval)

**When:** Code-reviewer approves + qa-tester finishes tests
**Your Role:** AUDIT drift.md and verify tests are valid (not circular)

### Part 1: Read Actual Code (Not Just Test Reports)

```bash
# Clone feature branch
git checkout feature/feature-name
git pull

# Read the actual implementation (not test output)
cat app/Services/FeatureService.php
cat app/Http/Controllers/FeatureController.php
cat database/migrations/XXXX_create_feature_tables.php

# Verify against Gate A spec
# Does code match intended-docs.md?
```

### Part 2: Spot-Check for Circular Tests

**Problem:** Tests pass because they mirror code flaws exactly

**Circular Test Detection Checklist:**
- [ ] Tests check constraints are ENFORCED, not just "code exists"
- [ ] Example: Rate limiting test manually makes 6 requests
  - Request 5: succeeds ✅
  - Request 6: fails with 429 (Too Many Requests) ✅
  - Error message is clear? ✅
- [ ] NOT just "rate limit feature exists" but "constraint works"
- [ ] Security tests verify actual behavior, not code comments
- [ ] Edge cases tested manually, not just code path coverage
- [ ] Negative tests actually trigger error conditions

**Red Flag Tests (BLOCK MERGE if found):**
- ❌ Test only checks "feature exists" (not "constraint enforced")
- ❌ Test mocks the constraint (defeats purpose)
- ❌ Test duplicates code logic exactly (mirrors code flaw)
- ❌ Security tests don't verify actual security
  - Example: "Rate limiting works" test but manual testing shows you CAN make 11 requests
  - Example: "CSRF protected" test but manual testing bypasses it
- ❌ Performance tests assume empty database
- ❌ Happy path only (no error cases)

### Part 3: Review drift.md

**drift.md = Differences between intended-docs.md and actual implementation**

```
Example:

## Intended Design
- Email sent synchronously (user waits for confirmation)

## Actual Implementation
- Email sent asynchronously via queue (user doesn't wait)

Reason: Synchronous caused 5-second delay in testing, user experience suffered

## Your Review
[ ] Is this change acceptable? YES / NO
[ ] Does it violate security? NO
[ ] Does it improve stability? YES
[ ] Is it documented? YES
[ ] Could this break existing usage? NO
```

### Part 4: Red Flag Checklist

**Block merge if ANY of these are true:**
- [ ] drift.md shows "simplified implementation" (cut corners?)
- [ ] "removed security check for performance" (NO COMPROMISE)
- [ ] "skipped error handling to save time" (technical debt risk)
- [ ] "database schema change not mentioned in Gate A" (unauthorized scope change)
- [ ] Tests only cover happy path (no error cases)
- [ ] Security tests don't actually verify constraint enforcement
- [ ] Performance tests assume empty database (N+1 hidden by seed data)
- [ ] No rollback plan for database migrations
- [ ] Passwords stored plaintext (should be bcrypt)
- [ ] API changes not documented
- [ ] Third-party integrations not tested

### Approval Decision:
```
Gate B Approved: _________________ Date: _______
                 (Tech-Lead Name)

I have personally verified:
  [ ] Code matches intended-docs.md
  [ ] Tests are valid (not circular)
  [ ] drift.md reviewed and approved
  [ ] No red flags detected
  [ ] Security constraints actually enforced
  [ ] Circular tests caught and fixed

DO NOT MERGE to staging without signature.
```

---

## GATE C: Staging-to-Production (BEFORE Deployment)

**When:** All staging tests pass + Playwright evidence generated
**Time:** 30-45 minutes
**Your Role:** AUTHORIZE production deployment (dual sign-off with SRE/DevOps)

### Pre-Deployment Checklist
- [ ] All tests passed in staging (screenshots attached)
- [ ] No new error logs in staging (last 24 hours)
- [ ] Performance baseline met (response times within spec)
- [ ] Database migrations successful and reversible
- [ ] Rollback procedure tested and documented
- [ ] On-call engineer notified and ready
- [ ] Feature flag working (if used for gradual rollout)
- [ ] Third-party services confirmed working (SES, S3, Twilio, FLKitOver)
- [ ] Monitoring alerts configured and tested
- [ ] Security headers verified

### Playwright Evidence
- [ ] Screenshots of key steps attached to PR
- [ ] Video recording of complete flow working?
- [ ] Browser console clean (no errors, no warnings)
- [ ] Performance metrics captured (load times, API response times)
- [ ] FLKitOver integration verified end-to-end

### Post-Deploy Monitoring (You + SRE, 30 minutes)
```
Immediate (30 minutes after deploy):
  [ ] Monitor error logs - any spikes?
  [ ] Check key performance metrics (response time, throughput)
  [ ] Manually test end-user workflow from UI
  [ ] Verify third-party integrations working (SES email delivery, S3 uploads)
  [ ] If issues found → ROLLBACK immediately
  [ ] If 30-min clean → Consider stable
```

### Auto-Rollback Triggers (You Can Execute Immediately - No Approval Needed)
```
ERROR RATE > 5% for 5 minutes → ROLLBACK
Database response time > 2 seconds → ROLLBACK
Third-party service failures (SES, S3) blocking feature → ROLLBACK
Security incident detected → ROLLBACK
Data corruption reported → ROLLBACK
FLKitOver webhook failures → ROLLBACK
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

### Approval Decision:
```
Gate C Approved: _________________ Date: _______
                 (Tech-Lead Name)

Gate C Authorized: ________________ Date: ______
                   (SRE/DevOps Name)

BOTH signatures required before production deployment.

Ready to monitor for 30 minutes post-deploy: YES / NO
```

---

## Infrastructure Gate (For AWS/Database Changes)

**When:** Infrastructure-as-Code (IaC) changes proposed (Terraform, CloudFormation, migrations)

### Review Checklist
- [ ] IaC syntax correct and follows team standards
- [ ] Changes don't break existing functionality
- [ ] Backup/restore procedure documented
- [ ] Rollback procedure tested
- [ ] Cost impact calculated and approved
- [ ] Security group/IAM policy changes reviewed
- [ ] Database migration has backward compatibility plan
- [ ] Zero-downtime deployment possible
- [ ] Monitoring/alerts configured for new resources

### Dual Sign-Off Required:
- [ ] Tech-Lead approval (business & architecture impact)
- [ ] DevOps/SRE approval (infrastructure & operational impact)

---

## Key Reminders

⚠️ **You are the final human authority** - AI agents work for you, don't approve based on agent reports alone
⚠️ **Always verify yourself** - Read actual code, not just test summaries
⚠️ **If you're unsure, REJECT** - Better to revise spec than rush bad code to production
⚠️ **Circular tests fool everyone** - Spot-check manually that constraints are ENFORCED
⚠️ **drift.md changes must be justified** - Demand clear reasoning for deviations
⚠️ **Three gates are mandatory** - They exist to catch problems before they reach customers
