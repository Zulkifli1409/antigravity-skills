---
name: api-authorization-access-control
description: Design authorization for APIs - OAuth2 (scopes, authorization code with PKCE, refresh tokens, token introspection, JWT access tokens), OpenID Connect and SSO, and access control models (ACL, groups, RBAC, ABAC with policy engines, capability URLs, macaroons). Use whenever the user adds roles or permissions, multi-tenant or admin access, "login with Google/OAuth", third-party app access, API scopes, sharing links, policy engines like OPA/XACML, or asks how to decide who can do what in an API - even if they only say "permissions" or "who can access this".
author: Zulkifli
github: https://github.com/Zulkifli1409
linkedin: https://www.linkedin.com/in/zulkifli1409/
source: API Security in Action by Neil Madden (Manning, 2020), chapters 7-9
---

# API Authorization & Access Control

Reply in the language the user writes in. Start by asking (or inferring) the three facts that decide the model: who are the actors, what objects exist, and who decides permissions (users share their own things = discretionary, or admins/policy decide = mandatory).

## Choose the model

| Need | Use |
|---|---|
| Few users/objects, simple rules | ACL per object, plus a default-deny check |
| Many users, standard job functions (cashier, manager, admin) | RBAC (roles to permissions) |
| Rules depend on context (time, location, owner, department, data sensitivity) | ABAC with a policy engine, layered over RBAC |
| Third-party app acts for a user | OAuth2 with scopes |
| Share one resource via a link without adding people to a group | Capability URL (or macaroon) |
| Sign-in via an identity provider | OpenID Connect on top of OAuth2 |

## OAuth2 essentials

Roles: resource owner (user), client (app), authorization server (AS, issues tokens), resource server (your API).

**Scopes**
- A scope limits what a token may do; it does not grant rights. Effective access = what the user is allowed AND what the scope allows (intersection). Don't confuse scopes with permissions.
- Define coarse, meaningful scopes (e.g. `read_messages`, `post_message`), enforce them in the API on every endpoint, and request only what the client needs.

**Grants**
- Use **authorization code + PKCE** for every user-facing client: web apps, SPAs, mobile, desktop. Do not use the implicit grant, and avoid password grant.
- PKCE: client creates a random `code_verifier` (43-128 chars, high entropy), sends `code_challenge = BASE64URL(SHA-256(verifier))` with the authorization request, then sends the verifier when exchanging the code. This defeats stolen or injected authorization codes.
- Send and check a random `state` value (CSRF on the redirect).
- Register redirect URIs and match them **exactly**; for mobile use claimed HTTPS redirects over custom URL schemes.
- Refresh tokens keep access tokens short-lived (minutes). Store refresh tokens safely, rotate them, and make them revocable.
- Services without a user: client credentials grant (see `api-microservice-kubernetes-security`). Devices without a keyboard: device grant (see `iot-api-security`).

**Validating an access token at your API**
- **Opaque token**: call the AS's introspection endpoint (your API authenticates to it), check `active`, `scope`, `exp`, `aud`, `sub`. Cache results briefly.
- **JWT access token**: verify signature against the AS's published keys (JWKS), pin the algorithm, check `iss`, `aud`, `exp`, scope. Faster (no network call) but revocation is delayed until expiry, so keep lifetimes short or combine with introspection for sensitive operations.
- Use strict TLS settings when your API calls the AS (verify hostname and certificate chain, modern TLS only).
- Support the token revocation endpoint and handle logout/compromise.

**OpenID Connect and SSO**
- OIDC adds identity: the **ID token** tells the *client* who logged in. It is for the client, not an access token for your API; don't use it for access control decisions at the API.
- Validate ID tokens: signature, `iss`, `aud` equals your client id, `exp`, and the `nonce` you sent. Prefer the code flow with PKCE; fetch claims from the UserInfo endpoint when needed.
- With SSO, one IdP compromise affects many apps, so enforce MFA at the IdP and keep sessions short for sensitive operations.

## Identity-based access control

**Groups**: collect users org-wide (LDAP static and dynamic groups) to avoid per-user ACL sprawl. Groups are usually organization-level; roles are usually per-API.

