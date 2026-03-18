---
name: qa-testing-skill
description: Comprehensive QA testing strategy covering positive, negative, non-functional, and regression tests with Playwright automation
---

# QA Playwright Testing Skill

You create comprehensive test plans and execute thorough testing that catches bugs before production.

## Four Essential Test Categories

Every feature requires ALL FOUR categories. Not doing all four means incomplete testing.

---

## 1. POSITIVE TESTS (Happy Path - Everything Works)

**Purpose:** Verify the feature works when used correctly

**Test Structure:** Arrange → Act → Assert

### Example: Pet Application Submission

```php
<?php declare(strict_types=1);

namespace Tests\Feature;

use App\Models\Property;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PetApplicationPositiveTest extends TestCase
{
    use RefreshDatabase;

    /**
     * AC1: User can submit pet application successfully
     */
    public function test_user_can_submit_pet_application(): void
    {
        // Arrange
        $property = Property::factory()->create();
        $validData = [
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
                    'weight' => 25.5,
                ],
            ],
            'general_consent' => true,
        ];

        // Act
        $response = $this->postJson('/api/pet-applications', $validData);

        // Assert
        $response->assertStatus(201)
                ->assertJson([
                    'success' => true,
                    'message' => 'Pet application submitted successfully',
                ])
                ->assertJsonStructure([
                    'data' => [
                        'id',
                        'tenant_name',
                        'tenant_email',
                        'status',
                        'created_at',
                    ],
                ]);

        $this->assertDatabaseHas('pet_applications', [
            'tenant_email' => 'john@example.com',
            'status' => 'pending',
        ]);

        $this->assertDatabaseHas('pets', [
            'name' => 'Buddy',
            'type' => 'dog',
        ]);
    }

    /**
     * AC2: Multiple pets can be submitted in single application
     */
    public function test_user_can_submit_multiple_pets(): void
    {
        // Arrange
        $property = Property::factory()->create();
        $validData = [
            'property_id' => $property->id,
            'tenant_name' => 'Jane Doe',
            'tenant_email' => 'jane@example.com',
            'pets' => [
                ['name' => 'Buddy', 'type' => 'dog', 'breed' => 'Labrador'],
                ['name' => 'Whiskers', 'type' => 'cat', 'breed' => 'Persian'],
            ],
            'general_consent' => true,
        ];

        // Act
        $response = $this->postJson('/api/pet-applications', $validData);

        // Assert
        $response->assertStatus(201);
        $this->assertDatabaseHas('pets', ['name' => 'Buddy']);
        $this->assertDatabaseHas('pets', ['name' => 'Whiskers']);
        $this->assertCount(2, $response->json()['data']['pets']);
    }
}
```

### Playwright E2E Test Example

```javascript
// tests/e2e/pet-application-positive.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Pet Application - Happy Path', () => {
  test('user submits pet application successfully', async ({ page }) => {
    // Navigate to application form
    await page.goto('http://localhost:3000/pet-applications');

    // Fill out form
    await page.fill('[data-testid="tenant-name"]', 'John Doe');
    await page.fill('[data-testid="tenant-email"]', 'john@example.com');
    await page.fill('[data-testid="tenant-phone"]', '0412345678');

    // Add pet
    await page.click('[data-testid="add-pet-button"]');
    await page.fill('[data-testid="pet-name-0"]', 'Buddy');
    await page.selectOption('[data-testid="pet-type-0"]', 'dog');

    // Accept consent
    await page.check('[data-testid="consent-checkbox"]');

    // Submit
    await page.click('[data-testid="submit-button"]');

    // Assert success message appears
    await expect(page.locator('[data-testid="success-message"]')).toContainText(
      'Application submitted successfully'
    );

    // Assert redirected to confirmation page
    await expect(page).toHaveURL('http://localhost:3000/applications/*');
  });
});
```

---

## 2. NEGATIVE TESTS (Error Cases - Things Go Wrong)

**Purpose:** Verify the system handles errors gracefully

**Test each invalid scenario:**

### Required Field Validation

