# Solution Specification

## 1. Goal

The goal is to reduce manual work in complaint handling, improve classification consistency, shorten response time and provide operational metrics for management.

The solution should not fully replace service specialists. It should automate repetitive steps and route uncertain or risky cases to humans.

## 2. Scope

### In scope

- Automatic email intake from `reklamacje@metalpol.pl`
- Attachment archiving in Azure Blob Storage
- AI-based extraction of structured complaint data
- AI-assisted defect classification
- SAP order and batch lookup
- Automatic JIRA `Complaint` ticket creation
- AI-generated customer response draft
- Human approval before sending customer-facing responses
- Creation of `Correction` tickets for confirmed defects
- Reporting metrics by defect type, production line, batch, order and processing time

### Out of scope for MVP

- Fully automatic complaint rejection or approval
- Fully autonomous customer communication
- Advanced image defect recognition
- SAP data modification
- Automatic financial compensation decisions

## 3. Proposed flow

1. Microsoft Graph webhook receives a new complaint email.
2. Intake service downloads email metadata, body and attachments.
3. Attachments are stored in Azure Blob Storage using SAS-secured access.
4. AI extraction service extracts:
   - order number,
   - batch number if present,
   - customer identifier,
   - language,
   - defect description,
   - attached image references.
5. Extracted data is validated against deterministic rules.
6. AI classification service assigns one of:
   - `visual`,
   - `dimensions`,
   - `material`,
   - `logistics`.
7. SAP lookup service checks order and batch data.
8. JIRA Integration Service creates a `Complaint` issue in project `REK`.
9. If confidence is low or SAP lookup fails, issue is routed to human review.
10. AI response service prepares a draft response in PL or EN.
11. Specialist reviews and approves the response.
12. If defect is confirmed, the system creates a `Correction` ticket for Quality Department.
13. Metrics are stored in PostgreSQL reporting layer.

## 4. Suggested technology stack

| Layer | Technology | Reason |
|---|---|---|
| Backend API | Python + FastAPI | Good fit for integrations, webhooks and AI workflows |
| Background processing | Celery/RQ or Azure Functions | Retry, async jobs, SAP/JIRA calls, attachment processing |
| Database | PostgreSQL | Structured reporting and auditability |
| Email integration | Microsoft Graph API | Native Microsoft 365 integration, OAuth2, webhooks |
| ERP integration | SAP REST API | Existing order and batch data |
| Ticketing | JIRA Cloud REST API | Existing workflow in `REK` project |
| Storage | Azure Blob Storage | Large image attachments and controlled access |
| AI layer | Azure OpenAI / OpenAI API with structured JSON output | Text extraction, classification and response drafting |
| Observability | Structured logs + metrics dashboard | Operational visibility and audit trail |

## 5. AI usage

AI is used only where probabilistic interpretation is useful.

### AI extraction

Input:

- email subject,
- email body,
- selected metadata,
- OCR/image description in a future phase.

Output example:

```json
{
  "order_id": "MP-2026-10445",
  "batch_id": "B-77A-2026",
  "customer_name": "AutoParts GmbH",
  "language": "EN",
  "defect_description": "visible scratch on coated surface",
  "missing_fields": [],
  "confidence": 0.91
}
```

### AI classification

Input:

- defect description,
- extracted structured fields,
- optionally SAP/QM data.

Output example:

```json
{
  "category": "visual",
  "confidence": 0.87,
  "reason": "Customer reports scratches and surface coating defect.",
  "requires_human_review": false
}
```

### AI response draft

Input:

- complaint summary,
- order validation status,
- batch validation status,
- defect category,
- customer language.

Output:

- draft response only,
- never sent automatically without approval in MVP.

## 6. Deterministic validation rules

AI output must be validated by deterministic logic.

Examples:

- If order ID is missing, route to human review.
- If classification confidence is below `0.75`, route to human review.
- If SAP returns `404` for order or batch, create ticket but mark validation as failed.
- If attachment download fails, create ticket with warning.
- If duplicate email/message ID exists, skip duplicate processing.
- If customer is unknown in PostgreSQL customer DB, assign manual verification.

## 7. Integrations

### Microsoft Graph API

Used for:

- new email webhook,
- message retrieval,
- attachment retrieval,
- sending approved responses.

Reason:

- native integration with Microsoft 365,
- avoids mailbox polling,
- supports OAuth2 and webhook-based processing.

### Azure Blob Storage

Used for:

- storing defect images,
- linking attachments in JIRA tickets.

Reason:

- images can be large,
- JIRA should not be the primary binary archive,
- SAS tokens allow controlled access.

### SAP ERP API

Used for:

- order validation,
- batch validation,
- production context lookup.

Constraints:

- rate limit: `100 req/min`,
- use retry with backoff,
- cache repeated order/batch lookups.

### JIRA Cloud API

Used for:

- creating `Complaint` issues,
- creating `Correction` issues,
- assigning review tasks,
- keeping operational workflow visible.

### PostgreSQL customer DB

Used for:

- customer lookup,
- enrichment,
- reporting dimensions.

Read-only access is enough for MVP.

## 8. Minimal data model

### complaints

```text
complaint_id
source_message_id
received_at
customer_id
order_id
batch_id
language
defect_category
classification_confidence
status
jira_complaint_key
jira_correction_key
response_sent_at
human_review_required
created_at
updated_at
```

### complaint_attachments

```text
attachment_id
complaint_id
blob_url
filename
content_type
size_bytes
created_at
```

### ai_decisions

```text
decision_id
complaint_id
decision_type
input_hash
output_json
confidence
model_name
created_at
```

## 9. Error handling

- Email processing must be idempotent using Microsoft Graph message ID.
- Attachment archiving failure should not block ticket creation.
- SAP API failure should create a JIRA ticket with `SAP validation pending`.
- AI failure should route to manual review.
- JIRA failure should retry and alert operator.
- Duplicate complaints should be detected by message ID and optionally order/batch similarity.

## 10. Metrics

The solution should provide:

- complaints per day/week/month,
- average time to first response,
- backlog size,
- defect category distribution,
- complaint count by production line,
- complaint count by batch,
- SAP validation success rate,
- AI classification confidence distribution,
- human review rate,
- correction ticket rate.

## 11. Security and compliance

- OAuth2 for Microsoft Graph.
- No credentials in repository.
- Store only required metadata.
- Use SAS tokens with limited lifetime.
- Log AI decisions, but avoid storing unnecessary personal data.
- Restrict access to Blob Storage and JIRA tickets.
- Keep audit trail for customer-facing responses.
