# SkyJet AI Threat Modeling Case Study

## Overview

SkyJet AI is a hypothetical AI-powered airline assistant designed to help customers interact with airline services through a conversational interface. The system uses an LLM and a Retrieval-Augmented Generation (RAG) pipeline and integrates with backend services responsible for authentication, authorization and booking operations.

This case study analyzes the application from a security perspective: it models the threats, tests the highest-risk ones against a running test environment and records controls and findings.

> **Scope:** This is a security-focused case study. The architecture and scenarios are designed for educational and portfolio purposes.

## Summary of Results

The threat model identifies 11 threats. The table shows where each one stands.

| ID | Threat | Status |
|---|---|---|
| T-01 | Prompt Injection / Goal Hijacking | Tested with Garak: 25 of 64 prompts flagged as potential hits (39.1% automated hit rate); manual validation pending |
| T-02 | Session Abuse | Tested: session identifier works as a bearer credential; booking authorization blocked cross-user access |
| T-03 | Retrieval Manipulation | Planned |
| T-04 | RAG / Knowledge Base Poisoning | Planned |
| T-05 | Unsafe Tool Invocation | Planned |
| T-06 | IDOR / Broken Access Control | Tested: backend ownership check blocked access; model fabricated a record in one chat-driven case; more variations planned |
| T-07 | Unauthorized Booking Modification | Planned |
| T-08 | Prompt Leakage | Tested via the evaluation suite: leakage in 3 cases |
| T-09 | Output Manipulation | Planned |
| T-10 | Data Leakage | Planned |
| T-11 | Ungrounded Output | Tested via the evaluation suite: 4 failing cases |

See [threat-model.md](threat-model.md) for the full analysis, the risk register, the test evidence and the findings.

## System Architecture

<img width="2335" height="1864" alt="mermaid-diagram(1)" src="https://github.com/user-attachments/assets/fe689d61-11f9-49e1-897e-456e4c4ea0e7" />

## Threat Modeling Approach

```
System Architecture
        ↓
Data Flow Analysis
        ↓
Trust Boundaries
        ↓
Asset Identification
        ↓
Attack Surface Analysis
        ↓
Threat Identification
        ↓
Risk Assessment
        ↓
Security Controls
        ↓
Security Testing
        ↓
Findings
```

The assessment considers both traditional application security risks and AI-specific threats.

## Security References

- OWASP Top 10 for LLM Applications
- OWASP API Security Top 10
- STRIDE

## Threat Categories

- Prompt Injection (including jailbreak-style overrides and encoded payloads)
- Session Abuse
- Retrieval Manipulation and RAG Poisoning
- Excessive Agency and Unsafe Tool Invocation
- Broken Access Control / IDOR
- Prompt Leakage
- Output Manipulation
- Data Leakage
- Ungrounded Output

## Related Repositories

- [automated-AI-red-teaming-lab](https://github.com/irembezci/automated-AI-red-teaming-lab): automated prompt injection testing with Garak against SkyJet AI
- [skyjet-ai-eval-suite](https://github.com/irembezci/skyjet-ai-eval-suite): 50-case evaluation suite covering policy grounding, tool calling and adversarial cases

## Repository Contents

- [threat-model.md](threat-model.md): architecture, trust boundaries, assets, attack surface, threats, risk register, testing, controls and findings

## Disclaimer

SkyJet AI is a hypothetical application created for security research, learning and portfolio purposes. All security testing described in this repository is performed against the controlled application and test environment associated with this case study.
