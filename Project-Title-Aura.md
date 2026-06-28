# Project Title: **Aura**
## Project Description
**Aura** is an AI-powered Chief of Staff designed to bridge the gap between meeting discussions and actual execution. Unlike standard transcription tools that merely summarize, Aura uses a **Multi-Agent Swarm** orchestrated by **Mastra** to extract structured commitments, validate them against hallucinations using **Enkrypt AI**, and autonomously manage the lifecycle of tasks. It features a "trust-but-verify" architecture where every AI-generated action is cross-referenced with the source transcript and checked for semantic contradictions against historical project data stored in **Qdrant**. By integrating directly with Linear, Jira, and Google Calendar, Aura transforms "brainstorming noise" into a verified, conflict-free execution plan.

---

# Product Requirements Document (PRD): Aura

## 1. Executive Summary
Aura is an autonomous execution system that transforms meeting recordings into actionable, verified workstreams. By leveraging a multi-agent orchestration layer, vector-based long-term memory, and dual-gate safety validation, Aura ensures that no commitment is lost, no task is hallucinated, and every follow-up is executed with professional precision.

## 2. Problem Statement
In modern organizations, meetings are the primary source of decisions, yet the transition from "talk" to "task" is manual, error-prone, and lacks accountability. 
*   **Information Decay:** Action items are forgotten or misremembered.
*   **Hallucination Risk:** AI summarizers often "invent" deadlines or assignees.
*   **Context Fragmentation:** New decisions often contradict prior commitments without anyone noticing.
*   **Manual Overhead:** Following up on tasks consumes significant leadership bandwidth.

## 3. Goals & Objectives
*   **Automate Extraction:** Achieve high-fidelity extraction of decisions, action items, and dependencies.
*   **Zero Hallucination:** Use dual-gate validation to ensure 100% alignment between transcripts and tasks.
*   **Semantic Consistency:** Identify and flag contradictions between new decisions and historical data.
*   **Autonomous Execution:** Reduce manual follow-up time by 80% through automated nudges and tool integrations (Linear/Jira).

## 4. Target Users / Stakeholders
*   **Project Managers:** To track execution health and dependency graphs.
*   **Executives/Chiefs of Staff:** To ensure organizational alignment without attending every meeting.
*   **Engineering Leads:** To sync meeting outcomes directly into sprint backlogs.
*   **Individual Contributors:** To receive clear, verified tasks and automated reminders.

## 5. Functional Requirements

### 5.1 Capture & Redaction
*   **Transcript Ingestion:** Support for a webhook-based ingestion of transcripts (MVP) and future live bot integration.
*   **Pause/Resume Control:** Allow participants to pause streaming during sensitive discussions.
*   **Post-Meeting Redaction:** A UI for organizers to strike out segments before downstream processing.
*   **Explicit Release:** No data processing occurs until the organizer clicks "Release."

### 5.2 Mastra Multi-Agent Swarm
*   **Extractor Agent:** Parses transcripts into structured JSON (decisions, tasks, deadlines, assignees).
*   **Memory Agent:** Queries Qdrant for historical context and flags semantic contradictions.
*   **Action Agent:** Interacts with external APIs (Linear, Jira, Calendar) to execute tasks.
*   **Human-in-the-Loop:** Low-confidence extractions must be routed to a manual confirmation queue.

### 5.3 Validation & Safety (Enkrypt AI)
*   **Gate 1 (Extraction):** Validates extracted commitments against the literal source transcript span to prevent hallucinations.
*   **Gate 2 (Outbound):** Scans autonomously drafted follow-up messages for toxicity, safety, and professional tone.

### 5.4 Memory & Search (Qdrant)
*   **Semantic Search:** Retrieve related past commitments within the same project.
*   **Conflict Detection:** Flag if a new decision (e.g., "Launch Friday") conflicts with a stored one ("Launch Monday").
*   **Multi-tenancy:** Namespace all vector data by `projectId`.

### 5.5 Dashboard & UI
*   **Execution Health View:** Global dashboard showing project status and blockers.
*   **Dependency Graph:** Visual representation of how tasks and decisions link across meetings.
*   **Human-Review Queue:** Interface for resolving low-confidence AI extractions.

## 6. Non-Functional Requirements
*   **Reliability:** Use Redis/BullMQ to handle long-running LLM workflows and retries.
*   **Observability:** Full tracing of agent reasoning and validation gates via LangSmith.
*   **Scalability:** Multi-tenant architecture supporting isolated project environments.
*   **Performance:** Real-time UI updates via WebSockets for workflow status.

