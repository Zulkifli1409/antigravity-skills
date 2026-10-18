---
name: web-security-code-review
description: Review web application code for security vulnerabilities and write a prioritized findings report with fixes. Use whenever the user shares code, a diff, a pull/merge request, or a repo and asks for a security review, audit, "is this safe", vulnerability check, or hardening feedback - especially for JavaScript/Node, React/Next.js, PHP/Laravel, or REST API backends. Also use when the user asks about XSS, CSRF, SQL/command/code injection, XXE, ReDoS, broken authorization, or risky dependencies in their own code, even if they do not say "security review".
author: Zulkifli
github: https://github.com/Zulkifli1409
linkedin: https://www.linkedin.com/in/zulkifli1409/
source: Web Application Security by Andrew Hoffman (O'Reilly, 2020), chapters 17-21 (Part III, Defense)
---

# Web Security Code Review

Based on the review method in Andrew Hoffman's *Web Application Security* (O'Reilly, 2020). Goal: find both the well-known vulnerability types AND the logic bugs that scanners miss, then report them in a way a developer can act on immediately.

Reply in the language the user writes in. Keep explanations short; the user usually wants findings and ready-to-paste fixes, not a lecture.

## Why context comes first

Scanners find archetypal bugs (XSS, injection). The expensive bugs are logic flaws that only make sense if you know who the users are and what the feature is worth. Example: an endpoint that trusts an `isMember: true` field sent by the client. No scanner flags that. So before reading code, build context.

## Step 1: Build context (ask only if it cannot be inferred)

Write down, in 3-6 lines:
- **Roles**: who uses this (guest, user, admin, tenant owner, cashier...) and what each may do
- **Feature**: what the code is supposed to do
- **Business impact**: what is costly or sensitive (money, PII, storage/bandwidth, admin power)

If the user pasted only a snippet and context is genuinely missing, state your assumptions in one line and proceed rather than interrogating them.

## Step 2: Review in this order

Order matters because it lets you understand the app organically and spend effort on user-facing, high-risk code first.

1. **Client code**: where the browser calls the server. This reveals the surface area, the payload shapes, and what users can reach.
2. **API layer**: follow each client call to its handler. Check auth, authorization, validation, how the payload is interpreted.
3. **Dependencies of those handlers**: DB access, loggers, file uploads, converters, helper libs, shell calls.
4. **Exposed but uncalled endpoints**: routes built for future features, debug routes, accidentally public internals.
5. **Everything else**, in descending order of risk.

For a diff or PR: start from the changed files, but still ask "what does this change expose that did not exist before?"

## Step 3: Check the four data questions on every flow

- How does data travel from A to B (protocol, format, encrypted?)
- How is it stored (plaintext? hashed? who can read it?)
- When it reaches the client, how is it rendered (text or DOM?)
- When it reaches the server, what operations run on it (query, shell, file path, regex, parser?)

## Step 4: Hunt both categories

**Archetypal vulnerabilities**: read the "Vulnerability patterns" section below for what to grep for and what the safe version looks like (XSS, CSRF, XXE, SQLi, command/code injection, ReDoS, logical DoS, dependencies).

**Custom logic bugs**: for every endpoint ask:
- Does the server decide authorization, or does it trust a client-supplied role/flag/price/user id?
- Can user A read or change user B's object by changing an id (IDOR)?
- Are the always-sensitive actions guarded: profile/settings update, password reset, private message read/write, paid features, admin/moderator actions?
- Is authorization a single central function, or re-implemented per endpoint? (Per-endpoint almost always has a gap somewhere.)
- Can steps be skipped, repeated, or reordered (coupon reuse, double-submit, race on balance)?

**Anti-patterns**: read the "Anti-patterns" section below (blacklists, boilerplate left in production, trust-by-default, client/server coupling).

## Step 5: Report

Use this structure. Keep each finding tight.

```
## Ringkasan / Summary
<2-3 lines: overall risk level, number of findings by severity, the one thing to fix first>

## Findings
### [SEVERITY] Short title  (file:line)
- **Problem**: what is wrong and how it can be abused (1-2 sentences, no exploit payload needed beyond a minimal example)
- **Fix**: ready-to-paste corrected code
- **Regression test**: one small test that fails if this comes back

## Anti-patterns / hardening notes
<brief bullets>

## Not reviewed
<what you could not see: other files, config, infra>
```

Severity: Critical (remote, no auth, data/RCE/takeover), High (auth'd attacker gets other users' data or admin power), Medium (needs user interaction or partial impact), Low (defense in depth). If the user wants a number, score with CVSS v3.1 base metrics; see the Vulnerability management section of the web-defense-hardening skill if it is available.

