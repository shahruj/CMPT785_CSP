# AI Session Artifact Skill

Use this skill when contributing to the CMPT 785 Case Study Project and any AI, agent, scanner, script, or tool output is used in the report.

The goal is to make every AI/tool-assisted claim traceable, verifiable, and easy to audit.

## Folder Convention

Store all AI/tool run artifacts in:

```text
AI_Sessions/
```

Use stable IDs:

```text
A0-full-visible-session-transcript.md
A1-codex-spring-security-research.md
A2-tool-or-agent-name-task.md
A3-semgrep-run.md
A4-github-history-cve-lookup.md
```

Use one file per meaningful AI session or tool run.

## Required Artifact Structure

Each session file should include:

```markdown
# A{N} - Short Session Title

Date:
Tool/model:
Operator:
Project phase:
Topic:

## Purpose

What the run was meant to produce.

## Inputs Given To The Tool

List repositories, commit hashes, files, URLs, prompts, prior outputs, scanner configs, rulesets, or assumptions provided.

## Role / Task Prompt Summary

Record the role/task prompt verbatim when possible. If not possible, provide a faithful summary.

## Tools / Commands Used

Include commands, tool names, versions, scanner rulesets, URLs opened, and configuration.

## Raw Output Or Transcript

Paste the raw output when short enough. For long outputs, summarize and link/name the attached raw file.

## Tool Output Incorporated Into Report

List the exact report claims, tables, or sections that used this session.

## Confirmed Findings

| Finding | Verification | Verdict |
|---|---|---|
| ... | source URL, command, file path/line, commit, or test | Confirmed |

## Refuted / Rejected Claims

| Claim | Reason | Verdict |
|---|---|---|
| ... | why source/code disproves it | Refuted or Rejected |

## Hypotheses Left For Follow-Up

| Hypothesis | Needed Verification |
|---|---|
| ... | git blame, commit, issue, source line, test, etc. |

## Appendix A Entry

Provide the exact row that should appear in the report's Workflow Log.
```

## Verdict Labels

Use only these labels in report-facing material:

- `Confirmed`: evidence resolves and supports the claim.
- `Refuted`: a tool/AI claim was checked and found wrong.
- `Rejected`: the suggestion is generic, irrelevant, old-topic carry-over, or not useful for this system.
- `Hypothesis`: plausible but not verified.

Only `Confirmed` findings count as evidence-backed findings.

## Evidence Rules

Every confirmed claim needs at least one of:

- Official documentation URL
- Security advisory URL
- GitHub issue, PR, or commit URL
- Local source file path and line number
- Command output
- Test result
- Scanner output plus human triage

No invented CVEs, commit hashes, file paths, line numbers, issue numbers, or URLs.

If a source path or URL cannot be resolved, mark the claim `Hypothesis` or remove it.

## Appendix A Requirements

Each report should have an Appendix A Workflow Log with one row per artifact:

```markdown
| ID | Tool/model | Date | Access/input | Role/task | Raw output file | Verification | Verdict |
|---|---|---|---|---|---|---|---|
```

Example:

```markdown
| A1 | ChatGPT/Codex with web search and local shell | 2026-09-30 | Course text, official Spring docs/advisories, local clone at `747f40d...` | Pivot Domain Analysis research to Spring Security | `AI_Sessions/A1-codex-spring-security-research.md` | Official URLs, local `rg`, source paths, Git commit hash | Mixed: confirmed/rejected/hypothesis as labelled |
```

## Contribution Record

When creating or updating an AI session artifact, also update Appendix B or a contribution record with:

- Who ran the tool/session
- Who verified the findings
- Which report section used the output
- Which claims remain hypotheses

The person who runs the tool and the person who verifies it may be different.

## Recommended Workflow For Future Agents

1. Read the assignment requirements.
2. Read the current report draft.
3. Read existing `AI_Sessions/*.md`.
4. Run the needed tool/search/agent session.
5. Save the raw output or structured transcript in `AI_Sessions/`.
6. Add or update the Appendix A row in the report.
7. Mark all output as Confirmed, Refuted, Rejected, or Hypothesis.
8. Keep report claims distinguishable from AI/tool suggestions.

## Current Project Context

Current topic: Spring Security.

Current starter files:

- `Report/domain-analysis-spring-security-draft.md`
- `Research/spring-security-domain-research.md`
- `spring-security-src/`

Current vulnerability candidates:

- `CVE-2026-47842`: deterministic AES/CBC encryption in `AesBytesEncryptor`
- `CVE-2026-47841`: WebAuthn user verification bypass via session serialization
- `CVE-2026-41707`: DPoP proof replay

