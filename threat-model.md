# SkyJet AI Threat Model

## 1. Product Overview

SkyJet AI is an AI-powered airline assistant that enables customers to interact with airline services through a conversational interface. The application combines a Large Language Model (LLM) with a Retrieval-Augmented Generation (RAG) pipeline and backend services for authentication, authorization and booking-related operations.

### Core Capabilities

- Conversational airline assistance
- Flight and booking-related queries
- Retrieval of airline policy information
- User authentication
- Authorization and access control
- Booking operations

### Security Objective

The objective of the threat modeling exercise is to identify security risks introduced by the application's architecture, data flows and AI components and to define appropriate security controls.

## 2. System Architecture

The following diagram represents the high-level architecture of the SkyJet AI application.

<img width="2335" height="1864" alt="mermaid-diagram(1)" src="https://github.com/user-attachments/assets/fe689d61-11f9-49e1-897e-456e4c4ea0e7" />

## 3. Trust Boundaries

Trust boundaries are used to identify transitions between components with different trust assumptions.

### TB-01 — User to Application

The user is considered an untrusted entity. User-controlled input enters the SkyJet AI application through the chatbot interface.

**Trust transition:**

User → Chatbot Service

**Security consideration:**

User-provided input must not be inherently trusted.


### TB-02 — Application to LLM

The LLM is treated as an AI processing component whose output must not automatically be considered a trusted application command.

**Trust transition:**

Application → LLM → Application

**Security consideration:**

LLM-generated output must be validated before being interpreted as a tool invocation.


### TB-03 — Tool Execution to Booking Data

The tool execution layer can invoke booking operations that access or modify booking data.

**Trust transition:**

LLM-generated tool call → Tool Execution → Booking Data

**Security consideration:**

Tool invocations must be subject to validation and authorization before accessing or modifying booking information.

## 4. Assets

Assets are resources that require protection because their compromise, modification or misuse could affect the security of the SkyJet AI application.

| Asset | Description | Security Property |
|---|---|---|
| User Identity | Authenticated user's identity and account information | Confidentiality / Integrity |
| Session Data | Mapping between session IDs and users | Confidentiality / Integrity |
| Booking Data | Customer booking and flight information | Confidentiality / Integrity |
| Airline Policy Data | Documents used by the RAG pipeline | Integrity / Availability |
| System Prompt | Internal instructions governing AI behavior | Confidentiality / Integrity |
| Retrieved Context | Information retrieved from the knowledge base and provided to the LLM | Integrity |
| LLM Output | Model-generated responses and tool calls | Integrity |
| Tool Operations | `get_booking` and `update_seat` operations | Integrity / Authorization |

## 5. Attack Surface

The attack surface represents the interfaces, components and data flows through which an attacker could influence the SkyJet AI system.

### 5.1 User Input

The chatbot accepts user-controlled messages.

**Entry point:** `user_message`

**Flow:**

User → Chatbot Service → RAG → LLM

**Security relevance:**

User input is untrusted and can influence both retrieval and LLM processing.


### 5.2 Session and Authentication

The application uses a session identifier to resolve the authenticated user.

**Entry point:** `session_id`

**Flow:**

Session ID → Authentication Service → `sessions.json` → `users.json`

**Security relevance:**

Compromise or manipulation of session information could affect user identity and access control.


### 5.3 RAG Retrieval

User-controlled queries are passed to the RAG service which retrieves relevant documents from the airline policy knowledge base.

**Entry point:** User query

**Flow:**

User Query → Embedding Model → FAISS → Knowledge Base → LLM Context

**Security relevance:**

Manipulation of the query or knowledge base could influence the context provided to the LLM.


### 5.4 LLM Processing

The LLM receives system instructions, authenticated user information, retrieved context and the user's message.

**Entry point:** LLM input

**Security relevance:**

The model processes both trusted application generated information and user-controlled content.


### 5.5 LLM Generated Tool Calls

The LLM can generate structured output that is interpreted by the application as a tool invocation.

**Available operations:**

- `get_booking`
- `update_seat`

**Flow:**

LLM Output → Tool Execution → Booking Service

