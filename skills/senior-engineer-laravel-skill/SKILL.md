---
name: senior-engineer-laravel-skill
description: Laravel 12 implementation standards with PHP 8.2+ patterns, strict typing, SOLID principles, and comprehensive documentation
---

# Senior Engineer Laravel Implementation Skill

You implement features with enterprise-grade quality: clean code, comprehensive documentation, robust error handling, and full test coverage.

## Coding Standards Foundation

### File Headers (MANDATORY in Every PHP File)
```php
<?php declare(strict_types=1);

namespace App\Services;

use Illuminate\Support\Facades\Log;
use App\Models\PetApplication;
use App\Exceptions\PetApplicationException;
```

**NEVER skip `declare(strict_types=1);` - it's mandatory.**

---

## PHP 8.2+ Code Patterns

### 1. Strict Typing for All Functions

```php
// ✅ CORRECT
public function submitApplication(array $data): PetApplication
{
    return $this->service->create($data);
}

public function getRateLimit(?string $userId): int
{
    return 10; // Use nullable types with ?
}

// ❌ INCORRECT
public function submitApplication($data) // No type hints
{
    return $this->service->create($data);
}
```

### 2. Constructor Property Promotion

```php
// ✅ CORRECT - Concise, no manual property assignment
public function __construct(
    private readonly PetApplicationModel $petApplication,
    private readonly FLKService $flkService,
    private readonly EmailNotificationService $emailService
) {}

// ❌ INCORRECT - Verbose, unnecessary manual assignment
public function __construct(
    private PetApplicationModel $petApplication,
    private FLKService $flkService
)
{
    $this->petApplication = $petApplication;
    $this->flkService = $flkService;
}
```

### 3. Proper Exception Handling

```php
// ✅ CORRECT - Specific exception, context logging
try {
    $result = $this->flkService->createDocument($data);
} catch (FLKApiException $e) {
    Log::error('FLK API error', [
        'error' => $e->getMessage(),
        'code' => $e->getCode(),
        'property_id' => $data['property_id'] ?? null
    ]);
    throw new PetApplicationException('Failed to create FLK document', 0, $e);
}

// ❌ INCORRECT - Bare exception, no logging
try {
    $result = $this->flkService->createDocument($data);
} catch (Exception $e) {
    throw new Exception('Error'); // No logging, no context
}
```

### 4. Use Log Facade, Never die()/var_dump()

```php
// ✅ CORRECT
Log::debug('Application submitted', ['application_id' => $app->id]);
Log::error('FLK API failed', ['status' => $response->status()]);

// ❌ INCORRECT
var_dump($data); // Never in production code
die('Debug: ' . $data); // Blocks execution
print_r($application); // Not loggable
```

---

## Laravel Model & Database Patterns

### Model with UUID, Relationships & Casts

```php
<?php declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\Concerns\HasUuids;

class PetApplication extends Model
{
    use HasFactory, HasUuids;

    // Explicitly allow only these fields (mass assignment protection)
    protected $fillable = [
        'property_id',
        'landlord_id',
        'tenant_name',
        'tenant_email',
        'tenant_phone',
        'status',
        'metadata',
    ];

    // Type casting for specific fields
    protected $casts = [
        'submitted_at' => 'datetime',
        'approved_at' => 'datetime',
        'general_consent' => 'boolean',
        'metadata' => 'json',
    ];

    // Always define relationships
    public function property(): BelongsTo
    {
        return $this->belongsTo(Property::class);
    }

    public function landlord(): BelongsTo
    {
        return $this->belongsTo(Landlord::class);
    }

    public function pets(): HasMany
    {
        return $this->hasMany(Pet::class);
    }

    // Query scopes for filtering
    public function scopePending($query)
    {
        return $query->where('status', 'pending');
    }

    public function scopeApproved($query)
    {
        return $query->where('status', 'approved');
    }
}
```

### Database Migration Pattern

```php
<?php declare(strict_types=1);

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('pet_applications', function (Blueprint $table) {
            // Use UUID primary key
            $table->uuid('id')->primary();

            // Foreign keys
            $table->uuid('property_id');
            $table->uuid('landlord_id')->nullable();

            // Data fields
            $table->string('tenant_name');
            $table->string('tenant_email');
            $table->string('tenant_phone')->nullable();
            $table->enum('status', ['pending', 'approved', 'rejected'])->default('pending');

            // JSON for flexible metadata
            $table->json('metadata')->nullable();

            // Timestamps
            $table->timestamps();

            // Foreign key constraints
            $table->foreign('property_id')
                  ->references('id')
                  ->on('properties')
                  ->onDelete('cascade');

            $table->foreign('landlord_id')
                  ->references('id')
                  ->on('landlords')
                  ->onDelete('set null');

            // Indexes for filtering
            $table->index(['property_id']);
            $table->index(['status']);
            $table->index(['created_at']);
            $table->index(['status', 'created_at']); // Composite for filtering
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('pet_applications');
    }
};
```

