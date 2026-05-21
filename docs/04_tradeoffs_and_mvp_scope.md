# Trade-offs and MVP Scope

## 1. Why not full automation?

Full automation would be risky because complaint handling affects customer communication, quality processes and potentially financial or legal outcomes.

MVP should automate intake, extraction, classification and ticket creation, but keep human approval for final responses and uncertain cases.

## 2. Why use AI at all?

AI is useful for interpreting unstructured email text in Polish and English, extracting fields and classifying defect descriptions.

However, AI should not be the system of record. SAP, JIRA and PostgreSQL remain authoritative systems.

## 3. Why not start with image recognition?

Image recognition may be useful later, but it is not required for MVP.

The biggest current pain is manual email handling, inconsistent classification and lack of metrics. These can be solved earlier with text extraction, SAP lookup and JIRA automation.

## 4. Why JIRA instead of a custom workflow app?

JIRA already exists and supports the current operational process. Using it reduces adoption risk and implementation cost.

A custom app could be considered later if JIRA becomes a bottleneck.

## 5. Why PostgreSQL reporting layer?

Excel does not provide reliable reporting, auditability or integration. PostgreSQL gives structured reporting without forcing SAP/JIRA to become analytical systems.

## 6. MVP priorities

### Phase 1

- Graph email ingestion
- Attachment archiving
- Structured data extraction
- SAP lookup
- JIRA Complaint creation
- Basic reporting

### Phase 2

- AI classification tuning
- Customer response drafts
- Human review queue
- Duplicate detection

### Phase 3

- Image-based defect support
- Production line anomaly detection
- Deeper SAP/QM analytics
- SLA prediction

## 7. Main risks

| Risk | Impact | Mitigation |
|---|---|---|
| Poor email quality or missing order numbers | Cannot validate complaint automatically | Human review route |
| Inconsistent historical labels | Weak AI examples and poor reporting baseline | Define target taxonomy explicitly |
| SAP API downtime or rate limits | Ticket creation delayed or incomplete | Async retry, backoff, pending validation status |
| Overtrust in AI classification | Wrong routing or wrong customer response | Confidence threshold and human approval |
| Duplicate complaints | Duplicate JIRA tickets and inflated metrics | Idempotency by message ID and similarity checks |
| Large/corrupted attachments | Processing failure | Store status separately and continue ticket creation |
| Multiple complaints in one email | Incorrect issue granularity | Parent issue + manual split |

## 8. Shadow mode rollout

Before enabling automated JIRA ticket creation, the system can run in shadow mode:

1. Process real complaint emails.
2. Extract and classify data.
3. Compare AI classification with human classification.
4. Measure confidence distribution and error patterns.
5. Tune thresholds and rules.
6. Enable automation only for high-confidence cases.

## 9. Success metrics

- Reduce average first response time from 2 days to less than 1 business day.
- Reduce manual data entry for standard complaints.
- Improve classification consistency.
- Provide reliable metrics by category, batch and production line.
- Keep low-confidence cases visible instead of silently failing.