Rules for good findings:
- Only report what you can point to in the code. Mark guesses as "needs verification".
- Prefer the permanent, application-wide fix (parameterized queries everywhere, central middleware) over a one-off patch. If only a partial fix is possible, say what remains open.
- Every fixed security bug should ship with a regression test, because a notable share of vulnerabilities are old bugs that come back.
- Do not write working attack tooling. Minimal proof-of-concept strings that demonstrate the flaw are fine.

---

# Reference

## Secure-coding anti-patterns

### 1. Blacklists
A blacklist is only safe if you know every bad input, now and in the future. You don't. The attacker buys another domain or finds another encoding.
- Flag: `blocked.includes(x)`, strip-the-bad-characters sanitizers, denylisted file extensions/domains/commands.
- Fix: flip to an allowlist. Maintenance burden is real, so reduce it with a submission/approval flow rather than abandoning the allowlist.
- Temporary or partial mitigations are acceptable only with a tracked plan and a ticket for the permanent fix.

### 2. Boilerplate and framework defaults in production
Frameworks often ship loose defaults that must be tightened.
- Flag: default 404/error pages that reveal framework and version, default landing pages, debug mode on, verbose stack traces, default credentials, databases reachable from the internet with no auth, sample routes.
- Fix: review every default before launch; bind services to localhost/private network; require auth; set production mode.

### 3. Trust by default
One OS user or one DB credential with every permission, used by every module.
- Flag: a single service account for logging + disk + database; the web process running as root/admin; one API key with full scope used everywhere.
- Fix: principle of least authority. Separate identities and permissions per module so a bug in one does not hand over the rest.

### 4. Client/server coupling
Server and client code so entangled that the server must parse/trust HTML and logic from the client (e.g. server templates mixing auth logic with markup).
- Flag: server accepts HTML fragments, or decides behavior from client-sent state; business rules enforced only in the UI.
- Fix: independent client and server, talking through a predefined format (JSON) and protocol. Server rejects anything outside the contract and re-checks every rule itself.

### 5. Per-endpoint authorization (signal of weak architecture)
Authorization re-implemented inside each handler.
- Flag: copy-pasted `if (user.role ...)` blocks; some routes missing them.
- Fix: one central authorization layer/middleware or policy class; endpoints without it fail CI (static analysis rule).

### 6. Repeated vulnerabilities = architecture smell
One XSS may be a mistake in one method. Several of the same type usually mean the architecture lacks a secure-by-default layer (shared sanitizer, shared query builder, shared auth middleware). Say so in the report and recommend the central fix instead of N patches.

## Vulnerability patterns: what to look for and the safe version

Contents: XSS, CSRF, XXE, SQL injection, command/code injection, ReDoS, logical DoS, third-party dependencies

Items marked (+) are modern additions beyond the book.

### XSS
**Look for**
- `innerHTML`, `outerHTML`, `document.write/writeln`, `DOMParser.parseFromString`, `document.implementation`
- Blob/SVG built from user data; `javascript:` URLs in `href`/`src`
- `eval`, `new Function`, `setTimeout/setInterval` with a string argument
- Server templates that print request params/user content unescaped (reflected/stored)
- React: `dangerouslySetInnerHTML`; Vue: `v-html`; Laravel Blade: `{!! !!}` (+)
- User-uploaded CSS (can leak form values through `background:url(...)` selectors)

**Safe version**
- Treat user data as text: `textContent`/`innerText`, framework default escaping.
- If HTML is required, sanitize with a well-tested library (+ DOMPurify) and build DOM with `createElement` + `appendChild` so you control tags and the payload controls only content.
- Entity-encode `& < > " '` for content placed in element bodies. This does NOT protect content inside `<script>`, CSS, or URLs.
- Validate link schemes (allow `http`/`https`/`mailto` only). Trick from the book: assign to an `<a>` element's `href` and read back the parsed value to reuse the browser's own URL handling; use `encodeURIComponent` for parts, never for a whole URL.
- Replace string-based `setTimeout("...")` with a function callback.
- Add a CSP (see hardening skill) as a second layer; it does not stop DOM XSS by itself.
- Centralize "append to DOM" in one helper so sanitization is routine.

### CSRF
**Look for**
- Any GET that changes state (update, delete, transfer, change tier) and accepts params from the URL
- GET and POST merged in one handler with an optional "updates" param
- State-changing POST with no token and no origin check
- Cookie-session auth with no SameSite attribute (+)
- Header checks written with broken logic (e.g. `if (!origin || referer)`); read them carefully

