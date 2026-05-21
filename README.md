# Metalpol AI Automation Case Study

> PL: Kontekst zadania został podany po polsku, ale dokumentacja rozwiązania jest przygotowana po angielsku ze względu na techniczny charakter integracji, automatyzacji AI i system designu.
> EN: The assignment context was provided in Polish, but the solution documentation is written in English to reflect common technical documentation practice for automation, integration and AI system design.

This repository contains a solution design for an AI-assisted complaint handling automation process for **Metalpol Sp. z o.o.**, a fictional automotive components manufacturer.

The goal is not to replace service specialists with a fully autonomous AI system. The goal is to automate repetitive work, improve classification consistency, shorten response time and create measurable operational visibility.

## Repository purpose

This case study focuses on:

* understanding the current complaint handling process,
* modelling the AS-IS and TO-BE workflows using Event Storming,
* identifying where automation and AI can realistically add value,
* designing integrations between Microsoft 365, SAP ERP, JIRA Cloud, PostgreSQL and Azure Blob Storage,
* defining MVP scope, risks, trade-offs and human-in-the-loop controls.

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

## Documentation

| Document                                                       | Description                                                                                                                               |
| -------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| [AS-IS Event Storming](docs/01_event_storming_as_is.md)        | Current manual complaint handling process, actors, commands, domain events, systems and pain points.                                      |
| [TO-BE Event Storming](docs/02_event_storming_to_be.md)        | Proposed AI-assisted process after automation, including extraction, classification, SAP validation, JIRA ticketing and human review.     |
| [Solution Specification](docs/03_solution_specification.md)    | Main technical specification covering flow, integrations, technology stack, AI usage, validation rules, data model, metrics and security. |
| [Trade-offs and MVP Scope](docs/04_tradeoffs_and_mvp_scope.md) | MVP priorities, rollout phases, key architectural decisions, risks and mitigations.                                                       |

## Diagrams

| Diagram                                                           | Description                                                                                   |
| ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| [AS-IS Event Storming Diagram](diagrams/as_is_event_storming.mmd) | Styled Mermaid process map of the current manual workflow.                                    |
| [TO-BE Event Storming Diagram](diagrams/to_be_event_storming.mmd) | Styled Mermaid process map of the proposed automated workflow.                                |
| [Architecture Context Diagram](diagrams/architecture_context.mmd) | High-level architecture view showing systems, services, AI layer, human review and reporting. |

## Examples

| Example                                                                 | Description                                                                     |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| [Sample Polish complaint email](examples/sample_complaint_email_pl.md)  | Example input complaint message in Polish.                                      |
| [Sample English complaint email](examples/sample_complaint_email_en.md) | Example input complaint message in English.                                     |
| [Sample extracted payload](examples/sample_extracted_payload.json)      | Example structured JSON payload produced by the extraction and validation flow. |

## Business context

Metalpol currently handles customer complaints manually:

1. Customer sends an email to `reklamacje@metalpol.pl` with defect photos, order number and description.
2. Service specialist reads the email and manually copies data into Excel.
3. Defect is classified manually as `visual`, `dimensions`, `material` or `logistics`.
4. Specialist checks order and batch information in SAP.
5. Complaint ticket is created in JIRA.
6. Customer response is written manually.
7. If the defect is confirmed, a correction ticket is created for the Quality Department.

## Main problems

* Around 40% of emails may land in spam or be read with delay.
* One specialist handles about 30 complaints per day, with peaks up to 80.
* Backlog can reach 2–3 days during high-volume periods.
* Defect classification is inconsistent between specialists.
* SAP and JIRA are not integrated.
* Excel acts as a manual register instead of a reliable process system.
* Management lacks metrics by category, production line, batch and backlog.

## Proposed solution

The proposed solution introduces an automated complaint intake and processing flow:

