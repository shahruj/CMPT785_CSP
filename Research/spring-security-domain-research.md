# Spring Security Domain Analysis Research Starter

Working repository clone: `spring-security-src`

Repository snapshot used locally:

- Git URL: https://github.com/spring-projects/spring-security.git
- Branch: `main`
- Local commit: `747f40da13de66b67249601fa9ceb8e0e99e6ef4`
- Clone type: shallow clone, depth 1
- Note: checkout required `core.longpaths=true` on Windows due long test file paths.

## Source Set

Primary sources gathered so far:

- Spring Security Reference: https://docs.spring.io/spring-security/reference/
- Project Modules and Dependencies: https://docs.spring.io/spring-security/reference/modules.html
- Servlet Applications: https://docs.spring.io/spring-security/reference/servlet/index.html
- CSRF reference: https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html
- OAuth2 reference: https://docs.spring.io/spring-security/reference/servlet/oauth2/
- Spring Security Advisories: https://spring.io/security/
- Spring Security source repo: https://github.com/spring-projects/spring-security
- Local source clone: `spring-security-src/`

## Product Overview Claims

| Label | Claim | Evidence | Notes |
|---|---|---|---|
| Confirmed | Spring Security is a framework for authentication, authorization, and protection against common attacks. | Reference landing page says this directly. | Good opening claim. |
| Confirmed | Spring Security supports both imperative and reactive applications. | Reference landing page states first-class support for both. | Useful deployment/context claim. |
| Confirmed | It is the de facto standard for securing Spring-based applications. | Reference landing page states this. | Quote/paraphrase carefully. |
| Confirmed | Current docs observed during research are for Spring Security 7.1.1, with stable lines 7.1.1, 7.0.7, and 6.5.11 listed. | Reference landing page version block. | Date this if included. |
| Confirmed | Spring Security is open source under Apache 2.0. | `spring-security-src/README.adoc`; `spring-security-src/LICENSE.txt`. | Selected-case-study fit. |
| Confirmed | Spring Security 6.0+ requires Spring 6.0+ and Java 17. | `spring-security-src/README.adoc` states this. | Version-specific; avoid overgeneralizing to older lines. |
| Confirmed | Spring Security uses Gradle and is split into many modules. | `settings.gradle` dynamically includes projects from Gradle build files; modules include `core`, `web`, `config`, `crypto`, `oauth2`, `saml2`, `webauthn`, etc. | Supports scale and source layout. |
| Confirmed | Spring maintains a public security advisory index with Spring Security CVEs. | Spring advisory page lists Spring Security CVEs such as CVE-2026-41707, CVE-2026-47841, and CVE-2026-47842. | Selected-case-study hard requirement. |
| Removed | Spring Security is itself a deployed identity provider SaaS. | The project is a library/framework used inside applications, not a standalone SaaS. Spring Authorization Server is related but separate in docs. | Keep deployment context precise. |
| Hypothesis | Most deployments are enterprise Java/Spring web applications. | Strongly plausible given Spring ecosystem and docs, but no quantified user base found yet. | Phrase as likely/common, not numeric. |

## Initial Asset Inventory

| Label | Asset | Why It Matters | Evidence | Origin / Verdict |
|---|---|---|---|---|
| Confirmed | Authentication decisions and authenticated principals | Incorrect authentication lets attackers impersonate users or services. | Docs define authentication as core purpose; source has `core/src/main/java/org/springframework/security/authentication` and many authentication filters/providers. | Added by us. |
| Confirmed | Authorization decisions and access-control rules | Incorrect authorization exposes protected resources or actions. | Modules page says core contains access-control classes; source has `core/src/main/java/org/springframework/security/authorization` and `web/.../AuthorizationFilter.java`. | Added by us. |
| Confirmed | Servlet filter chain | Servlet integration uses standard filters; ordering and matching decide which protections run. | Servlet docs; source has `web/src/main/java/org/springframework/security/web/FilterChainProxy.java` and `SecurityFilterChain.java`. | Added by us. |
| Confirmed | CSRF tokens and request validation | CSRF bypass allows state-changing actions from another site. | CSRF docs say protection is on by default for unsafe HTTP methods; source has `web/src/main/java/org/springframework/security/web/csrf/CsrfFilter.java`. | Added by us. |
| Confirmed | OAuth2/OIDC tokens and DPoP proofs | Bearer tokens, JWTs, and DPoP proofs protect APIs and clients; replay or validation bugs can cause impersonation. | OAuth2 docs; source has OAuth2 modules and `DPoPProofJwtDecoderFactory`, `DPoPProofReplayValidator`. | Added by us. |
| Confirmed | WebAuthn/passkey ceremonies and user verification state | WebAuthn bypass can let stolen authenticators log in without required verification. | CVE-2026-47841 advisory; source has `webauthn` module and `UserVerificationRequirement`. | Added by us. |
| Confirmed | Cryptographic helper APIs and encrypted application data | Deterministic encryption or weak defaults can leak sensitive data patterns. | CVE-2026-47842; source has `crypto/.../AesBytesEncryptor.java`, `AesCbcBytesEncryptor.java`, and `AesGcmBytesEncryptor.java`. | Added by us. |
| Confirmed | Session state and distributed session serialization | Session serialization can change object identity/state and affect security decisions. | CVE-2026-47841 explicitly depends on distributed HTTP session stores. | Added by us. |
| Confirmed | Security configuration DSL/XML namespace | Misconfiguration or parser/config bugs affect entire app security posture. | Modules page says `spring-security-config.jar` contains namespace parsing and Java config code. | Added by us. |
| Rejected | Packet capture files | Generic carry-over from an earlier topic; not a Spring Security product asset. | No relevance to framework domain. | Rejected. |

## Example Attack Scenarios

