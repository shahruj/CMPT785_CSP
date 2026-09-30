# A1 - Codex Spring Security Initial Research Session

Date: 2026-09-30  
Tool/model: ChatGPT/Codex coding agent, local shell, web search/browser  
Operator: A M Shahruj Rashid with Codex assistance  
Project phase: CSP Part 1 - Domain Analysis  
Topic: Spring Security

## Purpose

Create a basic evidence-backed research base for:

- Product overview
- Project assets
- Example attacks
- Vulnerability-history candidates
- Methodology and workflow appendix

## Inputs Given To The Tool

1. Course assignment text pasted from Canvas.
2. User instruction that the final selected topic is Spring Security.
3. Local workspace:
   - `C:\Users\amsha\OneDrive\Documents\ChatGPT\CMPT785`
4. Internet access for official Spring documentation and advisories.
5. GitHub repository access for Spring Security source.

## Role / Task Prompt Summary

The tool was asked to start research for the CMPT 785 Domain Analysis Report using Spring Security as the selected topic. The output needed to support the assignment requirement that AI/tool statements be traceable, verified, and marked as Confirmed, Refuted/Rejected, or Hypothesis.

## Tools / Commands Used

### Web Sources Opened

- Spring Security Reference: https://docs.spring.io/spring-security/reference/
- Spring Security Modules: https://docs.spring.io/spring-security/reference/modules.html
- Spring Security Servlet Applications: https://docs.spring.io/spring-security/reference/servlet/index.html
- Spring Security CSRF Reference: https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html
- Spring Security OAuth2 Reference: https://docs.spring.io/spring-security/reference/servlet/oauth2/
- Spring Security Advisories: https://spring.io/security/
- CVE-2026-41707 advisory: https://spring.io/security/cve-2026-41707/
- CVE-2026-47841 advisory: https://spring.io/security/cve-2026-47841/
- CVE-2026-47842 advisory: https://spring.io/security/cve-2026-47842/

### Repository Commands

```powershell
git ls-remote --heads https://github.com/spring-projects/spring-security.git
```

Purpose: verify repository exists and inspect available branches.

```powershell
git clone --depth 1 --branch main https://github.com/spring-projects/spring-security.git spring-security-src
```

Purpose: create local repository clone for source verification.

Note: the first checkout hit Windows long-path errors. The clone was repaired using:

```powershell
git config core.longpaths true
git restore --source=HEAD :/
git -c core.longpaths=true reset --hard HEAD
```

Final verified commit:

```text
747f40da13de66b67249601fa9ceb8e0e99e6ef4
```

### Source Inspection Commands

Representative `rg` searches used:

```powershell
rg -n "Spring Security is|authentication|authorization|common attacks|de-facto|Apache License|module" README.adoc settings.gradle build.gradle gradle.properties
```

```powershell
rg -n "class DPoPProofJwtDecoderFactory|DPoPProofJwtDecoderFactory|jti|replay|cache" oauth2 -g "*.java"
```

```powershell
rg -n "class UserVerificationRequirement|UserVerificationRequirement|readResolve|equals\(|== UserVerificationRequirement|userVerification" webauthn -g "*.java"
```

```powershell
rg -n "class AesBytesEncryptor|class AesCbcBytesEncryptor|class AesGcmBytesEncryptor|NULL_IV_GENERATOR|ivGenerator|CipherAlgorithm.CBC|Encryptors\.standard|Encryptors\.stronger|deprecated" crypto/src/main/java -g "*.java"
```

```powershell
rg -n "class CsrfFilter|DEFAULT_CSRF_MATCHER|doFilterInternal|CsrfTokenRepository|CsrfTokenRequestHandler" web/src/main/java/org/springframework/security/web/csrf -g "*.java"
```

```powershell
rg -n "interface SecurityFilterChain|class FilterChainProxy|class UsernamePasswordAuthenticationFilter|class BearerTokenAuthenticationFilter|class AuthorizationFilter|class AuthorizationManager" web core oauth2 -g "*.java"
```

## Tool Output Incorporated Into Report

The session created or updated:

- `Research/spring-security-domain-research.md`
- `Report/domain-analysis-spring-security-draft.md`
- `spring-security-src/` local source clone

Earlier non-Spring draft files were deleted/replaced:

- `Research/previous-topic-domain-research.md`
- `Report/domain-analysis-previous-topic-draft.md`

## Confirmed Findings From This Session

| Finding | Verification | Verdict |
|---|---|---|
| Spring Security provides authentication, authorization, and protection against common attacks. | Official reference page: https://docs.spring.io/spring-security/reference/ | Confirmed |
| Spring Security supports imperative and reactive applications. | Official reference page. | Confirmed |
| Spring Security integrates with servlet applications through standard servlet filters. | Servlet docs: https://docs.spring.io/spring-security/reference/servlet/index.html | Confirmed |
| Spring Security has public security advisories. | Advisory index: https://spring.io/security/ | Confirmed |
| Spring Security source was cloned locally. | `git rev-parse HEAD` returned `747f40da13de66b67249601fa9ceb8e0e99e6ef4`. | Confirmed |
| Spring Security is Apache 2.0 licensed. | `spring-security-src/README.adoc`; `spring-security-src/LICENSE.txt`. | Confirmed |
| `CVE-2026-47842` concerns deterministic AES/CBC encryption in `AesBytesEncryptor`. | Official advisory and local source under `crypto/src/main/java/org/springframework/security/crypto/encrypt/`. | Confirmed |
| `CVE-2026-47841` concerns WebAuthn user verification bypass with distributed sessions. | Official advisory and local source under `webauthn/`. | Confirmed |
| `CVE-2026-41707` concerns DPoP proof replay. | Official advisory and local source under `oauth2/oauth2-jose/`. | Confirmed |

## Rejected / Refuted Tool Suggestions

| Suggestion | Reason | Verdict |
|---|---|---|
| Packet capture files as a Spring Security asset | This was old-topic carry-over. Spring Security is not a packet analyzer. | Rejected |
| Packet parser crash as an example attack | Same old-topic carry-over; not specific to Spring Security. | Rejected |
| Spring Security as a standalone cloud SaaS identity provider | Spring Security is a framework/library embedded into applications. | Rejected |

## Hypotheses Left For Follow-Up

| Hypothesis | Needed Verification |
|---|---|
| Enterprise Java/Spring applications are the dominant deployment context. | Need a credible source or avoid quantified wording. |
| Exact fix commit for CVE-2026-47842. | Need GitHub PR/commit history or unshallow clone/tags. |
| Introducing commit for selected vulnerability. | Need `git blame`/history on affected branch. |
| Developer count from introduction to fix. | Need exact introducing and fixing commits first. |

## Notes For Appendix A

Use this file as the raw/session-output artifact for Appendix A entry `A1`.

Suggested Appendix A entry:

| ID | Tool/model | Date | Access/input | Role/task | Raw output file | Verification | Verdict |
|---|---|---|---|---|---|---|---|
| A1 | ChatGPT/Codex with web search and local shell | 2026-09-30 | Course text, official Spring docs/advisories, local shallow clone at `747f40d...` | Produce Spring Security Domain Analysis starter report material | `AI_Sessions/A1-codex-spring-security-research.md` | Official URLs, local `rg`, source paths, Git commit hash | Mixed: confirmed/rejected/hypothesis as labelled |


