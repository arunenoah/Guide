# Skills Generic Update Checklist

**Goal:** Make all skills generic for ANY project and ANY third-party API

---

## Skill 1: tech-lead-gates-skill/SKILL.md

**Changes needed:**

- [ ] Update description to remove FLKitOver mention
  ```
  ❌ "Gate A design, Gate B drift audit, Gate C production authorization"
  ✅ "Gate A design, Gate B drift audit, Gate C production authorization - generic for any project/API"
  ```

- [ ] Gate A Review Checklist - Replace specific APIs with generic
  ```
  ❌ Third-party APIs clearly specified (SES, S3, Twilio, FLKitOver)
  ✅ Third-party APIs clearly specified (name, endpoints, authentication method)
  ```

- [ ] Rejection Examples - Update to be generic
  ```
  ❌ "Fix security issue" → Which issue?
  ✅ "Call external API" → No authentication, retry logic, or timeout specified
  ```

- [ ] Part 1: Code Inspection - Make project-agnostic
  ```
  ❌ cat app/Services/PasswordResetService.php
  ✅ cat app/Services/[FeatureService].php
  ```

- [ ] Part 4: Red Flag Checklist - Update API-specific
  ```
  ❌ FLKitOver webhook failures → ROLLBACK
  ✅ Third-party service failures (timeouts, auth errors) → ROLLBACK
  ```

- [ ] Infrastructure Gate - Add third-party API considerations
  ```
  ❌ [existing checks]
  ✅ Third-party API endpoints/credentials configured in production
  ```

---

## Skill 2: tech-writer-spec-skill/SKILL.md

**Changes needed:**

- [ ] Update description
  ```
  ❌ "Technical documentation standards for Laravel applications including intended specs, drift analysis, and final documentation"
  ✅ "Technical documentation standards including intended specs, drift analysis, final documentation - generic for any tech stack/API"
  ```

- [ ] Anti-Pattern Examples - Add third-party API anti-patterns
  ```
  ❌ "improve security"
  ✅ Add: "call external API" (vague on which API, how to handle errors)
  ```

- [ ] Update FLKitOver Integration section → Generic Third-Party API Integration
  ```
  ❌ FLKitOver Integration
  ✅ Third-Party API Integration

  Template for ANY API:
  - Service name: [Name]
  - Purpose: [What it does]
  - Authentication: [Type]
  - Endpoints: [List]
  - Error handling: [How to handle failures]
  - Webhooks: [Yes/No - if yes, signature verification method]
  - Rate limits: [Specify]
  - Timeouts: [Specify]
  - Retries: [Strategy]
  ```

- [ ] Update AWS-specific section → External Services section
  ```
  ❌ AWS S3 for file storage
     AWS SES for email delivery
     Twilio for SMS notifications

  ✅ ### External Services
     - Storage service: [Service name] (S3, Cloudinary, etc.)
     - Email service: [Service name] (SES, SendGrid, Mailgun, etc.)
     - SMS service: [Service name] (Twilio, Vonage, AWS SNS, etc.)
  ```

- [ ] Update template variables
  ```
  ❌ [feature-name] can still be used
  ✅ But make sure examples don't assume PetApplication, password-reset, etc.
  ```

---

## Skill 3: senior-engineer-laravel-skill/SKILL.md

**Changes needed:**

- [ ] Update description
  ```
  ❌ "Laravel 12 implementation standards with PHP 8.2+ patterns, strict typing, SOLID principles, and comprehensive documentation"
  ✅ Same description is fine (it's about Laravel patterns, not specific APIs)
  ```

- [ ] Remove PetApplicationService example
  ```
  ❌ public function __construct(
       private readonly PetApplicationModel $petApplication,
       private readonly FLKService $flkService,
       private readonly EmailNotificationService $emailService,
       private readonly AwsS3Service $s3Service
     ) {}

     public function submitApplication(array $data): PetApplication {
         // Create FLK document
         $flkDocument = $this->flkService->createDocument($data);
         // ...
     }

  ✅ public function __construct(
       private readonly FeatureModel $model,
       private readonly ThirdPartyApiService $apiService,
       private readonly NotificationService $notificationService,
       private readonly StorageService $storageService
     ) {}

     public function submitRequest(array $data): Resource {
         try {
             // Call third-party API
             $apiResponse = $this->apiService->create($data);
             if (!$apiResponse['success']) {
                 throw new ThirdPartyApiException('API request failed');
             }
             // Store and return result
         } catch (ThirdPartyApiException $e) {
             Log::error('Third-party API error', [
                 'error' => $e->getMessage(),
                 'api_name' => 'Third-Party API'
             ]);
             throw new RequestException('Failed to process request', 0, $e);
         }
     }
  ```

- [ ] Update Exception Handling pattern
  ```
  ❌ catch (FLKApiException $e) {
       Log::error('FLK API error', [...]);
       throw new PetApplicationException(...);
     }

  ✅ catch (ThirdPartyApiException $e) {
       Log::error('Third-party API error', [...]);
       throw new FeatureException(...);
     }
  ```

