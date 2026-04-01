# BPMN Patterns → System Function Mappings

## Common BPMN Elements and Their System Counterparts

---

### 1. Start Events

| BPMN Pattern           | System Function                                              |
|------------------------|--------------------------------------------------------------|
| Timer Start Event      | Scheduled Job / Cron trigger                                 |
| Message Start Event    | Webhook / Event consumer / API endpoint                      |
| Signal Start Event     | Internal event bus subscription                              |
| Manual Start           | UI-triggered action with authentication check                |

---

### 2. Tasks / Activities

| BPMN Task Type         | System Function                                              |
|------------------------|--------------------------------------------------------------|
| User Task              | Form / UI screen with validation + save action               |
| Service Task           | API call / Background job / Integration endpoint             |
| Script Task            | Internal computation / Data transformation function          |
| Send Task              | Notification service (email, SMS, push, webhook)             |
| Receive Task           | Async message listener / Polling endpoint                    |
| Manual Task            | Human-only step → system records completion + timestamp      |
| Business Rule Task     | Rules engine evaluation / Conditional branching logic        |

---

### 3. Gateways / Decisions

| BPMN Gateway           | System Implementation                                        |
|------------------------|--------------------------------------------------------------|
| Exclusive (XOR)        | if/else branch → single Business Rule                        |
| Inclusive (OR)         | Multi-condition evaluation → multiple parallel branches      |
| Parallel (AND)         | Fork/join pattern → concurrent execution + synchronization   |
| Event-based            | State machine awaiting one of N events                       |
| Complex                | Rules engine with weighted conditions                        |

**Key rule**: Every gateway → at least one BR-XXX entry in the Business Rules section.

---

### 4. Events (Intermediate)

| BPMN Event             | System Function                                              |
|------------------------|--------------------------------------------------------------|
| Timer Intermediate     | Scheduled reminder / Escalation trigger / SLA monitor        |
| Message Intermediate   | Callback handler / Integration acknowledgment                |
| Error Intermediate     | Exception handler + compensating transaction                 |
| Escalation             | Supervisor notification + reassignment workflow              |
| Compensation           | Rollback / Undo / Saga compensation step                     |

---

### 5. End Events

| BPMN End Event         | System Function                                              |
|------------------------|--------------------------------------------------------------|
| None End               | Status update to terminal state + audit log                  |
| Message End            | Outbound notification / downstream system update             |
| Error End              | Error state persistence + alert to operations team           |
| Terminate End          | Cancel all in-flight sub-processes + cleanup                 |

---

### 6. Swimlanes / Pools

| BPMN Element           | System Mapping                                               |
|------------------------|--------------------------------------------------------------|
| Pool                   | System boundary / Microservice boundary                      |
| Lane (same pool)       | Role / Permission group within the system                    |
| Message flow (between pools) | API contract / Integration interface                  |

---

### 7. Data Objects

| BPMN Element           | System Mapping                                               |
|------------------------|--------------------------------------------------------------|
| Data Object            | Entity / Domain Object                                       |
| Data Store             | Database table / Repository                                  |
| Data Input/Output      | DTO (Data Transfer Object) / API request/response schema     |

---

## Common Process Patterns and System Architectures

### Approval Workflow Pattern

```
Process: Submit → Review → Approve/Reject → Notify
System:
  FR: Submit entity (create + validate)
  FR: Route to approver queue (role-based assignment)
  FR: Record approval decision (audit trail)
  FR: Notify submitter of outcome (async notification)
  FR: Handle escalation if not actioned within SLA (timer-based)
  BR: Define approval authority matrix by entity type/value
  Entity: ApprovalRequest {id, entity_ref, submitted_by, assigned_to, status, decision, timestamp}
```

### Data Entry + Validation Pattern

```
Process: Enter data → Validate → Save → Confirm
System:
  FR: Present form with field constraints
  FR: Execute client-side validation (immediate feedback)
  FR: Execute server-side validation (authoritative)
  FR: Persist validated data with version/timestamp
  FR: Return confirmation with reference ID
  BR: Define validation rules per field (format, range, mandatory, uniqueness)
```

### Notification / Escalation Pattern

```
Process: Action occurs → Wait → If no response → Escalate
System:
  FR: Send initial notification on trigger event
  FR: Start SLA timer on notification send
  FR: Check timer expiry (scheduled job)
  FR: Escalate to next role if timer expires without acknowledgment
  FR: Log all notification events with timestamps
  BR: Define SLA windows per notification type and priority
```