## 7. System Architecture Overview
The system is divided into five functional layers:
1.  **Ingestion Layer:** Webhook/Bot capturing raw audio/text.
2.  **Orchestration Layer:** Mastra Swarm managing agent logic and background jobs (Redis).
3.  **Data Layer:** PostgreSQL (Relational State), Qdrant (Vector Memory), and S3 (Object Storage).
4.  **External Services:** Enkrypt AI (Safety), Linear/Jira (Tasks), Google Calendar (Scheduling).
5.  **Client Layer:** Next.js Dashboard with Clerk Authentication.

## 8. Tech Stack
*   **Frontend:** Next.js, React, Tailwind CSS, Lucide React, Clerk (Auth).
*   **Backend/Orchestration:** Node.js, TypeScript, Mastra Swarm, LangGraph, Zod.
*   **Databases:** PostgreSQL (Supabase/Neon), Qdrant (Vector DB), Redis (BullMQ).
*   **AI/Safety:** OpenAI (LLM), Enkrypt AI (Safety Gates), Tavily/Serper (Search Tool).
*   **Observability:** LangSmith, OpenTelemetry.
*   **Infrastructure:** Vercel (Frontend), Railway/Render (Backend), AWS S3/R2 (Storage).

## 9. Data Requirements
### 9.1 Relational Schema (PostgreSQL)
*   `Users` / `Projects` / `Meetings`
*   `Commitments`: ID, text, assignee, deadline, status, source_span_id.
*   `Enkrypt_Audit_Log`: Results of Gate 1 and Gate 2 checks.
*   `Task_Sync_Log`: Status of Linear/Jira ticket creation.

### 9.2 Vector Collections (Qdrant)
*   `commitments`: Embeddings of decision/action text + metadata.
*   `meeting_summaries`: Coarse retrieval for meeting-level context.
*   `risk_flags`: Embeddings of identified blockers and contradictions.

## 10. API Specifications
*   `POST /webhook/transcript-released`: Triggers the Mastra Extraction workflow.
*   `GET /api/commitments`: Returns all tasks for a project/user.
*   `POST /api/commitments/:id/confirm`: Manually resolves a low-confidence extraction.
*   `GET /api/search`: Semantic search across meeting history.
*   `PATCH /api/meetings/:id/redact`: Updates transcript before release.

## 11. Security Requirements
*   **Authentication:** Managed by Clerk (JWT-based).
*   **Authorization:** Row-level security (RLS) in Postgres and `projectId` filtering in Qdrant.
*   **Data Protection:** PII redaction capabilities; encryption at rest for S3 transcripts.
*   **Auditability:** Every autonomous action must have a corresponding Enkrypt AI "Pass" log.

## 12. Deployment & Infrastructure
*   **CI/CD:** GitHub Actions for automated testing and deployment.
*   **Environment:** Containerized Node.js service on Railway.
*   **Scaling:** Horizontal scaling of Mastra workers to handle concurrent transcript processing.

## 13. Success Metrics
*   **Extraction Precision:** % of AI extractions accepted by humans without modification.
*   **Safety Rate:** % of hallucinations caught by Enkrypt AI Gate 1.
*   **Execution Velocity:** Reduction in time from "Meeting End" to "Ticket Created."
*   **Conflict Resolution:** Number of contradictory decisions flagged before becoming blockers.

## 14. Timeline & Milestones
*   **Phase 1 (Week 1):** Core Ingestion, Mastra Extraction Workflow, and PostgreSQL state.
*   **Phase 2 (Week 2):** Enkrypt AI Gate 1 integration and Qdrant Memory/Conflict detection.
*   **Phase 3 (Week 3):** Action Agent (Linear/Jira/Slack) and Enkrypt Gate 2.
*   **Phase 4 (Week 4):** Frontend Dashboard, Human-Review UI, and Telemetry (LangSmith).

## 15. Open Questions & Risks
*   **Transcription Quality:** How does the system handle low-quality audio or heavy accents? (Mitigation: Human-in-the-loop queue).
*   **LLM Latency:** Multi-agent swarms can be slow. (Mitigation: Redis/BullMQ for async processing).
*   **API Rate Limits:** Managing limits for Linear/Jira/OpenAI during high-volume periods.