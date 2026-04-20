# Failure Cases and State Transition Guide

## Overview

This document explains how failure-oriented records should move through the
`RSL-to-Kazene Royalty Bridge Specification v0.2` state model.

The goal is simple:

- do not silently drop incomplete records
- do not auto-settle suspicious records
- do not confuse temporary incompleteness with final rejection

In this bridge model, failure is not treated as an accidental edge case.
It is treated as a first-class operational state.

---

## Core States

The bridge defines the following primary states:

- `pending`
- `blocked_or_pending`
- `ready`
- `rejected`
- `settled`
- `reversed`

This document focuses on the failure-oriented states:

- `pending`
- `blocked_or_pending`
- `rejected`

These three states are where operational safety is won or lost.

---

## State Meanings

### `pending`

`pending` means the record is structurally valid enough to exist in the system,
but it is not yet complete enough for settlement.

Typical causes:

- `rsl_report_id` exists but `trace_event_id` has not arrived yet
- signature exists but verification has not been executed
- the event is newly ingested and still waiting for synchronization

This is a **normal waiting state**, not an error state.

---

### `blocked_or_pending`

`blocked_or_pending` means the record is no longer just “waiting normally”.
It is materially incomplete, suspicious, or operationally blocked.

Typical causes:

- one sync side remains missing beyond the allowed waiting window
- signature verification failed
- duplicate conflict was detected
- a required dependency exists in theory but cannot be trusted in practice

This is a **risk-aware holding state**.

It prevents the system from doing something reckless while still preserving the
record for later resolution.

---

### `rejected`

`rejected` means the record is no longer eligible for settlement.

Typical causes:

- both `rsl_report_id` and `trace_event_id` are missing
- integrity failure is terminal
- compatibility level or extension usage violates settlement policy
- duplicate conflict is irreconcilable
- the record is confirmed out of scope

This is a **final exclusion state**.

A rejected record may still be retained for audit, but it must not proceed into
automated royalty settlement.

---

## Why Three Failure-Oriented States Are Necessary

A robust bridge cannot collapse everything into “error”.

That would mix together:

- temporary incompleteness
- operational blockage
- definitive invalidity

These are not the same.

A good royalty bridge must distinguish between:

1. **not ready yet**
2. **not safe yet**
3. **not valid anymore**

That distinction is exactly what
`pending`, `blocked_or_pending`, and `rejected` provide.

---

## State Transition Philosophy

The bridge follows this principle:

> Wait when the record is incomplete.  
> Block when the record is unsafe.  
> Reject when the record is definitively invalid.

This prevents two major classes of system failure:

1. **premature settlement**
2. **silent data loss**

---

## Transition Map