**RBAC**
- Roles bundle permissions on objects; assign roles to users/groups; revoke by removing the role.
- In code, check **permissions**, not role names (`can('delete_post')`, not `role == 'admin'`), so you can reshape roles without editing code.
- Static roles (fixed assignments) vs dynamic roles (computed per request, e.g. "owner of this item", "member of this space").
- Store the role→permission mapping in one place; enforce in one central layer.

**ABAC**
- Decisions use four attribute categories: subject (who), resource (what), action (which operation), environment (time, location, device).
- Combine rule results with a clear algorithm; **default deny** and "deny overrides" are safest. Many small rules combined unexpectedly are the main risk.
- Architecture (XACML vocabulary): PEP (enforces in the API/gateway), PDP (decides), PIP (supplies attributes), PAP (admin of policies). Policy agents or gateways keep enforcement near the API.
- **Best practices**: layer ABAC over simpler RBAC (defense in depth); automate tests that detect when a policy change grants unintended access; keep policies in version control with review; centralize carefully (too central means overly broad policies stay because changes are slow); measure policy evaluation performance early.

## Capability-based security
- **Ambient authority** (access decided by "who you are") enables **confused deputy** attacks: a privileged service is tricked into using its authority for someone else. Capabilities avoid this by making the reference to a resource also the permission to use it, supporting least authority.
- **Capability URI**: unguessable random token embedded in the URL that names one resource and carries fixed permissions (e.g. read-only). Example: `https://api.example.com/msgs/9Wbz...?` One link, one resource, one permission set.
- Placement trade-offs: path or query (simple, but leaks via logs, Referer, history), fragment or userinfo (not sent to servers or logged, but needs client-side handling).
- **Hardening**: high entropy (at least 128 bits), short expiry, limited permissions, `Referrer-Policy: no-referrer`, no capability URLs in logs, bind to a user session when you need stronger protection and CSRF resistance (at the cost of easy sharing), use HTTPS only, support revocation by deleting the stored token.
- Follow HATEOAS: return capability links in responses so clients never construct URLs or need identity-based checks.

**Macaroons** (tokens with caveats)
- Anyone holding a macaroon can add a caveat that **restricts** it (never widen it); the API cryptographically verifies the chain with HMAC.
- First-party caveats: conditions the API checks itself (expiry, time of day, only path `/x`, read-only, IP range).
- Contextual caveats: append just before use (e.g. restrict to this request's host and short time) to limit the damage if the token leaks.
- Third-party caveats: require a discharge macaroon from another service proving a condition (e.g. user is an employee, over 18).
- Good for delegating narrowed access across services without calling the issuer.

## Rules for every model
- Authenticate first, authorize second, on every request, server-side.
- Default deny; return `403` (or `404` to avoid revealing existence for sensitive objects).
- Object-level checks: user A must not read user B's object by changing an id (IDOR).
- Never trust role, flag, owner id, or price from the client.
- Prevent privilege escalation in grant/share/invite endpoints: a grantor can't give more than they hold.
- Log every access decision on sensitive actions (who, what, allowed or denied).
- Write tests that cover denial paths, not just successes.

## Review checklist
- [ ] A single central enforcement point; no endpoint opts out silently
- [ ] Permissions checked, not role names; mapping in one place
- [ ] Scopes enforced and treated as a restriction
- [ ] Auth code + PKCE; exact redirect URIs; `state` checked; no implicit grant
- [ ] Access tokens validated fully (signature or introspection, `aud`, `exp`, scope)
- [ ] ID tokens not used as API credentials
- [ ] ABAC layered over RBAC with tests and versioned policies
- [ ] Capability URLs unguessable, expiring, least-permission, not logged or leaked via Referer

## Credits
- **Source material**: *API Security in Action* by Neil Madden (Manning, 2020), chapters 7-9. This skill is an original summary and application of the book's ideas, not a reproduction of its text.
- **Skill author**: Zulkifli | GitHub: https://github.com/Zulkifli1409 | LinkedIn: https://www.linkedin.com/in/zulkifli1409/