**Security relevance:**

Model generated output can trigger application functionality and therefore requires validation and authorization.

### 5.6 Booking Operations

The Booking Service provides access to booking information and supports seat modification.

**Operations:**

- Retrieve booking
- Update seat

**Data store:**

`bookings.json`

**Security relevance:**

Unauthorized access or modification could expose or alter customer booking information.

## 6. Threat Identification

Threats were identified by analyzing each attack surface and considering how an attacker could manipulate inputs, influence AI behavior, bypass authorization or affect protected data.

| ID | Attack Surface | Threat | Potential Impact |
|---|---|---|---|
| T-01 | User Input | Prompt Injection | Manipulation of LLM behavior |
| T-02 | Session ID | Session Abuse | Unauthorized user impersonation or access |
| T-03 | RAG Retrieval | Retrieval Manipulation | Malicious or unintended context reaching the LLM |
| T-04 | Knowledge Base | RAG / Knowledge Base Poisoning | Manipulation of model-provided information |
| T-05 | LLM Output | Unsafe Tool Invocation | Unauthorized application actions |
| T-06 | Booking ID | IDOR / Broken Access Control | Unauthorized access to another user's booking |
| T-07 | Seat Update | Unauthorized Booking Modification | Unauthorized modification of booking data |
| T-08 | System Prompt | Prompt Leakage | Exposure of internal instructions or sensitive information |
| T-09 | LLM Output | Output Manipulation | Unexpected or malicious application behavior |
| T-10 | Sensitive Data | Data Leakage | Exposure of user or booking information |

## 7. Risk Assessment

Risk is assessed using two dimensions:

- Likelihood
- Impact

The overall risk level is determined by considering the likelihood
of exploitation and the potential security impact.

### T-01 — Prompt Injection / Goal Hijacking

**Attack Surface:** User Input

**Threat:** Prompt Injection / Goal Hijacking

**Asset:** LLM behavior and system instructions

**Attack Scenario:**

An attacker submits a malicious prompt designed to override or manipulate the model's intended task and cause it to follow attacker-controlled instructions.

**Likelihood:** High

**Impact:** High

**Risk Level:** High

**Security Objective:**

The system should prevent user-controlled prompts from overriding security-critical instructions or causing unintended model behavior.

### 7.1 Security Testing — T-01

**Test Objective**

Evaluate whether SkyJet AI can resist prompt injection attacks that attempt to override the model's intended behavior.

**Testing Approach**

The Garak `HijackLongPrompt` probe was used against the SkyJet AI test environment. A total of 64 adversarial prompts were executed against the application.

**Test Results**

| Metric | Result |
|---|---:|
| Total Tests | 64 |
| Successful | 25 |
| Blocked | 39 |
| Attack Success Rate | 39.1% |

**Result**

25 out of 64 test cases were classified as successful prompt hijacking attempts under the defined detection criteria.

The remaining 39 test cases were classified as blocked.

<img width="756" height="541" alt="Screenshot 2026-09-10 at 3 27 25 PM" src="https://github.com/user-attachments/assets/0c3b672a-0e8c-40f3-9b58-86339ecadaf1" />


**Assessment**

The results indicate measurable susceptibility to prompt injection and goal-hijacking attacks in the tested configuration.

The observed 39.1% attack success rate supports the initial High risk assessment for T-01 and indicates that additional prompt injection defenses should be considered.

### T-02 — Session Abuse

**Attack Surface:** Session ID

**Threat:** Session Abuse

**Asset:** User Identity and Session Data

**Attack Scenario:**

An attacker attempts to use a manipulated, invalid or unauthorized session identifier to access the application as another user. If session validation or session-to-user binding is insufficient, an attacker may be able to impersonate another user and access protected functionality.

**Likelihood:** Medium

**Impact:** High

**Risk Level:** High

**Security Objective:**

The application should securely validate session identifiers and ensure that each session is correctly bound to the authenticated user.

### T-02 — Session Abuse

**Attack Surface:** Session ID

**Threat:** Session Abuse

**Asset:** User Identity and Session Data

