---
name: api-token-authentication
description: Implement and harden token-based API authentication - session cookies, CSRF defense (SameSite, double-submit), CORS configuration, Bearer tokens, Web Storage vs cookies, hashing and HMAC for stored tokens, self-contained tokens, JWT validation pitfalls, encrypted tokens, and token revocation/logout. Use whenever the user builds login, sessions, "remember me", JWT auth, refresh logic, a SPA or mobile app talking to an API, hits CORS or CSRF errors, asks where to store tokens, or asks whether JWT or sessions are better - even if they never say "token".
author: Zulkifli
github: https://github.com/Zulkifli1409
linkedin: https://www.linkedin.com/in/zulkifli1409/
source: API Security in Action by Neil Madden (Manning, 2020), chapters 4-6
---

# API Token Authentication

Reply in the language the user writes in. Give the recommended choice first, with a short reason, then code or config for their stack.

## Pick the mechanism

| Situation | Recommended |
|---|---|
| Browser client served from the SAME site as the API | Server-side session cookie (HttpOnly, Secure, SameSite) + CSRF defense |
| Browser client on a different origin, mobile or desktop app, third-party clients | Bearer tokens in the `Authorization` header |
| Need horizontal scale and low DB load | Self-contained tokens (JWT) PLUS an allowlist/revocation record |
| Third-party access or delegated login | OAuth2/OIDC (see `api-authorization-access-control`) |

Do not pick JWT just because it is popular; a database-backed opaque token is simpler and revocable. JWT's value is scale and cross-service verification.

## Session cookies (same-site browser clients)

