# Metalpol AI Automation Case Study

> **PL:** Kontekst zadania został podany po polsku, ale dokumentacja rozwiązania została przygotowana po angielsku ze względu na techniczny charakter integracji, automatyzacji AI i system designu.  
> **EN:** The assignment context was provided in Polish, but the solution documentation is written in English to reflect common technical documentation practice for AI automation, integrations, and system design.

This repository contains a **solution design** for an AI-assisted complaint handling process for **Metalpol Sp. z o.o.**, a fictional manufacturer of automotive metal components.

The proposal is intentionally designed as a **practical, auditable automation system**, not as a fully autonomous “AI does everything” concept.  
The goal is to reduce manual work, improve classification consistency, shorten response time, and create operational visibility while keeping humans in the loop for risky or ambiguous cases.

---

## Recommended reading order

1. [AS-IS Event Storming](docs/01_event_storming_as_is.md)
2. [TO-BE Event Storming](docs/02_event_storming_to_be.md)
3. [Solution Specification](docs/03_solution_specification.md)
4. [Trade-offs and MVP Scope](docs/04_tradeoffs_and_mvp_scope.md)

If you want the shortest possible overview, read:
- this README,
- the TO-BE document,
- and the Solution Specification.

---

## What this repository contains

This case study covers:

- understanding the current complaint handling process,
- modelling the **AS-IS** and **TO-BE** workflows using Event Storming,
- identifying where automation and AI realistically add value,
- designing integrations between **Microsoft 365**, **SAP ERP**, **JIRA Cloud**, **PostgreSQL**, and **Azure Blob Storage**,
- defining **MVP scope**, **risks**, **trade-offs**, and **human-in-the-loop** controls.

This repository does **not** contain production implementation code.  
It is a **system design and automation analysis case study** prepared for a recruitment task.

---

## Repository structure

```text
metalpol-ai-automation/
├── README.md
├── docs/
│   ├── 01_event_storming_as_is.md
│   ├── 02_event_storming_to_be.md
│   ├── 03_solution_specification.md
│   └── 04_tradeoffs_and_mvp_scope.md
├── diagrams/
│   ├── as_is_event_storming.mmd
│   ├── to_be_event_storming.mmd
│   └── architecture_context.mmd
└── examples/
    ├── sample_complaint_email_pl.md
    ├── sample_complaint_email_en.md
    └── sample_extracted_payload.json
```

---

## Documentation

| Document                                                       | Purpose                                                                                                                                      |
| -------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [AS-IS Event Storming](docs/01_event_storming_as_is.md)        | Describes the current manual complaint process, actors, systems, domain events, and pain points.                                             |
| [TO-BE Event Storming](docs/02_event_storming_to_be.md)        | Shows the proposed AI-assisted process after automation, including extraction, validation, SAP enrichment, JIRA ticketing, and human review. |
| [Solution Specification](docs/03_solution_specification.md)    | Main technical specification describing architecture, flow, integrations, AI usage, validation rules, reporting, and operational decisions.  |
| [Trade-offs and MVP Scope](docs/04_tradeoffs_and_mvp_scope.md) | Explains MVP priorities, rollout decisions, constraints, and architectural trade-offs.                                                       |

---

## Diagrams

| Diagram                                                           | Purpose                                                                                       |
| ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| [AS-IS Event Storming Diagram](diagrams/as_is_event_storming.mmd) | Visual representation of the current manual complaint workflow.                               |
| [TO-BE Event Storming Diagram](diagrams/to_be_event_storming.mmd) | Visual representation of the target automated workflow.                                       |
| [Architecture Context Diagram](diagrams/architecture_context.mmd) | High-level architecture showing systems, integrations, AI layer, human review, and reporting. |

---

## Examples

| Example                                                                 | Purpose                                                                   |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| [Sample Polish complaint email](examples/sample_complaint_email_pl.md)  | Example complaint message in Polish.                                      |
| [Sample English complaint email](examples/sample_complaint_email_en.md) | Example complaint message in English.                                     |
| [Sample extracted payload](examples/sample_extracted_payload.json)      | Example structured output produced by the extraction and validation flow. |

---

## Business context

Metalpol currently handles complaints manually:

1. A customer sends an email to `reklamacje@metalpol.pl` with a defect description, order number, and photos.
2. A service specialist manually copies the data into Excel.
3. The complaint is manually categorized (`visual`, `dimensions`, `material`, `logistics`).
4. Order and batch information are manually checked in SAP.
5. A `Complaint` issue is created in JIRA.
6. A customer response is written manually.
7. If the defect is confirmed, a `Correction` ticket is created for the Quality Department.

---

## Main problems

The current process creates several operational issues:

* email intake is unreliable and delayed,
* manual data entry introduces bottlenecks,
* complaint classification is inconsistent between specialists,
* SAP and JIRA are disconnected,
* Excel acts as a fragile operational register,
* management lacks reliable metrics across categories, lines, batches, and backlog.

