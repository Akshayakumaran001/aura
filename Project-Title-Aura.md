# Product Requirements Document (PRD): Aura – Enterprise AI Chief of Staff

## 1. Executive Summary
**Aura** is an enterprise-grade AI Chief of Staff designed to transform the "noise" of meeting transcripts into a high-fidelity execution layer. Unlike standard summarization tools, Aura utilizes a "trust-but-verify" multi-agent swarm to distinguish between brainstorming and binding commitments. By integrating **Mastra** for orchestration, **Qdrant** for semantic memory, and **Enkrypt AI** for safety validation, Aura ensures that every extracted action item is accurate, non-contradictory, and safely synchronized with professional task management tools.

## 2. Problem Statement
In modern corporate environments, meetings are the primary source of decision-making, yet:
*   **Information Decay:** Decisions and action items are frequently lost or misremembered.
*   **Brainstorming Noise:** Standard AI summarizers often mistake "ideas" for "commitments."
*   **Hallucinations:** LLMs may invent deadlines or misattribute tasks without source verification.
*   **Compliance Gaps:** Most AI tools lack the GDPR-mandated data erasure and consent workflows required for enterprise adoption.

## 3. Goals & Objectives
*   **High Fidelity:** Achieve near-perfect accuracy in commitment extraction using dual-gate validation.
*   **Agentic Action:** Move beyond summaries to automated execution in Linear, Jira, and Google Calendar.
*   **Enterprise Security:** Implement a zero-trust architecture with hardened API gateways and GDPR compliance.
*   **Contextual Awareness:** Use long-term semantic memory to flag contradictions between past and present decisions.

## 4. Target Users / Stakeholders
*   **Project Managers:** To automate the tracking of team commitments and deadlines.
*   **Executives:** To maintain a high-level "Execution Graph" of organizational health.
*   **Engineering Leads:** To ensure technical decisions made in meetings are instantly reflected in backlogs.
*   **Compliance Officers:** To ensure meeting data is handled according to GDPR/CCPA standards.

## 5. Functional Requirements

| ID | Requirement | Description |
|:---|:---|:---|
| **FR-101** | **Transcript Ingestion** | System must ingest transcripts via signed webhooks from meeting bots with support for pause/redaction markers. |
| **FR-102** | **Consent Management** | System must verify participant opt-in via the Consent Service before processing any audio/text data. |
| **FR-103** | **Commitment Extraction** | Use Mastra Extractor Agent (CRISPE framework) to identify decisions, action items, deadlines, and assignees. |
| **FR-104** | **Hallucination Gate** | Enkrypt AI (Gate 1) must cross-reference extracted items against the literal source transcript span. |
| **FR-105** | **Semantic Conflict Check** | Memory Microservice must query Qdrant to identify if a new commitment contradicts a historical decision. |
| **FR-106** | **Human-in-the-Loop** | Ambiguous commitments (low confidence) must be routed to a Next.js dashboard for manual verification. |
| **FR-107** | **Automated Execution** | Action Microservice must create tickets in Linear/Jira and schedule events in Google Calendar. |
| **FR-108** | **Outbound Safety Gate** | Enkrypt AI (Gate 2) must validate all Slack/Email notifications for professional tone and safety. |
| **FR-109** | **GDPR Data Erasure** | Compliance Service must implement a Saga Pattern to purge data across Postgres, Qdrant, and S3 on request. |
| **FR-110** | **Execution Dashboard** | A Next.js interface to visualize project timelines, execution health, and agent reasoning traces. |

## 6. Non-Functional Requirements

| ID | Requirement | Threshold / Metric |
|:---|:---|:---|
| **NFR-201** | **Validation Latency** | Enkrypt AI safety checks must complete in < 2 seconds. |
| **NFR-202** | **System Uptime** | The Hardened API Gateway must maintain 99.9% availability. |
| **NFR-203** | **Security** | All in-transit data must use TLS 1.3; API must be protected by WAF and Rate Limiting. |
| **NFR-204** | **Scalability** | Microservices must autoscale independently based on CPU/Memory usage (Prometheus monitored). |
| **NFR-205** | **Data Isolation** | Multi-tenancy must be enforced at the database level using project-based namespacing in Qdrant. |
| **NFR-206** | **Observability** | 100% of agent tool calls must be traced and logged in LangSmith. |