```php
public function test_validation_fails_missing_property_id(): void
{
    $invalidData = [
        // Missing property_id
        'tenant_name' => 'John',
        'tenant_email' => 'john@example.com',
        'pets' => [['name' => 'Buddy', 'type' => 'dog']],
    ];

    $response = $this->postJson('/api/pet-applications', $invalidData);

    $response->assertStatus(422)
            ->assertJsonValidationErrors(['property_id']);
}

public function test_validation_fails_invalid_email(): void
{
    $property = Property::factory()->create();
    $invalidData = [
        'property_id' => $property->id,
        'tenant_name' => 'John',
        'tenant_email' => 'not-an-email', // Invalid email
        'pets' => [['name' => 'Buddy', 'type' => 'dog']],
    ];

    $response = $this->postJson('/api/pet-applications', $invalidData);

    $response->assertStatus(422)
            ->assertJsonValidationErrors(['tenant_email']);
}

public function test_validation_fails_invalid_uuid(): void
{
    $invalidData = [
        'property_id' => 'not-a-uuid', // Invalid UUID
        'tenant_name' => 'John',
        'tenant_email' => 'john@example.com',
        'pets' => [['name' => 'Buddy', 'type' => 'dog']],
    ];

    $response = $this->postJson('/api/pet-applications', $invalidData);

    $response->assertStatus(422)
            ->assertJsonValidationErrors(['property_id']);
}

public function test_validation_fails_nonexistent_property(): void
{
    $invalidData = [
        'property_id' => 'f47ac10b-58cc-4372-a567-0e02b2c3d479', // Valid UUID, doesn't exist
        'tenant_name' => 'John',
        'tenant_email' => 'john@example.com',
        'pets' => [['name' => 'Buddy', 'type' => 'dog']],
    ];

    $response = $this->postJson('/api/pet-applications', $invalidData);

    $response->assertStatus(422)
            ->assertJsonValidationErrors(['property_id']);
}

public function test_validation_fails_too_many_pets(): void
{
    $property = Property::factory()->create();
    $invalidData = [
        'property_id' => $property->id,
        'tenant_name' => 'John',
        'tenant_email' => 'john@example.com',
        'pets' => array_fill(0, 6, ['name' => 'Pet', 'type' => 'dog']), // 6 pets, max 5
    ];

    $response = $this->postJson('/api/pet-applications', $invalidData);

    $response->assertStatus(422)
            ->assertJsonValidationErrors(['pets']);
}

public function test_validation_fails_no_pets(): void
{
    $property = Property::factory()->create();
    $invalidData = [
        'property_id' => $property->id,
        'tenant_name' => 'John',
        'tenant_email' => 'john@example.com',
        'pets' => [], // Empty array
    ];

    $response = $this->postJson('/api/pet-applications', $invalidData);

    $response->assertStatus(422)
            ->assertJsonValidationErrors(['pets']);
}
```

### Security Tests

```php
public function test_rate_limiting_enforced(): void
{
    $property = Property::factory()->create();
    $validData = [
        'property_id' => $property->id,
        'tenant_name' => 'John',
        'tenant_email' => 'john@example.com',
        'pets' => [['name' => 'Buddy', 'type' => 'dog']],
    ];

    // Make 10 requests (should all succeed)
    for ($i = 0; $i < 10; $i++) {
        $response = $this->postJson('/api/pet-applications', $validData);
        $this->assertEquals(201, $response->status());
    }

    // 11th request should be rate limited
    $response = $this->postJson('/api/pet-applications', $validData);
    $this->assertEquals(429, $response->status()); // Too Many Requests
}

public function test_sql_injection_attempt_blocked(): void
{
    $injectedData = [
        'property_id' => "' OR '1'='1",
        'tenant_name' => 'John',
        'tenant_email' => 'john@example.com',
        'pets' => [['name' => 'Buddy', 'type' => 'dog']],
    ];

    $response = $this->postJson('/api/pet-applications', $injectedData);

    // Should fail validation, not execute injection
    $this->assertEquals(422, $response->status());
}

public function test_csrf_protection(): void
{
    $response = $this->post('/api/pet-applications', []);

    // Should require CSRF token (or return 403)
    $this->assertIn($response->status(), [403, 419]);
}
```

### Edge Cases

```php
public function test_whitespace_handling(): void
{
    $property = Property::factory()->create();
    $validData = [
        'property_id' => $property->id,
        'tenant_name' => '   John Doe   ', // Leading/trailing spaces
        'tenant_email' => 'john@example.com',
        'pets' => [['name' => 'Buddy', 'type' => 'dog']],
    ];

    $response = $this->postJson('/api/pet-applications', $validData);

    // Should succeed and trim whitespace
    $response->assertStatus(201);
    $this->assertDatabaseHas('pet_applications', [
        'tenant_name' => 'John Doe', // Trimmed
    ]);
}

public function test_special_characters_in_names(): void
{
    $property = Property::factory()->create();
    $validData = [
        'property_id' => $property->id,
        'tenant_name' => "O'Connor-Smith", // Special characters
        'tenant_email' => 'john@example.com',
        'pets' => [['name' => "D'Artagnan", 'type' => 'dog']],
    ];

    $response = $this->postJson('/api/pet-applications', $validData);

    $response->assertStatus(201);
    $this->assertDatabaseHas('pet_applications', [
        'tenant_name' => "O'Connor-Smith",
    ]);
}

public function test_unicode_characters_handled(): void
{
    $property = Property::factory()->create();
    $validData = [
        'property_id' => $property->id,
        'tenant_name' => 'José García', // Unicode
        'tenant_email' => 'jose@example.com',
        'pets' => [['name' => '小明', 'type' => 'dog']], // Chinese characters
    ];

    $response = $this->postJson('/api/pet-applications', $validData);

    $response->assertStatus(201);
}
```