**Safe version**
- GET is read-only. Split read and write handlers; writes use POST/PUT/PATCH/DELETE.
- Anti-CSRF token per session/user: bound to the user id, has an expiry, built with a server-only secret; sent on every form and AJAX call.
- Verify `Origin` and `Referer` against an allowlist; reject if both are missing on a state-changing request. Treat this as a first layer only, since an XSS on an allowed origin defeats it.
- Apply it as middleware on all routes, not per route. (+) `SameSite=Lax/Strict` on session cookies.

### XXE
**Look for**
- Any server-side XML parsing: SOAP, SAML, SVG/DOCX/XLSX upload processing, RSS, config import
- Parser created with defaults (Java parsers are often permissive by default)

**Safe version**
- Disable DTDs/external entities in the parser (for Java, disallow `doctype-decl`); verify per library, never assume the default.
- If the data is just structured key/values, use JSON instead of XML.
- Remember XXE is a gateway: read-only file access can escalate to internal network access and RCE.

### SQL injection
**Look for**
- String concatenation or template literals building queries with request data
- ORM raw-query escape hatches (`raw`, `query(`, `whereRaw`, `$queryRawUnsafe`) with interpolation (+)
- Multiple DB adapters in one app (grep each adapter's query call)

**Safe version**
- Prepared statements / bind variables everywhere. The query's intent is fixed before data arrives.
- If something cannot be parameterized (table or column names, ORDER BY), use an allowlist.
- DB-specific escaping functions are a mitigation only, never the primary defense.
- Run the app's DB user with least privilege.

### Command injection / code injection
**Look for**
- `exec`, `spawn` with shell, backticks, `system`, `eval`, `child_process`, template-string commands
- High-risk targets: task schedulers, image/video compression CLIs (ffmpeg-style), backup scripts, loggers, anything calling the host OS, any interpreter
- Endpoints that accept a "command" or "script" from the client and run it

**Safe version**
- Never pass user-chosen commands to a server interpreter. Expose a small allowlist of named actions, plus allowed syntax, order, and frequency.
- Call binaries with an argument array, no shell; validate each argument against an allowlist or strict pattern.
- Run each module under its own low-privilege identity (principle of least authority) so one injection does not reach files, logs, and DB at once.

### Regex DoS (ReDoS)
**Look for**
- Nested quantifiers over a repeated group, like `(a+)+`, `(a|aa)+`, `(.*a){n}`
- Regexes applied to user input without a length cap
- Any place a user can supply a regex

**Safe version**
- Rewrite to remove ambiguity, cap input length, or use a linear-time engine (+).
- Add a regex linter/static check in CI. Never accept user-supplied regexes.

### Logical DoS
**Look for**
- Unbounded loops/recursion on user input, huge page sizes, no pagination limit
- Expensive endpoints (export, report, search, image processing) with no rate limit or queue
- Unbounded uploads, zip/XML expansion, unbounded JSON depth
- Client-side loops that freeze older phones

**Safe version**
- Think in risk levels (high/medium/low), not vulnerable/secure. Cap sizes and counts, rate-limit, time out, queue background jobs, log duration of every request and job.

### Third-party dependencies
**Look for**
- Wide semver ranges (`^`, `~`, `*`), no lockfile committed, packages pulled from git branches
- Known-vulnerable versions deep in the tree (different versions of the same library)
- Dependencies running with the main app's full privileges

**Safe version**
- Commit the lockfile; pin audited versions; run an automated scan against a CVE database (npm audit, Snyk-style tools, NIST NVD).
- Isolate risky integrations in a separate service or process and talk to them with JSON only.

## Review checklist
- [ ] Context established (roles, assets, business impact documented)
- [ ] Client, API, and internal data flows traced
- [ ] No client-controlled trust (roles, flags, prices validated server-side)
- [ ] IDOR and object-level authorization verified on every data access
- [ ] Parameterized queries and context-aware escaping enforced
- [ ] Sensitive endpoints guarded with rate limiting and CSRF protection
- [ ] Lockfiles committed and automated dependency scanning active in CI
- [ ] Every reported finding includes a ready-to-paste fix and a regression test

---

## Credits
- **Source material**: *Web Application Security* by Andrew Hoffman (O'Reilly, 2020), chapters 17-21 (Part III, Defense). This skill is an original summary and application of the book's ideas, not a reproduction of its text. Items marked (+) are modern additions.
- **Skill author**: Zulkifli | GitHub: https://github.com/Zulkifli1409 | LinkedIn: https://www.linkedin.com/in/zulkifli1409/
