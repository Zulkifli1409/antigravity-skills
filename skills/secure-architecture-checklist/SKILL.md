---
name: secure-architecture-checklist
description: Analyze a planned or existing web feature/system design for security risks BEFORE or during coding, and produce a security architecture review with a requirements-to-risks table and concrete decisions. Use whenever the user describes a new app, feature, SaaS, e-commerce, POS, billing, login/registration, multi-tenant or role-based system, database schema, or API design and wants it to be secure - or asks "what should I think about security-wise", threat model, data flow, authentication, authorization, password storage, 2FA, TLS, PII/payment data, search, or third-party integrations. Trigger even if the user only says "I'm planning to build X" and security has not been mentioned.
author: Zulkifli
github: https://github.com/Zulkifli1409
linkedin: https://www.linkedin.com/in/zulkifli1409/
source: Web Application Security by Andrew Hoffman (O'Reilly, 2020), chapters 7 and 17-19
---

# Secure Architecture Checklist

Based on the architecture-phase method in Andrew Hoffman's *Web Application Security*. Core idea: most security work is moving and storing data safely, and the cheapest time to fix a flaw is before code exists (NIST figure quoted in the book: fixing in design is roughly 30-60x cheaper than in production). Re-architecting after customers depend on you is the expensive case, especially for platforms where customers build on top of you (stores, tenants, plugins).

Reply in the language the user writes in. Be concrete and decision-oriented: each risk ends in a decision or a question the user must answer.

## Workflow

### 1. Collect the requirements
Take the feature list from the user (or extract it from the conversation). If it is vague, state assumptions in one line and proceed. Don't ask more than one clarifying question.

### 2. Derive facts from the requirements
Turn each requirement into a plain statement of what the system will hold or do. Example: "users can save cards" becomes "we store financial data". Typical derived facts:
- We store credentials / personal identifiers / financial data
- There are guests and logged-in users, so authentication AND authorization exist
- Users search content, so search index and permissions can drift
- Users upload files or content others will view, so XSS/upload risk
- Third parties integrate or we integrate third parties
- Multiple tenants, roles, or branches share data

### 3. Map facts to risk areas
For each fact, ask the questions in the matching reference file:
- Identity, sessions, passwords, 2FA, TLS, authorization → the "Authentication and credentials" section below
- Data at rest, PII/financial data, search, third parties, service separation, least authority → the "Data and integration" section below

### 4. Check the architecture signals
Secure architectures put security in the structure so each feature inherits it; insecure ones rely on each developer remembering. Look for:
- One central authorization layer vs. checks copy-pasted in each endpoint
- One shared sanitizer / query layer / validation library vs. ad-hoc per feature
- Client and server separated by a defined contract (JSON) vs. entangled
- Layers of defense (validate at API, escape at render, CSP in browser) vs. a single control
- Reuse of proven components vs. home-made crypto/auth/search

### 5. Produce the review

```
## Ringkasan
<2-3 lines: biggest architectural risks and the 3 decisions to make first>

## Requirements -> facts -> risks
| Requirement | What we now hold/do | Risk area |
|---|---|---|

## Decisions (recommended defaults)
<numbered list; each: decision, why, cost/trade-off>

## Open questions for you
<only the ones that change the design: laws/regions for PII, who can see what, payment provider, etc.>

## Later phases
Code review focus areas, tests to write first, what to monitor.
```

Keep recommended defaults specific (name the algorithm, the header, the pattern), and flag trade-offs honestly (performance, UX, cost). If the user already chose something weak (MD5 passwords, JWT in localStorage, shared DB root user), say so plainly and give the replacement.

## Order of work in a secure lifecycle
1. Architecture review (this skill) → 2. Security review of each merge request (`web-security-code-review`) → 3. Automated discovery (static/dynamic analysis, regression tests) → 4. Vulnerability management. Remind the user that code review comes after architecture review, not instead of it.

## Stay within scope
This is design guidance for building and protecting the user's own systems. It does not include attack instructions.

---

# Reference

## Authentication, credentials, sessions, authorization

Contents: Data in transit, Password policy, Password hashing, 2FA, Authentication vs authorization, Authorization checklist

Items marked (+) are modern additions beyond the book.

### Data in transit
- Everything over TLS (HTTPS). Never the older SSL versions. This removes the easy man-in-the-middle credential theft.
- Decide this first; it affects every data flow. Every major web server has simple TLS setup; free certificates exist (Let's Encrypt).
- (+) Redirect HTTP to HTTPS, set HSTS, mark cookies `Secure` and `HttpOnly`, `SameSite`.

### Password policy
- Strength is about unpredictable patterns (entropy), not just length or special characters. Attackers start with dictionaries of the most common passwords and common structures, then brute force.
- Reject passwords found in a common-passwords list (book suggests top-1000; (+) larger breached-password lists are better).
- Reject passwords containing the user's first name, last name, birthdate, or parts of their address.

### Password hashing
- Never store plaintext. Hash on first receipt, compare hashes at login. Hashing is one-way, so even your own staff or a rogue employee cannot read passwords.
- Use a deliberately SLOW algorithm so each guess costs an attacker time: **bcrypt** (gets slower as hardware improves) or **PBKDF2** with the highest iteration count your hardware tolerates. (+) Argon2id is also a strong modern choice.
- Outcome comparison after a DB leak: plaintext = all compromised; fast hash like MD5 = many cracked via rainbow tables/brute force; bcrypt = very unlikely any cracked.
- Don't invent your own scheme. Use the framework's built-in password hasher.

### 2FA
- Offer authenticator-app TOTP (e.g. Google Authenticator style). It blocks remote logins not initiated by the account owner: the attacker needs password AND device.
- Hardware tokens suit employees and admins more than consumers (distribution pain). SMS is weaker than app-based but still far better than no 2FA.
- Recommend mandatory 2FA for admin/owner roles.

### Authentication vs authorization
- Authentication: who are you? Authorization: what may you access? You need both whenever there are different user tiers.
- Basic auth (credentials on each request) leaks easily. Digest is better. Token-based schemes and OAuth are common today; OAuth lets a big provider vouch for identity but concentrates risk (one compromised provider account touches many sites).
- Stateless tokens must be signed/encrypted with a server-only secret and carry expiry.

### Authorization checklist
- Centralize it: one policy layer that every endpoint passes through. Re-implementing checks per endpoint reliably leaves gaps.
- Always enforce server-side; never trust a role, flag, price, or user id sent by the client.
- Always-guard list: settings/profile updates, password reset, private message read/write, anything paid, any elevated function (moderation, admin).
- Check object-level access, not only role (user A must not read user B's order by changing an id).
- Multi-tenant: every query is scoped by tenant; test cross-tenant access explicitly.
- Log denied attempts.

## Data storage, search, third parties, separation

Contents: PII and financial data, Search engines, Third-party dependencies, Least authority, Client/server separation, Logging and monitoring

### PII and financial data
- First question: is it legal to store this in the countries you operate in, and which laws apply? That can decide the design.
- Assume the database will be breached one day: PII should not be exposed in an easily abusable format (encrypt sensitive fields, minimize what you keep, restrict who and what can read it).
- Smaller teams: outsource card and bank data to a compliant payment/storage specialist so you never hold raw numbers.
- Collect only what the feature needs.

### Search
A custom search engine usually needs its own datastore (e.g. Elasticsearch) beside the primary database. That creates a sync problem with security consequences:
- Permissions changed in the primary DB must propagate to the search index.
- Objects deleted in the primary DB must also disappear from the index; otherwise deleted or private data stays searchable.
- Decide the single source of truth and how authorization is applied at query time.
- Treat search input as untrusted (query injection).

### Third-party dependencies and integrations
- Dependencies bring their own dependencies (a tree). Different versions of the same library may coexist and one may be vulnerable.
- Automate checks against a CVE database whenever a dependency is added, and keep checking.
- Pin audited versions and commit the lockfile; consider a private mirror for critical packages.
- Isolation: run risky integrations on their own server/process with minimal rights and talk via JSON over HTTP, treating them like pure functions. Downsides: latency, and any confidential data you send them can still be exposed.
- Prefer allowlisting integration partners (verify them) over blocking known-bad ones.

### Principle of least authority
Each module gets only the data and permissions it needs. Separate OS users / DB roles for logging, disk writes, and database access. Background tools (backup scripts, media converters) should not run as admin. The point: an injection bug in one module must not expose everything.

### Client/server separation
Develop client and server independently and let them talk through a predefined format and protocol (JSON over HTTP). The server then validates one known shape instead of parsing arbitrary HTML/logic, and each side has fewer security mechanisms to maintain.

### Prefer secure formats and defaults
- JSON over XML when you don't need XML features (avoids XXE class).
- Secure-by-default framework settings; remove demo/boilerplate pages; bind internal services to private interfaces.

### Logging and monitoring
- Log every request with its response time and the duration of background jobs; this is how abuse and DoS become visible.
- When a vulnerability is known but unfixed, add targeted logging so you can see whether it is being exploited meanwhile.
- Keep logs free of passwords, tokens, and full card data.

---

## Credits
- **Source material**: *Web Application Security* by Andrew Hoffman (O'Reilly, 2020), chapters 7 and 17-19. This skill is an original summary and application of the book's ideas, not a reproduction of its text. Items marked (+) are modern additions.
- **Skill author**: Zulkifli | GitHub: https://github.com/Zulkifli1409 | LinkedIn: https://www.linkedin.com/in/zulkifli1409/
