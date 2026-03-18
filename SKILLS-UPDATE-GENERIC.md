# Skills Update - Generic for ANY Project/API

**Status:** Update existing skills to be generic (not FLKitOver or PetApplication specific)

---

## Changes Required

### 1. Replace Project-Specific Names

**Before (Specific):**
```
FLKitOver, PetApplication, password-reset, PasswordResetService,
PasswordResetController, password_resets table, pet_applications,
tenant_name, pet_type, general_consent
```

**After (Generic):**
```
[Third-Party API Name], [Feature Name], [FeatureService], [FeatureController],
[feature]_[resource] table, [resource], relevant_field_1, relevant_field_2,
user_consent
```

---

### 2. Replace API-Specific Flows

**FLKitOver-Specific Flow:**
```
❌ "User submits feature request
   System creates FLK document with credentials
   FLK returns document ID
   Webhook notifies when document signed
   System updates feature status"
```

**Generic Third-Party API Flow:**
```
✅ "User submits feature request
   System calls third-party API ([API Name]) with authentication
   API returns response with ID
   System handles webhook notifications (if applicable)
   System updates feature status based on API response"
```

---

### 3. Replace AWS-Specific Services

**Specific:**
```
AWS S3 for file storage
AWS SES for email delivery
Twilio for SMS notifications
```

**Generic:**
```
External file storage service (S3, Cloudinary, etc.)
Email delivery service (SES, SendGrid, Mailgun, etc.)
SMS service (Twilio, Vonage, AWS SNS, etc.)
```

---

## Skills to Update

### 1. tech-writer-spec-skill/SKILL.md

**Update sections:**
- Remove "FLKitOver Integration" section
- Replace with generic "Third-Party API Integration" section
- Replace examples:
  - ❌ `"FLKitOver integration required"`
  - ✅ `"[Third-party API name] integration required"`

**Changes:**
```markdown
❌ BEFORE:
## FLKitOver Integration
1. User submits feature request
2. System creates FLK document with credentials
3. FLK returns document ID
4. Webhook notifies when document signed
5. System updates feature status

✅ AFTER:
## Third-Party API Integration

### [API Name] Integration
1. User submits feature request
2. System calls [API Name] endpoint
3. API returns response with data
4. System handles API response (success/error)
5. System handles webhooks (if applicable)
6. System updates feature status

### Authentication
- Method: [OAuth 2.0 / API Key / Basic Auth / Bearer Token]
- Credentials: Environment variables (never hardcoded)

### Error Handling
- Timeout: [specify seconds]
- Retries: [specify count and backoff strategy]
- Rate limiting: [specify limits]
- Error scenarios: [list expected errors and handling]

### Webhook Handling (if applicable)
- Signature verification: [HMAC-SHA256 or similar]
- Timeout handling: [retry logic]
- Idempotency: [ensure duplicate webhooks don't break system]
```

---

### 2. senior-engineer-laravel-skill/SKILL.md

**Update sections:**
- Remove "FLKitOver Integration Pattern" code examples
- Replace with generic "Third-Party API Integration Pattern"
- Remove "PetApplicationService" example
- Replace with generic "[FeatureName]Service" example

**Changes:**
```php
❌ BEFORE:
// FLKitOver Integration Pattern
public function __construct(
    private readonly FLKService $flkService,
    private readonly EmailNotificationService $emailService,
    private readonly AwsS3Service $s3Service
) {}

public function submitApplication(array $data): PetApplication
{
    // Create FLK document
    $flkDocument = $this->flkService->createDocument($data);
    // ...
}

✅ AFTER:
// Third-Party API Integration Pattern
public function __construct(
    private readonly ThirdPartyApiService $apiService,
    private readonly NotificationService $notificationService,
    private readonly StorageService $storageService
) {}

public function submitRequest(array $data): Resource
{
    try {
        // Call third-party API
        $apiResponse = $this->apiService->create($data);

        // Handle response
        if (!$apiResponse['success']) {
            throw new ThirdPartyApiException('API request failed');
        }

        // Store result
        $resource = $this->storeResult($apiResponse);
        return $resource;

    } catch (ThirdPartyApiException $e) {
        Log::error('Third-party API error', [
            'error' => $e->getMessage(),
            'api_name' => 'Third-Party API',
            'request_data' => $data
        ]);
        throw new RequestException('Failed to process request', 0, $e);
    }
}
```

---

### 3. code-reviewer-quality-skill/SKILL.md

**Update sections:**
- Remove "FLKitOver Integration Security" section
- Replace with generic "Third-Party API Integration Security" section

