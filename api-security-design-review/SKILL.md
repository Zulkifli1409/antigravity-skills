---
name: api-security-design-review
description: Design and review the security of a REST/HTTP API from scratch using threat modeling (assets, security goals, STRIDE) and the core controls (HTTPS, authentication, access control, audit logging, rate limiting, input validation, safe output and security headers). Use whenever the user is designing, building, or auditing an API backend, asks "is my API secure", wants a threat model, API security checklist, input validation or injection defenses, rate limiting, audit logging, password storage for an API, or security headers for JSON responses - even if they only say they are building an endpoint, a mobile backend, or a REST service.
author: Zulkifli
github: https://github.com/Zulkifli1409
linkedin: https://www.linkedin.com/in/zulkifli1409/
source: API Security in Action by Neil Madden (Manning, 2020), chapters 1-3
---

# API Security Design & Review

Reply in the language the user writes in. Be concrete: every risk ends with a control and ready-to-paste code or config for the user's stack. Skip theory the user did not ask for.

## 1. Define what "secure" means (do this first)

Security is not a feature you bolt on; it is a set of goals attached to specific assets. Work in four iterating steps:

1. **Assets**: what is worth protecting (user data, tokens, money, availability, admin power, the session cookie you just created).
2. **Security goals** per asset: confidentiality, integrity, availability (CIA), plus accountability (who did what), non-repudiation, privacy.
3. **Constraints**: turn each goal into a testable rule. Example: goal "confidentiality of messages" becomes "user A cannot read user B's messages" and "an unauthenticated caller reads nothing". These become your first security tests.
4. **Implement and re-check**: new mechanisms create new assets (a login creates session tokens that now need protecting). Revisit the model regularly.

## 2. Threat model with STRIDE

Draw the data flows (client, API, DB, other services) and mark trust boundaries. For each flow and component, ask each STRIDE question and pick a control:

| Threat | Meaning | Typical control |
|---|---|---|
| Spoofing | pretending to be someone else | authentication, TLS |
| Tampering | altering data or messages | TLS, MAC/signatures, access control, DB permissions |
| Repudiation | denying an action | audit logging |
| Information disclosure | leaking private data | encryption, access control, safe errors |
| Denial of service | blocking legitimate use | rate limiting, quotas, timeouts |
| Elevation of privilege | doing what you should not | access control, least privilege, validation |

Not every threat deserves equal effort: a cycling-club API does not need to defend against a nation state. Rank by realistic likelihood and impact, and write down what you chose NOT to defend. Threat modeling is a recurring activity; the plan matters less than the planning.

## 3. The five core mechanisms

1. **Encryption in transit**: HTTPS everywhere, plus an HSTS header so browsers never downgrade (`Strict-Transport-Security: max-age=31536000; includeSubDomains`).
2. **Identification and authentication**: verify who is calling before doing anything.
3. **Access control (authorization)**: decide what that identity may do, on every request. Authenticate first, then authorize.
4. **Audit logging**: record who did what, when, and whether it succeeded.
5. **Rate limiting**: protect availability.

## 4. Secure development basics

**Injection**
- Use prepared statements / parameterized queries for every SQL call. Never concatenate request data into a query.
- Give the API's DB account only the permissions it needs (no DROP/CREATE/ALTER for the runtime account). This limits damage if injection slips through.

**Input validation** (treat all input as hostile)
- Validate type, length, range, and format against an allowlist of what is acceptable, not a list of bad things.
- Regular expressions are fine, but beware ReDoS (nested quantifiers like `(a+)+`); cap the input length first.
- Reject early with a 4xx and a generic message. Don't echo the raw input back in errors.
- Validate the `Content-Type` and reject anything unexpected with 415; parse JSON with a real parser.