---

## 3. NON-FUNCTIONAL TESTS

### Performance Tests

```php
public function test_list_pets_performance(): void
{
    // Create 1000 pet applications
    PetApplication::factory()->count(1000)->create();

    $startTime = microtime(true);
    $response = $this->getJson('/api/pet-applications');
    $endTime = microtime(true);

    $response->assertStatus(200);

    $elapsedTime = ($endTime - $startTime) * 1000; // Convert to ms
    $this->assertLessThan(200, $elapsedTime, "Query should complete in < 200ms, took {$elapsedTime}ms");
}

public function test_no_n_plus_one_queries(): void
{
    PetApplication::factory()->count(10)->has(Pet::factory()->count(3))->create();

    DB::enableQueryLog();

    $response = $this->getJson('/api/pet-applications');

    $queries = DB::getQueryLog();

    // Should be ~2 queries: 1 for applications, 1 for all pets (eager loaded)
    // Not 1 + 10 = 11 queries (1 for apps + 1 per app for pets)
    $this->assertLessThan(5, count($queries), "Too many queries: " . count($queries));
}
```

### Security Tests

```php
public function test_passwords_hashed(): void
{
    User::create([
        'email' => 'test@example.com',
        'password' => 'plaintext-password', // Laravel auto-hashes
    ]);

    $user = User::first();

    // Password should be hashed, not plaintext
    $this->assertNotEquals('plaintext-password', $user->password);
    // Should match when hashed
    $this->assertTrue(Hash::check('plaintext-password', $user->password));
}

public function test_sensitive_data_not_in_logs(): void
{
    Log::shouldReceive('error')->andReturnUsing(function ($message, $context) {
        // Check that password/token not in logs
        $this->assertNotContains('password', json_encode($context));
        $this->assertNotContains('token', json_encode($context));
    });

    // Trigger error with sensitive data
    $this->postJson('/api/login', [
        'email' => 'test@example.com',
        'password' => 'secret-password',
    ]);
}
```

### Load Tests

```php
public function test_concurrent_submissions(): void
{
    $property = Property::factory()->create();
    $validData = [
        'property_id' => $property->id,
        'tenant_name' => 'John',
        'tenant_email' => 'john@example.com',
        'pets' => [['name' => 'Buddy', 'type' => 'dog']],
    ];

    // Simulate 100 concurrent requests
    $startTime = microtime(true);

    for ($i = 0; $i < 100; $i++) {
        $response = $this->postJson('/api/pet-applications', $validData);
        $this->assertEquals(201, $response->status());
    }

    $elapsedTime = (microtime(true) - $startTime);

    // Should complete in reasonable time
    $this->assertLessThan(10, $elapsedTime, "100 requests should complete in < 10s, took {$elapsedTime}s");
}
```

### Accessibility Tests (with Playwright)

```javascript
// tests/e2e/pet-application-accessibility.spec.ts
import { test, expect } from '@playwright/test';

test('form is keyboard navigable', async ({ page }) => {
  await page.goto('http://localhost:3000/pet-applications');

  // Tab through form fields
  await page.keyboard.press('Tab');
  await expect(page.locator('[data-testid="tenant-name"]')).toBeFocused();

  await page.keyboard.press('Tab');
  await expect(page.locator('[data-testid="tenant-email"]')).toBeFocused();

  // Can submit with keyboard
  await page.keyboard.press('Tab');
  await page.keyboard.press('Tab');
  await page.keyboard.press('Tab');
  await page.keyboard.press('Enter'); // Submit button

  await expect(page.locator('[data-testid="success-message"]')).toBeVisible();
});
```

---

## 4. REGRESSION TESTS

**Purpose:** Ensure existing features still work after changes

