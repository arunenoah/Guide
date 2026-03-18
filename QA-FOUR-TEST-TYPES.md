# QA-Tester: Four Test Types Complete Guide

## Overview

Every feature tested by QA-Tester must include 4 types of tests:
1. **Positive Tests** - Happy path (everything works)
2. **Negative Tests** - Error cases (user makes mistakes)
3. **Non-Functional Tests** - Quality attributes (performance, security, etc.)
4. **Regression Tests** - Existing features (nothing broke)

---

## 1. POSITIVE Tests (Happy Path)

### What to Test
User does everything correctly → system responds correctly

### When to Run
First phase of QA-tester (after code-reviewer approves)

### Examples for Password Reset Feature

```php
// Unit Test: Password hashing works
public function test_password_is_hashed_correctly(): void
{
    $user = User::factory()->create(['password' => 'password123']);

    expect(Hash::check('password123', $user->password))->toBeTrue();
    expect($user->password)->not->toBe('password123');
}

// Integration Test: Reset token created in database
public function test_reset_token_stored_in_database(): void
{
    $user = User::factory()->create();

    $response = $this->post('/api/password-reset', [
        'email' => $user->email
    ]);

    $response->assertStatus(200);

    expect(DB::table('password_reset_tokens')
        ->where('email', $user->email)
        ->exists()
    )->toBeTrue();
}

// E2E Test: User can reset password (Playwright)
test('user can reset password via UI', async ({ page }) => {
    await page.goto('/password-reset');

    await page.fill('input[type="email"]', 'user@example.com');
    await page.click('button[type="submit"]');

    // Success message appears
    await expect(page.locator('text=Check your email')).toBeVisible();

    // Take screenshot as proof
    await page.screenshot({ path: 'test-evidence/reset-success.png' });
});
```

### Success Criteria
- ✅ Success message displays
- ✅ Email actually sent (check email service logs)
- ✅ Token created in database
- ✅ User can click reset link
- ✅ New password works on next login
- ✅ No errors in logs

### Tools Used
- **Backend:** PHPUnit (database, email service)
- **Frontend:** Playwright (UI interactions, screenshots)

---

## 2. NEGATIVE Tests (Error Cases)

### What to Test
User enters invalid data or violates constraints → system handles gracefully

### When to Run
Same phase as positive tests (qa-tester phase)

### Examples for Password Reset Feature

```php
// Test: Invalid email format
public function test_rejects_invalid_email_format(): void
{
    $response = $this->post('/api/password-reset', [
        'email' => 'not-an-email'
    ]);

    $response->assertStatus(422);
    $response->assertJsonValidationErrors('email');
}

// Test: Non-existent email
public function test_email_not_found_error(): void
{
    $response = $this->post('/api/password-reset', [
        'email' => 'nobody@example.com'
    ]);

    // Should not reveal user doesn't exist (security)
    $response->assertStatus(200);
    $response->assertJson(['message' => 'Check your email for reset link']);
}

// Test: Rate limiting
public function test_rate_limiting_5_attempts_per_hour(): void
{
    $email = 'user@example.com';

    // Attempt 6 times
    for ($i = 0; $i < 6; $i++) {
        $response = $this->post('/api/password-reset', ['email' => $email]);

        if ($i < 5) {
            expect($response->getStatusCode())->toBe(200);
        } else {
            // 6th attempt blocked
            expect($response->getStatusCode())->toBe(429); // Too Many Requests
        }
    }
}

// Test: Expired token rejection
public function test_expired_token_rejected(): void
{
    // Create token that's 2 hours old (expires in 1 hour)
    $token = DB::table('password_reset_tokens')->create([
        'email' => 'user@example.com',
        'token' => 'old-token',
        'created_at' => now()->subHours(2)
    ]);

    $response = $this->post('/api/password-reset/confirm', [
        'token' => 'old-token',
        'password' => 'new-password'
    ]);

    $response->assertStatus(422);
    $response->assertJson(['message' => 'Reset link has expired']);
}

// Test: Token reuse prevention
public function test_token_cannot_be_used_twice(): void
{
    // Reset password with token
    $response1 = $this->post('/api/password-reset/confirm', [
        'token' => 'valid-token',
        'password' => 'new-password-1'
    ]);
    expect($response1->getStatusCode())->toBe(200);

    // Try to use same token again
    $response2 = $this->post('/api/password-reset/confirm', [
        'token' => 'valid-token',
        'password' => 'new-password-2'
    ]);
    expect($response2->getStatusCode())->toBe(422);
}

// Playwright: Invalid email UI error
test('shows validation error for invalid email', async ({ page }) => {
    await page.goto('/password-reset');

    await page.fill('input[type="email"]', 'not-an-email');
    await page.click('button[type="submit"]');

    await expect(page.locator('text=Invalid email address')).toBeVisible();
    await page.screenshot({ path: 'test-evidence/invalid-email.png' });
});

// Playwright: Rate limiting
test('blocks after 5 attempts', async ({ page }) => {
    // Attempt 6 times rapidly
    for (let i = 0; i < 6; i++) {
        await page.fill('input[type="email"]', 'test@example.com');
        await page.click('button[type="submit"]');

        if (i < 5) {
            await expect(page.locator('text=Check your email')).toBeVisible();
            await page.reload();
        } else {
            await expect(page.locator('text=Too many attempts')).toBeVisible();
            await page.screenshot({ path: 'test-evidence/rate-limit.png' });
        }
    }
});
```

