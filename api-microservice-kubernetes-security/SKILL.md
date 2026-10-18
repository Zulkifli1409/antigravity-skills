---
name: api-microservice-kubernetes-security
description: Secure microservice APIs and Kubernetes deployments - pod security contexts, SSRF and DNS rebinding defense, TLS and mutual TLS between services, service mesh (Istio/Linkerd), network policies, ingress hardening, service-to-service authentication (API keys, JWT bearer, OAuth2 client credentials, certificate-bound tokens), secrets and key management, and propagating user identity across internal calls (confused deputy, phantom tokens, token exchange). Use whenever the user deploys services on Kubernetes or Docker, builds internal APIs, fetches user-supplied URLs, stores secrets, or asks how services should authenticate each other - even if they only say "microservices" or "my containers".
author: Zulkifli
github: https://github.com/Zulkifli1409
linkedin: https://www.linkedin.com/in/zulkifli1409/
source: API Security in Action by Neil Madden (Manning, 2020), chapters 10-11
---

# Microservice & Kubernetes API Security

Reply in the language the user writes in. Core mindset: **zero trust**. A request is not trustworthy because it came from inside your network; require authentication to internal services too, so that one breached pod can't roam freely (lateral movement).

## 1. Harden the workload (Kubernetes basics)
- Run containers as a non-root user with a security context:
  ```yaml
  securityContext:
    runAsNonRoot: true
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
    capabilities: { drop: ["ALL"] }
  ```
- Enforce this cluster-wide so nobody can deploy privileged pods. The book uses pod security policies; (+) those were removed from Kubernetes in v1.25, so use Pod Security Admission (restricted profile) or a policy engine instead.
- Give each service its own service account with minimal RBAC; don't mount service-account tokens into pods that don't call the Kubernetes API.
- Set CPU/memory limits and requests (DoS containment), pin image digests, scan images, keep images small.
- One database account per service with least privilege.

## 2. SSRF (server-side request forgery)
Any feature that fetches a user-supplied URL (link preview, webhook, image import, PDF render) can be aimed at internal services or the cloud metadata endpoint (the 2019 Capital One breach used SSRF to steal cloud credentials from the metadata service).

Defenses, in order of strength:
1. **Strict allowlist** of permitted hosts/URL patterns. Blocklisting private IP ranges is incomplete (alternate encodings, redirects, DNS tricks) and only a secondary layer.
2. Fetch with a client that **doesn't follow redirects automatically**; re-validate every redirect target, cap redirect count.
3. Resolve the hostname once, validate the resulting IP is public, and connect to that IP (avoids time-of-check/time-of-use).
4. Allow only `http`/`https`; restrict ports; set short timeouts and response size caps; don't return raw fetched content or error detail to the caller.
5. Run the fetcher as an isolated microservice with network policies that forbid access to internal services and to the metadata IP (e.g. 169.254.169.254); (+) require IMDSv2 on AWS.
6. Make internal services authenticate callers (zero trust), so SSRF finds nothing open.