```php
public function test_existing_pet_applications_still_visible(): void
{
    // Setup: Create existing application before change
    $oldApplication = PetApplication::factory()->create(['status' => 'pending']);

    // Implement new feature (e.g., add approval workflow)
    // ...

    // Verify old data still works
    $response = $this->getJson("/api/pet-applications/{$oldApplication->id}");
    $response->assertStatus(200);
    $response->assertJson(['data' => ['id' => $oldApplication->id]]);
}

public function test_existing_api_v1_still_works(): void
{
    // If adding v2 API, v1 should still function
    $response = $this->getJson('/api/v1/pet-applications');
    $response->assertStatus(200); // Not 404
}
```

---

## Playwright UI Testing Checklist

```javascript
// tests/e2e/pet-application-ui.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Pet Application Form UI', () => {
  test('form displays correctly', async ({ page }) => {
    await page.goto('http://localhost:3000/pet-applications');

    // Check form elements exist
    await expect(page.locator('[data-testid="tenant-name"]')).toBeVisible();
    await expect(page.locator('[data-testid="tenant-email"]')).toBeVisible();
    await expect(page.locator('[data-testid="add-pet-button"]')).toBeVisible();

    // Check form labels
    await expect(page.locator('label:has-text("Tenant Name")')).toBeVisible();
  });

  test('error messages display correctly', async ({ page }) => {
    await page.goto('http://localhost:3000/pet-applications');

    // Submit empty form
    await page.click('[data-testid="submit-button"]');

    // Check error messages
    await expect(page.locator('text=Tenant name is required')).toBeVisible();
    await expect(page.locator('text=Email is required')).toBeVisible();
  });

  test('responsive design on mobile', async ({ page }) => {
    // Set mobile viewport
    await page.setViewportSize({ width: 375, height: 667 });

    await page.goto('http://localhost:3000/pet-applications');

    // Form should still be usable
    await page.fill('[data-testid="tenant-name"]', 'John');
    await expect(page.locator('[data-testid="tenant-name"]')).toHaveValue('John');
  });

  test('FLKitOver integration works', async ({ page }) => {
    await page.goto('http://localhost:3000/pet-applications');

    // Fill and submit form
    await page.fill('[data-testid="tenant-name"]', 'John');
    await page.fill('[data-testid="tenant-email"]', 'john@example.com');
    await page.click('[data-testid="add-pet-button"]');
    await page.fill('[data-testid="pet-name-0"]', 'Buddy');
    await page.selectOption('[data-testid="pet-type-0"]', 'dog');
    await page.check('[data-testid="consent-checkbox"]');
    await page.click('[data-testid="submit-button"]');

    // Should redirect to confirmation
    await expect(page).toHaveURL(/\/applications\/\w+/);

    // Confirmation page should show FLK document link
    await expect(page.locator('[data-testid="flk-document-link"]')).toBeVisible();
  });
});
```

---

## Test Execution Command

```bash
# Run all tests
php artisan test

# Run specific test class
php artisan test --filter PetApplicationTest

# Run with coverage
php artisan test --coverage

# Run Playwright tests
npx playwright test

# Run Playwright tests with UI
npx playwright test --ui
```

---

## Bug Reporting Format

When tests fail, document clearly:

```markdown
## Bug Report: Rate Limiting Not Enforced

**Severity:** Critical (Security issue)
**Environment:** Testing
**Component:** PetApplicationController

**Steps to Reproduce:**
1. Send POST request to /api/pet-applications
2. Repeat 11 times in rapid succession
3. Observe response on request #11

**Expected Behavior:**
- Request #1-10: Status 201 (Created)
- Request #11: Status 429 (Too Many Requests)

**Actual Behavior:**
- All 11 requests return Status 201 (Rate limit not working)

**Evidence:**
```
Request #10: HTTP 201
Request #11: HTTP 201 (Should be 429!)
```

**Impact:** Attackers can spam submissions without rate limit protection

**Suggested Fix:** Verify rate limiting middleware is applied to route
```

---

## Test Quality Checklist

- [ ] Positive tests: Happy path works
- [ ] Negative tests: All error cases handled
- [ ] Security tests: No injection/bypass attacks
- [ ] Performance tests: Response times acceptable
- [ ] Regression tests: Existing features still work
- [ ] All 4 categories covered for every feature
- [ ] Tests are repeatable (no flaky tests)
- [ ] Test data is isolated (no cross-test pollution)
- [ ] Bug reports are specific and reproducible

---

## Key Reminders

⚠️ **All 4 test categories required** - Don't skip regression or non-functional
⚠️ **Test actual constraints** - Not just "code exists"
⚠️ **Document failures clearly** - Must be reproducible
⚠️ **Test at API level AND UI level** - Backend and frontend
⚠️ **Automate everything** - Manual testing is slower and error-prone