---

## Service Layer Pattern (Composition Over Inheritance)

```php
<?php declare(strict_types=1);

namespace App\Services;

use App\Models\PetApplication;
use App\Exceptions\PetApplicationException;
use Illuminate\Support\Facades\Log;

class PetApplicationService
{
    /**
     * Constructor with dependency injection.
     * Use readonly properties (PHP 8.1+) to prevent accidental mutation.
     */
    public function __construct(
        private readonly PetApplicationModel $petApplicationModel,
        private readonly FLKService $flkService,
        private readonly EmailNotificationService $emailService,
        private readonly AwsS3Service $s3Service
    ) {}

    /**
     * Submit a new pet application.
     *
     * @param array $data Application data from validated form request
     * @return PetApplication The created application
     * @throws PetApplicationException If submission fails
     */
    public function submitApplication(array $data): PetApplication
    {
        try {
            // 1. Create application in database
            $application = $this->petApplicationModel->create([
                'property_id' => $data['property_id'],
                'tenant_name' => $data['tenant_name'],
                'tenant_email' => $data['tenant_email'],
                'tenant_phone' => $data['tenant_phone'] ?? null,
                'status' => 'pending',
            ]);

            // 2. Create pets (child records)
            if (!empty($data['pets'])) {
                $application->pets()->createMany($data['pets']);
            }

            // 3. Create FLKitOver document
            $flkDocument = $this->flkService->createDocument([
                'applicant_name' => $data['tenant_name'],
                'property_id' => $data['property_id'],
                'application_id' => $application->id,
            ]);

            // Store FLK document ID in metadata
            $application->update([
                'metadata' => array_merge(
                    $application->metadata ?? [],
                    ['flk_document_id' => $flkDocument['id']]
                ),
            ]);

            // 4. Send notification email
            $this->emailService->sendApplicationSubmitted($application);

            Log::info('Pet application submitted', [
                'application_id' => $application->id,
                'property_id' => $data['property_id'],
            ]);

            return $application;

        } catch (FLKApiException $e) {
            Log::error('FLK document creation failed', [
                'error' => $e->getMessage(),
                'property_id' => $data['property_id'],
            ]);
            throw new PetApplicationException('Failed to create application document', 0, $e);

        } catch (\Exception $e) {
            Log::error('Unexpected error in submitApplication', [
                'error' => $e->getMessage(),
                'property_id' => $data['property_id'] ?? null,
            ]);
            throw new PetApplicationException('Failed to submit application', 0, $e);
        }
    }

    /**
     * Update application status.
     *
     * @param PetApplication $application
     * @param string $status New status
     * @return PetApplication Updated application
     */
    public function updateStatus(PetApplication $application, string $status): PetApplication
    {
        $application->update(['status' => $status]);

        Log::info('Application status updated', [
            'application_id' => $application->id,
            'new_status' => $status,
        ]);

        return $application;
    }
}
```

---

## Form Request Validation Pattern

```php
<?php declare(strict_types=1);

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePetApplicationRequest extends FormRequest
{
    public function authorize(): bool
    {
        // User must be authenticated
        return auth()->check();
    }

    public function rules(): array
    {
        return [
            // Required fields
            'property_id' => ['required', 'uuid', 'exists:properties,id'],
            'tenant_name' => ['required', 'string', 'max:255'],
            'tenant_email' => ['required', 'email', 'max:255'],

            // Pets (nested)
            'pets' => ['required', 'array', 'min:1', 'max:5'],
            'pets.*.name' => ['required', 'string', 'max:255'],
            'pets.*.type' => ['required', 'string', 'in:dog,cat,bird,other'],
            'pets.*.breed' => ['nullable', 'string', 'max:255'],
            'pets.*.age' => ['nullable', 'integer', 'min:0', 'max:30'],
            'pets.*.weight' => ['nullable', 'numeric', 'min:0', 'max:100'],

            // Optional fields
            'tenant_phone' => ['nullable', 'string', 'regex:/^[\d\s\-\+\(\)]+$/'],
            'general_consent' => ['accepted'],
        ];
    }

    /**
     * Custom error messages for clarity.
     */
    public function messages(): array
    {
        return [
            'property_id.exists' => 'The selected property does not exist.',
            'pets.min' => 'You must provide at least one pet.',
            'pets.max' => 'You can submit a maximum of 5 pets.',
            'general_consent.accepted' => 'You must accept the terms and conditions.',
        ];
    }
}
```

---

## Controller Pattern (Thin Controllers)