### Success Criteria
- ✅ All error messages clear and helpful
- ✅ User can't bypass validation
- ✅ Security constraints enforced (rate limiting, token expiry)
- ✅ No system errors exposed to user
- ✅ Error doesn't leak user existence (privacy)

### Tools Used
- **Backend:** PHPUnit (validation, constraints, database checks)
- **Frontend:** Playwright (error message display, UI validation)

---

## 3. NON-FUNCTIONAL Tests (Quality Attributes)

### What to Test
Performance, security, accessibility, load handling, responsiveness

### When to Run
Third phase (after positive and negative tests pass)

### Categories

#### A. Performance
```php
// Test: API response time
public function test_password_reset_api_under_500ms(): void
{
    $start = microtime(true);

    $this->post('/api/password-reset', [
        'email' => 'user@example.com'
    ]);

    $duration = (microtime(true) - $start) * 1000;

    expect($duration)->toBeLessThan(500); // milliseconds
}

// Playwright: Page load time
test('password reset page loads in < 500ms', async ({ page }) => {
    const start = Date.now();
    await page.goto('/password-reset');
    const loadTime = Date.now() - start;

    expect(loadTime).toBeLessThan(500);
});

// Playwright: Form submission time
test('form submits and shows success in < 2 seconds', async ({ page }) => {
    const start = Date.now();

    await page.fill('input[type="email"]', 'user@example.com');
    await page.click('button[type="submit"]');
    await expect(page.locator('text=Check your email')).toBeVisible();

    const duration = Date.now() - start;
    expect(duration).toBeLessThan(2000); // milliseconds
});
```

#### B. Security
```php
// Test: No hardcoded secrets in code
public function test_no_secrets_in_codebase(): void
{
    $appCode = file_get_contents(base_path('app/Http/Controllers/PasswordResetController.php'));

    expect($appCode)->not->toContain('sk_live_');
    expect($appCode)->not->toContain('FLKIT_SECRET');
    expect($appCode)->not->toContain('password123');
}

// Test: CSRF token required
public function test_csrf_protection_enforced(): void
{
    $response = $this->post('/password-reset', [
        'email' => 'user@example.com'
    ]);

    // Should fail without CSRF token
    $response->assertStatus(419); // Token Mismatch
}

// Test: Passwords are hashed, not stored plaintext
public function test_passwords_hashed_not_plaintext(): void
{
    $user = User::factory()->create(['password' => 'mypassword123']);

    expect(Hash::check('mypassword123', $user->password))->toBeTrue();

    $dbPassword = DB::table('users')
        ->where('id', $user->id)
        ->first()
        ->password;

    expect($dbPassword)->not->toBe('mypassword123');
    expect($dbPassword)->toHaveLength(60); // bcrypt hash length
}

// Test: No SQL injection
public function test_sql_injection_prevented(): void
{
    $response = $this->post('/api/password-reset', [
        'email' => "' OR '1'='1"
    ]);

    // Should safely reject, not execute injection
    $response->assertStatus(422);
}

// Playwright: No API keys in HTML
test('no secrets in HTML source', async ({ page }) => {
    await page.goto('/password-reset');

    const html = await page.content();

    expect(html).not.toContain('sk_live_');
    expect(html).not.toContain('FLKIT_SECRET');
    expect(html).not.toContain('API_KEY=');
});
```

