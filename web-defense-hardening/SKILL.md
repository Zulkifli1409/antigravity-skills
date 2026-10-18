---
name: web-defense-hardening
description: Plan and implement defenses for web application vulnerabilities - mitigation steps per vulnerability class (XSS, CSRF, XXE, SQL/command injection, DoS/DDoS, vulnerable dependencies), CSP and security header setup, security regression tests, static/dynamic analysis in CI, and triage of reported vulnerabilities with CVSS severity scoring and fix tracking. Use whenever the user asks how to prevent, fix, mitigate, harden, or secure against an attack or vulnerability, wants a CSP, CSRF middleware, a security checklist before launch, a bug bounty or disclosure policy, wants to score or prioritize a vulnerability, or got a pentest/scanner/bug-report result to deal with - even if they do not name the vulnerability class.
author: Zulkifli
github: https://github.com/Zulkifli1409
linkedin: https://www.linkedin.com/in/zulkifli1409/
source: Web Application Security by Andrew Hoffman (O'Reilly, 2020), chapters 17-28 (Part III, Defense)
---

# Web Defense & Hardening

Based on Part III (Defense) of Andrew Hoffman's *Web Application Security*. The lifecycle: secure architecture → security code review → vulnerability discovery → vulnerability analysis and management → regression tests → broad mitigation. Each layer catches what the previous one missed, so recommend layers, not a single silver bullet.

Reply in the language the user writes in. Give ready-to-paste code or config for the user's stack when possible; ask the stack only if it changes the answer.

## Pick the task

| User wants | Do this |
|---|---|
| Fix or prevent a specific vulnerability | Read the "Mitigations" section below, give the permanent app-wide fix first, then layered defenses, then a regression test |
| CSP / headers | the "Mitigations" section below → "CSP and headers" section; produce a header the user can paste and explain what it blocks |
| Got a bug report / scanner / pentest result | the "Vulnerability management" section below → reproduce, score, prioritize, fix, regression test |
| Pre-launch hardening checklist | Build it from the "Launch checklist" below, tailored to the user's stack |
| Set up automation in CI | the "Vulnerability management" section below → "Discovery automation" |
| Disclosure policy / bug bounty | the "Vulnerability management" section below → "Programs" |

## Principles to apply every time

1. **Permanent, application-wide fixes beat one-off patches.** A fix applied at one call site leaves every other call site open. Prefer central middleware, shared helpers, and framework features.
2. **Layer defenses.** Example for XSS: encode on output + sanitize where HTML is required + CSP + safe DOM APIs. A single control will eventually be bypassed.
3. **Allowlist, not blacklist.** Anything that enumerates bad things is incomplete by design.
4. **Least authority.** Limit blast radius with separate identities and minimal permissions.
5. **Never close a security bug on a partial fix.** If the fix is temporary, open a new tracked issue describing the still-vulnerable surface, with its own score.
6. **Every closed security bug ships a regression test.** A meaningful share of vulnerabilities are earlier bugs that reappear after rollbacks, merges, or variations.
7. **Log while waiting.** If a known vulnerability is not fixed yet, add logging around it so exploitation is visible.
8. **Mitigate more widely than the bug.** Fix the instance, then ask "where else does this pattern exist?" and close the whole class.

## Launch checklist (adapt to stack)

- TLS everywhere; HSTS; secure, HttpOnly, SameSite cookies
- Passwords hashed with bcrypt/PBKDF2/Argon2; common-password rejection; 2FA for admins
- Central authorization layer; object-level checks; tenant scoping
- Parameterized queries only; no user-controlled commands reach a shell
- Output encoding by default; no unsanitized `innerHTML`/`v-html`/`dangerouslySetInnerHTML`; CSP deployed
- State-changing endpoints are not GET; CSRF token and origin checks as middleware
- XML parsers have external entities/DTDs disabled (or switch to JSON)
- Rate limits, size limits, timeouts; regexes linted; logging of request duration
- Dependencies scanned, versions pinned, lockfile committed
- Debug mode off; default pages and verbose errors removed; internal services not public
- Static analysis in CI; vulnerability regression tests on commit/push hooks (or daily at minimum)
- A way for outsiders to report vulnerabilities safely

## Output style

For a "how do I fix X" request: 1) one-sentence root cause, 2) the fix in code for their stack, 3) one layered defense on top, 4) a regression test. Skip long theory. For a triage request, use the report format in the "Vulnerability management" section below.

