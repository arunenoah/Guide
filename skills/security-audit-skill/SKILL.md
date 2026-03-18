---
name: security-audit-skill
description: Comprehensive security audit standards for Laravel applications covering OWASP Top 10, FLKitOver integrations, and AWS service security
---

# Security Audit Skill

You conduct thorough security audits that identify vulnerabilities before code reaches production.

## Security Audit Workflow

1. **Receive implementation** for audit
2. **Review code systematically** using OWASP Top 10 checklist
3. **Test security controls** manually
4. **Check configurations** for security defaults
5. **Document findings** with severity and remediation
6. **Provide specific fixes** with code examples
7. **Report completion** when all critical issues resolved

---

## OWASP Top 10 Web Application Vulnerabilities

### 1. BROKEN AUTHENTICATION

**Check:**
- Are passwords hashed (bcrypt/Argon2, not MD5)?
- Are sessions timeout configured?
- Are session tokens secure?
- Is MFA implemented if required?

```php
// ❌ VULNERABLE - Plaintext password storage
$user->update(['password' => $request->password]);

// ✅ SECURE - Password hashed by Laravel
$user->update(['password' => Hash::make($request->password)]);

// ❌ VULNERABLE - Weak hash
$password = md5($request->password);

// ✅ SECURE - Laravel default (Bcrypt)
$user = User::create([
    'email' => $request->email,
    'password' => Hash::make($request->password), // Automatically bcrypt
]);
```

**Session Security:**
```php
// config/session.php - Check settings
'driver' => 'database', // Use database, not cookies
'lifetime' => 120, // Minutes
'expire_on_close' => false,
'encrypt' => true, // Encrypt session data
'secure' => env('SESSION_SECURE_COOKIE', true), // HTTPS only
'http_only' => true, // No JavaScript access
'same_site' => 'lax', // CSRF protection
```

---

### 2. BROKEN ACCESS CONTROL (Authorization)

**Check:**
- Can users access resources they don't own?
- Are authorization checks present?
- Are policies/gates used?

```php
// ❌ VULNERABLE - No authorization check
public function delete($id)
{
    PetApplication::find($id)->delete(); // Anyone can delete anything
}

// ✅ SECURE - Authorization policy
public function delete($id)
{
    $application = PetApplication::findOrFail($id);
    $this->authorize('delete', $application); // Verify ownership
    $application->delete();
}

// Policy definition:
class PetApplicationPolicy
{
    public function delete(User $user, PetApplication $application): bool
    {
        return $user->id === $application->user_id; // Only owner can delete
    }
}
```

**Test Authorization:**
```php
public function test_user_cannot_delete_others_application(): void
{
    $owner = User::factory()->create();
    $attacker = User::factory()->create();

    $application = PetApplication::factory()->create(['user_id' => $owner->id]);

    // Attacker tries to delete owner's application
    $response = $this->actingAs($attacker)
                     ->deleteJson("/api/pet-applications/{$application->id}");

    // Should be forbidden
    $this->assertEquals(403, $response->status());

    // Application should still exist
    $this->assertDatabaseHas('pet_applications', ['id' => $application->id]);
}
```

---

### 3. INJECTION (SQL Injection, Command Injection, etc.)

**Check:**
- Are raw SQL queries used with user input?
- Are all queries parameterized?

```php
// ❌ VULNERABLE - SQL Injection
public function search($query)
{
    $sql = "SELECT * FROM users WHERE name LIKE '$query'";
    return DB::select($sql);
}

// ✅ SECURE - Eloquent prevents injection
public function search($query)
{
    return User::where('name', 'like', $query)->get();
}

// ✅ SECURE - If raw query needed, use bindings
public function search($query)
{
    return DB::select("SELECT * FROM users WHERE name LIKE ?", [$query]);
}
```

**Test SQL Injection:**
```php
public function test_sql_injection_prevented(): void
{
    $maliciousQuery = "' OR '1'='1";

    $response = $this->postJson('/api/search', ['query' => $maliciousQuery]);

    // Should fail gracefully, not execute injection
    $response->assertStatus(422); // Validation error
}
```

---