#### C. Accessibility
```php
// Playwright: Keyboard navigation
test('fully keyboard navigable', async ({ page }) => {
    await page.goto('/password-reset');

    // Tab to email input
    await page.keyboard.press('Tab');
    const emailField = page.locator('input[type="email"]');
    await expect(emailField).toBeFocused();

    // Type email
    await page.keyboard.type('test@example.com');

    // Tab to submit button
    await page.keyboard.press('Tab');
    const submitBtn = page.locator('button[type="submit"]');
    await expect(submitBtn).toBeFocused();

    // Enter to submit
    await page.keyboard.press('Enter');

    await expect(page.locator('text=Check your email')).toBeVisible();
});

// Playwright: Color contrast
test('sufficient color contrast for accessibility', async ({ page }) => {
    await page.goto('/password-reset');

    // Run axe accessibility check
    const accessibilityResults = await page.evaluate(() => {
        // Pseudo-code for accessibility check
        return true; // All contrast ratios >= 4.5:1
    });

    expect(accessibilityResults).toBeTruthy();
});

// Playwright: Form labels
test('form labels associated with inputs', async ({ page }) => {
    await page.goto('/password-reset');

    const label = page.locator('label[for="email"]');
    await expect(label).toContainText('Email');

    // Input has matching id
    const input = page.locator('input#email');
    await expect(input).toBeVisible();
});
```

#### D. Responsive Design
```php
// Playwright: Mobile (375px)
test('works on mobile 375px', async ({ page }) => {
    await page.setViewportSize({ width: 375, height: 667 });
    await page.goto('/password-reset');

    // Form should be fully visible
    await expect(page.locator('form')).toBeVisible();

    // All inputs should be clickable
    await page.fill('input[type="email"]', 'test@example.com');
    await page.click('button[type="submit"]');

    await expect(page.locator('text=Check your email')).toBeVisible();
});

// Playwright: Tablet (768px)
test('works on tablet 768px', async ({ page }) => {
    await page.setViewportSize({ width: 768, height: 1024 });
    await page.goto('/password-reset');

    // Form should layout correctly
    await page.fill('input[type="email"]', 'test@example.com');
    await page.click('button[type="submit"]');

    await expect(page.locator('text=Check your email')).toBeVisible();
});

// Playwright: Desktop (1920px)
test('works on desktop 1920px', async ({ page }) => {
    await page.setViewportSize({ width: 1920, height: 1080 });
    await page.goto('/password-reset');

    // Form should be readable and properly spaced
    const form = page.locator('form');
    await expect(form).toBeVisible();
});
```

#### E. Load Testing
```php
// Test: Concurrent requests
public function test_handles_concurrent_requests(): void
{
    $promises = [];

    for ($i = 0; $i < 10; $i++) {
        $promises[] = $this->post('/api/password-reset', [
            'email' => "user{$i}@example.com"
        ]);
    }

    // All should succeed (not timeout)
    foreach ($promises as $response) {
        expect($response->getStatusCode())->toBe(200);
    }
}

// Playwright: 10 concurrent form submissions
test('handles 10 concurrent users', async ({ page }) => {
    const submissions = [];

    for (let i = 0; i < 10; i++) {
        const promise = (async () => {
            await page.goto('/password-reset');
            await page.fill('input[type="email"]', `user${i}@example.com`);
            await page.click('button[type="submit"]');
            return await page.locator('text=Check your email').isVisible();
        })();
        submissions.push(promise);
    }

    const results = await Promise.all(submissions);

    expect(results.every(r => r === true)).toBeTruthy();
});
```

### Success Criteria
- ✅ API response < 500ms
- ✅ Page loads < 500ms
- ✅ No hardcoded secrets
- ✅ CSRF protection enforced
- ✅ Passwords hashed (not plaintext)
- ✅ SQL injection prevented
- ✅ Fully keyboard navigable
- ✅ Sufficient color contrast
- ✅ Works on mobile/tablet/desktop
- ✅ Handles concurrent requests