```php
<?php declare(strict_types=1);

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\StorePetApplicationRequest;
use App\Http\Resources\PetApplicationResource;
use App\Services\PetApplicationService;
use App\Exceptions\PetApplicationException;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Log;

class PetApplicationController extends Controller
{
    public function __construct(
        private readonly PetApplicationService $petApplicationService
    ) {}

    /**
     * Create a new pet application.
     *
     * @param StorePetApplicationRequest $request Validated request
     * @return JsonResponse
     */
    public function store(StorePetApplicationRequest $request): JsonResponse
    {
        try {
            $application = $this->petApplicationService->submitApplication(
                $request->validated()
            );

            return response()->json([
                'success' => true,
                'data' => new PetApplicationResource($application),
                'message' => 'Pet application submitted successfully',
            ], 201);

        } catch (PetApplicationException $e) {
            Log::warning('Pet application submission failed', [
                'error' => $e->getMessage(),
                'user_id' => auth()->id(),
            ]);

            return response()->json([
                'success' => false,
                'message' => 'Failed to submit application',
                'errors' => ['general' => [$e->getMessage()]],
            ], 422);

        } catch (\Exception $e) {
            Log::error('Unexpected error in PetApplicationController::store', [
                'error' => $e->getMessage(),
                'user_id' => auth()->id(),
            ]);

            return response()->json([
                'success' => false,
                'message' => 'Server error',
            ], 500);
        }
    }
}
```

---

## API Resource (Response Transformer)

```php
<?php declare(strict_types=1);

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class PetApplicationResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'property_id' => $this->property_id,
            'tenant_name' => $this->tenant_name,
            'tenant_email' => $this->tenant_email,
            'tenant_phone' => $this->tenant_phone,
            'status' => $this->status,
            'pets' => PetResource::collection($this->whenLoaded('pets')),
            'created_at' => $this->created_at->toIso8601String(),
            'updated_at' => $this->updated_at->toIso8601String(),
        ];
    }
}
```

---

## Test Patterns

### Feature Test Example

```php
<?php declare(strict_types=1);

namespace Tests\Feature;

use App\Models\Property;
use App\Models\PetApplication;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PetApplicationTest extends TestCase
{
    use RefreshDatabase;

    public function test_can_submit_pet_application_successfully(): void
    {
        // Arrange
        $property = Property::factory()->create();
        $data = [
            'property_id' => $property->id,
            'tenant_name' => 'John Doe',
            'tenant_email' => 'john@example.com',
            'tenant_phone' => '0412345678',
            'pets' => [
                [
                    'name' => 'Buddy',
                    'type' => 'dog',
                    'breed' => 'Labrador',
                    'age' => 3,
                ],
            ],
            'general_consent' => true,
        ];

        // Act
        $response = $this->postJson('/api/pet-applications', $data);

        // Assert
        $response->assertStatus(201)
                ->assertJson(['success' => true])
                ->assertJsonStructure([
                    'data' => [
                        'id',
                        'tenant_name',
                        'status',
                        'created_at',
                    ],
                ]);

        $this->assertDatabaseHas('pet_applications', [
            'tenant_email' => 'john@example.com',
            'status' => 'pending',
        ]);
    }

    public function test_validation_fails_without_required_fields(): void
    {
        // Act
        $response = $this->postJson('/api/pet-applications', []);

        // Assert
        $response->assertStatus(422)
                ->assertJsonValidationErrors([
                    'property_id',
                    'tenant_name',
                    'pets',
                ]);
    }
}
```

---

## Code Quality Checklist Before Invoking Code-Reviewer

- [ ] All files have `declare(strict_types=1);`
- [ ] All functions have type hints (parameters and return types)
- [ ] All methods have PHPDoc with @param and @return
- [ ] No use of `die()`, `var_dump()`, or `print_r()`
- [ ] All exceptions caught with specific exception types
- [ ] All database operations use Eloquent (not raw SQL)
- [ ] Models use UUID primary keys
- [ ] Services use constructor dependency injection
- [ ] Controllers are thin (no business logic)
- [ ] Form Requests used for validation
- [ ] Tests cover happy path and error cases
- [ ] Database migrations create proper indexes
- [ ] No N+1 queries (use eager loading)
- [ ] FLKitOver integration errors caught and logged

---

## Implementation Workflow

1. **Read spec thoroughly** - Understand all requirements before coding
2. **Create migration** - Design database schema
3. **Create model** - Define relationships and casts
4. **Create service** - Implement business logic
5. **Create controller** - Handle HTTP requests
6. **Create requests** - Validation rules
7. **Create tests** - Unit, feature, integration
8. **Verify no red flags** - Run through code quality checklist
9. **Invoke code-reviewer** - Submit for review

---

## Key Reminders

⚠️ **Strict typing is mandatory** - `declare(strict_types=1);` every file
⚠️ **Services own business logic** - Controllers only handle HTTP
⚠️ **Log everything important** - Debugging production issues requires logs
⚠️ **Handle errors explicitly** - Never let exceptions bubble up uncaught
⚠️ **UUID architecture throughout** - No numeric IDs
⚠️ **Test as you code** - Don't leave testing for the end
