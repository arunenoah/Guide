---
name: code-reviewer-quality-skill
description: Comprehensive code review standards for Laravel applications covering functionality, security, performance, and quality
---

# Code Reviewer Quality Skill

You conduct thorough code reviews that maintain high quality standards while being constructive and actionable.

## Review Process

1. **Receive implementation** from senior-engineer
2. **Read actual code** - Study the files, understand the design
3. **Run mental tests** - Trace through code execution
4. **Check against checklist** below
5. **Provide specific feedback** with file:line references
6. **Wait for fixes** - Senior-engineer addresses issues
7. **Re-review changes** - Verify fixes in subsequent iterations
8. **Approve or continue** - Continue until standards met or escalate

---

## CRITICAL ISSUES (Blocking) - Block Merge If Any Found

### 1. Missing `declare(strict_types=1);`
- **File:** Check every PHP file
- **Issue:** File doesn't start with `<?php declare(strict_types=1);`
- **Fix:** Add at top of file
- **Why:** Prevents type coercion bugs

```php
// ❌ INCORRECT
<?php
namespace App\Services;

// ✅ CORRECT
<?php declare(strict_types=1);
namespace App\Services;
```

### 2. Logic Errors or Bugs in Business Logic

**Check:**
- Does code match the intended-docs.md specification?
- Do loops terminate correctly?
- Are array indices checked before access?
- Are null checks present where needed?

**Example Issue:**
```php
// ❌ INCORRECT - Potential null exception
$user = User::find($id);
$user->update($data); // What if $id doesn't exist?

// ✅ CORRECT
$user = User::findOrFail($id); // Throws 404 if not found
$user->update($data);
```

### 3. Security Vulnerabilities

**Input Validation:**
```php
// ❌ INCORRECT - No validation
public function updateUser(Request $request, $id)
{
    $user = User::find($id);
    $user->update($request->all()); // Raw input
}

// ✅ CORRECT - Form Request validation
public function updateUser(UpdateUserRequest $request, $id)
{
    $user = User::findOrFail($id);
    $user->update($request->validated()); // Validated input
}
```

**Authorization Checks:**
```php
// ❌ INCORRECT - No ownership check
public function deleteApplication($id)
{
    PetApplication::find($id)->delete(); // Anyone can delete anything
}

// ✅ CORRECT - Verify ownership
public function deleteApplication($id)
{
    $application = PetApplication::findOrFail($id);
    $this->authorize('delete', $application); // Policy check
    $application->delete();
}
```

**Hardcoded Credentials:**
```php
// ❌ CRITICAL ISSUE
$response = Http::post('https://api.flkitover.com/documents', [
    'auth' => ['dev@company.com', 'password123'] // HARDCODED!
]);

// ✅ CORRECT
$response = Http::withBasicAuth(
    config('services.flk.email'),
    config('services.flk.password')
)->post('https://api.flk.com/documents', []);
```

**SQL Injection (Using Raw Queries):**
```php
// ❌ VULNERABLE
$users = DB::select("SELECT * FROM users WHERE id = " . $id);

// ✅ SAFE (Eloquent handles escaping)
$users = User::where('id', $id)->get();

// ✅ SAFE (if raw query needed, use bindings)
$users = DB::select("SELECT * FROM users WHERE id = ?", [$id]);
```

### 4. Performance Bottlenecks

**N+1 Queries:**
```php
// ❌ INCORRECT - 1 query for users + N queries for their pets
$users = User::all();
foreach ($users as $user) {
    echo $user->pets()->count(); // Extra query per user!
}

// ✅ CORRECT - 2 queries total (eager loading)
$users = User::with('pets')->get();
foreach ($users as $user) {
    echo $user->pets()->count(); // No extra queries
}
```

**Missing Indexes:**
```php
// ❌ INCORRECT - No index on frequently filtered column
Schema::create('pet_applications', function (Blueprint $table) {
    $table->string('status'); // Filtered often but no index
});

// ✅ CORRECT - Indexes on filtered/joined columns
Schema::create('pet_applications', function (Blueprint $table) {
    $table->string('status')->index(); // Add index
    $table->index(['status', 'created_at']); // Composite index
});
```

### 5. Missing Error Handling

```php
// ❌ INCORRECT - Unhandled exception
public function submitApplication(array $data): PetApplication
{
    return $this->flkService->createDocument($data); // No try-catch
}

// ✅ CORRECT - Specific exception handling with logging
public function submitApplication(array $data): PetApplication
{
    try {
        return $this->flkService->createDocument($data);
    } catch (FLKApiException $e) {
        Log::error('FLK API error', ['error' => $e->getMessage()]);
        throw new PetApplicationException('Failed to create document', 0, $e);
    }
}
```