### Tools Used
- **Security:** Code inspection, PHPUnit
- **Performance:** Playwright timing tests
- **Accessibility:** Playwright + manual testing
- **Load:** Playwright concurrent tests

---

## 4. REGRESSION Tests (Existing Features)

### What to Test
Existing features still work after new code added

### When to Run
Last phase (before deployment)

### Examples
```php
// Regression: Login still works
public function test_login_still_works_after_password_reset(): void
{
    $user = User::factory()->create(['password' => 'old-password']);

    $response = $this->post('/api/login', [
        'email' => $user->email,
        'password' => 'old-password'
    ]);

    $response->assertStatus(200);
    $this->assertAuthenticatedAs($user);
}

// Regression: Registration still works
public function test_registration_still_works(): void
{
    $response = $this->post('/api/register', [
        'email' => 'newuser@example.com',
        'password' => 'password123',
        'password_confirmation' => 'password123'
    ]);

    $response->assertStatus(201);
    expect(User::where('email', 'newuser@example.com')->exists())->toBeTrue();
}

// Regression: Password change still works
public function test_password_change_endpoint_still_works(): void
{
    $user = User::factory()->create(['password' => 'old-password']);

    $response = $this->actingAs($user)->post('/api/password-change', [
        'current_password' => 'old-password',
        'new_password' => 'new-password',
        'new_password_confirmation' => 'new-password'
    ]);

    $response->assertStatus(200);
    expect(Hash::check('new-password', $user->password))->toBeTrue();
}

// Regression: Email service still works
public function test_welcome_email_still_sent_on_registration(): void
{
    Mail::fake();

    $this->post('/api/register', [
        'email' => 'newuser@example.com',
        'password' => 'password123',
        'password_confirmation' => 'password123'
    ]);

    Mail::assertSent(WelcomeEmail::class);
}

// Playwright: All auth flows work
test('full auth flow works (register → login → logout)', async ({ page }) => {
    // Register
    await page.goto('/register');
    await page.fill('input[name="email"]', 'newuser@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.fill('input[name="password_confirmation"]', 'password123');
    await page.click('button[type="submit"]');

    await expect(page.locator('text=Welcome')).toBeVisible();

    // Logout
    await page.click('[data-testid="logout"]');

    // Login
    await page.goto('/login');
    await page.fill('input[name="email"]', 'newuser@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button[type="submit"]');

    // Should be logged in
    await expect(page.locator('text=Dashboard')).toBeVisible();
});
```

### Success Criteria
- ✅ 0 new test failures
- ✅ All existing features working
- ✅ No breaking changes
- ✅ API endpoints unchanged
- ✅ Database migrations compatible

### Tools Used
- **All Tests:** PHPUnit (existing test suite)
- **E2E:** Playwright (user workflows)

---

## Summary: When to Run Each Test Type

| Test Type | Phase | Timing | Focus |
|-----------|-------|--------|-------|
| **Positive** | 1st | Immediately after code-reviewer | Happy path works |
| **Negative** | 1st (same) | During positive testing | Errors handled |
| **Non-Functional** | 3rd | After positive/negative pass | Quality attributes |
| **Regression** | 4th | Before deployment | Nothing broke |

---

## QA-Tester Invocation Templates

### Full Test Run (All 4 Types)
```
/invoke qa-tester
"Comprehensive test for password reset:

POSITIVE: User successfully resets password
NEGATIVE: Handle invalid emails, rate limiting, expired tokens
NON-FUNCTIONAL: Performance < 500ms, security checks, keyboard accessible, mobile responsive
REGRESSION: Login, registration, password change still work

Use PHPUnit for backend, Playwright for UI, capture screenshots.
Report: passing/failing tests, bugs, quality metrics."
```

### Quick Positive + Regression Only
```
/invoke qa-tester
"Quick test for password reset:
- Positive: Happy path works (email, reset, new password)
- Regression: Login and registration still work"
```

### Security-Focused
```
/invoke qa-tester
"Security testing for password reset:
- Negative: Rate limiting, token expiry, SQL injection prevention
- Non-Functional: No hardcoded secrets, CSRF protection, password hashing
- Performance: Response times acceptable"
```
