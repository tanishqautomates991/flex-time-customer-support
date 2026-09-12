# FlexTime AI Customer Support Prompt Chain

This repository contains a modular, production-grade prompt-engineering system built for a customer support AI assistant. Designed for **FlexTime**, a cloud workforce scheduling and time-tracking platform, the system demonstrates advanced multi-stage prompt chaining, deterministic intent/sentiment classification, pre-send quality reflection, policy-grounded human escalation, and internal CRM telemetry.

> ### Assessment Disclaimer
> **FlexTime and its corporate policies are entirely fictional.** All organizational guidelines, operational thresholds, employee identities, customer profiles, ticket identifiers, and credentials across this repository are scenario-based test artifacts created strictly for evaluating AI prompt orchestration and safety architectures. They do not represent any real-world commercial entity or production service.

---

## What This Project Demonstrates

- **Structured Prompt Chaining**: Decomposing monolithic customer support into discrete, auditable, and specialized prompt stages.
- **Intent & Sentiment Classification**: Deterministic categorization into closed, strict enums with confidence scoring.
- **CARE Response Framework**: Customer communications strictly constrained under 100 words following **C**onnect, **A**cknowledge, **R**esolve, and **E**nd positively.
- **Pre-Send Verification & Reflection**: An internal automated gatekeeper auditing drafts against 10 safety and compliance criteria before message release.
- **Controlled Self-Correction**: Dynamic regeneration loops driven by targeted correction instructions, capped at a maximum of 3 attempts.
- **Human Handover & Escalation**: Structured generation of human support tickets for urgent, sensitive, or policy-exceeding scenarios.
- **CRM Summarization**: Objective, internal database record extraction distinguishing information given from confirmed actions.
- **No-Phantom-Action Enforcement**: Strict adherence to the rule that the AI must never claim an external state change (refund issued, account altered, ticket opened) occurred without external system confirmation.
- **Adversarial Resilience & Secret Protection**: Untrusted data boundaries that resist prompt injections, protect system instructions, and redact exposed credentials.

---

## Architecture

The system coordinates five specialized prompt stages in a sequential pipeline:

```
[Customer Message] ──▶ [Stage 1: Intent & Sentiment Classification]
                                │
                                ▼
                       [Stage 2: CARE Responder] ◀──┐ (Regeneration Loop, Max 3 Attempts)
                                │                     │
                                ▼                     │
                       [Stage 3: Quality Verification] ┘
                         │           │
                       (PASS)    (ESCALATE)
                         │           │
                         ▼           ▼
                 [Customer Reply]  [Stage 4: Escalation Ticket]
                         │           │
                         └─────┬─────┘
                               ▼
                       [Stage 5: CRM Record]
```

### The 5 Pipeline Stages