1. Microsoft Graph webhook detects a new complaint email.
2. Intake service downloads email metadata, body and attachments.
3. Attachments are archived in Azure Blob Storage.
4. AI extraction service converts unstructured email content into structured JSON.
5. Deterministic validator checks required fields, schema and idempotency.
6. AI classification service assigns a defect category with confidence score.
7. SAP Lookup Service validates order and batch data.
8. JIRA Integration Service creates a `Complaint` issue.
9. Low-confidence, incomplete or failed validations are routed to human review.
10. AI Response Draft Service prepares a customer response draft in Polish or English.
11. Service specialist approves or edits the draft before sending.
12. If defect is confirmed, a `Correction` issue is created for the Quality Department.
13. Metrics and audit data are stored for reporting.

## Technology stack

| Layer                 | Technology                                            | Reason                                                             |
| --------------------- | ----------------------------------------------------- | ------------------------------------------------------------------ |
| Backend API           | Python + FastAPI                                      | Suitable for webhooks, integrations and AI workflow orchestration. |
| Background processing | Celery/RQ or Azure Functions                          | Handles retries, async SAP/JIRA calls and attachment processing.   |
| Database              | PostgreSQL                                            | Provides structured state, auditability and reporting capability.  |
| Email integration     | Microsoft Graph API                                   | Native Microsoft 365 integration with OAuth2 and webhook support.  |
| ERP integration       | SAP REST API                                          | Provides order, batch and production context.                      |
| Ticketing             | JIRA Cloud REST API                                   | Keeps the existing operational workflow in project `REK`.          |
| Storage               | Azure Blob Storage                                    | Stores large defect images outside JIRA.                           |
| AI layer              | Azure OpenAI / OpenAI API with structured JSON output | Supports text extraction, classification and response drafting.    |
| Observability         | Structured logs and reporting dashboard               | Enables monitoring, troubleshooting and management metrics.        |

## AI usage boundaries

AI is used where interpretation of unstructured text is valuable:

* extracting structured complaint data from emails,
* classifying defect type,
* generating response drafts,
* optionally supporting image-based defect analysis in a later phase.

AI is not treated as the source of truth. SAP, JIRA and PostgreSQL remain authoritative systems. Customer-facing responses require human approval in the MVP.

## Human-in-the-loop strategy

Human review is required when:

* required fields are missing,
* AI confidence is below threshold,
* SAP order or batch lookup fails,
* customer identity is unclear,
* multiple complaints appear in one email,
* AI output is malformed,
* the case may affect customer relationship, quality decision or financial outcome.

## MVP scope

The MVP should focus on:

* email ingestion,
* attachment archiving,
* structured data extraction,
* deterministic validation,
* AI-assisted classification,
* SAP order and batch lookup,
* JIRA Complaint creation,
* human review routing,
* basic reporting and metrics.

Advanced image recognition, full autonomous complaint decisions and automatic financial outcomes are intentionally outside MVP scope.

## Key design decisions

| Decision                                                   | Rationale                                                                       |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Use Microsoft Graph webhook instead of mailbox polling     | Reduces delay and avoids manual mailbox monitoring.                             |
| Store images in Azure Blob Storage                         | Defect photos can be large and should not make JIRA the primary binary archive. |
| Keep JIRA as workflow system                               | Reduces adoption risk because the existing process already uses JIRA.           |
| Use AI for extraction and classification only where useful | Avoids pretending that AI should own deterministic business rules.              |
| Require human approval for customer responses              | Prevents uncontrolled customer-facing or quality-impacting decisions.           |
| Add PostgreSQL reporting layer                             | Provides metrics that Excel cannot reliably deliver.                            |

## Success metrics

Potential success metrics:

* reduced average time to first response,
* lower manual data entry effort,
* improved classification consistency,
* lower backlog during seasonal peaks,
* percentage of complaints processed automatically,
* percentage of cases routed to human review,
* SAP validation success rate,
* defect distribution by type, line and batch,
* correction ticket rate.

## Commit history intention

The repository is built incrementally to reflect the analysis workflow:

1. initialize repository structure,
2. document AS-IS process,
3. document TO-BE process,
4. add architecture context,
5. add solution specification,
6. add MVP trade-offs,
7. add sample inputs and extracted payloads,
8. polish README and navigation.

## Notes

This repository does not contain production implementation code. It is a system design and automation analysis case study prepared for a recruitment task.