### 4. INSECURE DESIGN (Missing Security Requirements)

**Check:**
- Are security requirements defined?
- Is rate limiting implemented?
- Are sensitive operations protected?

```php
// ❌ INSECURE - No rate limiting
Route::post('/api/login', [LoginController::class, 'store']);

// ✅ SECURE - Rate limiting on login attempts
Route::post('/api/login', [LoginController::class, 'store'])
    ->middleware('throttle:5,1'); // 5 attempts per minute
```

**Rate Limiting Test:**
```php
public function test_rate_limiting_enforced_on_login(): void
{
    // Make 6 login attempts
    for ($i = 0; $i < 6; $i++) {
        $response = $this->postJson('/api/login', [
            'email' => 'test@example.com',
            'password' => 'password',
        ]);
    }

    // 6th request should be rate limited
    $this->assertEquals(429, $response->status()); // Too Many Requests
}
```

---

### 5. BROKEN ACCESS TO SENSITIVE DATA

**Check:**
- Are passwords/tokens exposed in API responses?
- Are PII fields visible to unauthorized users?
- Is sensitive data encrypted in transit (HTTPS)?

```php
// ❌ VULNERABLE - Password exposed in response
return response()->json($user); // Includes password hash!

// ✅ SECURE - Use API Resource to hide sensitive fields
return new UserResource($user);

// UserResource.php
public function toArray($request)
{
    return [
        'id' => $this->id,
        'email' => $this->email,
        'name' => $this->name,
        // NOT including 'password' or 'password_hash'
    ];
}
```

**Test Data Exposure:**
```php
public function test_password_not_exposed_in_api_response(): void
{
    $user = User::factory()->create(['password' => Hash::make('secret')]);

    $response = $this->getJson("/api/users/{$user->id}");

    // Response should NOT contain password
    $this->assertNotContains('password', json_encode($response->json()));
}
```

---

### 6. BROKEN OBJECT LEVEL AUTHORIZATION (IDOR)

**Insecure Direct Object Reference - accessing resources via URL/ID without permission check**

```php
// ❌ VULNERABLE - IDOR
public function show($applicationId)
{
    $application = PetApplication::find($applicationId);
    return response()->json($application); // Anyone can view any application
}

// ✅ SECURE - Authorization check
public function show($applicationId)
{
    $application = PetApplication::findOrFail($applicationId);
    $this->authorize('view', $application); // Verify ownership
    return response()->json($application);
}
```

**Test IDOR:**
```php
public function test_cannot_view_others_application(): void
{
    $owner = User::factory()->create();
    $attacker = User::factory()->create();

    $application = PetApplication::factory()->create(['user_id' => $owner->id]);

    // Attacker tries to view owner's application
    $response = $this->actingAs($attacker)
                     ->getJson("/api/pet-applications/{$application->id}");

    // Should be forbidden
    $this->assertEquals(403, $response->status());
}
```

---

### 7. CROSS-SITE SCRIPTING (XSS)

**Check:**
- Is user input escaped in HTML output?
- Are Laravel blade templates used (auto-escaping)?

```php
// ❌ VULNERABLE - XSS
<div>{{ $userName }}</div> <!-- Could contain <script> tags -->

// ✅ SECURE - Blade auto-escapes by default
<div>{{ $userName }}</div> <!-- Safe, automatically escaped -->

// ✅ EXPLICIT - If raw HTML needed
<div>{!! $safeHtml !!}</div> <!-- Only if you trust the content -->
```

**Test XSS Prevention:**
```php
public function test_xss_prevented_in_user_name(): void
{
    $xssPayload = '<script>alert("XSS")</script>';

    $response = $this->postJson('/api/pet-applications', [
        'tenant_name' => $xssPayload,
        'tenant_email' => 'test@example.com',
        'property_id' => Property::factory()->create()->id,
        'pets' => [['name' => 'Buddy', 'type' => 'dog']],
    ]);

    $response->assertStatus(201);

    // Check that script tags are escaped, not executed
    $this->assertDatabaseHas('pet_applications', [
        'tenant_name' => htmlspecialchars($xssPayload),
    ]);
}
```

---

### 8. INSECURE DESERIALIZATION

