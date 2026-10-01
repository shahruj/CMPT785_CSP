# Domain Analysis Report: Spring Security

Status: starter draft for Part 1.

Topic: Spring Security  
Local source snapshot: `747f40da13de66b67249601fa9ceb8e0e99e6ef4`

## 1. Product Overview

Spring Security is an open-source Java security framework that provides authentication, authorization, and protection against common attacks for Spring-based and other Java applications. Its users include application developers, platform teams, enterprise security teams, and organizations building servlet, reactive, OAuth2/OIDC, SAML2, WebAuthn, LDAP, and method-security features into applications.

Unlike a standalone deployed service, Spring Security is usually embedded as libraries inside web applications, API services, authorization servers, and resource servers. Its domain is security-sensitive because framework mistakes can directly affect login, access-control, token validation, session handling, CSRF protection, cryptographic APIs, and passkey/WebAuthn flows across many downstream applications.

### Verification Table

| Label | Claim | Evidence | Kept/Removed |
|---|---|---|---|
| Confirmed | Spring Security provides authentication, authorization, and protection against common attacks. | https://docs.spring.io/spring-security/reference/ | Kept |
| Confirmed | Spring Security supports imperative and reactive applications. | https://docs.spring.io/spring-security/reference/ | Kept |
| Confirmed | Spring Security integrates with servlet applications through standard servlet filters. | https://docs.spring.io/spring-security/reference/servlet/index.html | Kept |
| Confirmed | Spring Security has many modules, including core, web, config, LDAP, OAuth2, ACL, CAS, test, and taglibs. | https://docs.spring.io/spring-security/reference/modules.html and local source directories | Kept |
| Confirmed | Spring Security is Apache 2.0 open-source software. | `spring-security-src/README.adoc`; `spring-security-src/LICENSE.txt` | Kept |
| Confirmed | The advisory index lists multiple Spring Security CVEs. | https://spring.io/security/ | Kept |
| Confirmed | The local source clone uses Gradle and dynamic module inclusion. | `spring-security-src/settings.gradle`; `spring-security-src/build.gradle` | Kept |
| Removed | Spring Security is itself a cloud SaaS identity provider. | It is a framework/library; deployment occurs inside applications that depend on it. | Removed |
| Hypothesis | Enterprise Java/Spring web apps are the dominant deployment context. | Plausible from Spring ecosystem and docs, but not yet quantified. | Keep only if worded carefully. |

## 2. Project Assets

| Label | Asset | Why It Matters | Verification | Origin |
|---|---|---|---|---|
| Confirmed | Authentication decisions and principals | A bug can let attackers impersonate users or services. | Reference docs; `core/src/main/java/org/springframework/security/authentication`. | Added by us |
| Confirmed | Authorization decisions and access rules | A bug can expose protected URLs, methods, or domain objects. | Modules page; `core/.../authorization`; `web/.../AuthorizationFilter.java`. | Added by us |
| Confirmed | Servlet security filter chain | Filter order and matching determine whether protections run. | Servlet docs; `web/src/main/java/org/springframework/security/web/FilterChainProxy.java`; `SecurityFilterChain.java`. | Added by us |
| Confirmed | CSRF tokens and validation | CSRF failures can allow unwanted state-changing actions. | CSRF docs; `web/src/main/java/org/springframework/security/web/csrf/CsrfFilter.java`. | Added by us |
| Confirmed | OAuth2/OIDC tokens and DPoP proofs | Token validation/replay bugs can cause API compromise. | OAuth2 docs; `oauth2/oauth2-jose/.../DPoPProofReplayValidator.java`. | Added by us |
| Confirmed | WebAuthn/passkey ceremony state | User verification failures can bypass intended biometric/PIN checks. | CVE-2026-47841; `webauthn/.../UserVerificationRequirement.java`. | Added by us |
| Confirmed | Cryptographic helper APIs | Unsafe encryption behavior can leak sensitive stored data patterns. | CVE-2026-47842; `crypto/.../AesBytesEncryptor.java`. | Added by us |
| Confirmed | Distributed session state | Serialized session objects can affect security decisions. | CVE-2026-47841 advisory. | Added by us |
| Confirmed | Security configuration DSL/XML namespace | Config parsing/building errors can disable protections or misapply rules. | Modules page says config module contains namespace parsing and Java configuration. | Added by us |
| Rejected | Packet capture files | Old-topic artifact from an earlier candidate; not a Spring Security product asset. | No supporting evidence. | Rejected |

