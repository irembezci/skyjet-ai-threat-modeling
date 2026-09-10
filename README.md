# SkyJet AI Threat Modeling Case Study

## Overview

SkyJet AI is a hypothetical AI-powered airline assistant designed to help customers interact with airline services through a conversational interface. The system uses an LLM and a Retrieval-Augmented Generation (RAG) pipeline and integrates with backend services responsible for authentication, authorization and booking operations. This case study demonstrates how an AI-powered product can be analyzed from a security perspective during the design and development lifecycle.

> **Scope:** This is a security-focused case study. The architecture and scenarios are designed for educational and portfolio purposes.


## Objectives

The goal of this case study is to:

- Analyze the security architecture of an AI-powered application
- Identify AI-specific and application-level attack surfaces
- Perform threat modeling using the system architecture and data flows
- Assess potential security risks
- Define appropriate security controls and mitigations
- Connect identified threats with practical security testing

## System Architecture

```mermaid
flowchart TD
    User[User]

    subgraph Application["SkyJet AI Application"]
        FastAPI[FastAPI Application]
        Auth[Authentication Service]
        Authorization[Authorization Service]
        Chatbot[Chatbot Service]
        LLM[LLM Service]
        RAG[RAG Service]
        Booking[Booking Service]
    end

    subgraph Data["Application Data"]
        Users[(Users)]
        Sessions[(Sessions)]
        Bookings[(Bookings)]
    end

    subgraph Knowledge["Knowledge Base"]
        Policies[(Airline Policies)]
    end

    User -->|Prompt / Request| FastAPI
    FastAPI --> Auth
    FastAPI --> Authorization

    Auth --> Chatbot
    Authorization --> Chatbot

    Chatbot --> LLM
    Chatbot --> RAG
    Chatbot --> Booking

    RAG --> Policies

    Booking --> Users
    Booking --> Sessions
    Booking --> Bookings
````

## Threat Modeling Approach

The threat modeling process follows the following workflow:

```text
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
```

The assessment considers both traditional application security risks and AI-specific threats.

### Security References

* OWASP Top 10 for LLM Applications
* OWASP GenAI Security Project
* MITRE ATLAS
* STRIDE
* NIST AI Risk Management Framework

## Threat Categories

The assessment focuses on threats including:

* Prompt Injection
* Jailbreaking
* Data Leakage
* Broken Access Control / IDOR
* RAG Poisoning
* Retrieval Manipulation
* Excessive Agency
* Encoding-based Evasion
* Sensitive Information Exposure
* Model-related Privacy Risks

The final threat list will be refined based on the actual system architecture and identified attack surfaces.


## Case Study Deliverables

This case study will document:

1. System Architecture
2. Data Flow Diagram
3. Trust Boundaries
4. Assets
5. Attack Surface
6. Threat Model
7. Risk Assessment
8. Security Controls
9. Security Testing
10. Findings and Recommendations


## Security Testing

Identified threats will be mapped to practical security tests where applicable. The objective is not only to identify theoretical risks but to validate whether the identified security controls are effective against realistic attack scenarios.

## Disclaimer

SkyJet AI is a hypothetical application created for security research, learning and portfolio purposes. All security testing described in this repository is performed against the controlled application and test environment associated with this case study.