---

## Proposed solution in one paragraph

The proposed solution introduces an **event-driven complaint intake pipeline**.
A new complaint email triggers automated ingestion through Microsoft Graph. The message and attachments are archived, complaint data is extracted and classified, deterministic validation is applied, SAP is queried for production context, a JIRA `Complaint` ticket is created, and a response draft is prepared for specialist approval. Low-confidence or incomplete cases are routed to human review. All steps are logged for reporting, auditability, and continuous improvement.

---

## Where AI is used — and where it is not

### AI is used for:

* extracting structured complaint data from unstructured email text,
* classifying complaint type,
* generating customer response drafts,
* optionally supporting image-based defect analysis in a future phase.

### AI is not used for:

* mailbox event detection,
* attachment storage,
* SAP lookup,
* JIRA issue creation,
* deterministic validation rules,
* retry logic,
* idempotency,
* workflow routing.

This is intentional.
The design treats AI as a **supporting interpretation layer**, not as the operational source of truth.

---

## Human-in-the-loop strategy

Human review is required when:

* required fields are missing,
* confidence is below threshold,
* SAP order or batch lookup fails,
* customer identity is unclear,
* multiple complaints appear in a single email,
* AI output is malformed,
* the case may affect customer relationship, quality decision, or financial outcome.

This allows the system to automate high-volume repetitive work without creating uncontrolled customer-facing risk.

---

## MVP scope

The MVP focuses on:

* email ingestion,
* attachment archiving,
* structured data extraction,
* deterministic validation,
* AI-assisted classification,
* SAP order and batch lookup,
* JIRA `Complaint` creation,
* human review routing,
* basic metrics and reporting.

The MVP intentionally excludes:

* advanced computer vision,
* fully autonomous complaint decisions,
* autonomous financial outcomes,
* complex predictive quality analytics.

---

## Technology stack

| Layer                 | Technology                            | Reason                                                                               |
| --------------------- | ------------------------------------- | ------------------------------------------------------------------------------------ |
| Backend API           | Python + FastAPI                      | Good fit for webhooks, integrations, orchestration, and structured automation flows. |
| Background processing | Celery / RQ or Azure Functions        | Supports asynchronous processing, retries, and integration workloads.                |
| Database              | PostgreSQL                            | Stores operational state, audit data, and reporting-friendly records.                |
| Email integration     | Microsoft Graph API                   | Native Microsoft 365 integration with OAuth2 and webhook support.                    |
| ERP integration       | SAP REST API                          | Source of truth for orders, batches, and production context.                         |
| Ticketing             | JIRA Cloud REST API                   | Preserves the existing operational workflow in project `REK`.                        |
| Storage               | Azure Blob Storage                    | Suitable archive for binary complaint attachments.                                   |
| AI layer              | Azure OpenAI / OpenAI API             | Supports extraction, classification, and draft generation with structured output.    |
| Observability         | Structured logs + reporting dashboard | Supports troubleshooting, auditability, and management reporting.                    |

---

## Key design decisions

| Decision                                             | Rationale                                                                  |
| ---------------------------------------------------- | -------------------------------------------------------------------------- |
| Use Microsoft Graph webhook instead of polling       | Reduces delay and removes manual mailbox monitoring.                       |
| Archive images in Azure Blob Storage                 | Keeps binary storage out of JIRA and improves scalability.                 |
| Keep JIRA as the operational workflow tool           | Reduces change-management risk by keeping the existing team process.       |
| Use AI only where interpretation is valuable         | Avoids pushing probabilistic models into deterministic business rules.     |
| Require human approval for customer-facing responses | Reduces operational and reputational risk.                                 |
| Add PostgreSQL as a reporting and audit layer        | Creates visibility and process metrics that Excel cannot provide reliably. |

---

## Success metrics

Suggested metrics for evaluating the solution:

* average time to first response,
* complaint backlog,
* percentage of automatically processed complaints,
* percentage routed to manual review,
* classification consistency,
* SAP validation success rate,
* complaint volume by category,
* complaint volume by production line and batch,
* correction ticket rate.

---

## Assumptions

This design assumes that:

* SAP remains the source of truth for orders and batches,
* JIRA remains the operational workflow system,
* complaint emails usually contain enough information to identify the order or customer,
* customer-facing communication requires human approval in the MVP,
* AI services can return structured outputs together with confidence scoring.

---

## Notes

The repository was built incrementally to reflect a realistic analysis workflow:

1. understand the current process,
2. model AS-IS,
3. design TO-BE,
4. define the architecture,
5. specify the solution,
6. document trade-offs and MVP boundaries,
7. add examples and navigation.

The objective of this repository is not to present a polished “AI vision deck”, but to show **system understanding, architecture thinking, and practical automation design decisions**.
