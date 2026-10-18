---
name: iot-api-security
description: Secure IoT and constrained-device APIs - DTLS and cipher choices for low-power devices, pre-shared keys vs X.509 certificates, end-to-end encryption and authentication across protocol hops (COSE, NaCl), nonce reuse and misuse-resistant encryption, key distribution and derivation, device authentication, replay prevention (timestamps, nonces, counters, ETag), OAuth2 device grant, and offline authorization. Use whenever the user builds or secures devices that talk to an API such as ESP32, Arduino, Raspberry Pi, sensors, MQTT or CoAP gateways, smart-home or tracking hardware, or asks how a device should authenticate to a server - even if the word IoT is not used.
author: Zulkifli
github: https://github.com/Zulkifli1409
linkedin: https://www.linkedin.com/in/zulkifli1409/
source: API Security in Action by Neil Madden (Manning, 2020), chapters 12-13
---

# IoT API Security

Reply in the language the user writes in. Constrained devices (little CPU, RAM, storage, battery, bandwidth) cannot always run standard web security stacks, so recommend the lightest option that still meets the goal and say what is being traded away.

## 1. Transport security on devices
- **UDP-based protocols (CoAP, etc.)**: use **DTLS** (datagram TLS). TCP-based: TLS.
- **Cipher suites** suited to small devices: AES-CCM (small code size, hardware-accelerated on many chips) or ChaCha20-Poly1305 (fast without AES hardware). Avoid CBC modes and anything without authenticated encryption.
- **Pre-shared keys (PSK) vs X.509 certificates**:
  - Certificates need parsing, chain validation, and signature code, which costs flash, RAM, and CPU. Good for capable devices and PKI-managed fleets.
  - PSK removes that overhead and uses fast symmetric crypto, but you must provision and protect a unique key per device.
  - For **forward secrecy** with PSK, use PSK combined with ephemeral Diffie-Hellman (ECDHE-PSK) on devices that can afford it, so a leaked key doesn't expose past traffic.
- Never share one key across the whole fleet; one extracted device would compromise everything.

## 2. End-to-end security across hops
Messages often cross different transports (device to gateway over BLE/CoAP, gateway to cloud over HTTPS/MQTT). Transport security stops at each hop, so a compromised gateway or broker sees and can alter everything. Add **end-to-end** protection of the message itself:
- Authenticated encryption of the payload (COSE for CBOR, JOSE for JSON; the book also notes NaCl-style boxes are simpler and harder to misuse).
- Authenticate the message and its metadata, not just the body.
- Keep keys only at the two real endpoints.

## 3. Nonces, entropy, misuse resistance
- AES-CCM/GCM and similar modes are **catastrophically broken if a nonce is reused with the same key** (an attacker can recover plaintext or forge messages).
- Constrained devices often have poor randomness at boot, so random nonces are risky. Safer options: a persistent monotonic counter in nonce, or a **misuse-resistant** mode such as SIV-AES, where an accidental nonce repeat only leaks that two messages were identical.
- Ensure a proper entropy source (hardware RNG) before generating keys; never seed with time or a fixed value.

## 4. Key management at fleet scale
- **Unique key per device**. Derive device keys from a master key with a KDF (HKDF) and the device id, so the server stores one master secret (kept in an HSM/KMS) instead of millions of keys.
- **Key distribution server** for provisioning and rotation; plan how a device gets its first key (factory provisioning, secure enrollment) and how it is revoked.
- **Ratcheting** (hash the key forward after use and delete the old one) gives forward secrecy even for devices that can't do public-key crypto.
- Use hardware protection where available: secure elements, TPM, trusted execution, and disabled debug ports, so a physical attacker can't read keys easily. Assume devices WILL be physically accessible.

## 5. Authenticating devices
- Give every device an identity (device profile): id, public key or encrypted PSK, owner, status, allowed operations.
- **Transport-layer** authentication (client certificate or PSK identity in the (D)TLS handshake) is simplest when a secure end-to-end connection exists.
- Without that, implement app-layer authentication where the device signs or MACs its requests.
- Store device credentials encrypted; protect the device registry; support disabling a stolen device immediately.

## 6. Replay attacks and freshness
End-to-end authenticated messages can be captured and replayed. Prove freshness with one of:
- **Timestamps** (needs reasonably synced clocks, accept a small window),
- **Nonces** the server remembers (state per device),
- **Challenge-response** (server sends a random challenge, device signs or MACs it),
- **Monotonic counters** (server stores the highest value seen per device and rejects lower or equal).
Preventing replay requires per-device state on the server; budget for it.

**REST pattern with ETags**: include the resource's current ETag in the authenticated request (e.g. via `If-Match`, covered by the MAC). Because the ETag changes whenever the resource changes, a replayed old request no longer matches and is rejected, giving replay protection without extra counters.

## 7. Authorizing devices and users (OAuth2)
- **OAuth2 device authorization grant** lets a keyboard-less device obtain a user-approved token: the device shows or sends a short code and URL, the user approves on their phone, the device polls the token endpoint. Use short code lifetimes and polling interval limits.
- The IETF **ACE-OAuth** work adapts OAuth2 to constrained environments (CBOR/CWT tokens, CoAP, DTLS).
- Issue narrow scopes and short-lived tokens to devices; bind tokens to the device key (proof-of-possession) so stolen tokens are useless.

## 8. Offline operation
Devices can't always reach the cloud. Design for it explicitly:
- **Self-contained tokens** carrying credentials and policy (what the holder may do, expiry, conditions) that the device verifies locally.
- **Proof-of-possession constraints** so a token only works with the matching device or client key.
- Keep offline authority limited in scope and time; sync and revoke once connectivity returns; log locally and upload later.
- Cache only what's needed; assume local storage can be extracted.

## 9. Review checklist
- [ ] (D)TLS with an AEAD cipher suite; unique credentials per device
- [ ] End-to-end protection if any hop is untrusted; keys held only at endpoints
- [ ] Nonce strategy is safe (counter or misuse-resistant mode) and RNG is sound
- [ ] Device keys derived or stored in HSM/KMS; rotation and revocation planned
- [ ] Replay prevented (timestamp, nonce, challenge, counter, or ETag)
- [ ] Device grant or equivalent for user-approved access; tokens short-lived and PoP-bound
- [ ] Offline access bounded and auditable
- [ ] Physical tamper assumptions documented (debug ports, secure element)

## Credits
- **Source material**: *API Security in Action* by Neil Madden (Manning, 2020), chapters 12-13. This skill is an original summary and application of the book's ideas, not a reproduction of its text.
- **Skill author**: Zulkifli | GitHub: https://github.com/Zulkifli1409 | LinkedIn: https://www.linkedin.com/in/zulkifli1409/