**Check:**
- Are untrusted objects deserialized?
- Is user input unserialized?

```php
// ❌ VULNERABLE - Deserializing user input
$object = unserialize($_POST['data']); // Dangerous!

// ✅ SECURE - Use JSON
$object = json_decode($request->input('data'), true); // Safe
```

---

### 9. USING COMPONENTS WITH KNOWN VULNERABILITIES

**Check:**
- Are dependencies up-to-date?
- Are security advisories addressed?

```bash
# Check for vulnerable dependencies
composer audit

# Update all packages
composer update

# Check for outdated packages
composer outdated
```

**Verify:**
```bash
# All packages should show latest versions
# No "CVE" warnings in audit output
```

---

### 10. INSUFFICIENT LOGGING AND MONITORING

**Check:**
- Are important security events logged?
- Can admins see unauthorized access attempts?
- Are logs protected?

```php
// ✅ GOOD - Log security events
Log::warning('Unauthorized access attempt', [
    'user_id' => auth()->id(),
    'resource' => 'pet_applications',
    'resource_id' => $applicationId,
    'action' => 'view',
]);

// ✅ GOOD - Log failed authentications
Log::warning('Failed login attempt', [
    'email' => $request->email,
    'ip' => $request->ip(),
    'timestamp' => now(),
]);

// ✅ GOOD - Audit sensitive operations
Log::info('Pet application approved', [
    'application_id' => $application->id,
    'approved_by' => auth()->id(),
    'approved_at' => now(),
]);
```

---

## FLKitOver Integration Security

### 1. Credential Management

```php
// ❌ VULNERABLE - Hardcoded credentials
$response = Http::post('https://api.flkitover.com/documents', [
    'auth' => ['dev@company.com', 'password123']
]);

// ✅ SECURE - Environment variables
$response = Http::withBasicAuth(
    config('services.flk.email'),
    config('services.flk.password')
)->post('https://api.flk.com/documents', []);
```

### 2. Webhook Signature Verification

```php
// ✅ SECURE - Verify FLK webhook signatures
public function handleFlkWebhook(Request $request)
{
    $signature = $request->header('X-FLK-Signature');
    $payload = $request->getContent();

    $expectedSignature = hash_hmac(
        'sha256',
        $payload,
        config('services.flk.webhook_secret')
    );

    // Prevent timing attacks with hash_equals
    if (!hash_equals($expectedSignature, $signature)) {
        Log::warning('Invalid FLK webhook signature');
        return response('Unauthorized', 401);
    }

    // Process webhook
    $data = $request->json()->all();
    // ...
}
```

### 3. Error Handling

```php
// ❌ VULNERABLE - Exposing FLK error details
try {
    $this->flkService->createDocument($data);
} catch (FLKApiException $e) {
    return response()->json([
        'error' => $e->getMessage(), // Might expose API details
    ], 500);
}

// ✅ SECURE - Log details, return generic message
try {
    $this->flkService->createDocument($data);
} catch (FLKApiException $e) {
    Log::error('FLK integration error', [
        'error' => $e->getMessage(),
        'code' => $e->getCode(),
        'application_id' => $data['application_id'] ?? null,
    ]);
    return response()->json([
        'error' => 'Failed to create document', // Generic message
    ], 500);
}
```

---

## AWS Service Security

### AWS S3 Security

```php
// ✅ SECURE - S3 configuration
config('filesystems.disks.s3' => [
    'driver' => 's3',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION'),
    'bucket' => env('AWS_BUCKET'),
    'visibility' => 'private', // Private by default
]);

// ✅ SECURE - Upload with private visibility
$url = Storage::disk('s3')->put(
    'documents/' . $fileName,
    $fileContent,
    ['visibility' => 'private'] // Explicitly private
);

// ✅ SECURE - Temporary signed URLs with expiry
$url = Storage::disk('s3')->temporaryUrl(
    'documents/' . $fileName,
    now()->addHours(24) // Expires in 24 hours
);

// ❌ VULNERABLE - Public file access
Storage::disk('s3')->put('documents/' . $fileName, $content, 'public');
```

### AWS SES Security