Cookie attributes, always the most restrictive that works:
- `Secure` (HTTPS only) and `HttpOnly` (JavaScript cannot read it, so XSS can't steal it).
- `SameSite=Strict` (or `Lax` if you need links from other sites to arrive logged in).
- Do NOT set `Domain` unless required; a cookie scoped to the parent domain is exposed to every subdomain, and abandoned subdomains get hijacked.
- Prefer the `__Host-` prefix (forces Secure, Path=/, no Domain).
- Don't set `Max-Age`/`Expires` on the auth cookie so closing the tab logs out; BUT browsers restore session cookies, so ALSO enforce an absolute lifetime and an idle timeout on the server.
- Token value: long random, from a cryptographically secure generator (at least 128 bits of entropy; the book uses 20 random bytes).

**Session fixation**: on successful login, discard any existing session and issue a fresh token. Never keep the pre-login session id.

**Logout**: delete the server-side session and clear the cookie. A "logout" that only removes the cookie client-side leaves a valid token alive.

## CSRF (cookie-based auth needs it)
- `SameSite` cookies stop most CSRF in modern browsers; keep it but don't rely on it alone (older browsers, same-site attacker subdomains).
- **Hash-based double-submit cookie**: send a CSRF token in a header (e.g. `X-CSRF-Token`) whose value is a hash (SHA-256, Base64url) of the session cookie value. The server recomputes the hash from the session cookie and compares in constant time. An attacker who can plant cookies from a subdomain cannot forge a matching hash for your session, which fixes the weakness of plain double-submit.
- Add the token on every non-GET request automatically via one client wrapper.
- GET must never change state.

## CORS (cross-origin web clients)
- CORS relaxes the browser's same-origin policy; it is not an authorization system and doesn't stop non-browser clients.
- Echo the `Origin` back only if it exactly matches an allowlist; never reflect arbitrary origins, never use `*` together with credentials, avoid the string `null`.
- Add `Vary: Origin` when the response differs by origin.
- Handle preflight (`OPTIONS`) with the right `Access-Control-Allow-Methods` and `-Headers` (include `Authorization` if you use bearer tokens). Return `Access-Control-Allow-Credentials: true` only for cookie flows that truly need it.
- Cookies across origins are increasingly blocked by browsers; for cross-origin use bearer tokens.

## Bearer tokens (cross-origin, mobile)
- Send `Authorization: Bearer <token>`. On failure respond `401` with `WWW-Authenticate: Bearer`.
- Opaque random token stored server-side with expiry, subject, and attributes; delete expired tokens regularly.
- **Where the browser stores it**: Web Storage (localStorage/sessionStorage) is immune to CSRF but readable by any script, so one XSS means token theft. If you use it, treat XSS prevention (CSP, output encoding) as a hard prerequisite. Same-site apps are safer with an HttpOnly cookie. Don't store long-lived tokens in localStorage.

## Hardening tokens stored in a database
1. **Store only a hash** (SHA-256) of the token, not the token. A DB leak then yields nothing usable. Hash on lookup.
2. **MAC the token** with HMAC-SHA-256: the issued token is `id.tag`; reject bad tags before touching the DB. Keep the HMAC key outside the database (key store/KMS), so a DB-only breach cannot forge tokens. Compare tags in constant time.
3. Use DB permissions and row-level security so the API role can only touch the token table the way it must.
4. Encrypt sensitive token attributes only when you really need them; database encryption has many failure modes.

## Self-contained tokens and JWT
- State lives in the token itself, signed (and optionally encrypted). Scales without a DB, but cannot be revoked by itself.
- Signature: HMAC (shared secret between issuer and verifier) or public-key signature (issuer signs, many services verify with the public key).
- **Encryption**: signed tokens are readable by anyone (Base64, not secret). Put sensitive claims in an encrypted token (JWE) or keep them out. Use authenticated encryption (AES-GCM, ChaCha20-Poly1305, or NaCl SecretBox), never encryption without authentication.

Standard claims and what to enforce:
- `iss` issuer: must match who you trust.
- `aud` audience: reject if your API's identifier is not present (stops a token for service A working on B).
- `exp`, `nbf`, `iat`: enforce, allow small clock skew only.
- `sub`: subject. `jti`: unique id, useful for replay detection and allowlists.

**JWT validation rules (historically the source of serious CVEs)**
- Pin the algorithm on the server per key. Never let the token's `alg` header choose; reject `alg: none`; never verify an RSA public key as an HMAC secret (algorithm confusion).
- Do not trust header-supplied keys or URLs (`jwk`, `jku`, `x5u`) unless allowlisted.
- Always verify signature, then `iss`, `aud`, `exp`. Fail closed with a generic error.
- Use a maintained library; avoid the error-prone optional parts of JOSE (for encrypted JWTs avoid RSA1_5; prefer modern algorithms).
- **Token type confusion**: if you issue different token kinds (access, refresh, ID, CSRF), make them non-interchangeable with a distinct `typ`/audience/key per kind, and check it.
- Keep lifetimes short (minutes for access tokens), and don't put secrets or excessive PII in claims.

## Revocation and logout with stateless tokens
- Pure stateless tokens can't be revoked before `exp`. Options, best first:
  1. **Hybrid/allowlist**: JWT for verification plus a DB record keyed by `jti`; a token is valid only if the record exists. Logout deletes the record. A secure default that still offloads claims parsing.
  2. Short-lived access tokens + refresh tokens stored server-side (revoke the refresh token).
  3. Blocklist of revoked `jti` until expiry (needs the same DB, and fails open if the list is lost).
- Changing a password or detecting compromise should revoke all of that user's tokens.

## Review checklist
- [ ] Cookie flags `Secure; HttpOnly; SameSite`, no wide `Domain`, `__Host-` prefix
- [ ] New session id issued at login; server-side idle and absolute timeouts; real logout
- [ ] CSRF protection on all state-changing requests; GET is read-only
- [ ] CORS origin allowlist exact-match; no `*` with credentials
- [ ] Tokens high entropy, stored hashed, HMAC key kept out of DB
- [ ] JWT: pinned algorithm, `aud`/`iss`/`exp` enforced, no `alg: none`, keys from allowlist
- [ ] Sensitive claims encrypted or omitted; short expiry
- [ ] Revocation strategy exists and logout works
- [ ] If Web Storage is used: strong XSS defenses and CSP in place

## Credits
- **Source material**: *API Security in Action* by Neil Madden (Manning, 2020), chapters 4-6. This skill is an original summary and application of the book's ideas, not a reproduction of its text.
- **Skill author**: Zulkifli | GitHub: https://github.com/Zulkifli1409 | LinkedIn: https://www.linkedin.com/in/zulkifli1409/