**DNS rebinding** beats validate-then-fetch: the attacker's DNS answers with a public IP during validation (TTL 0), then an internal IP for the real request. Counter by:
- strictly validating the `Host` header on your services against known hostnames,
- using TLS for internal traffic (the certificate won't match),
- filtering DNS responses that map external names to internal IPs.

## 3. Secure internal communications
- **TLS for all service-to-service traffic.** Without it a compromised node can read and alter everything.
- **Service mesh (Linkerd, Istio)** injects a sidecar proxy that provides automatic **mutual TLS** with short-lived workload certificates and service identities, with little application code.
- **Network policies**: default-deny ingress and egress per namespace, then allow only needed flows (frontend to API, API to DB). Istio **authorization policies** can restrict by service identity, which is easier than IP-based rules.
- **Ingress controller**: terminate external TLS there, apply rate limiting and consistent TLS settings, and put it inside the mesh so ingress-to-backend is also mTLS. Strip client-supplied headers that your services trust (e.g. forwarded client-certificate headers) so clients can't spoof identity.

## 4. Service-to-service authentication
Pick one, weakest to strongest:
- **API key**: simple static secret; weak (long-lived, hard to rotate, no identity detail). Acceptable only for low-risk cases over TLS.
- **JWT bearer authentication**: the service presents a signed (or encrypted) JWT it can generate; verify issuer, audience, expiry. Short lifetimes.
- **OAuth2 client credentials grant**: service authenticates to the AS and gets an access token in its own name. Fine for simple cases.
- **Service accounts**: regular accounts flagged for machine use; flexible, but protect them strongly because they often hold elevated privileges.
- **JWT bearer grant** (RFC 7523): deploy a short-lived JWT to the service at startup and exchange it for access/refresh tokens, so no long-lived powerful credential sits on disk.
- **Mutual TLS**: both sides present certificates. Verify the chain, validity, revocation status where possible, and derive identity from the certificate (SAN/subject) rather than trusting any cert from your CA. With a mesh, the sidecar does this for you.
- **Certificate-bound access tokens** (OAuth2 mTLS, RFC 8705): the token carries the certificate thumbprint (`cnf.x5t#S256`); the API checks it matches the client cert on the TLS connection, so a stolen token is useless without the private key.

Use different credentials per service, and per environment. Scope tokens narrowly (audience = specific service).

## 5. Secrets and key management
- Kubernetes Secrets are only base64-encoded by default and can leak via environment variables (crash dumps, child processes, `kubectl describe`), mounted volumes, anyone with read access to the secret API, backups, or etcd. Turn on encryption at rest for etcd and restrict secret access with RBAC.
- Prefer a **secret vault / KMS** (HashiCorp Vault, cloud KMS/Secrets Manager) with audit logs, dynamic short-lived credentials, and rotation. The vault needs an initial credential; use a short-lived JWT or workload identity for that bootstrap.
- Never bake secrets into images, source control, or CI logs. Mount as files rather than env vars when possible.
- Keep cryptographic keys in an HSM/KMS (PKCS#11 interface) when you can; derive per-purpose keys with a KDF (HKDF) rather than reusing one key; plan and test **key rotation** and revocation.
- Separate signing keys, encryption keys, and MAC keys; never reuse a key across purposes.

## 6. Calls made on behalf of a user
When a user request fans out to internal APIs, don't let backend services act with their own broad authority (confused deputy). Pass the user's identity and enforce access at every hop.
- **Phantom token pattern**: clients get opaque tokens; the API gateway introspects them and swaps in a signed JWT with claims for internal calls. Internal services verify the JWT; external clients never see it.
- **OAuth2 token exchange (RFC 8693)**: a service exchanges the incoming token for a new, narrower token for the downstream service (audience-restricted).
- **Macaroons** with caveats work across trust boundaries to narrow permissions as a request travels.
- Never forward the raw user token blindly to services that don't need it, and never trust an unauthenticated "user id" header between services.

## 7. Review checklist
- [ ] Containers non-root, no privilege escalation, read-only FS, dropped capabilities
- [ ] Policy enforced cluster-wide; per-service service accounts and DB users
- [ ] URL-fetching features: allowlist, redirect re-validation, no internal reachability
- [ ] `Host` header validated; TLS inside the cluster
- [ ] mTLS via mesh; default-deny network policies; ingress rate-limited and header-sanitized
- [ ] Service authentication chosen deliberately; short-lived, audience-scoped credentials
- [ ] Secrets in a vault or encrypted-at-rest store, rotated, not in env/images/repos
- [ ] User identity propagated safely (phantom token / token exchange), no confused deputy

## Credits
- **Source material**: *API Security in Action* by Neil Madden (Manning, 2020), chapters 10-11. This skill is an original summary and application of the book's ideas, not a reproduction of its text. Items marked (+) are modern additions.
- **Skill author**: Zulkifli | GitHub: https://github.com/Zulkifli1409 | LinkedIn: https://www.linkedin.com/in/zulkifli1409/