```php
// ✅ SECURE - Use Laravel Mail with SES
Mail::send('emails.application-submitted', $data, function ($message) use ($user) {
    $message->to($user->email)
            ->from(config('mail.from.address'), config('mail.from.name'));
});

// Environment variables (not hardcoded)
// AWS_SES_REGION=us-east-1
// AWS_SES_KEY=your-access-key
// AWS_SES_SECRET=your-secret
```

---

## Laravel Security Configuration

### CSRF Protection

```php
// ✅ SECURE - CSRF middleware enabled by default
// config/app.php - Middleware list includes VerifyCsrfToken

// In forms
<form method="POST" action="/api/applications">
    @csrf <!-- Include CSRF token -->
    <!-- form fields -->
</form>

// In AJAX
headers: {
    'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').getAttribute('content')
}
```

### Security Headers

```php
// ✅ SECURE - Middleware for security headers
class Middleware
{
    protected $middleware = [
        // ... other middleware
        \App\Http\Middleware\SetSecurityHeaders::class,
    ];
}

// SetSecurityHeaders.php
public function handle($request, Closure $next)
{
    $response = $next($request);

    $response->header('X-Content-Type-Options', 'nosniff');
    $response->header('X-Frame-Options', 'DENY');
    $response->header('X-XSS-Protection', '1; mode=block');
    $response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');

    return $response;
}
```

---

## Security Audit Checklist

- [ ] All critical OWASP Top 10 issues resolved
- [ ] No hardcoded credentials
- [ ] All user input validated
- [ ] All passwords hashed (Bcrypt/Argon2)
- [ ] Authorization checks present
- [ ] Rate limiting implemented
- [ ] Sensitive data not exposed in responses
- [ ] FLKitOver integration secure (credentials, webhooks)
- [ ] AWS S3 files private by default
- [ ] Logging captures security events
- [ ] Dependencies checked for vulnerabilities
- [ ] HTTPS enforced
- [ ] CSRF protection enabled
- [ ] Security headers configured
- [ ] Error messages don't expose details

---

## Security Report Format

```markdown
## Security Audit Report: [feature-name]

### Summary
- Risk Level: CRITICAL / HIGH / MEDIUM / LOW
- Vulnerabilities Found: X critical, Y high, Z medium
- FLKitOver Integration: SECURE / AT RISK
- AWS Service Security: COMPLIANT / ISSUES FOUND

### Critical Vulnerabilities (Must Fix Before Production)

1. **Hardcoded Credentials - FLKitOver API**
   - Location: `app/Services/FLKService.php:23`
   - Issue: Email and password hardcoded in source code
   - Risk: Credentials visible in git history, exposed if repo breached
   - Fix: Move to environment variables
   ```php
   // BEFORE (❌ VULNERABLE)
   $auth = ['dev@company.com', 'password123'];

   // AFTER (✅ SECURE)
   $auth = [config('services.flk.email'), config('services.flk.password')];
   ```

2. **SQL Injection in Search**
   - Location: `app/Http/Controllers/SearchController.php:45`
   - Issue: Raw query with user input: `DB::select("SELECT * WHERE name = '$query'")`
   - Risk: Database breach, data exfiltration
   - Fix: Use Eloquent or parameterized queries

### High Priority Issues

1. **Missing Authorization Check**
   - Users can view/delete other users' applications
   - Add authorization policy check

2. **No Rate Limiting on API**
   - Attackers can spam endpoints
   - Add throttle middleware

### Recommendations

1. Implement comprehensive input validation
2. Add security headers middleware
3. Enable HTTPS enforcement
4. Set up security monitoring and alerting
5. Regular penetration testing

### Sign-Off

Security audit approved: _________________ Date: _______

Cannot proceed to production without security approval.
```

---

## Key Reminders

⚠️ **Security is not optional** - Every vulnerability is a potential breach
⚠️ **Defense in depth** - Multiple layers are better than one
⚠️ **Assume compromise** - Treat every user input as potentially malicious
⚠️ **Log everything** - Can't investigate incidents without audit trails
⚠️ **Keep dependencies updated** - Vulnerable packages are common attack vectors
