# PRD: AI Triage Agent for Support Inbox

> Portfolio sample — a product requirements document written to show how I think
> about scoping, requirements, and rollout. Fictional product, real PM craft.

## 1. Problem

Support teams drown in inbound tickets. Today at a typical B2B SaaS company:
- Median first response time is **7 hours**; 22% of tickets wait > 24h
- ~40% of tickets are repetitive (password resets, billing questions, status checks)
- Agents context-switch constantly, and urgent issues (outages, security) get buried

**Who feels it:** support agents (burnout, backlog), customers (slow responses), and the business (churn risk on slow first response — every hour of wait increases churn likelihood).

## 2. Goal & success metrics

**Goal:** Cut median first response time from 7h to under 1h within one quarter of launch.

| Metric | Baseline | Target (Q1) |
|---|---|---|
| Median first response time | 7h | < 1h |
| % tickets auto-resolved | 0% | 25% |
| CSAT (on agent-handled tickets) | 4.1 | ≥ 4.1 (no regression) |
| Escalation accuracy (auto-triaged → correct team) | — | ≥ 90% |

**Non-goals:** replacing human agents; handling billing disputes involving money movement (always human); supporting languages beyond English in v1.

## 3. Users

- **Support agents** — need a clean triage queue, not a black box
- **Support leads** — need visibility into what the agent touched and why
- **End customers** — need fast, correct answers or a fast handoff

## 4. Requirements

### Functional

1. **Ingest**: Connect to the ticket source (Zendesk/Intercom API). Poll every 60s; new tickets appear in the triage queue within 2 minutes.
2. **Classify**: Every new ticket gets `category` (billing / account / technical / security / other), `urgency` (P1–P4), and `sentiment`. P1 = outage/security/data-loss language.
3. **Auto-resolve**: If confidence ≥ 0.9 AND category is in the safe list (password reset, billing FAQ, status), the agent drafts and sends a response from the help-center corpus, then marks the ticket solved. The customer can always reply "talk to a human" to reopen.
4. **Route**: Everything else is assigned to the right queue with a suggested priority and a 2-line summary. P1s page the on-call lead immediately.
5. **Audit trail**: Every agent action logs the ticket, the decision, the confidence, and the sources used. Leads can filter by "handled by agent" and overturn any decision in one click.
6. **Kill switch**: A single toggle pauses all agent actions; the queue reverts to manual instantly.

### Non-functional

- P95 classification latency < 3s per ticket
- No PII leaves the region of the customer's data residency
- Every model output is logged for 90 days (debuggability + compliance)

## 5. UX notes

- The triage queue shows agent-handled tickets with a distinct badge and confidence score — agents should trust but verify.
- The customer never sees "AI" branding unless the company opts in; the reply reads like a normal support response with a "was this helpful?" prompt.

## 6. Rollout

1. **Shadow mode (2 wks)**: agent classifies and drafts, sends nothing. Measure classification accuracy vs. human labels.
2. **10% of tickets (2 wks)**: auto-resolve on, human review of every agent action.
3. **50% → 100%**: expand as escalation accuracy holds above 90%.
4. Kill-switch drill in week 1 so the team trusts the off-ramp.

## 7. Risks & mitigations

| Risk | Mitigation |
|---|---|
| Wrong auto-response damages trust | Confidence threshold + safe-category list + one-click reopen |
| P1 misclassified as low urgency | Separate P1 detector with keyword + sentiment ensemble; page on-call on any P1 signal |
| Agents feel replaced | Position as triage, not replacement; agents handle the complex 60%, CSAT guardrail metric |

## 8. Open questions

- Which ticket source do we integrate first — Zendesk or Intercom?
- Do we need multi-language in v1 for the EU customer base?
- Who owns the help-center corpus quality — support ops or product?