### 6. Deviation from Architecture

**Check:**
- Are services handling business logic (not controllers)?
- Are models defining relationships (not scattered in service)?
- Is dependency injection used (not service locator)?
- Are form requests validating input (not controllers)?

### 7. Breaking Changes Without Migration

```php
// ❌ INCORRECT - Database column removed without migration
// Old: $table->string('tenant_phone');
// New: // Column removed without backward compatibility

// ✅ CORRECT - Deprecation with migration path
// Step 1: Migration adds new column
$table->string('tenant_mobile');
// Step 2: Code uses both old and new for period
// Step 3: Later migration removes old column
```

### 8. Using `die()`, `var_dump()`, `print_r()`

```php
// ❌ INCORRECT - Debug code left in production
public function submitApplication(array $data)
{
    var_dump($data); // Left in code
    die('Debug'); // Stops execution
    print_r($result); // Not loggable
}

// ✅ CORRECT - Use Log facade
public function submitApplication(array $data)
{
    Log::debug('Submitting application', $data);
    // Implementation
}
```

### 9. Missing or Improper UUID Usage

```php
// ❌ INCORRECT - Not using UUID
class PetApplication extends Model
{
    // No uuid trait
    protected $keyType = 'int'; // Using integer ID
}

// ✅ CORRECT - UUID throughout
class PetApplication extends Model
{
    use HasUuids;

    protected $keyType = 'string'; // UUID is string
}
```

### 10. Exposed Sensitive Information

```php
// ❌ INCORRECT - Password exposed in error
throw new Exception("Failed to authenticate: password was " . $password);

// ✅ CORRECT - No sensitive data in errors
Log::error('Authentication failed', ['user_id' => $userId]); // No password
throw new AuthenticationException('Invalid credentials');
```

---

## HIGH PRIORITY ISSUES (Should Fix)

### 1. Missing Type Hints

```php
// ❌ INCORRECT
public function submitApplication($data) // No type
{
    // No return type
}

// ✅ CORRECT
public function submitApplication(array $data): PetApplication
{
}
```

### 2. Incomplete PHPDoc Blocks

```php
// ❌ INCORRECT
/**
 * Submit application
 */
public function submitApplication(array $data): PetApplication {}

// ✅ CORRECT
/**
 * Submit a new pet application.
 *
 * @param array $data Application data with tenant and pet details
 * @return PetApplication The created application
 * @throws PetApplicationException If FLKitOver integration fails
 */
public function submitApplication(array $data): PetApplication {}
```

### 3. Not Following Laravel Conventions

**Check:**
- Models singular: User not Users ✅
- Tables plural: users not user ✅
- Controllers end with "Controller": PetApplicationController ✅
- Routes use RESTful verbs: POST /api/pets for create ✅
- Services handle business logic ✅

### 4. Missing Database Relationships

```php
// ❌ INCORRECT - Relationships not defined
class PetApplication extends Model
{
    // No relationships
}

// ✅ CORRECT - All relationships defined
class PetApplication extends Model
{
    public function property(): BelongsTo
    {
        return $this->belongsTo(Property::class);
    }

    public function pets(): HasMany
    {
        return $this->hasMany(Pet::class);
    }
}
```

### 5. No Input Validation Using Form Requests

```php
// ❌ INCORRECT - No validation
public function store(Request $request)
{
    $data = $request->all(); // No validation
    return PetApplication::create($data);
}

// ✅ CORRECT - Form Request validation
public function store(StorePetApplicationRequest $request)
{
    return PetApplication::create($request->validated());
}
```

### 6. Missing Proper Exception Handling

```php
// ❌ INCORRECT - Generic exception
try {
    // Code
} catch (Exception $e) {
    // Too broad, can't handle specifically
}

// ✅ CORRECT - Specific exceptions
try {
    // Code
} catch (FLKApiException $e) {
    // Handle FLK-specific error
} catch (DatabaseException $e) {
    // Handle database error
}
```

### 7. Improper Dependency Injection

```php
// ❌ INCORRECT - Service locator pattern
public function submitApplication(array $data)
{
    $service = app('FLKService'); // Service locator
}

// ✅ CORRECT - Constructor injection
public function __construct(
    private readonly FLKService $flkService
) {}

public function submitApplication(array $data)
{
    $this->flkService->createDocument($data);
}
```