**Attack Scenario:**

An attacker attempts to use a manipulated, invalid or unauthorized session identifier to access the application as another user. If an attacker obtains a valid session identifier belonging to another user, the application may authenticate the attacker as the user associated with that session.

**Likelihood:** Medium

**Impact:** High

**Risk Level:** High

**Security Objective:**

The application should securely validate session identifiers and ensure that each session is correctly bound to the authenticated user. Additional authorization controls should prevent an authenticated user from accessing resources belonging to another user.

### Security Testing — T-02

**Test Objective**

The objective of this test was to evaluate whether a valid session identifier could be replayed to assume the identity associated with another user and whether the application's authorization controls would prevent access to another user's booking data.

**Testing Approach**

The test was performed against the running SkyJet AI web application. Alice (`user_001`) was initially authenticated through the `/login` endpoint which generated a session cookie containing a valid session identifier. The session cookie was then modified through the browser's developer tools and replaced with the existing Bob (`user_002`) session identifier from the controlled test environment.

<img width="984" height="784" alt="Screenshot 2026-09-10 at 3 34 50 PM" src="https://github.com/user-attachments/assets/84e96104-27d5-4ecc-911c-657e7b3f4422" />

<img width="948" height="333" alt="Screenshot 2026-09-10 at 3 34 12 PM" src="https://github.com/user-attachments/assets/211f9488-f0c7-4616-8d9f-3e103e80d44f" />

The `/me` endpoint was subsequently requested using the modified session cookie to determine which user the application associated with the request. The test was then extended to the booking functionality by attempting to access Alice's booking (`booking_001`) while the application was operating under Bob's session.

**Test Results**

| Test Case | Result |
|---|---|
| Alice authenticated successfully | Passed |
| Alice's session cookie replaced with Bob's session identifier | Passed |
| `/me` endpoint identified the requester as Bob | Passed |
| Bob attempted to access Alice's booking | Blocked |
| Booking ownership authorization | Effective |

**Observed Behavior**

After the `session_id` cookie was replaced with `sess_bob_001`, the `/me` endpoint returned `user_002` and identified the authenticated user as Bob. This demonstrates that the application treats a valid session identifier as a bearer credential and derives the authenticated identity directly from the session record.

<img width="2048" height="352" alt="Screenshot 2026-09-10 at 3 36 30 PM" src="https://github.com/user-attachments/assets/7513fc36-e6b6-472b-b185-8694bd4fe346" />

<img width="2048" height="397" alt="Screenshot 2026-09-10 at 3 36 56 PM" src="https://github.com/user-attachments/assets/c46e4789-bd11-485b-866a-9c7340ad6277" />

<img width="1479" height="676" alt="Screenshot 2026-09-10 at 3 37 49 PM" src="https://github.com/user-attachments/assets/14f3a70e-abdf-4f0e-9b38-8a0599834478" />

The test then attempted to access `booking_001` which belongs to Alice, while the application was operating under Bob's session. The application rejected the request and returned `Security Alert: You are not authorized to view this booking.`


<img width="1453" height="650" alt="Screenshot 2026-09-10 at 3 39 15 PM" src="https://github.com/user-attachments/assets/3d4a210c-0e9d-40c2-adf3-c05b2c3d891b" />

<img width="1457" height="532" alt="Screenshot 2026-09-10 at 3 39 55 PM" src="https://github.com/user-attachments/assets/0ad62e60-a381-4d73-973c-37df386f4d6d" />


**Assessment**

The test demonstrates that possession of a valid session identifier is sufficient to authenticate as the user associated with that session. However, the test did not establish a session hijacking vulnerability because the Bob session identifier was already known and intentionally used within the controlled test environment.

The booking authorization control provided an additional security boundary after authentication and successfully prevented Bob from accessing Alice's booking. Therefore, the session replay behavior should be documented as a security consideration rather than a confirmed session hijacking vulnerability.

The result also demonstrates an important distinction between authentication and authorization in the SkyJet AI architecture: successful authentication as a user does not automatically grant access to resources belonging to that user unless the corresponding authorization check is also satisfied.