**Changes:**
```
❌ BEFORE:
## FLKitOver Integration Security

// VULNERABLE - Hardcoded credentials
$response = Http::post('https://api.flkitover.com/documents', [
    'auth' => ['dev@company.com', 'password123']
]);

// SECURE - Proper credential handling
class FLKService {
    public function __construct(
        private readonly string $apiUrl,
        private readonly string $email,
        private readonly string $password
    ) {}
}

✅ AFTER:
## Third-Party API Integration Security

Check for each external API:
- [ ] Credentials in environment variables (not hardcoded)
- [ ] API keys/secrets never logged
- [ ] Proper authentication method (OAuth, Bearer token, API key)
- [ ] Timeout specified (connection + read)
- [ ] Retry logic with exponential backoff
- [ ] Error handling catches API-specific exceptions
- [ ] Webhook signatures verified (if applicable)
- [ ] Rate limiting respected
- [ ] Error responses don't expose API details

// VULNERABLE - Hardcoded credentials
$response = Http::post('https://api.example.com/create', [
    'auth' => ['user@company.com', 'secret123']
]);

// SECURE - Proper credential handling
class ExternalApiService {
    public function __construct(
        private readonly string $apiUrl,
        private readonly string $apiKey,
        private readonly int $timeout = 30
    ) {}

    public function create(array $data): array {
        return Http::timeout($this->timeout)
                   ->withToken($this->apiKey)
                   ->post($this->apiUrl . '/create', $data)
                   ->json();
    }
}
```

---

### 4. qa-testing-skill/SKILL.md

**Update sections:**
- Remove "FLKitOver Integration Testing" examples
- Replace with generic "Third-Party API Testing" patterns
- Replace "PetApplicationTest" with "[FeatureName]Test"

**Changes:**
```php
❌ BEFORE:
## FLKitOver Integration Testing

public function test_flk_document_creation_workflow(): void {
    $this->mock(FLKService::class, function ($mock) {
        $mock->shouldReceive('createDocument')
             ->once()
             ->andReturn(['document_id' => 'test-doc-id']);
    });
}

✅ AFTER:
## Third-Party API Integration Testing

Test every external API call:

1. MOCKING (Unit Tests - test your code, not API)
public function test_api_call_successful(): void {
    // Mock the third-party API response
    $this->mock(ExternalApiService::class, function ($mock) {
        $mock->shouldReceive('create')
             ->once()
             ->with($this->validData())
             ->andReturn(['success' => true, 'id' => 'api-123']);
    });

    $result = $this->service->submit($this->validData());
    $this->assertEquals('api-123', $result->api_id);
}

2. ERROR HANDLING (Unit Tests - API failures)
public function test_api_timeout_handled(): void {
    $this->mock(ExternalApiService::class, function ($mock) {
        $mock->shouldReceive('create')
             ->once()
             ->andThrow(new TimeoutException('API timeout'));
    });

    $this->expectException(RequestException::class);
    $this->service->submit($this->validData());
}

3. INTEGRATION TESTS (Optional - if staging API available)
public function test_api_integration_with_staging(): void {
    $this->skipIfNoStagingApiAccess();

    $response = $this->apiService->create([
        'name' => 'Test',
        'email' => 'test@example.com'
    ]);

    $this->assertTrue($response['success']);
    $this->assertNotEmpty($response['id']);
}

4. WEBHOOK TESTING (if applicable)
public function test_webhook_signature_verification(): void {
    $payload = ['event' => 'completed', 'id' => '123'];
    $signature = hash_hmac('sha256', json_encode($payload), config('api.webhook_secret'));

    $response = $this->post('/webhook', $payload, [
        'X-API-Signature' => $signature
    ]);

    $response->assertStatus(200);
}

5. RETRY LOGIC (if applicable)
public function test_api_retry_on_temporary_failure(): void {
    $apiMock = $this->mock(ExternalApiService::class);
    $apiMock->shouldReceive('create')
            ->times(3)  // Fails 2x, succeeds on 3rd
            ->andReturnValues([
                throw new TemporaryException('Temporary failure'),
                throw new TemporaryException('Temporary failure'),
                ['success' => true, 'id' => '123']
            ]);

    $result = $this->service->submitWithRetry($this->validData());
    $this->assertEquals('123', $result->api_id);
}
```

---

### 5. security-audit-skill/SKILL.md

**Update sections:**
- Remove "FLKitOver Integration Security" section
- Replace with generic "Third-Party API Security Checklist"
- Remove "AWS Service Security" as separate section
- Combine into "External Service Security"