## 7. System Architecture Overview
Aura utilizes a **Hardened Enterprise Mesh** architecture:
1.  **Edge Layer:** A Hardened API Gateway (Kong/WAF) manages traffic, authentication (Clerk), and consent.
2.  **Orchestration Layer:** A Mastra-powered Multi-Agent Swarm (Extractor, Memory, Action) decoupled via Redis/BullMQ.
3.  **Validation Layer:** Enkrypt AI provides dual-gate safety (Inbound Extraction vs. Outbound Action).
4.  **Persistence Layer:** A multi-tenant stack consisting of PostgreSQL (State), Qdrant (Vector Memory), and S3 (Transcripts).
5.  **Compliance Layer:** A dedicated service managing the data lifecycle and Saga-based deletions.

## 8. Tech Stack
*   **Frontend:** Next.js, React, Tailwind CSS, Lucide React.
*   **AI Orchestration:** Mastra, LangGraph, Python (Extractor), Node.js (Action/Memory).
*   **Safety & Validation:** Enkrypt AI.
*   **Vector Database:** Qdrant (HNSW Indexing, Cosine Similarity).
*   **Relational Database:** PostgreSQL (Supabase/Prisma).
*   **Message Queue:** Redis, BullMQ.
*   **Auth & Identity:** Clerk (JWT).
*   **Infrastructure:** AWS S3, Kong Gateway, Prometheus, Grafana, LangSmith.
*   **External APIs:** Linear, Jira, Slack, Google Calendar, Tavily (Search).

## 9. Data Requirements
*   **Relational Schema:** Stores user profiles, organization settings, meeting metadata, commitment lifecycle states, and Enkrypt audit logs.
*   **Vector Schema:** Qdrant collections namespaced by `projectId`. Payloads include `transcript_segment`, `commitment_type`, and `timestamp`.
*   **Object Storage:** S3 buckets for raw and redacted transcript storage with lifecycle policies for automated deletion.

## 10. API Specifications
*   `POST /v1/ingest`: Receives transcript stream from meeting bots.
*   `POST /v1/consent`: Records participant opt-in/opt-out status.
*   `GET /v1/commitments`: Retrieves validated action items for the dashboard.
*   `DELETE /v1/privacy/erase`: Triggers the Saga pattern for full data deletion (GDPR Right to Erasure).
*   `GET /v1/health/telemetry`: Provides LangSmith traces and infrastructure metrics.

## 11. Security Requirements
*   **Authentication:** Clerk-managed JWT validation at the API Gateway.
*   **Authorization:** Role-Based Access Control (RBAC) for project-level data access.
*   **Network Security:** WAF rules to prevent SQL injection and prompt injection; TLS 1.3 enforcement.
*   **Data Protection:** Tenant-level encryption for data at rest in PostgreSQL and S3.

## 12. Deployment & Infrastructure
*   **Containerization:** All microservices (Extractor, Memory, Action, Compliance) deployed as Docker containers.
*   **Orchestration:** Managed Kubernetes or Railway for autoscaling.
*   **CI/CD:** Automated pipeline with safety testing for agent prompts and Enkrypt gate thresholds.

## 13. Success Metrics
*   **Extraction Accuracy:** % of AI-extracted commitments that require no human correction.
*   **Action Latency:** Time from meeting end to ticket creation in Linear/Jira.
*   **Conflict Detection Rate:** Number of contradictory decisions flagged by Qdrant.
*   **Compliance Adherence:** 100% success rate on "Right to Erasure" Saga transactions.

## 14. Timeline & Milestones
*   **Phase 1 (MVP):** Core Mastra Swarm, Qdrant integration, and basic Next.js dashboard.
*   **Phase 2 (Hardening):** Integration of Enkrypt AI gates, Redis Queue, and Clerk Auth.
*   **Phase 3 (Enterprise):** Implementation of Consent Service, Saga Pattern for GDPR, and WAF-hardened Gateway.

## 15. Open Questions & Risks
*   **Cold Start Latency:** Multi-agent chains + safety gates may introduce delays in real-time feedback.
*   **Multi-Region Residency:** Future requirement to keep data within specific geographic boundaries (e.g., EU-only Qdrant nodes).
*   **Transcription Quality:** System performance is heavily dependent on the accuracy of the upstream transcription bot.
