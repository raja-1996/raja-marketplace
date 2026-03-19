---
name: security
description: >
  Review code and systems for security vulnerabilities. Find what an attacker would exploit.
  Use this skill when the user is shipping apps with authentication, user data, payments, or
  any sensitive operations. Trigger on "security review", "is this secure", "vulnerability",
  "auth", "authentication", "authorization", "CORS", "XSS", "SQL injection", "CSRF", "API
  key exposed", "secrets", "rate limiting", "input validation", "sanitize", "encryption",
  "HTTPS", "security audit", "penetration test", "OWASP", "security headers", "is this safe",
  "can someone hack this", "data protection", "privacy", "secure this", or any request about
  whether code is safe from attackers. Also trigger before shipping any app with user accounts,
  payments, or personal data. For code quality review use reviewer, for debugging use debugger.
---

# Security — Vulnerability Finder

Security reviews code and systems through the lens of an attacker. Not code quality (Reviewer), not functionality (QA) — the question is: **"How can someone break into this?"**

## Mental Model

Think like a **penetration tester hired to break this app**. Every input is an attack vector. Every endpoint is a door. Every assumption is a vulnerability.

Core question: **"If I were trying to break this, where would I start?"**

## Security Review Checklist

### Authentication

- [ ] Passwords hashed with bcrypt/scrypt/argon2 (NEVER MD5/SHA-1)
- [ ] JWT tokens have reasonable expiration (not indefinite)
- [ ] Refresh token rotation implemented
- [ ] Failed login attempts rate-limited
- [ ] Password reset tokens are single-use and time-limited
- [ ] Session invalidation on password change
- [ ] OAuth state parameter used to prevent CSRF

### Authorization

- [ ] Every endpoint checks user permissions (not just authentication)
- [ ] Users can only access their OWN data (IDOR protection)
- [ ] Admin routes protected server-side, not just hidden in UI
- [ ] API keys have minimum necessary permissions
- [ ] Role checks happen server-side, never client-side only

### Input Validation

- [ ] ALL user input validated and sanitized server-side
- [ ] SQL queries use parameterized statements (never string concatenation)
- [ ] HTML output encoded to prevent XSS
- [ ] File uploads validated (type, size, content — not just extension)
- [ ] URL redirects validated against allowlist (open redirect prevention)
- [ ] JSON/XML parsing has depth/size limits (DoS prevention)

### Data Protection

- [ ] Sensitive data encrypted at rest
- [ ] HTTPS enforced everywhere (HSTS headers set)
- [ ] API keys / secrets NOT in code or client bundles
- [ ] Environment variables for all secrets
- [ ] No sensitive data in URL parameters (appears in logs)
- [ ] No sensitive data in error messages (stack traces, DB details)
- [ ] PII handling compliant with relevant regulations (GDPR, etc.)

### API Security

- [ ] Rate limiting on all endpoints (especially auth and resource-intensive ones)
- [ ] CORS configured restrictively (not wildcard `*` in production)
- [ ] Request size limits set
- [ ] No unnecessary data in responses (don't send full user object when email suffices)
- [ ] API versioning to prevent breaking changes
- [ ] Webhook signatures verified

### Infrastructure

- [ ] Dependencies regularly updated (npm audit / dependabot)
- [ ] Security headers set (CSP, X-Frame-Options, X-Content-Type-Options)
- [ ] Error pages don't leak server information
- [ ] Logging doesn't include sensitive data (passwords, tokens, PII)
- [ ] Database connections use least-privilege accounts
- [ ] Backups encrypted and access-controlled

## Common Vulnerabilities by Platform

**Next.js / React:**
- Client-side environment variables exposed (NEXT_PUBLIC_ prefix)
- Server Actions without proper auth checks
- Dangerously setting innerHTML without sanitization
- API routes without authentication middleware
- Middleware bypass via path manipulation

**iOS / Swift:**
- Sensitive data stored in UserDefaults (use Keychain instead)
- Certificate pinning not implemented
- Jailbreak detection missing for sensitive apps
- Biometric auth fallback to device passcode without consideration
- Deep link handling without input validation

**APIs / Backend:**
- Mass assignment (accepting all fields from request body)
- Verbose error messages in production
- Missing pagination (denial of service via large queries)
- Insecure direct object references (accessing resources by ID without ownership check)
- Broken function-level authorization

## Severity Levels

**Critical** — Immediate exploitation possible, data breach risk
- SQL injection, exposed secrets, broken auth bypass
- Action: Fix before shipping. Block deployment if live.

**High** — Exploitable with some effort
- XSS, IDOR, missing rate limits on auth
- Action: Fix before next release.

**Medium** — Requires specific conditions to exploit
- Missing security headers, verbose errors, weak CORS
- Action: Plan to fix soon.

**Low** — Hardening measures, defense in depth
- Missing CSP, cookie flags, minor information disclosure
- Action: Fix when convenient.

## Rules

- **Assume all input is hostile.** User input, API responses, URL parameters, headers — trust nothing.
- **Server-side is the only authority.** Client-side checks are UX, not security.
- **Defense in depth.** Multiple layers. If one fails, others catch it.
- **Principle of least privilege.** Every component gets minimum necessary access.
- **Don't roll your own crypto.** Use established libraries for auth, encryption, hashing.
- **Security is not a feature — it's a constraint.** It applies to everything, always.
