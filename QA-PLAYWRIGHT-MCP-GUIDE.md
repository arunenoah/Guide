# QA-Tester + Playwright MCP Integration Guide

## Why Playwright MCP for QA-Tester?

**The Problem Without Playwright:**
- QA-tester can only run unit/integration tests (PHPUnit)
- Can't validate UI interactions end-to-end
- Can't generate visual evidence (screenshots)
- Misses bugs that only show up in browser
- No way to generate test reports with visual proof

**The Solution: Playwright MCP**
- Run end-to-end tests against real browser
- Capture screenshots as evidence of pass/fail
- Validate user workflows: login → actions → logout
- Generate test reports with visual documentation
- Test API + UI together, not separately

## What Playwright MCP Can Do

### 1. **Run E2E Tests**
```javascript
// Playwright can run tests that PHP/Laravel tests can't
test('user can reset password', async ({ page }) => {
  // Navigate to login page
  await page.goto('/login');

  // Click "Forgot Password"
  await page.click('[data-testid="forgot-password"]');

  // Fill in email
  await page.fill('input[type="email"]', 'test@example.com');

  // Submit
  await page.click('button[type="submit"]');

  // Verify success message appears
  await expect(page.locator('text=Check your email')).toBeVisible();
});
```

### 2. **Generate Visual Evidence**
```javascript
// Capture screenshots for documentation
await page.screenshot({ path: 'screenshots/password-reset-success.png' });

// Video recording of test flow
page.on('video', video => video.saveAs('test-videos/reset-flow.webm'));

// Full page snapshots
expect(page).toHaveScreenshot('password-reset-complete.png');
```

### 3. **Validate Responsive Design**
```javascript
// Test on mobile, tablet, desktop
test('works on mobile', async ({ page }) => {
  await page.setViewportSize({ width: 375, height: 667 });
  // Test mobile flow
});

test('works on tablet', async ({ page }) => {
  await page.setViewportSize({ width: 768, height: 1024 });
  // Test tablet flow
});
```

### 4. **Test FLKitOver Integration Visually**
```javascript
// Validate FLKitOver form workflow
test('can submit FLKitOver form in iframe', async ({ page }) => {
  // Navigate to pet application form
  await page.goto('/pet-application');

  // Fill form fields
  await page.fill('input[name="property_id"]', 'uuid');
  await page.fill('input[name="tenant_name"]', 'John Doe');

  // Validate FLKitOver fields appear
  const flkForm = page.frameLocator('iframe[name="flk-form"]');
  await expect(flkForm.locator('[name="document_type"]')).toBeVisible();

  // Submit and verify success
  await page.click('button[type="submit"]');
  await expect(page.locator('text=Application submitted')).toBeVisible();

  // Screenshot as evidence
  await page.screenshot({ path: 'test-evidence/flk-submission.png' });
});
```

### 5. **Generate Test Reports**
```javascript
// Playwright generates HTML reports automatically
// Reports include:
// - Test results (pass/fail)
// - Execution time
// - Screenshots/videos
// - Stack traces for failures
// - Timeline of test execution
```

## QA-Tester Workflow with Playwright

### Phase 1: Manual Testing (Run Tests)
```bash
# QA-Tester invoked by tech-lead after code-reviewer approves

QA-Tester does:
1. Run PHPUnit tests (back-end logic)
2. Run Playwright tests (front-end + integration)
3. Run security tests (OWASP, injection, auth)
4. Validate FLKitOver integration
5. Test AWS service integrations (S3, SES)
6. Generate evidence: screenshots, videos, reports
```

### Phase 2: Report Bugs (If Tests Fail)
```markdown
## Bug Report: Password Reset Not Submitting

### Steps to Reproduce
1. Navigate to /password-reset
2. Enter email: test@example.com
3. Click "Send Reset Link"

### Expected
Success message appears, email is sent

### Actual
Form still shows, no error message, email not sent

### Evidence
- Screenshot: form-stuck.png
- Video: reset-flow-failure.webm
- Browser console logs attached

### Database Check
- Email 'test@example.com' exists in users table
- No reset tokens created
- No logs in Laravel log file
```

### Phase 3: Verify Fixes (With Playwright)
```javascript
// QA-Tester re-runs Playwright tests after engineer fixes code

test('password reset flow works end-to-end', async ({ page }) => {
  // Verify the bug is fixed
  await page.goto('/password-reset');
  await page.fill('input[type="email"]', 'test@example.com');
  await page.click('button[type="submit"]');

  // Should now see success message
  await expect(page.locator('text=Check your email')).toBeVisible();

  // Capture success evidence
  await page.screenshot({ path: 'test-evidence/fix-verified.png' });
});
```

## Setting Up Playwright for QA-Tester

### Installation
```bash
npm install -D @playwright/test

# Create playwright.config.ts
npx playwright codegen http://localhost:8000
```

### Configuration
```javascript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'], // Generate HTML report
    ['json'], // Machine-readable report
    ['junit'], // CI integration
  ],
  use: {
    baseURL: 'http://localhost:8000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  webServer: {
    command: 'php artisan serve',
    url: 'http://localhost:8000',
    reuseExistingServer: !process.env.CI,
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
});
```

### Example Test Structure
```
tests/e2e/
├── auth/
│   ├── login.spec.ts
│   ├── password-reset.spec.ts
│   └── logout.spec.ts
├── pet-application/
│   ├── submit-application.spec.ts
│   ├── flk-integration.spec.ts
│   └── file-upload.spec.ts
├── aws-integration/
│   ├── s3-upload.spec.ts
│   └── email-notification.spec.ts
└── fixtures/
    ├── auth.ts (logged-in user)
    └── data.ts (test data setup)
```