**Safe output** (even for pure JSON APIs, browsers may render responses)
- Always return the correct `Content-Type: application/json; charset=utf-8`; a missing or wrong type lets browsers guess and treat data as HTML.
- Send on every response, including error responses (these are the ones people forget):
  - `X-Content-Type-Options: nosniff`
  - `Cache-Control: no-store` for sensitive responses
  - `Content-Security-Policy: default-src 'none'; frame-ancestors 'none'` for API responses
  - `X-Frame-Options: DENY` (legacy clickjacking defense)
- Build JSON with a serializer, never by string concatenation.

## 5. Rate limiting (availability)
- Enforce at the edge (load balancer, reverse proxy, API gateway); add a per-instance limit as defense in depth.
- Limit per client identity or IP, with stricter limits on login and other expensive endpoints.
- Return `429 Too Many Requests` with a `Retry-After` header.
- Also set body size limits, timeouts, and pagination caps.

## 6. Authentication basics
- HTTP Basic is acceptable only over HTTPS, and awkward for browsers. See the `api-token-authentication` skill for sessions, bearer tokens, and JWT.
- Store passwords with a slow, memory-hard hash (scrypt, or bcrypt/Argon2id). Never plain or fast hashes like MD5/SHA-1 or unsalted SHA-256. Tune cost so a login takes roughly 100ms or more on your hardware.
- Use constant-time comparison for secrets. Return the same error for "unknown user" and "wrong password".
- Rate-limit and monitor login attempts.

## 7. Audit logging
- Log security-relevant events before and after they happen (request received, then result), with a request id, authenticated user, action, target, outcome, timestamp, and source address.
- Include failures and denied attempts; that is where attacks show up.
- Protect the logs: separate storage, append-only where possible, restricted access, so an attacker or rogue admin cannot erase their trail (repudiation).
- Never log secrets (passwords, tokens, full card numbers). Sanitize user-supplied values written to logs to prevent log injection.

## 8. Access control basics
- Default deny. Check on every request, server-side, against the object being touched, not just "is logged in".
- Start with simple ACLs (who may do what on which object); move to roles or attributes when ACLs grow unmanageable (see `api-authorization-access-control`).
- **Avoid privilege escalation**: a user must never be able to grant someone more permissions than the user holds, create an account more powerful than themselves, or modify their own permissions. Check this on every "add member / change role / share" endpoint.
- Make sure authentication is enforced on every route by default (a global filter), with explicit public exceptions, rather than opt-in per route.

## 9. Output: review report format

```
## Summary
<risk level, top 3 issues, what to fix first>

## Assets and goals
<short list>

## Threats (STRIDE)
| Threat | Where | Existing control | Gap / Fix |

## Findings
### [SEVERITY] Title (file:line or endpoint)
- Problem: ...
- Fix: <code or config>
- Test: <one test that fails if it regresses>

## Not covered
<what you could not see>
```

Severity: Critical (unauthenticated data access/takeover), High (authenticated user reaches other users' data or admin power), Medium (needs user interaction or limited impact), Low (hardening).

## 10. Quick checklist
- [ ] Assets, goals, and trust boundaries written down; STRIDE pass done
- [ ] HTTPS + HSTS; no plaintext listeners
- [ ] Every route authenticated by default; every object access authorized
- [ ] Parameterized queries; least-privilege DB user
- [ ] Input validated by allowlist with length caps; wrong Content-Type rejected
- [ ] Correct Content-Type and security headers on ALL responses, errors included
- [ ] Rate limits and size limits; 429 + Retry-After
- [ ] Passwords hashed with scrypt/bcrypt/Argon2id
- [ ] Audit log of security events, protected, no secrets in it
- [ ] No privilege escalation paths in grant/share endpoints
- [ ] Security tests for each constraint (user A cannot read user B, anonymous reads nothing)

## Credits
- **Source material**: *API Security in Action* by Neil Madden (Manning, 2020), chapters 1-3. This skill is an original summary and application of the book's ideas, not a reproduction of its text.
- **Skill author**: Zulkifli | GitHub: https://github.com/Zulkifli1409 | LinkedIn: https://www.linkedin.com/in/zulkifli1409/
