# rsl-royalty-bridge-v0.2
Four-layer integration model connecting RSL Collective, the RSL-to-Kazene Bridge, and Kazene Royalty OS for permission, reconciliation, and trace-based royalty return.

# RSL Collective Full Integration Model

RSL Collective handles **permission and reporting**.  
The RSL-to-Kazene Bridge handles **synchronization and auditability**.  
Kazene Royalty OS handles **trace-based allocation and royalty return**.

## What this model does

This model connects three roles without collapsing them into one system:

- **RSL / RSL Collective**  
  External permission, collective licensing, repertoire scope, and reporting.

- **RSL-to-Kazene Bridge**  
  Bidirectional synchronization between external reports and internal trace events.

- **Kazene Royalty OS**  
  Internal trace weighting, royalty pool allocation, recomputation, and reversal.

## Why this architecture is strong

The key idea is separation of responsibilities:

- **Permission** is not the same as **contribution**
- **External reporting** is not the same as **internal allocation**
- **Auditability** requires both sides to be linked, not guessed

This is why the bridge exists.

## Four-layer architecture

1. **Discovery / Authorization Layer**  
   RSL terms, machine-readable permissions, license discovery

2. **Collective Licensing / Reporting Layer**  
   RSL Collective repertoire, licensing relationship, period reports

3. **Bridge / Reconciliation Layer**  
   `rsl_report_id` ↔ `trace_event_id`, verification, idempotency, status control

4. **Trace Allocation / Royalty Return Layer**  
   Kazene-side trace weighting, pool allocation, recomputation, reversal

## Operational principle

The integrated model follows this sequence:

**permission → report → reconcile → allocate**

More concretely:

1. content is licensed externally
2. usage is reported through the collective layer
3. report objects are reconciled with internal trace events
4. royalties are allocated internally through trace-aware logic

## Responsibility split

- **RSL Collective** answers:  
  _“Was this use licensed and reportable?”_

- **Bridge** answers:  
  _“Does the external report actually match internal trace evidence?”_

- **Kazene Royalty OS** answers:  
  _“How should value be returned inside the royalty pool?”_

## One-sentence definition

The full integration model is a four-layer architecture in which  
RSL Collective manages external permission and reporting,  
the Bridge enforces synchronization and auditability,  
and Kazene Royalty OS performs internal trace-based royalty allocation.

## Design philosophy

Do not merge everything.  
Instead, keep the system legible:

- **institutional layer**
- **bridge layer**
- **allocation layer**

That separation is what makes the model scalable, auditable, and adoptable.