## QA-Tester Instructions (Use This)

When invoking qa-tester for testing:

```
/invoke qa-tester
"Test the password reset feature:

1. Run all PHPUnit tests (unit + integration)
2. Run Playwright e2e tests:
   - Test happy path: email → reset link → new password
   - Test error cases: invalid email, expired token, token reuse
   - Test security: rate limiting, CSRF protection
   - Capture screenshots for each major step
3. Check database state:
   - Password reset tokens created correctly
   - Tokens expire properly
   - Emails sent successfully
4. Validate email delivery (check SES/SendGrid logs)
5. Generate test report with evidence:
   - Playwright HTML report
   - Screenshots of key steps
   - Video of complete flow
6. Report any failures with:
   - Screenshot of failure
   - Exact steps to reproduce
   - Database state at time of failure
   - Browser console logs"
```

## What QA-Tester Should Check with Playwright

### Front-End (UI/UX)
- ✅ Buttons clickable, forms submittable
- ✅ Validation messages clear and helpful
- ✅ Loading states visible during operations
- ✅ Success/error messages appear correctly
- ✅ Responsive on mobile/tablet/desktop
- ✅ Keyboard navigation works (accessibility)

### Integration (Laravel Backend + UI)
- ✅ Form data saved to database
- ✅ Email notifications sent
- ✅ Redirects happen correctly
- ✅ Session management works
- ✅ CSRF protection doesn't block legitimate requests
- ✅ Rate limiting allows legitimate traffic

### Security (Playwright + Manual)
- ✅ Input validation prevents XSS
- ✅ SQL injection attempts fail
- ✅ Unauthorized access blocked
- ✅ Session tokens work correctly
- ✅ Sensitive data not exposed in HTML
- ✅ API endpoints require authentication

### FLKitOver Integration (Critical)
- ✅ FLKitOver form iframe loads
- ✅ Form data flows to FLKitOver API
- ✅ Document signature workflow completes
- ✅ Callbacks from FLKitOver processed correctly
- ✅ Errors from FLKitOver handled gracefully
- ✅ Screenshots captured at each FLKitOver step

## Test Report Template (QA-Tester Output)

```markdown
# Test Results: Password Reset Feature

## Summary
- **Total Tests:** 24
- **Passed:** 22 ✅
- **Failed:** 2 ❌
- **Skipped:** 0
- **Duration:** 2m 34s

## Unit Tests (PHPUnit)
- Password hashing: ✅
- Token generation: ✅
- Token validation: ✅
- Email sending: ✅

## Integration Tests (PHPUnit)
- Reset token stored in DB: ✅
- Email sent via SES: ✅
- Token expires correctly: ✅

## E2E Tests (Playwright)
- Happy path (email → link → new password): ✅
- Invalid email handling: ✅
- Expired token error: ❌ (See Bug #1)
- Token reuse prevention: ✅
- Mobile responsive layout: ✅
- Keyboard navigation: ✅

## Bugs Found
### Bug #1: Expired Token Error Message (Critical)
- **Steps:** Get reset link, wait 5+ minutes, click link
- **Expected:** "Token expired, request new one"
- **Actual:** Generic 404 page
- **Screenshot:** [test-evidence/expired-token.png]
- **Video:** [test-evidence/expired-flow.webm]
- **Fix by:** Senior-Engineer
- **Status:** Pending

### Bug #2: Email Not Sent on First Attempt (High)
- **Frequency:** 1/3 attempts (intermittent)
- **Evidence:** SES logs show 1 failed, 2 successful
- **Database:** All reset tokens created correctly
- **Investigation:** May be SES rate limiting or timeout
- **Fix by:** Senior-Engineer

## Database Checks
- Reset tokens created: ✅
- Token expiry set to 1 hour: ✅
- User email updated after reset: ✅
- Old passwords not readable in DB: ✅

## Security Validation
- Rate limiting (5 attempts/hour): ✅
- CSRF token required: ✅
- Email addresses not exposed: ✅
- Tokens unique and unpredictable: ✅

## Performance
- Form loads in < 500ms: ✅
- Email sent in < 2 seconds: ✅
- Page renders in < 1 second: ✅

## Recommendation
**NOT READY for UAT** - Fix 2 bugs first, then re-test with Playwright.
```

## Key Benefits of Playwright for QA-Tester

| Benefit | Impact |
|---------|--------|
| **Visual Evidence** | Test reports now include screenshots, videos, not just pass/fail |
| **Real Browser Testing** | Catches bugs PHPUnit can't (CSS issues, timing, race conditions) |
| **FLKitOver Validation** | Can actually test iframe interactions, document signing flows |
| **Faster Bug Reporting** | Screenshots + videos = engineers fix bugs faster |
| **Regression Prevention** | E2E tests prevent bugs from coming back |
| **Documentation** | Test videos serve as feature documentation |

## Summary

- ✅ **QA-Tester now has 4 MCPs:** MySQL, GitHub, Playwright, Context-Mode
- ✅ **Playwright enables:** E2E testing, visual evidence, UI validation, FLKitOver testing
- ✅ **QA-Tester can now:** Run backend tests + frontend tests + integration tests
- ✅ **Test reports include:** Screenshots, videos, database checks, security validation
- ✅ **Senior-Engineer gets:** Clear bug reports with exact reproduction steps and evidence