This skill is for defending systems the user owns or is authorized to protect. If a request turns into building an attack against someone else's system, decline that part and offer the defensive angle instead.

---

# Reference

## Mitigations by vulnerability class

Contents: XSS, CSP and headers, CSRF, XXE, SQL injection, Other injection, DoS (regex, logical, DDoS), Dependencies

Items marked (+) are modern additions beyond the book.

### XSS
Rule of thumb: never let unsanitized user-supplied data reach the DOM, and pass it as a string when HTML is not required.

1. Use text-only APIs: `textContent`/`innerText` instead of `innerHTML`. (innerText's own escaping has had browser-specific bypasses, so do not treat it as the only layer.)
2. Use a central function for every DOM insertion so sanitization is uniform.
3. If limited HTML is required, sanitize with a maintained library (+ DOMPurify) and construct nodes with `createElement`/`appendChild`.
4. Avoid risky sinks: `innerHTML`, `outerHTML`, `document.write(ln)`, `DOMParser.parseFromString`, `document.implementation`, Blob and SVG built from user data (SVG can run script).
5. Entity-encode the big five: `& < > " '`. Works for element content only; it does not protect `<script>`, CSS, or URL contexts.
6. Links: allow only safe schemes (`http`, `https`, `mailto`); block `javascript:` URLs. `encodeURIComponent` is for URL components, not whole URLs.
7. Don't use string-form `setTimeout`/`setInterval`/`eval`; pass functions.
8. User CSS: best is to disallow uploads; next best is to let users change a fixed set of fields and generate the stylesheet yourself; hardest is sanitizing every URL-loading property. CSS can leak form values via background requests.
9. Add CSP (below). Remember CSP does not stop DOM XSS on its own.
10. (+) Prefer frameworks that escape by default (React, Vue, Blade), and treat their "raw HTML" escape hatches as review triggers.

### CSP and headers
Deliver as an HTTP header (preferred) or a `<meta http-equiv="Content-Security-Policy">`.

- `script-src 'self' https://trusted.example` restricts where scripts may load from. Inline script and `eval` are blocked by default once CSP is on.
- Avoid `'unsafe-inline'` and `'unsafe-eval'`; refactor code that depends on them. Avoid wildcard hosts (`*.example.com`): a future user-upload subdomain would silently become a script source.
- Start with the policy at project start when you already know where code comes from; it is easy to change later.
- Starter to adapt:
```
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; object-src 'none'; base-uri 'self'; frame-ancestors 'none'
```
- (+) Complementary headers: `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, cookie flags (`Secure; HttpOnly; SameSite=Lax`). (+) Roll out with `Content-Security-Policy-Report-Only` first to see what would break.

### CSRF
1. **GET never changes state.** Split combined read/update handlers; use POST/PUT/PATCH/DELETE for writes.
2. **Verify `Origin` and `Referer`** against an allowlist; reject state-changing requests that carry neither. First line of defense only: an XSS on an allowed origin defeats it, and user-generated content makes it weaker.
3. **Anti-CSRF tokens**: cryptographically random or signed, tied to the user, time-limited, sent on every form and AJAX call, verified server-side; log and reject failures.
4. **Stateless token recipe** for stateless APIs: user id + timestamp + nonce/signature using a key that exists only on the server. Don't convert a stateless API to stateful just for this.
5. **Apply as middleware to all routes** and wrap the client's request layer so the token is always attached. The weakest unprotected route is the one that gets attacked.
6. (+) `SameSite` cookies as an extra layer.

Sketch (framework-agnostic logic):
```js
// run before every route
function csrfShield(req, res, next) {
  const safe = ['GET', 'HEAD', 'OPTIONS'];
  if (!safe.includes(req.method)) {
    const origin = req.headers.origin || req.headers.referer;
    if (!origin || !allowedOrigins.some(o => origin.startsWith(o))) return res.sendStatus(403);
    if (!verifyCsrfToken(req.headers['x-csrf-token'], req.user)) return res.sendStatus(403);
  }
  next();
}
```

### XXE
- Disable DTDs/external entities in the XML parser (single config line in most libraries; e.g. disallow `doctype-decl` in Java factories). Check the library docs; do not assume the default is safe.
- If you only exchange structured data, use JSON (or another non-XML format) instead.
- Keep XML only where its features are needed (rendering, mixed content, schema validation), and then lock down the parser.
- XXE can start read-only and escalate to internal network access and remote code execution, so treat it as high severity.

### SQL injection
1. **Prepared statements / bind variables** everywhere: compile the query first, supply values after. User data can no longer change the query's intent.
2. Find all query sites: search each DB adapter's import and call pattern (`.query(`, query builders, raw ORM calls, DSLs that generate SQL).
3. Non-parameterizable parts (identifiers, ORDER BY): allowlist.
4. DB-specific escaping functions are a fallback mitigation, not a substitute.
5. Least-privilege DB accounts; no admin credentials in the app.

```js
// unsafe
db.query("SELECT * FROM users WHERE name = '" + name + "'");
// safe
db.query("SELECT * FROM users WHERE name = ?", [name]);
```

### Other injection (command, code, interpreters)
- High-risk targets: schedulers, compression/media tools, backup scripts, DBs, loggers, any host OS call, any interpreter.
- Never execute client-supplied commands. Offer a vetted allowlist of actions with fixed syntax, order, and frequency limits.
- Invoke tools with argument arrays (no shell), validate each argument.
- Run each module with least authority, separate OS users and permissions.
- Audit dependencies too: sub-dependencies often fall in the risky categories.

### DoS
**Regex DoS**: detect nested/repeated-group backtracking patterns (like `(a[ab]*)+`) in code review and with a regex linter in CI; never accept user-supplied regexes; don't integrate libraries that do.

**Logical DoS**: list the code paths that consume critical resources (CPU, memory, DB, disk, bandwidth). Rate-limit, cap sizes/counts/pagination, set timeouts, queue heavy jobs. Rate each exposed function high/medium/low for DoS risk instead of vulnerable/secure. Log request and job durations so abuse is visible.

**DDoS** can't be prevented, only mitigated: bandwidth-management/CDN/scrubbing services that inspect traffic before it reaches your server; blackholing suspicious traffic to a sink server (effective for small attacks, can catch legit users, weak against large ones). Understand normal traffic patterns first, because oversensitive filters block real users.

### Dependencies
- Model the whole dependency tree (e.g. `npm ls`) including sub-dependencies and multiple versions of the same package.
- Automate scans against CVE data (NIST NVD, Snyk-style tools) when adding or updating packages and on a schedule.
- Pin exact audited versions; removing the caret pins only the top level, so also lock the whole tree (lockfile/shrinkwrap). For maximum assurance reference Git SHAs or run a private package mirror, since a maintainer can in rare cases republish an existing version number.
- Isolate hard-to-trust components in a separate server/process and exchange JSON only; don't persist state there.

## Vulnerability discovery, analysis, and management

Contents: Discovery automation, Programs, Handling a report (reproduce, score, prioritize, fix, regression test), CVSS v3.1 quick reference, Regression test pattern, Report format

### Discovery automation
Automation is cheap, durable, and good at routine flaws, but weak at logic bugs and chained vulnerabilities. Use all three:

1. **Static analysis** (reads code, no execution). Fast, runs in editor/CI, many false positives, weaker on dynamic languages like JavaScript. Examples named in the book: Checkmarx (paid, many languages), PMD (Java), Bandit (Python), Brakeman (Ruby). Configure for the OWASP Top 10. Detectable patterns:
   - XSS: `innerHTML` and DOM sinks (including string-form `setInterval`)
   - Reflected XSS: values taken from URL params and echoed
   - SQLi: user strings used in queries
   - CSRF: state-changing GET
   - DoS: evil regexes
   - Policy rules: reject endpoints without the authorization import; reject input handling that bypasses the shared validation library
   (+) Semgrep, ESLint security plugins, CodeQL are common modern options.
2. **Dynamic analysis** (runs the app in a production-like environment). Fewer false positives, finds real runtime issues, costs more to set up and run. Examples: IBM AppScan, Veracode (paid). (+) OWASP ZAP (free).
3. **Vulnerability regression tests**: simplest and highest value. Run on commit/push hooks (reject on failure) or at least daily.

### Programs
- **Responsible disclosure policy**: publish how outsiders may test without legal risk, how to submit, a report template, and a quiet period while you fix. Without a clear path, savvy users stay silent.
- **Bug bounty** (HackerOne/Bugcrowd-style platforms supply legal templates and triage tooling): adds incentive. Reproduce reports before paying so you don't pay for false positives.
- **Third-party penetration tests**: assign scope (new and high-risk areas), can include source access under agreement; run before release and again for high-risk areas.
- Internal red/blue teams and incentives for engineers to log known issues also help.

### Handling a report
1. **Reproduce** in a staging environment that mirrors production (automated setup, mock users/data, not publicly reachable). This filters false positives and user-config mistakes, reveals root cause, and informs effort. Log the result immediately.
2. **Score** (below).
3. **Prioritize** using risk, not score alone: financial risk, difficulty of exploitation, type of data exposed, contractual obligations (SLAs, customer agreements), existing mitigations.
4. **Track** to completion with deadlines derived from risk. Check contracts for violated commitments. Add extra logging while the fix is in progress.
5. **Fix** permanently and application-wide when possible. If only a temporary fix is possible, file a new issue for the remaining surface with its own score. Never close early.
6. **Regression test** shipped with the fix.
7. **Look for siblings**: same pattern elsewhere in the codebase.

### CVSS v3.1 quick reference
Base score inputs (eight):
- **Attack Vector**: Network (worst) / Adjacent / Local / Physical
- **Attack Complexity**: Low / High (many setup steps or conditions outside attacker control = High)
- **Privileges Required**: None (guest) / Low (normal user) / High (admin)
- **User Interaction**: None / Required (e.g. must click a link)
- **Scope**: Unchanged / Changed (impact spreads beyond the vulnerable component)
- **Confidentiality / Integrity / Availability impact**: None / Low / High

Score 0-10 → severity: 0.1-3.9 Low; 4.0-6.9 Medium; 7.0-8.9 High; 9.0-10 Critical.

Other layers:
- **Temporal**: exploit maturity (theory → working exploit), remediation level (no fix → official fix), report confidence.
- **Environmental**: how much confidentiality/integrity/availability matter in YOUR app (health or tax data vs. throwaway sandbox; 24/7 uptime vs. none).
- Chained or unusual vulnerabilities, and IoT or physical-device apps, may need a custom scoring extension.

When scoring for the user, list each input with a one-line justification and give the severity band. Say if you are estimating the numeric score rather than calculating it with the official calculator.

### Regression test pattern
A security regression test looks like a functional test; its purpose is what differs. It asserts the fix still holds.

Example: a subscription-change endpoint was a state-changing GET (CSRF). Fix: POST only. Test: the endpoint allows exactly one method, POST.
```js
test('changeSubscriptionTier accepts only POST', async () => {
  const res = await request(app).options('/api/changeSubscriptionTier');
  const allowed = (res.headers['allow'] || '').split(',').map(s => s.trim());
  expect(allowed).toEqual(['POST']);
});
test('GET on changeSubscriptionTier is rejected', async () => {
  const res = await request(app).get('/api/changeSubscriptionTier?newTier=business');
  expect([404, 405]).toContain(res.status);
});
```
Other examples: sanitizer test with a known XSS string must come out inert; a query with a quote character must return no extra rows; a malicious regex input must finish under a time limit; an XML payload with an external entity must be rejected; a non-owner request for another user's object must return 403/404.

### Report format (triage output)
```
### Vulnerability: <title>
- Reproduced: yes/no/not yet (steps, environment)
- Root cause: <1-2 lines>
- CVSS v3.1 base: AV:_/AC:_/PR:_/UI:_/S:_/C:_/I:_/A:_ -> <score> (<severity>)
- Priority: <P0-P3> and why (data, contracts, mitigations)
- Fix plan: <permanent fix> ; temporary mitigation: <if any> ; remaining surface ticket: <if any>
- Interim logging: <what to log>
- Regression test: <test>
- Related spots to check: <where else the pattern occurs>
```

---

## Credits
- **Source material**: *Web Application Security* by Andrew Hoffman (O'Reilly, 2020), chapters 17-28 (Part III, Defense). This skill is an original summary and application of the book's ideas, not a reproduction of its text. Items marked (+) are modern additions.
- **Skill author**: Zulkifli | GitHub: https://github.com/Zulkifli1409 | LinkedIn: https://www.linkedin.com/in/zulkifli1409/
