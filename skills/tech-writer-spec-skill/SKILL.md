---
name: tech-writer-spec-skill
description: Technical documentation standards for Laravel applications including intended specs, drift analysis, and final documentation
---

# Tech-Writer Specification Skill

You create clear, precise documentation that prevents AI hallucinations and ensures implementation accuracy.

## Core Documentation Principle

**Specifications must be testable, not subjective.**

### Anti-Pattern Language (NEVER Use These)
- ❌ "should be fast" → Use metric: "< 500ms response time"
- ❌ "user should be happy" → Use acceptance criteria: "completion in < 2 minutes"
- ❌ "intuitive interface" → Use specific design: "modal appears after button click"
- ❌ "optimize performance" → Use metric: "reduce database queries from N+1 to 2"
- ❌ "improve security" → Use specific control: "implement bcrypt password hashing"
- ❌ "handle errors gracefully" → Use specific behavior: "return 422 status with field errors"

### Pattern Language (ALWAYS Use These)
- ✅ "Response time < 500ms for 95th percentile"
- ✅ "User completes workflow in < 2 minutes"
- ✅ "Modal appears 200ms after button click"
- ✅ "Reduce database queries from 12 to 2 using eager loading"
- ✅ "Passwords hashed with bcrypt, never stored plaintext"
- ✅ "Return HTTP 422 with field-specific validation errors"

---

## Phase 1: Initial Documentation (intended-docs.md)

**When:** Receive tech-lead's feature design
**Deliverable:** [feature-name]-intended-docs.md

### Template: [feature-name]-intended-docs.md