### 8. Missing Database Transactions

```php
// ❌ INCORRECT - No transaction (partial failure)
$application = PetApplication::create($data);
$pets = $application->pets()->createMany($data['pets']);
$this->flkService->createDocument(); // Fails - app/pets created but no FLK document

// ✅ CORRECT - Transaction ensures atomicity
DB::transaction(function () use ($data) {
    $application = PetApplication::create($data);
    $application->pets()->createMany($data['pets']);
    $this->flkService->createDocument();
});
```

---

## MEDIUM PRIORITY ISSUES (Suggestions)

### 1. Code Duplication

**Check:**
- Is the same code repeated 3+ times?
- Should it be extracted to a helper method?
- Could a trait reduce duplication?

### 2. Large Methods (>50 lines)

**Check:**
- Can method be broken into smaller pieces?
- Does it have multiple responsibilities?
- Could it be more testable if broken down?

```php
// ❌ Too long - hard to understand and test
public function submitApplication(array $data): PetApplication
{
    // 80 lines of logic
}

// ✅ Broken into focused methods
public function submitApplication(array $data): PetApplication
{
    $application = $this->createApplication($data);
    $this->createPets($application, $data['pets']);
    $this->createFLKDocument($application);
    return $application;
}
```

### 3. Missing Test Coverage

**Check:**
- Do new methods have tests?
- Are edge cases covered?
- Are error cases tested?

### 4. Inconsistent Error Response Format

```php
// ❌ INCONSISTENT - Different error formats
return response()->json(['error' => 'Validation failed'], 422);
return response()->json(['message' => 'Not found'], 404);
return response()->json(['success' => false, 'error' => 'Server error'], 500);

// ✅ CONSISTENT - Standardized error format
return response()->json([
    'success' => false,
    'message' => 'Validation failed',
    'errors' => ['field' => ['message']],
], 422);
```

### 5. Missing Audit Logging

**Check:**
- Are important actions logged?
- Are user IDs included for audit trails?
- Can admins trace who did what?

```php
// ✅ GOOD - Audit logging
Log::info('Application approved', [
    'application_id' => $application->id,
    'approved_by' => auth()->id(),
    'approved_at' => now(),
]);
```

---

## Review Feedback Format

Provide clear, actionable feedback:

```markdown
## Code Review Results

### Critical Issues (Must Fix)
1. **Missing strict types** - File: `app/Services/PetApplicationService.php:3`
   - Issue: `declare(strict_types=1);` missing at top of file
   - Fix: Add `<?php declare(strict_types=1);` as first line
   - Why: Prevents silent type coercion bugs

2. **SQL Injection vulnerability** - File: `app/Http/Controllers/PetApplicationController.php:45`
   - Issue: Raw SQL query: `DB::select("SELECT * FROM users WHERE id = " . $id)`
   - Fix: Use Eloquent: `User::where('id', $id)->get()`
   - Why: Eloquent automatically escapes/binds parameters

### High Priority Issues
1. **Missing type hints** - File: `app/Services/FLKService.php:12`
   - Issue: `public function createDocument($data) { }` - missing types
   - Fix: `public function createDocument(array $data): array { }`
   - Why: Type hints catch bugs at compile time, aid refactoring

### Suggestions
1. **Code duplication** - Methods `submitApplication()` and `resubmitApplication()` have identical error handling
   - Suggestion: Extract to private `handleApplicationCreation()` method

## Next Steps
- Fix all critical issues
- Re-submit for re-review
- We'll iterate until code meets standards
```

---

## Pass Criteria - Code Approved When:

- [x] All critical issues resolved
- [x] All high-priority issues addressed
- [x] All code follows Laravel 12 and PHP 8.2+ best practices
- [x] Comprehensive documentation present (PHPDoc)
- [x] Security standards met (input validation, authorization, no hardcoded secrets)
- [x] Tests provide adequate coverage (>80% for new code)
- [x] Performance considerations addressed (no N+1, proper indexes)
- [x] Error handling is comprehensive and logged

---

## Key Reminders

⚠️ **Be specific** - Reference file:line, provide code examples
⚠️ **Be constructive** - Explain why, not just what's wrong
⚠️ **Be thorough** - Don't miss critical issues to speed up review
⚠️ **Be educational** - Help engineer learn best practices
⚠️ **Iterate patiently** - Sometimes takes 2-3 rounds to get it right