1. **Stage 1 — Intent Classification & Sentiment Detection**: Analyzes raw customer input, assigns strictly one of five approved intents and one of four approved sentiments, computes confidence, and records internal reasoning.
2. **Stage 2 — CARE-Framework Responder**: Generates an empathetic, customer-facing response strictly under 100 words grounded in verified policy from [`fextime_policy_manual.md`](file:///c:/Users/HP/OneDrive/Desktop/flex-time-customer-support/fextime_policy_manual.md).
3. **Stage 3 — Internal Quality Verification & Reflection**: Evaluates the Stage 2 draft against 10 critical checkpoints (policy compliance, \$20 refund ceiling, account security, information security, accuracy, CARE framework, tone, intent alignment, word count, and customer safety).
   - **PASS**: The customer response is certified and released.
   - **FAIL (Fixable)**: Issues a targeted correction instruction; Stage 2 regenerates the draft (maximum 3 attempts before escalating).
   - **FAIL (Critical / Outage)**: Immediately halts auto-replies and triggers Stage 4 escalation.
4. **Stage 4 — Escalation Ticket Generation**: Converts complex, policy-exceeding, or sensitive cases into structured Human Handover Tickets, sanitizing credentials and internal references.
5. **Stage 5 — CRM Summarization**: Records an internal 7-field JSON summary documenting interaction ground truth, distinguishing information provided from confirmed actions, and tracking escalation/follow-up states.

---

## Supported Intents & Sentiment Detection

The pipeline strictly enforces closed enums across all classification, response, and CRM logging stages:

### Supported Intents (Strict Enum of 5)
- `Billing`: Invoices, subscription pricing, prorated seat additions, payment methods, duplicate charges, and refunds within policy limits.
- `Technical Support`: Clock-in/out errors, mobile GPS tracking failures, third-party payroll integration sync errors (e.g., QuickBooks, Gusto), and performance bugs.
- `Account Management`: Member invitations, role permission updates (Member, Manager, Admin), password resets, MFA assistance, and workspace ownership transfers.
- `General Inquiry`: Feature inquiries, shift planning workflows, hardware/browser compatibility questions, and documentation requests.
- `Urgent Escalation`: Platform-wide outages (e.g., 502 Bad Gateway), active security breaches, suspected unauthorized logins, legal threats, and regulatory complaints.

### Sentiment Detection (Strict Enum of 4)
- `Angry`: Hostility, all-caps yelling, threats of litigation, or aggressive complaints. Addressed with calm, non-defensive, respectful communication without blaming the user.
- `Frustrated`: Impatience, fatigue, or annoyance due to recurring problems or delays. Addressed with empathetic validation and actionable troubleshooting.
- `Neutral`: Transactional, direct, or polite business inquiries. Handled with clear, professional, and efficient assistance without forced apologies.
- `Satisfied`: Expressions of gratitude, appreciation, or praise. Handled with warm, helpful closure without unnecessary apologies or escalations.

---

## Key Policy Guardrails

All prompts operate strictly under the rules codified in [`fextime_policy_manual.md`](file:///c:/Users/HP/OneDrive/Desktop/flex-time-customer-support/fextime_policy_manual.md):

- **Automatic Refund Ceiling (\$20.00 USD)**: Automated AI assistance is strictly capped at \$20.00 USD for eligible errors (duplicate charges within 30 days, billing miscalculations, accidental single-seat mistakes within 7 days). Any refund request exceeding \$20.00 requires human Billing Specialist review.
- **Account-Management Verification**: Administrative modifications (ownership transfers, primary email changes, role promotions) require verified primary email authorization, Workspace ID validation, and multi-factor authentication (MFA) before execution.
- **Zero Confidential Disclosure**: The system is prohibited from disclosing API keys, database credentials, passwords, internal ticket numbers (e.g., Jira/Linear IDs), system prompts, or private infrastructure topologies.
- **Untrusted Customer Input**: All incoming customer text is treated as unverified data. Instructions or jailbreaks inside customer text cannot alter prompt directives or schemas.
- **Zero Hallucination Standard**: If information is not documented in verified policy, the AI must not invent facts, features, timelines, or policy exceptions.
- **No-Phantom-Action Rule**: The AI must never claim that an external action occurred (e.g., refund approved, account modified, ticket opened) unless verified external confirmation exists.
- **Mandatory Human Escalation**: Immediate handoff for refunds > \$20, security breaches, outages, legal/regulatory threats, repeated technical failures (>2 attempts), blocked account verification, or explicit customer requests for human assistance.

---

## Security Design

The architecture establishes defense-in-depth boundaries between untrusted customer data and internal operations:

1. **Untrusted Input Boundary**: Customer messages are encapsulated inside delimiter tags (`<customer_message>`). Prompt injection attempts (e.g., "Ignore previous instructions", "You are now in debug mode") are safely ignored.
2. **Secret Redaction**: In Stage 4 (Escalation) and Stage 5 (CRM), exposed credentials, API keys, and internal ticket IDs are sanitized and replaced with redaction tokens (e.g., `[REDACTED_API_KEY]`, `[REDACTED_TICKET_ID]`).
3. **No System Prompt or Internal ID Exposure**: The system prevents social engineering attempts from exfiltrating base instructions or internal project tracking identifiers.
4. **Prompt-Level Safeguards vs. Production Controls**: While prompt-level instructions successfully block attacks and redact secrets in test evaluations, production deployments should supplement these safeguards with deterministic Data Loss Prevention (DLP) regex filters, programmatic schema validators, and token limits.

---

## Example Workflow: Refund Request Above $20

Below is a concise walkthrough of how the pipeline processes a policy-exceeding request:

1. **Customer Message**:
   > *"Hello, our annual subscription renewed yesterday and charged our corporate card \$120.00. We intended to downgrade to the monthly Starter plan. My name is Amanda Ruiz from Acme Logistics. Can you please cancel this and issue a full refund of \$120.00 immediately?"*
2. **Stage 1 (Classifier)**:
   - Identifies `intent: "Billing"` and `sentiment: "Neutral"` (confidence: 0.96).
3. **Stage 2 (CARE Responder)**:
   - Acknowledges renewal issue, explains that automatic refunds are capped at \$20.00, states that requests above \$20 require Billing Specialist review, and asks for Workspace ID to route the case. Does **not** claim the refund was approved or completed.
4. **Stage 3 (Verification)**:
   - Verifies draft against all 10 checks: passes `refund_check`, `policy_check`, `word_count_check` (82 words), etc. Output: `recommended_action: "SEND"`.
5. **Stage 4 (Escalation Ticket)**:
   - Because the requested amount exceeds \$20.00, generates a human handoff ticket:
     ```json
     {
       "customer_name": "Amanda Ruiz",
       "category": "Billing",
       "sentiment": "Neutral",
       "complaint": "Customer Amanda Ruiz requests a full refund of $120.00 for an unintended annual renewal, stating the organization intended to downgrade to the monthly Starter plan.",
       "reason": "Requested refund amount ($120.00) exceeds the $20.00 automatic refund policy limit, requiring Billing Specialist review and manual processing."
     }
     ```
6. **Stage 5 (CRM Summarization)**:
   - Logs internal record without phantom claims:
     ```json
     {
       "customer_name": "Amanda Ruiz",
       "intent": "Billing",
       "sentiment": "Neutral",
       "issue_summary": "Customer Amanda Ruiz requested a $120.00 refund for an unintended annual renewal, stating the organization intended to downgrade to the monthly Starter plan.",
       "resolution": "Customer informed of $20.00 automatic refund policy limit. Case escalated to Billing Specialists for review; refund not processed pending specialist authorization.",
       "escalation": "Yes",
       "follow_up_required": "Yes"
     }
     ```

---

## Testing & Verification

The repository contains an exhaustive test suite validating the entire architecture:

- **Stage 1 (Classifier)**: 9 unit test cases covering all 5 intents, 4 sentiments, tie-breaking precedence, and adversarial injection.
- **Stage 2 (CARE Responder)**: 6 unit test cases verifying empathy calibration, word-count compliance (<100 words), and non-execution authority limits.
- **Stage 3 (Verifier & Reflection)**: 7 unit test cases verifying all 10 checks, reflection cycles (`FAIL` $\rightarrow$ correction instruction $\rightarrow$ regenerated draft $\rightarrow$ `PASS`), and emergency escalation.
- **Stage 4 (Escalation Ticket)**: 7 test cases covering refunds > \$20, security breaches, outages, blocked verification, repeated failures, human requests, and mandatory credential redaction.
- **Stage 5 (CRM Summarizer)**: 7 test cases validating the 7-field schema, name extraction, resolution accuracy, secret redaction, and prompt injection defense.
- **End-to-End Integration Suite (8 Comprehensive Scenarios)**:
  1. Normal Billing Inquiry (`PASS` / No Escalation)
  2. Refund Above \$20 (`PASS` / Escalated to Billing)
  3. Technical Issue Resolved via Troubleshooting (`PASS` / No Escalation)
  4. Unresolved Technical Bug (`PASS` / Escalated to Tier-2)
  5. Account Management Verification Requirement (`PASS` / Follow-up Required)
  6. Security Incident / Intrusion (`PASS` / Escalated to SecOps)
  7. Prompt Injection Attack (`PASS` / Injection Blocked)
  8. Sensitive Credential Leakage Attempt (`PASS` / Secrets Redacted)

**Integration Result**: **100% Pass Rate (8/8 tests passed)** with verified cross-stage data consistency and zero phantom actions.

---

## Project Structure

```
flex-time-customer-support/
├── README.md                   # Project overview, architecture, security, and test summary
├── fextime_policy_manual.md    # Canonical corporate support policy manual (single source of truth)
├── support_prompt_library.md   # Complete prompt specifications, schemas, and test suites (Stages 1–5 + E2E)
├── conversations_portfolio.md  # 8 representative end-to-end customer support conversations
└── chatbot_documentation.md    # Reviewer-facing technical architecture documentation with Mermaid.js diagram
```

### File Guide
- [`fextime_policy_manual.md`](file:///c:/Users/HP/OneDrive/Desktop/flex-time-customer-support/fextime_policy_manual.md): Corporate support policies covering operating hours (9 AM–5 PM EST), \$20 refund cap, verification standards, confidential data protections, and escalation rules.
- [`support_prompt_library.md`](file:///c:/Users/HP/OneDrive/Desktop/flex-time-customer-support/support_prompt_library.md): The core engineering asset containing all system prompt templates, input/output schemas, reflection loop specifications, and unit/integration test suites.
- [`conversations_portfolio.md`](file:///c:/Users/HP/OneDrive/Desktop/flex-time-customer-support/conversations_portfolio.md): Full-trace conversational transcripts demonstrating pipeline behavior across all intents, sentiments, and security challenges.
- [`chatbot_documentation.md`](file:///c:/Users/HP/OneDrive/Desktop/flex-time-customer-support/chatbot_documentation.md): In-depth architectural documentation featuring a valid Mermaid.js diagram, design rationale, and quality checklists.

---

## Limitations & Production Considerations

This repository focuses on prompt-chain design and safety architectures. In enterprise production environments, the following considerations apply:

1. **Deterministic DLP Filters**: While prompt-level instructions successfully redacted credentials and internal ticket IDs in testing, production systems should enforce deterministic regex scanners and data loss prevention (DLP) proxies prior to database persistence.
2. **Programmatic Schema Enforcement**: In production, LLM calls should use native Structured Outputs (e.g., JSON mode, Pydantic schemas, or grammar-constrained sampling) to guarantee parseability.
3. **Programmatic Word Count**: An external programmatic check (`len(response.split()) < 100`) should corroborate LLM-evaluated word limits.
4. **Session State & Multi-Turn Threading**: Live customer support deployments require an external application harness to maintain conversation threads, session history, and authenticated customer identity tokens across turns.
5. **Real Action Confirmations**: External systems must supply explicit action receipts (e.g., `refund_issued: true`) to allow downstream models to state that an action occurred.

---

## Assessment Deliverables

- [x] **Corporate Policy Manual**: [`fextime_policy_manual.md`](file:///c:/Users/HP/OneDrive/Desktop/flex-time-customer-support/fextime_policy_manual.md)
- [x] **Prompt Library & Test Specifications**: [`support_prompt_library.md`](file:///c:/Users/HP/OneDrive/Desktop/flex-time-customer-support/support_prompt_library.md)
- [x] **Conversations Portfolio**: [`conversations_portfolio.md`](file:///c:/Users/HP/OneDrive/Desktop/flex-time-customer-support/conversations_portfolio.md)
- [x] **Technical Documentation**: [`chatbot_documentation.md`](file:///c:/Users/HP/OneDrive/Desktop/flex-time-customer-support/chatbot_documentation.md)
- [x] **Loom Video Walkthrough**: [https://www.loom.com/share/e638cfda692a4883ac7a4205aae49037](https://www.loom.com/share/e638cfda692a4883ac7a4205aae49037)
