# Fintech Architecture Principles

This note describes the engineering principles I use when thinking about financial and regulated platforms. It is not a production blueprint; it is a concise statement of architectural priorities and trade-offs.

## 1. The ledger is a system of record, not a balance cache

Balances should be derived from durable accounting events rather than treated as the primary truth. For money movement, I prefer an append-oriented journal with explicit debit and credit entries and clear account boundaries.

A useful invariant is simple:

> Every accepted financial event must remain explainable after the fact.

That means retaining enough information to reconstruct what happened, why it happened, and which business rule allowed it.

## 2. Idempotency is part of the domain model

Retries are normal in distributed systems. A payment or transfer API that treats duplicate requests as an edge case is not finished.

Financial commands should carry stable idempotency identifiers, and the platform should distinguish between:

- a safe retry of the same command
- a genuinely new financial instruction
- a conflicting request reusing an identifier

The objective is not just API cleanliness. It is preventing accidental double execution.

## 3. Reconciliation is a first-class workflow

No distributed financial system should assume that every internal state transition will perfectly match every external provider.

Reconciliation should be designed alongside the transaction flow, not added later as an operations script.

That includes:

- internal ledger-to-ledger reconciliation
- platform-to-provider reconciliation
- settlement verification
- exception queues
- replay and recovery procedures
- explicit ownership of unresolved differences

## 4. Auditability should be structural

An audit trail is more useful when it emerges naturally from the architecture rather than from application logging alone.

Important state changes should capture:

- who or what initiated the change
- when it occurred
- the previous and resulting state
- the policy or rule involved
- correlation and transaction identifiers
- external references where applicable

Operational logs can expire. Core business evidence often cannot.

## 5. Authentication and authorization are different problems

Authentication establishes identity. Authorization decides whether that identity may perform a specific action on a specific resource.

Financial platforms should model authorization explicitly and avoid relying on UI visibility as a security control.

Particular attention belongs on:

- service-to-service identity
- administrative elevation
- privileged workflows
- scoped API access
- approval boundaries
- separation of duties

## 6. Compliance orchestration should preserve evidence

Compliance checks are often implemented as yes/no gates. That is rarely sufficient.

A useful compliance workflow also records:

- which checks were performed
- which data was evaluated
- which version of a rule or list was used
- the outcome
- the source of the outcome
- whether a human override occurred
- why that override was permitted

This makes the compliance decision reproducible rather than merely recorded.

## 7. Event-driven does not mean eventually correct by accident

Events are valuable for decoupling, but financial events need clear ownership, ordering assumptions, replay behavior, and duplicate handling.

For every event, I want to know:

1. Who owns the source of truth?
2. Can the event be delivered more than once?
3. Can events arrive out of order?
4. What happens if a consumer is unavailable?
5. Can the event safely be replayed?
6. How is reconciliation performed if processing diverges?

## 8. AI should assist controlled decisions, not obscure them

AI can be useful in financial operations for classification, search, anomaly triage, summarization, and operator support.

Where a decision has financial, legal, or compliance consequences, the architecture should preserve clear control boundaries and explainability.

My preferred pattern is:

**AI proposes → deterministic controls evaluate → human or policy approves where required → system records the evidence.**

## 9. Failure modes belong in the design

Architecture documents often describe only the successful transaction path.

The more useful questions are:

- What if the provider times out after accepting the payment?
- What if we persist the command but fail before publishing the event?
- What if the event is delivered twice?
- What if settlement differs from authorization?
- What if a compliance provider is unavailable?
- What if reconciliation detects a discrepancy three days later?

Design quality is often most visible in these answers.

## 10. Modernization must preserve domain knowledge

Older platforms are rarely valuable because of their technology stack. They are valuable because they encode business rules accumulated over years.

A modernization effort should identify those rules explicitly before replacing the implementation.

The goal is not to reproduce old software with newer syntax.

The goal is to preserve the valuable domain knowledge while improving security, operability, testability, data access, and the ability to change.