```markdown
# Feature: [Feature Name] - Intended Design

## 1. Business Purpose & Overview
- Who uses this feature?
- Why do they need it?
- What problem does it solve?
- What are success metrics? (e.g., "reduce application processing time from 2 hours to 15 minutes")

## 2. Acceptance Criteria (Testable, Specific)

### AC1: Primary Workflow
- **Given:** [precondition]
- **When:** [user action]
- **Then:** [specific outcome]

### AC2: Error Handling
- **Given:** [invalid input]
- **When:** [user submits form]
- **Then:** [specific error response]

### AC3: Security Requirement
- **Given:** [security constraint]
- **When:** [threat scenario]
- **Then:** [specific mitigation]

(Use structured acceptance criteria, not narrative)

## 3. Technical Architecture

### Database Schema
```sql
CREATE TABLE features (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    status VARCHAR(50) DEFAULT 'pending',
    metadata JSON,
    created_at TIMESTAMP,

    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### Service Layer (Composition Pattern)
- FeatureService
  - Dependencies: FeatureModel, NotificationService, AwsS3Service
  - Methods: submitFeature(), updateStatus(), deleteFeature()

### API Endpoints
- POST /api/features - Create feature
- GET /api/features/{id} - Retrieve feature
- PUT /api/features/{id} - Update feature
- DELETE /api/features/{id} - Delete feature

### Request/Response Format
```json
// POST /api/features
{
  "property_id": "uuid",
  "name": "string (required, max 255)",
  "description": "string (optional)",
  "priority": "high|medium|low"
}

// Response 201
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "string",
    "status": "pending",
    "created_at": "2025-01-01T12:00:00Z"
  }
}
```

## 4. Security Requirements

### Authentication & Authorization
- Require user authentication for all endpoints
- Verify user owns resource before allowing updates
- Rate limit: max 10 requests per minute per user

### Input Validation
- name: required, string, max 255 characters
- priority: enum (high, medium, low)
- description: optional, max 2000 characters

### Data Protection
- Passwords hashed with bcrypt (never plaintext)
- Sensitive data encrypted at rest
- PII fields masked in logs

### FLKitOver Integration Security
- Use agent-specific credentials (not shared)
- Validate webhook signatures
- Never expose credentials in error messages

## 5. FLKitOver Integration

### Document Workflow
1. User submits feature request
2. System creates FLK document with credentials
3. FLK returns document ID
4. Webhook notifies when document signed
5. System updates feature status

### Integration Points
- Create document: POST /documents (FLK API)
- Get status: GET /documents/{id} (FLK API)
- Receive webhook: POST /api/webhook/flk-signature

## 6. AWS Service Integration

### S3 for File Storage
- Upload location: /features/{feature_id}/documents/
- File access: signed URLs (24-hour expiry)
- Privacy: private visibility (not public)

### SES for Email Notifications
- Template: feature_submitted
- Recipients: user email, admin email
- Content: feature details, action link

## 7. Performance Requirements

### Response Time Targets
- List features: < 200ms (< 1000 items)
- Create feature: < 500ms (includes S3 upload)
- Update status: < 100ms

### Database Optimization
- Index on user_id for user's features query
- Index on status for filtering
- No N+1 queries in list endpoint

## 8. Testing Strategy

### Unit Tests (Service Layer)
- FeatureService::submitFeature() - happy path
- FeatureService::submitFeature() - invalid data
- FeatureService::updateStatus() - authorization check

### Feature Tests (API Endpoints)
- POST /api/features - success case
- POST /api/features - validation errors
- GET /api/features/{id} - authorization check (can't view others' features)
- DELETE /api/features/{id} - owner can delete, non-owner cannot

### Integration Tests
- FLKitOver document creation
- SES email delivery
- S3 file upload and signed URL generation
- Webhook handling

### Non-Functional Tests
- Performance: Create 1000 features, list all < 500ms
- Security: Attempt to access user's features without auth (fails)
- Load: 100 concurrent requests to create feature

## 9. Migration Strategy

### Database Migration
- No breaking changes
- Backward compatible
- Rollback procedure: drop table (if new), revert columns (if modified)

### API Versioning
- New endpoints: /api/v2/features (if breaking)
- Backward compatibility: v1 endpoints still work for 6 months

## 10. Environment Variables Required
```env
FLK_API_URL=https://api.staging.flkitover.com
FLK_TIMEOUT=30
AWS_S3_BUCKET=feature-storage
AWS_SES_REGION=us-east-1
```

## 11. Dependencies
- Laravel 12
- PHP 8.2+
- MySQL UUID support
- AWS SDK for Laravel
- FLKitOver API access

## 12. Known Constraints & Assumptions
- FLKitOver API available (staging environment)
- S3 bucket pre-created with proper permissions
- SES sender identity verified
- Database connection stable
```

### Documentation Standards
- Use numbered headings for clarity
- Include code examples for all API endpoints
- Use SQL syntax highlighting for migrations
- Include request/response examples with status codes
- Be specific: "< 200ms" not "fast"

---

## Phase 2: Final Documentation & Drift Analysis

**When:** Implementation complete and tested
**Deliverables:** Updated README.md + drift.md

### Update README.md

Add new sections:
- New API endpoints with curl examples
- New environment variables
- New database tables
- Integration setup instructions

### Create drift.md Template

```markdown
# Feature Drift Analysis: [feature-name]

## Summary
- Total changes: X major, Y minor
- Overall alignment: High/Medium/Low
- Major deviations: [list]

## Detailed Drift Analysis

### DRIFT #1: [Section Name]
- **Intended:** [What was planned]
- **Actual:** [What was implemented]
- **Reason:** [Why it changed]
- **Impact:** [Positive/Negative/Neutral consequences]
- **Approval:** [Tech-lead approves this drift: YES/NO]

### DRIFT #2: Database Schema
- **Intended:** Simple features table with 5 columns
- **Actual:** Added audit_log_id column, json metadata column
- **Reason:** Requirements discovered during implementation
- **Impact:** POSITIVE - Enhanced audit capabilities, easier to extend
- **Approval:** YES - Improves observability

### DRIFT #3: API Response Format
- **Intended:** Simple flat JSON response
- **Actual:** Nested structure with relationships
- **Reason:** FLKitOver integration required additional data
- **Impact:** POSITIVE - Client has all needed data, fewer requests
- **Approval:** YES - Improves UX

## Unchanged Elements
- ✅ UUID architecture maintained
- ✅ Service layer composition pattern followed
- ✅ Security implementations (rate limiting, input validation) unchanged
- ✅ Laravel 12 and PHP 8.2+ used as planned

## Quality Improvements
- Enhanced error handling with custom exceptions
- Comprehensive PHPDoc documentation
- Test coverage: 87% (exceeds 80% requirement)
- Performance: API response < 150ms (exceeds < 200ms target)

## Tech-Lead Approval
All drifts reviewed and approved: _________________ Date: _______

Cannot proceed to production without drift.md signature.
```

### Drift Analysis Checklist
- [ ] Compare intended-docs.md to actual code
- [ ] List each difference with reason
- [ ] Assess impact (positive/negative)
- [ ] Document all database schema changes
- [ ] Document all API changes
- [ ] Document all FLKitOver integration changes
- [ ] Get tech-lead to approve each drift

---

## Quality Standards for All Documentation

### Clarity Checklist
- [ ] No subjective language ("should be fast" → "< 200ms")
- [ ] All metrics specific with numbers
- [ ] All workflows include edge cases
- [ ] All APIs have example requests/responses
- [ ] Code examples compile and run
- [ ] No ambiguous requirements (like "handle errors")

### Completeness Checklist
- [ ] Database schema defined (SQL)
- [ ] All endpoints documented with validation rules
- [ ] Security requirements explicit
- [ ] Performance targets with numbers
- [ ] Testing strategy for all scenarios
- [ ] Environment variables listed
- [ ] FLKitOver integration steps clear
- [ ] AWS service usage documented

### Reviewability Checklist
- [ ] Tech-lead can understand without asking questions
- [ ] Engineer can implement without ambiguity
- [ ] QA can test all acceptance criteria
- [ ] Code reviewer can verify against spec

---

## Common Documentation Mistakes (Avoid These)

❌ **Mistake:** "The system should be scalable"
✅ **Fix:** "Support 10,000 concurrent users with < 500ms response time"

❌ **Mistake:** "Improve error handling"
✅ **Fix:** "Return HTTP 422 with field-specific validation errors: {'email': ['Invalid format']}"

❌ **Mistake:** "Integrate with FLKitOver"
✅ **Fix:** "POST to FLK /documents endpoint with user credentials, store returned document_id, verify webhook signature, update feature status to 'signed'"

❌ **Mistake:** "Test thoroughly"
✅ **Fix:** "Unit tests: 100% coverage of PetApplicationService. Feature tests: all CRUD operations. Integration tests: FLK document flow, S3 uploads, SES delivery. Performance test: 1000 requests < 2 seconds"

---

## Key Reminders

⚠️ **Specifications prevent hallucinations** - Vague specs → AI makes assumptions → wrong implementation
⚠️ **Testable specs only** - If you can't test it, don't write it
⚠️ **Numbers not adjectives** - "< 200ms" not "fast"
⚠️ **Edge cases matter** - Spec what happens when things fail
⚠️ **Drift.md is mandatory** - Documents are never perfect, track what actually changed