```text
new ingest
   |
   v
pending
   | \
   |  \ unresolved issue / suspicion / timeout
   |   \
   |    v
   |  blocked_or_pending
   |      | \
   |      |  \ terminal invalidity
   |      |   \
   |      |    v
   |      |  rejected
   |      |
   |      \ issue resolved
   |         \
   v          v
ready ------> settled
                |
                v
             reversed

Detailed Transition Rules
1. pending -> ready

Move from pending to ready when all settlement-critical inputs are aligned.

Minimum expected conditions:

sync_ids.rsl_report_id is present
sync_ids.trace_event_id is present
idempotency.idempotency_key is present
idempotency.dedupe_hash is present
no unresolved duplicate conflict exists
verification status is acceptable under implementation policy

Typical example:

external access record arrived first
trace event arrived shortly after
no integrity issue found
settlement can now be computed

This is the normal success path.

2. pending -> blocked_or_pending

Move from pending to blocked_or_pending when the record is no longer merely waiting.

Typical triggers:

one synchronization side is still missing after the policy window
verification is stalled due to dependency failure
duplicate suspicion appears
input shape is valid, but operational confidence drops

Typical example:

rsl_report_id exists
trace_event_id is still null after the expected ingestion window
record should not remain in vague limbo forever
therefore it is upgraded to blocked_or_pending

This transition prevents “eternal pending”.

3. blocked_or_pending -> ready

Move from blocked_or_pending to ready when the blocking condition has been resolved.

Typical resolution paths:

missing trace_event_id arrives
signature verification succeeds after retry
duplicate conflict is cleared by audit or reconciliation
extension namespace review confirms compatibility

Typical example:

signature initially failed because the wrong key set was loaded
verifier retries with the correct key
verification becomes valid
record becomes safe for settlement

This is the recovery path.

4. blocked_or_pending -> rejected

Move from blocked_or_pending to rejected when the issue is no longer recoverable.

Typical triggers:

signature verification failure is confirmed as terminal
duplicate conflict is irreconcilable
settlement-critical extension usage violates policy
audit determines the record is out of scope or malformed beyond recovery

Typical example:

idempotency_key reused with materially different payload
audit confirms it is not a safe replay
record is not trustworthy enough to settle
result: rejected

This is the controlled exclusion path.

5. pending -> rejected

This transition should be rare, but it is valid when the record is immediately disqualified.

Typical triggers:

both rsl_report_id and trace_event_id are missing
record is structurally present but fails minimum sync identity requirements
compatibility policy already makes settlement impossible

Typical example:

an ingested object contains access and charge data
but neither bridge-side synchronization anchor exists
there is nothing to reconcile against
result: immediate rejected

This avoids pretending that an unsynchronizable record is merely “waiting”.

6. ready -> rejected

This transition should also be rare, but must exist.

Sometimes a record becomes ready based on available evidence, then fails later audit.

Typical triggers:

late audit reveals false synchronization
beneficiary mapping is invalid under policy
integrity issue is discovered after readiness

Typical example:

record reached readiness
later review reveals namespace misuse in settlement-critical logic
record is withdrawn from settlement eligibility
result: rejected

This protects the system from “readiness illusion”.

7. settled -> reversed

This is not a failure state in the same sense as the others, but it is part of
failure-aware operations.

Typical triggers:

charge reversal
correction request
audit invalidation of a prior settlement
recomputation after late-arriving evidence

Typical example:

settlement already executed
later, external charge is reversed
the royalty outcome must be reversed as well
result: reversed

A mature bridge must be able to admit that history sometimes needs correction.
A bridge without reversal is a bridge with amnesia.

Canonical Failure Scenarios
Scenario A: Missing Trace Arrival
Initial condition
rsl_report_id exists
trace_event_id is null
verification not yet checked
Correct state
pending
Escalation condition
trace still missing beyond policy window
Next state
blocked_or_pending
Resolution path
if trace later arrives and checks pass -> ready
if sync never becomes possible -> rejected
Scenario B: Signature Verification Failure
Initial condition
sync pair exists
signature bundle exists
verification result = failed
Correct state
blocked_or_pending or rejected
Recommended default
start with blocked_or_pending
move to rejected only after terminal confirmation
Why

A failed verification may be:

a genuine integrity failure
a wrong key problem
a temporary verifier mismatch

Do not reject too early unless the failure is known to be final.

Scenario C: Duplicate Conflict
Initial condition
same idempotency_key
same dedupe_scope
different business-significant payload or different dedupe_hash
Correct state
blocked_or_pending
Why

This is not a safe replay.
It is a conflict requiring review.

Resolution path
if confirmed as policy-approved recomputation -> ready
if confirmed as invalid or abusive duplication -> rejected
Scenario D: Both Sync Anchors Missing
Initial condition
rsl_report_id = null
trace_event_id = null
Correct state
rejected
Why

The bridge has no bilateral anchor.
Without either side of the bridge, this is not a bridge record in any meaningful settlement sense.

Scenario E: Informational or Experimental Compatibility Only
Initial condition
compatibility_level = informational or experimental
Correct state
pending, blocked_or_pending, or rejected
Not allowed
automated ready
automated settled
Why

These levels are useful for audit, observation, and rollout testing, but should
not drive core settlement decisions.

Operational Guidance
1. Do not let pending become permanent

A record should not live forever in pending.

Implementations should define timeout or escalation policies such as:

ingest SLA window
verification SLA window
sync reconciliation window

If those windows are exceeded, the record should move to blocked_or_pending.

2. Treat blocked_or_pending as meaningful

blocked_or_pending is not a vague middle zone.
It should trigger real operational behavior such as:

manual review queue
reconciliation retry
verifier retry
namespace policy review
duplicate investigation

This state exists to protect the settlement layer from uncertainty.

3. Use rejected deliberately, not emotionally

A record should be rejected only when there is a clear reason.

Good rejection practice means:

attach reason_codes
preserve evidence
preserve lineage where possible
avoid silent discard

A rejected record is still valuable as audit history.

4. Preserve recovery paths

Many blocked records are not bad records.
They are just unresolved records.

Design the system so that:

late-arriving trace can recover a blocked record
corrected verification can recover a blocked record
recomputation can be tracked without erasing prior lineage

Good systems do not confuse temporary darkness with final disappearance.

Recommended Reason Codes

The following reason codes are useful examples:

awaiting_trace_sync
awaiting_rsl_sync
verification_not_checked
signature_verification_failed
missing_sync_ids
duplicate_conflict_detected
duplicate_conflict_confirmed
namespace_policy_review_required
compatibility_not_settlement_eligible
manual_audit_required
not_eligible_for_settlement
charge_reversal_propagated
recomputation_required

Implementations may extend this vocabulary, but should keep reason codes stable
enough for audit readability.

Recommended Monitoring Metrics

Implementations should monitor at least:

count of pending records older than SLA window
count of blocked_or_pending records by reason code
percentage of records moving from blocked_or_pending -> ready
percentage of records moving from blocked_or_pending -> rejected
count of settled -> reversed
frequency of duplicate conflicts
frequency of verification failures

These metrics reveal whether the bridge is operating as a disciplined protocol
or as a polite fiction.

Final Principle

The bridge should behave like a cautious settlement system, not an optimistic logger.

That means:

incomplete is not complete
suspicious is not ready
invalid is not recoverable
correction is not shameful

The strength of v0.2 is not that it assumes everything will go well.

Its strength is that it knows how to behave when things do not.
