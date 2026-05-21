# Metalpol AI Complaint Automation

Recruitment task for the **AI Automation Analyst** role.

This repository describes an automation proposal for handling customer complaints at **Metalpol Sp. z o.o.**, a fictional automotive components manufacturer.

The goal is not to replace service specialists with a magical AI oracle, because reality has already suffered enough. The goal is to automate repetitive work, improve classification consistency, shorten response time and create measurable operational visibility.

## Navigation

1. [AS-IS Event Storming](docs/01_event_storming_as_is.md)
2. [TO-BE Event Storming](docs/02_event_storming_to_be.md)
3. [Solution Specification](docs/03_solution_specification.md)
4. [Trade-offs and MVP Scope](docs/04_tradeoffs_and_mvp_scope.md)

## Diagrams

- [AS-IS Event Storming Mermaid](diagrams/as_is_event_storming.mmd)
- [TO-BE Event Storming Mermaid](diagrams/to_be_event_storming.mmd)
- [Architecture Context Mermaid](diagrams/architecture_context.mmd)

## Example payloads

- [Sample extracted complaint payload](examples/sample_extracted_payload.json)
- [Sample Polish complaint email](examples/sample_complaint_email_pl.md)
- [Sample English complaint email](examples/sample_complaint_email_en.md)

## Problem summary

Current complaint handling is mostly manual:

- complaints arrive by email,
- data is copied into Excel,
- categorization is subjective,
- SAP and JIRA are disconnected,
- metrics are missing,
- backlog grows during seasonal peaks.

## Proposed automation

The proposed solution uses:

- Microsoft Graph API for complaint email ingestion,
- Azure Blob Storage for image archiving,
- AI-based extraction and classification,
- SAP ERP API for order and batch validation,
- JIRA Cloud API for complaint and correction tickets,
- PostgreSQL reporting layer for metrics,
- human-in-the-loop review for uncertain or high-impact cases.

## Key principles

- AI supports extraction, classification and response drafting.
- SAP, JIRA and PostgreSQL remain authoritative systems.
- Customer-facing responses require human approval in the MVP.
- Every AI decision is logged with input hash, output, model name and confidence score.
- The MVP focuses on intake, classification, ticket creation and metrics before advanced image analysis.