- [ ] Update Model example (remove PetApplication fields)
  ```
  ❌ protected $fillable = [
       'property_id',
       'landlord_id',
       'tenant_name',
       'tenant_email',
       'status',
       'metadata',
     ];

  ✅ protected $fillable = [
       'user_id',
       '[relevant_field_1]',
       '[relevant_field_2]',
       'status',
       'metadata',
     ];
  ```

- [ ] Update Migration example
  ```
  ❌ Schema::create('pet_applications', function (Blueprint $table) {
       $table->uuid('id')->primary();
       $table->uuid('property_id');
       $table->string('tenant_name');
       // ...
     });

  ✅ Schema::create('[feature_name]_[resources]', function (Blueprint $table) {
       $table->uuid('id')->primary();
       $table->uuid('user_id');
       $table->string('[relevant_field]');
       // ...
     });
  ```

- [ ] Update Service example name
  ```
  ❌ class PetApplicationService
  ✅ class [FeatureName]Service
  ```

- [ ] Update Request Validation example
  ```
  ❌ class StorePetApplicationRequest extends FormRequest
  ✅ class Create[FeatureName]Request extends FormRequest
  ```

- [ ] Update Controller example
  ```
  ❌ class PetApplicationController extends Controller
  ✅ class [FeatureName]Controller extends Controller
  ```

- [ ] Update Resource example
  ```
  ❌ class PetApplicationResource extends JsonResource
  ✅ class [FeatureName]Resource extends JsonResource
  ```

- [ ] Update Test example names
  ```
  ❌ class PetApplicationTest extends TestCase
  ✅ class [FeatureName]Test extends TestCase
  ```

---

## Skill 4: code-reviewer-quality-skill/SKILL.md

**Changes needed:**

- [ ] Update description to be generic
  ```
  ❌ "Comprehensive code review standards for Laravel applications covering functionality, security, performance, and quality"
  ✅ Same is fine (applies to all Laravel code)
  ```

- [ ] Remove PetApplicationController example
  ```
  ❌ class PetApplicationController extends Controller
  ✅ class [FeatureName]Controller extends Controller
  ```

- [ ] Update SQL Injection examples
  ```
  ❌ $user = User::find($id);
  ✅ $resource = [ResourceModel]::find($id);
  ```

- [ ] Replace FLKitOver Integration Security section
  ```
  ❌ ## FLKitOver Integration Security

     // ❌ VULNERABLE - Hardcoded credentials
     $response = Http::post('https://api.flkitover.com/documents', [
         'auth' => ['dev@company.com', 'password123']
     ]);

     // ✅ SECURE - Proper credential handling
     class FLKService {
         public function __construct(...) {}
     }

  ✅ ## Third-Party API Integration Security

     Check for each external API:
     - [ ] Credentials in environment variables (not hardcoded)
     - [ ] API keys/secrets never logged
     - [ ] Proper authentication method
     - [ ] Timeout specified
     - [ ] Retry logic with exponential backoff
     - [ ] Error handling catches API-specific exceptions
     - [ ] Webhook signatures verified (if applicable)

     // ❌ VULNERABLE - Hardcoded credentials
     $response = Http::post('https://api.example.com/create', [
         'auth' => ['user@company.com', 'secret123']
     ]);

     // ✅ SECURE - Proper credential handling
     class ExternalApiService {
         public function __construct(
             private readonly string $apiUrl,
             private readonly string $apiKey,
             private readonly int $timeout = 30
         ) {}
     }
  ```

- [ ] Update security headers section (can stay the same)

- [ ] Update error response example
  ```
  ❌ 'errors' => ['general' => [$e->getMessage()]]
  ✅ 'errors' => ['error' => [$e->getMessage()]]
     (Generic, not PetApplication specific)
  ```

---

## Skill 5: qa-testing-skill/SKILL.md

**Changes needed:**

- [ ] Update description
  ```
  ❌ "Comprehensive QA testing strategy covering positive, negative, non-functional, and regression tests with Playwright automation"
  ✅ Same is fine (applies to all testing)
  ```

- [ ] Remove PetApplicationTest examples
  ```
  ❌ class PetApplicationTest extends TestCase {
       public function test_can_submit_pet_application_successfully(): void {
           $property = Property::factory()->create();
           // ...
       }
     }

  ✅ class [FeatureName]Test extends TestCase {
       public function test_can_submit_[feature]_successfully(): void {
           // Example: Create required resources
           $[prerequisite] = [PrerequisiteModel]::factory()->create();
           // ...
       }
     }
  ```

- [ ] Remove FLKitOver Integration Testing section
  ```
  ❌ ## FLKitOver Integration Testing

     public function test_flk_document_creation_workflow(): void {
         $this->mock(FLKService::class, function ($mock) {
             $mock->shouldReceive('createDocument')
                  ->once()
                  ->andReturn(['document_id' => 'test-doc-id']);
         });
     }

  ✅ ## Third-Party API Integration Testing

     Test every external API call:

     1. MOCKING (Unit Tests)
     2. ERROR HANDLING (Unit Tests)
     3. INTEGRATION TESTS (Optional - if staging available)
     4. WEBHOOK TESTING (if applicable)
     5. RETRY LOGIC (if applicable)

     Example for ANY API:
     public function test_api_call_successful(): void {
         $this->mock(ExternalApiService::class, function ($mock) {
             $mock->shouldReceive('create')
                  ->once()
                  ->andReturn(['success' => true, 'id' => 'api-123']);
         });
         // ...
     }
  ```