## 3. Example Attacks

### Our Scenarios

1. **DPoP proof replay through cache pressure.** An attacker intercepts a valid DPoP proof and then floods the server with dummy proofs to force cache pressure. If the original `jti` is evicted or not retained correctly, the attacker can replay the proof and impersonate the victim. This targets authentication/integrity and maps to CVE-2026-41707.

2. **WebAuthn user verification bypass after session serialization.** An application requires WebAuthn user verification and uses a distributed session store. If framework code compares the verification requirement by object identity rather than value, deserialization can make the requirement check fail. An attacker with a stolen authenticator may authenticate without satisfying PIN/biometric verification. This maps to CVE-2026-47841.

3. **Encrypted data correlation through deterministic AES-CBC.** An application stores sensitive values encrypted with affected `AesBytesEncryptor` CBC behavior. An attacker with read access to the database can identify equal plaintext values by comparing ciphertext and can attempt dictionary attacks. This maps to CVE-2026-47842.

4. **CSRF-sensitive state change.** A victim is logged into a Spring Security-protected web application. If CSRF protection is disabled or bypassed, a malicious site can cause the victim browser to send a state-changing request. This targets integrity of user actions. This is a general scenario supported by the CSRF docs, not yet tied to a specific Spring Security CVE.

### Tool-Suggested Scenarios and Verdicts

| Tool Suggestion | Verdict | Reason |
|---|---|---|
| Packet parser crash | Rejected | Old-topic artifact; not Spring Security's domain. |
| Generic SQL injection | Hypothesis/reject | Could occur in apps, but needs Spring Security-specific source/advisory evidence. |
| Weak password hashing | Hypothesis | Relevant module exists, but must be tied to a concrete API, default, or advisory before counting. |

## 4. Vulnerability History

Each group member should own one vulnerability. The table below is a starting shortlist.

| Candidate | Impact | Why It Is Useful | Current Evidence |
|---|---|---|---|
| `CVE-2026-47842`, deterministic AES/CBC encryption | Confidentiality | Compact source area and clear cryptographic root cause. | Advisory; `crypto/.../AesBytesEncryptor.java`; replacement encryptors. |
| `CVE-2026-47841`, WebAuthn user verification bypass | Authentication/integrity | Clear object identity/value equality mistake under distributed sessions. Selected for Shahruj's Vulnerability History 4. | Advisory; `webauthn/.../UserVerificationRequirement.java`; WebAuthn operations/tests; see `Report/vulnerability-history-4-cve-2026-47841.md`. |
| `CVE-2026-41707`, DPoP proof replay | Authentication/integrity | Modern OAuth2 proof-of-possession issue with a cache/replay root cause. | Advisory; `DPoPProofJwtDecoderFactory`; `DPoPProofReplayValidator`. |

### Vulnerability Detail: CVE-2026-47841

Shahruj's prepared deep dive is in `Report/vulnerability-history-4-cve-2026-47841.md`.

Short version: affected Spring Security WebAuthn/passkey flows used object identity (`==`) to check whether `UserVerificationRequirement` was `REQUIRED`. In distributed-session deployments, serialization/deserialization could produce an equivalent `"required"` object that was not the same object reference as the static constant, causing Spring Security to treat user verification as not required. The public fix commit is `a447020c9236e7e258517b0fd327ea50331a65fc` (`Improve Equivalence Tests`), and `git blame` traces the vulnerable checks back to `b0e8730d70ee548cd383ba358ec87e268b52c29b` (`Add Passkeys Support`).

Important scope note: this does not mean anyone can log in without a passkey. The attacker still needs the victim's authenticator or equivalent credential material; the bypass is of PIN/biometric user verification, not of authenticator possession.

### Vulnerability Detail Starter: CVE-2026-47842

Name: `CVE-2026-47842`  
Advisory: https://spring.io/security/cve-2026-47842/  
Affected feature: `AesBytesEncryptor` using CBC with null/fixed IV behavior  
Affected source area: `crypto/src/main/java/org/springframework/security/crypto/encrypt/`

In our words: affected Spring Security encryption APIs can produce deterministic AES/CBC ciphertext when using a fixed all-zero IV. If two plaintexts are equal under the same password/salt, their ciphertexts match, letting an attacker with read access to encrypted storage correlate records and mount dictionary attacks.