### Our Scenarios

1. Confirmed/Hypothesis mix: DPoP proof replay after cache pressure.
   - Attacker intercepts a valid DPoP proof, then sends dummy proofs to force cache pressure.
   - If the legitimate `jti` is evicted, the attacker replays the valid proof.
   - Impact: unauthorized access and impersonation.
   - Evidence: CVE-2026-41707; source area `oauth2/oauth2-jose/.../DPoPProofReplayValidator.java`.

2. Confirmed/Hypothesis mix: WebAuthn user verification bypass with distributed sessions.
   - Application uses WebAuthn with `userVerification = REQUIRED` and stores sessions in Redis/JDBC/Hazelcast.
   - Serialization/deserialization changes object identity, causing identity comparison to fail.
   - Attacker with a user's authenticator may authenticate without required PIN/biometric verification.
   - Evidence: CVE-2026-47841; source has `UserVerificationRequirement` and tests asserting post-deserialization identity.

3. Confirmed/Hypothesis mix: Encrypted database value correlation.
   - Application uses deprecated/affected `AesBytesEncryptor` CBC mode with fixed/null IV behavior.
   - Attacker with read access to encrypted rows observes equal ciphertext values and correlates users or fields.
   - Impact: confidentiality loss, pattern leakage, possible dictionary attack.
   - Evidence: CVE-2026-47842; source has deprecated `AesBytesEncryptor` and replacements.

4. Confirmed/Hypothesis mix: CSRF protection bypass or misapplication.
   - Application performs state-changing actions using unsafe HTTP methods.
   - If CSRF protection is disabled/misconfigured or a framework bug mishandles token validation, attacker can cause the victim browser to perform actions.
   - Evidence: CSRF docs and `CsrfFilter`; no specific CVE chosen yet.

### Tool-Suggested Scenarios To Treat Carefully

| Scenario | Current Verdict | Reason |
|---|---|---|
| Buffer overflow in packet parser | Rejected as old-topic/generic | Spring Security is primarily Java framework code, not a network packet parser. |
| SQL injection in Spring Security itself | Hypothesis/reject unless tied to JDBC module source/advisory | Applications using it can have SQLi, but the framework asset needs concrete source evidence. |
| Weak password storage | Hypothesis | Spring Security has password encoders, but claims need source/advisory evidence and should distinguish framework defaults from app misuse. |

## Vulnerability History Candidates

### Strong Candidate 1: CVE-2026-47842 / deterministic AES-CBC encryption

- Advisory: https://spring.io/security/cve-2026-47842/
- Severity: Medium
- Affected lines: Spring Security 7.1.0, 7.0.0-7.0.6, 6.5.0-6.5.11, 6.4.0-6.4.18, 5.8.0-5.8.27, 5.7.0-5.7.25.
- Impact: confidentiality; fixed IV makes equal plaintext produce equal ciphertext for a given password/salt.
- Relevant source:
  - `crypto/src/main/java/org/springframework/security/crypto/encrypt/AesBytesEncryptor.java`
  - `crypto/src/main/java/org/springframework/security/crypto/encrypt/AesCbcBytesEncryptor.java`
  - `crypto/src/main/java/org/springframework/security/crypto/encrypt/AesGcmBytesEncryptor.java`
  - `crypto/src/main/java/org/springframework/security/crypto/encrypt/Encryptors.java`
- Why strong: advisory is detailed, source is compact, root cause is explainable, and CIA category is clear.
- More work needed:
  - Identify fix commit(s) and introducing commit using GitHub history after unshallowing or using GitHub web/API.
  - Compare actual diff against advisory migration guidance.

### Strong Candidate 2: CVE-2026-47841 / WebAuthn user verification bypass

- Advisory: https://spring.io/security/cve-2026-47841/
- Severity: High
- Impact: authentication bypass under specific distributed-session conditions.
- Relevant source:
  - `webauthn/src/main/java/org/springframework/security/web/webauthn/api/UserVerificationRequirement.java`
  - `webauthn/src/main/java/org/springframework/security/web/webauthn/management/Webauthn4JRelyingPartyOperations.java`
  - `webauthn/src/test/java/org/springframework/security/web/webauthn/api/UserVerificationRequirementTests.java`
- Why strong: clear coding mistake (`==` identity comparison vs value equality) and security consequence.
- More work needed:
  - Verify exact vulnerable code in an affected tag/branch.
  - Find fix commit and compare `==` to `.equals()` or serialization fix.

### Strong Candidate 3: CVE-2026-41707 / DPoP proof replay

- Advisory: https://spring.io/security/cve-2026-41707/
- Severity: High
- Impact: unauthorized access and impersonation after replay of intercepted proof.
- Relevant source:
  - `oauth2/oauth2-jose/src/main/java/org/springframework/security/oauth2/jwt/DPoPProofJwtDecoderFactory.java`
  - `oauth2/oauth2-jose/src/main/java/org/springframework/security/oauth2/jwt/DPoPProofReplayValidator.java`
  - `oauth2/oauth2-jose/src/test/java/org/springframework/security/oauth2/jwt/DPoPProofJwtDecoderFactoryTests.java`
- Why strong: source now has tests/comments around cache-full behavior that directly match advisory mechanics.
- More work needed:
  - Verify the vulnerable previous behavior in an affected tag.
  - Find fix commit and compare cache eviction/rejection semantics.

## Immediate Next Research Tasks

1. Pick one vulnerability per group member from the shortlist.
2. For each vulnerability, identify exact fix commit/PR and affected source files.
3. Fetch enough Git history for selected files/tags to run `git blame`.
4. Build comparison tables: tool claims vs verified advisory/source/git facts.
5. Save raw AI/tool outputs in `AI_Sessions/` and cite them from Appendix A.