- [ ] Update Playwright E2E example
  ```
  ❌ test.describe('Pet Application - Happy Path', () => {
       test('user submits pet application successfully', async ({ page }) => {
           await page.goto('http://localhost:3000/pet-applications');
           await page.fill('[data-testid="tenant-name"]', 'John Doe');
           // ...
       })
     })

  ✅ test.describe('[Feature Name] - Happy Path', () => {
       test('user completes [feature] workflow successfully', async ({ page }) => {
           await page.goto('http://localhost:3000/[feature-path]');
           await page.fill('[data-testid="[field-name]"]', '[valid-value]');
           // ...
       })
     })
  ```

- [ ] Update Test Structure names
  ```
  ❌ [feature-name]Test → ✅ [FeatureName]Test
  ❌ PetApplicationPositiveTest → ✅ [FeatureName]PositiveTest
  ❌ PetApplicationSecurityTest → ✅ [FeatureName]SecurityTest
  ```

- [ ] Update database references
  ```
  ❌ $this->assertDatabaseHas('pet_applications', [...])
  ✅ $this->assertDatabaseHas('[feature_name]_[resources]', [...])
  ```

---

## Skill 6: security-audit-skill/SKILL.md

**Changes needed:**

- [ ] Update description
  ```
  ❌ "Comprehensive security audit standards for Laravel applications covering OWASP Top 10, FLKitOver integrations, and AWS service security"
  ✅ "Comprehensive security audit standards for Laravel applications covering OWASP Top 10, third-party API integrations, and external service security"
  ```

- [ ] Remove FLKitOver section entirely
  ```
  ❌ ## FLKitOver Integration Security

     Webhook signature verification
     API credential rotation
     Document access control
     Audit trail maintenance
     Secure document storage

  ✅ ## External Service Integration Security

     For ANY third-party service (SaaS, API, Cloud):
     - Authentication & Credentials
     - API Call Security
     - Error Handling
     - Webhook Security (if applicable)
     - Data Protection
     - Monitoring & Logging
  ```

- [ ] Replace AWS section with generic External Services
  ```
  ❌ ## AWS Service Security

     IAM role-based access control
     S3 bucket policies and encryption
     VPC configuration for database access
     CloudTrail for API auditing
     Secrets Manager for credential storage

  ✅ ## Cloud/External Service Security

     - [ ] IAM/RBAC properly configured
     - [ ] Storage encryption at rest
     - [ ] Network access controls
     - [ ] Audit trails enabled
     - [ ] Credentials management (Secrets Manager or equivalent)
  ```

- [ ] Update FLKitOver credential examples
  ```
  ❌ config('services.flk.email')
     config('services.flk.password')

  ✅ config('services.external_api.key')
     config('services.external_api.secret')
  ```

- [ ] Update webhook signature example
  ```
  ❌ $signature = $request->header('X-FLK-Signature');
  ✅ $signature = $request->header('X-API-Signature');
  ```

---

## Summary Checklist

### Generic Name Replacements

Replace across ALL skills:

- [ ] FLKitOver → [Third-Party API] or [External Service]
- [ ] PetApplication → [Feature] or [Resource]
- [ ] tenant_name → [relevant_field_1] or [field_name]
- [ ] pet_type → [relevant_field_2] or [field_type]
- [ ] password-reset → [feature-name]
- [ ] PasswordReset → [FeatureName]
- [ ] PasswordResetService → [FeatureName]Service
- [ ] password_resets → [feature_name]_[resources]
- [ ] SES/S3/Twilio → [Email Service]/[Storage Service]/[SMS Service]

### Validation

After updates, verify:

- [ ] No FLKitOver-specific examples (unless showing generic pattern)
- [ ] No PetApplication references
- [ ] All examples use [FeatureName], [ApiName] placeholders
- [ ] Any project can use these skills
- [ ] Skills work with different tech stacks (not just Laravel)
- [ ] Skills work with any third-party service
- [ ] Test new skills with different examples:
  - Stripe payment integration
  - SendGrid email service
  - Twilio SMS service
  - Slack webhook integration
  - Salesforce CRM integration

---

## Testing After Update

Try invoking agents with different projects:

```
/invoke tech-writer "Using tech-writer-spec-skill, create spec
for Stripe payment integration."

/invoke qa-tester "Using qa-testing-skill, test SendGrid email integration
with all 4 categories."

/invoke security-reviewer "Using security-audit-skill, audit Slack webhook
integration for security."
```

**Result:** Skills should work seamlessly with ANY project/API! ✅

---

**Estimated Time:** 2-3 hours to update all 6 skills
**Benefit:** Skills become truly reusable across all projects