CIA category: Confidentiality.

Likely coding/design mistake: an encryption helper API allowed CBC mode with a default/null IV generator, making IVs deterministic rather than unpredictable.

More work needed:

- Find the exact fix commit/PR.
- Compare old `AesBytesEncryptor` behavior with new `AesCbcBytesEncryptor`/`AesGcmBytesEncryptor` replacements.
- Trace the introducing commit and count developers touching the source between introduction and fix.

### Tool Claim Comparison Starter

| Claim | Tool Claimed | Verified Result | Evidence | Verdict |
|---|---|---|---|---|
| Spring Security has public security advisories | Yes | Correct | Spring advisory index lists Spring Security CVEs. | Confirmed |
| CVE-2026-47842 affects `AesBytesEncryptor` | Yes | Correct | Advisory and local `crypto/.../AesBytesEncryptor.java`. | Confirmed |
| CVE-2026-47841 is about WebAuthn session serialization | Yes | Correct | Advisory and local `webauthn` source. | Confirmed |
| Exact fix commit for CVE-2026-47842 | TBD | Not yet verified | Need GitHub history/PR or tags. | Hypothesis |
| Introducing commit | TBD | Not yet verified | Need `git blame` on affected branch/history. | Hypothesis |

## 5. Methodology and Tool Notes

Initial methodology: center the work on Spring Security-specific evidence from official Spring documentation, Spring's public advisory index, and a local clone of the source repository. Claims are kept only when tied to an official URL, local source path, or command-verifiable repository fact. Old-topic artifacts and generic application-security claims are marked rejected or hypothesis until tied to Spring Security source/advisories.

Tools used so far:

- Web search and official Spring documentation/advisory pages for product and vulnerability facts.
- Local shallow clone of Spring Security `main`.
- `rg` against source for module, CSRF, OAuth2, WebAuthn, and crypto code paths.
- Git commands to verify repository state and commit hash.

Most reliable so far: official reference docs, official advisories, local source paths.  
Least reliable so far: generic web-app attack suggestions that are not specifically caused by Spring Security code.

Concrete change for Part 2: build the architecture from Spring Security modules (`core`, `web`, `config`, `crypto`, `oauth2`, `webauthn`, etc.) and then use tools to challenge missing trust boundaries.

## Appendix A: Workflow Log Starter

| ID | Tool/model | Date | Access/input | Role/task | Raw output file | Verification | Verdict |
|---|---|---|---|---|---|---|---|
| A0 | ChatGPT/Codex visible conversation transcript | 2026-09-30 | Visible Codex conversation and tool-action context | Preserve full visible session text and project pivot history | `AI_Sessions/A0-full-visible-session-transcript.md` | Checked against current chat context and generated workspace files | Transcript artifact; use A1+ for verified findings |
| A1 | ChatGPT/Codex with web search and local shell | 2026-09-30 | Course assignment text, official Spring docs/advisories, local shallow clone at `747f40d...` | Produce Spring Security Domain Analysis starter report material | `AI_Sessions/A1-codex-spring-security-research.md` | Claims checked against official URLs, local `rg`, and Git source paths | Mixed: confirmed/rejected/hypothesis as labelled |
| A2 | ChatGPT/Codex with web search and local shell | 2026-10-02 | Group preliminary report, official Spring advisory, local Spring Security repository with tags/history | Research and draft Vulnerability History 4 for `CVE-2026-47841` | `AI_Sessions/A2-cve-2026-47841-history-research.md` | Advisory URLs, `git diff`, `git show`, `git blame`, tag checks | Mixed: confirmed/refuted/hypothesis as labelled |

## Appendix B: Contribution Record Starter

| Team Member | Tasks | AI/tool sessions run | Findings verified |
|---|---|---|---|
| Shahruj | Topic pivot, initial Spring Security research setup, source clone, starter report skeleton, Vulnerability History 4 deep dive for `CVE-2026-47841` | A1, A2 | Product overview, assets, vulnerability shortlist, WebAuthn user verification bypass root cause/fix/introduction |
| TBD | Vulnerability 1 | TBD | TBD |
| TBD | Vulnerability 2 | TBD | TBD |
| TBD | Vulnerability 3 | TBD | TBD |