**Changes:**
```markdown
❌ BEFORE:
## FLKitOver Integration Security

Webhook signature verification
API credential rotation
Document access control
Audit trail maintenance
Secure document storage

## AWS Service Security

IAM role-based access control
S3 bucket policies and encryption
VPC configuration for database access
CloudTrail for API auditing
Secrets Manager for credential storage

✅ AFTER:
## External Service Integration Security

For ANY third-party service (SaaS, API, Cloud), check:

### Authentication & Credentials
- [ ] API keys/credentials in environment variables (never source code)
- [ ] Credentials never logged (even in error messages)
- [ ] Proper auth method (OAuth 2.0 preferred, API Key acceptable with rotation)
- [ ] Token expiration handled (refresh before expiry)
- [ ] Multi-environment credentials (dev != staging != prod)

### API Call Security
- [ ] Timeout specified (prevent hanging connections)
- [ ] HTTPS enforced (never HTTP)
- [ ] Certificate validation enabled
- [ ] Request validation (validate before sending)
- [ ] Response validation (don't trust API response blindly)

### Error Handling
- [ ] Specific exception types (not generic Exception)
- [ ] API error details never exposed to users
- [ ] Internal logging with full details (safe for dev troubleshooting)
- [ ] Graceful degradation (app works if service temporarily fails)
- [ ] Retry logic with exponential backoff

### Webhook Security (if applicable)
- [ ] Signature verification (HMAC-SHA256 or similar)
- [ ] Timestamp validation (prevent replay attacks)
- [ ] Idempotency keys (prevent duplicate processing)
- [ ] Webhook endpoint authentication
- [ ] Only process webhooks from trusted IP ranges

### Data Protection
- [ ] User data not sent to third-party unnecessarily
- [ ] Data minimization (send only required fields)
- [ ] Sensitive data encryption in transit
- [ ] Data retention policy (delete unneeded data)
- [ ] Vendor compliance check (do they meet your security standards?)

### Monitoring & Logging
- [ ] Failed API calls logged
- [ ] Rate limit approaching: alert
- [ ] Service outages: alert
- [ ] Unauthorized errors: alert (possible credential compromise)
- [ ] Response time anomalies: alert

### Configuration
- [ ] API base URL in environment variable
- [ ] Rate limit settings documented
- [ ] Timeout values documented
- [ ] Retry policy documented
- [ ] Circuit breaker configured (fail gracefully after N failures)

### Cloud Service Specific (S3, SES, DynamoDB, etc.)
- [ ] IAM roles follow least privilege
- [ ] Resource policies restrict access
- [ ] Encryption at rest enabled
- [ ] Encryption in transit enabled
- [ ] Access logging enabled
- [ ] Audit trails retained (CloudTrail for AWS)
- [ ] No public access by default
```

---

## How to Apply Updates

### Option 1: Batch Update (Recommended)

1. **Backup current skills:**
   ```bash
   cp -r skills skills_backup_2026-03-17
   ```

2. **Update each skill file:**
   - Replace project-specific names with generic placeholders
   - Replace API-specific examples with generic patterns
   - Update descriptions to be framework-agnostic

3. **Test with example:**
   ```
   /invoke tech-writer "Using tech-writer-spec-skill, create spec
   for payment processing integration.

   Third-party service: Stripe
   Requirements: [your requirements]"
   ```

### Option 2: Create New Generic Versions

Keep old versions, create new `-generic` versions:
```
/skills/
├── tech-lead-gates-skill/SKILL.md (current - project specific)
├── tech-lead-gates-skill-generic/SKILL.md (new - generic)
├── ...
```

Then teams choose which version to use.

---

## Generic Placeholders to Use

Replace these everywhere:

```
❌ FLKitOver
✅ [Third-Party API Name] or [External Service Name]

❌ PetApplication
✅ [Resource Name] or [Feature Name]

❌ PasswordResetService
✅ [FeatureName]Service

❌ tenant_name, pet_type, general_consent
✅ [relevant_field_1], [relevant_field_2], [user_consent]

❌ SES, S3, Twilio
✅ [Email Service], [Storage Service], [SMS Service]

❌ /api/pet-applications
✅ /api/[resource-name]

❌ password_resets table
✅ [feature_name]_[resource] table
```

---

## Validation Checklist

After updating, verify:

- [ ] No mention of "FLKitOver" (unless explaining generic pattern)
- [ ] No mention of "PetApplication" or specific project names
- [ ] All examples use generic placeholders: [FeatureName], [ApiName]
- [ ] Third-party API patterns generic for ANY service
- [ ] Agents still reference correct skills
- [ ] Examples work for different projects (payment, email, SMS, docs, etc.)

---

## Benefits After Update

✅ Skills work for ANY project (not just one)
✅ Skills work with ANY third-party API
✅ Can reuse skills across different teams/projects
✅ New projects don't need new skills, just updated examples
✅ Skills remain authoritative for quality standards

---

## Example: After Update

### Before (Project-Specific)
```
"Integrate with FLKitOver to create documents"
- FLKitOver credentials
- FLKitOver webhook verification
- FLKitOver error handling
```

### After (Generic)
```
"Integrate with [Third-Party API] to [specific purpose]"
- [Third-Party API] authentication ([OAuth/API Key/Bearer Token])
- Webhook signature verification (if applicable)
- [Third-Party API] error handling and retries
- Timeout and rate limiting
```

**Result:** Same skill works for Stripe payments, SendGrid email, Twilio SMS, Stripe Connect, and any other API.

---

**Next Steps:**

1. Review this update guide
2. Update each skill with generic names
3. Test with different project examples
4. Document new generic standards

This makes skills truly reusable across all projects! 🎯
