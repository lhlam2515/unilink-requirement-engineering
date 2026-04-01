# Functional Requirement Templates for Common System Types

## How to Use This File

Load specific sections relevant to the system type being designed.
These templates provide starter FRs that MUST be adapted to the specific context — never copy verbatim.

---

## 1. Approval Workflow System

### Core FRs

```
FR-T01: Submit Item for Approval
  The system SHALL accept a submission request from an authorized Actor, validate all
  required fields, assign a unique reference ID, record the submitter identity and
  timestamp, and route the submission to the designated approver queue.

FR-T02: Assign to Approver
  The system SHALL automatically assign submitted items to the appropriate approver
  based on [routing criteria: role, value threshold, category, etc.]. If no approver
  is available, the system SHALL escalate to the backup approver defined in BR-XXX.

FR-T03: Record Approval Decision
  The system SHALL record the approver's decision (APPROVED / REJECTED / RETURNED),
  the rationale (if required by BR-XXX), the decision timestamp, and the approver identity.
  The system SHALL prevent modification of a finalized decision without an audit trail entry.

FR-T04: Notify Submitter
  The system SHALL notify the submitter via [channel] within [time] of a decision being
  recorded, including the decision outcome, reference ID, and (if rejected) the stated reason.

FR-T05: Escalate on SLA Breach
  The system SHALL monitor the time elapsed since assignment. If no decision is recorded
  within [SLA window] defined in BR-XXX, the system SHALL escalate to [escalation role]
  and update the item status to ESCALATED.
```

---

## 2. Data Pipeline / ETL System

### Core FRs

```
FR-T10: Ingest Source Data
  The system SHALL accept data from [source type] via [protocol/format], validate the
  structure against the defined schema, and reject records that fail validation with
  an error code and reason. Valid records SHALL be queued for processing.

FR-T11: Transform Data
  The system SHALL apply transformation rules [BR-XXX] to each valid record, producing
  output records conforming to the target schema. Transformation failures SHALL be
  logged with the source record ID and failure reason.

FR-T12: Load to Target
  The system SHALL write transformed records to [target system], handling duplicates
  per BR-XXX (INSERT / UPSERT / REJECT), and return a load summary (records processed,
  succeeded, failed).

FR-T13: Report Pipeline Results
  The system SHALL generate a processing report for each pipeline run including:
  run_id, start_time, end_time, records_received, records_processed, records_failed,
  and a reference to the error log.
```

---

## 3. Inventory / Stock Management System

### Core FRs

```
FR-T20: Record Stock Movement
  The system SHALL record every stock movement (inbound / outbound / adjustment) with:
  item_id, quantity, movement_type, reference_document, actor, and timestamp.
  The system SHALL update the on-hand quantity atomically.

FR-T21: Enforce Minimum Stock Rule
  The system SHALL evaluate on-hand quantity after every outbound movement. If quantity
  falls below the reorder threshold defined in BR-XXX, the system SHALL automatically
  create a replenishment alert and optionally trigger a purchase order per BR-XXX.

FR-T22: Reserve Stock for Orders
  The system SHALL reserve (soft-lock) stock against a confirmed order, reducing
  available quantity without reducing on-hand quantity. Reserved stock SHALL be
  released if the order is cancelled within [time window].
```

---

## 4. User Authentication & Authorization

### Core FRs (infer and add these if missing from the process)

```
FR-T30: Authenticate User [ASSUMED if not stated]
  The system SHALL verify actor identity using [mechanism] before granting access to
  any system function. Failed authentication attempts SHALL be logged. After [N]
  consecutive failures, the account SHALL be locked per BR-XXX.

FR-T31: Authorize Action [ASSUMED if not stated]
  The system SHALL evaluate the authenticated actor's role against the required
  permission for each system function before execution. Unauthorized attempts SHALL
  be rejected with error code 403 and logged with actor identity and attempted action.

FR-T32: Maintain Audit Log [ASSUMED if not stated]
  The system SHALL record every state-changing operation with: actor_id, action,
  entity_type, entity_id, before_state, after_state, and timestamp. Audit logs
  SHALL be immutable and retained for [period] per BR-XXX.
```

---

## 5. Notification System

### Core FRs

```
FR-T40: Send Triggered Notification
  The system SHALL dispatch a notification when a defined trigger event occurs,
  using the channel (email / SMS / push / in-app) specified in the actor's
  notification preferences or the system default per BR-XXX.

FR-T41: Track Notification Delivery
  The system SHALL record the dispatch timestamp, delivery status (SENT / DELIVERED /
  FAILED), and any delivery error for each notification. Failed notifications SHALL
  be retried [N] times per BR-XXX before being marked PERMANENTLY_FAILED.

FR-T42: Support Notification Preferences
  The system SHALL allow actors to configure their notification channel preferences
  and opt-out of non-mandatory notification categories. Mandatory notifications
  (defined in BR-XXX) SHALL not be suppressible.
```

---

## Inferred Requirements Checklist

When transforming any process, always check whether these universal FRs are needed and add them
with `[ASSUMED]` tag if the process did not explicitly state them:

- [ ] Authentication before any actor-facing function
- [ ] Authorization check before state-changing function
- [ ] Audit log for all state changes
- [ ] Input validation for all data entry points
- [ ] Error handling and user feedback for all failure paths
- [ ] Notification on key state transitions (submit, approve, reject, complete)
- [ ] SLA timer and escalation for human-dependent steps
- [ ] Soft-delete (not hard-delete) for business-critical entities
- [ ] Idempotency for externally triggered operations
- [ ] Rate limiting for API-exposed functions
