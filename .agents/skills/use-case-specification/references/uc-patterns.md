# Common Use Case Patterns

## Table of Contents
1. [CRUD Patterns](#crud-patterns)
2. [Authentication & Authorization](#authentication--authorization)
3. [Search & Filter](#search--filter)
4. [Notification & Messaging](#notification--messaging)
5. [Payment & Transaction](#payment--transaction)
6. [File & Document Management](#file--document-management)
7. [Reporting & Export](#reporting--export)

---

## CRUD Patterns

### Create (Submit / Register / Add)
- **Trigger**: Actor initiates creation action
- **Main Flow**: Actor fills form → System validates inputs → System persists record → System confirms
- **Alternate**: Duplicate detected → System warns → Actor decides
- **Exception**: Validation fails → System highlights errors → Actor corrects

### Read / View
- **Trigger**: Actor requests to see a record
- **Main Flow**: Actor requests view → System retrieves data → System displays
- **Exception**: Record not found → System notifies → Use case ends

### Update (Edit / Modify)
- **Precondition**: Record exists and actor has permission
- **Main Flow**: Actor opens record → System displays current data → Actor modifies → System validates → System saves
- **Exception**: Concurrent edit conflict → System detects → System prompts actor to resolve

### Delete (Remove / Archive)
- **Precondition**: Record exists, actor authorized
- **Alternate**: Actor cancels → System discards → Use case ends
- **Exception**: Record has dependencies → System blocks deletion → System explains constraint

---

## Authentication & Authorization

### Key Rule
> "Authenticate User" and "Authorize Action" are almost always `<<include>>` use cases, not standalone goals.
> Use them as building blocks inside goal-driven use cases.

### UC Template: Authenticate User (<<include>>)
- **Trigger**: Called by another use case that requires identity verification
- **Main Flow**: System prompts credentials → Actor submits → System verifies → System grants session
- **Exception**: Max attempts exceeded → System locks account → System notifies actor

---

## Search & Filter

### Pattern Notes
- Always include: empty result state, too-many-results state, filter combinations
- Distinguish between "search" (intent-driven, keyword) and "filter" (criteria-driven, structured)

### Key Alternate Flows
- AF: No results found → System displays empty state with suggestions
- AF: Results exceed display limit → System paginates → Actor navigates
- AF: Actor refines search → System re-executes with new criteria

---

## Notification & Messaging

### Pattern Notes
- Notifications are usually secondary actor behavior (the notification service)
- Keep sending notifications as a system response step, not a standalone use case
- Exception: "Manage Notification Preferences" IS a valid standalone UC

---

## Payment & Transaction

### Critical Exception Flows to Always Include
- EF: Payment declined → System notifies → Actor may retry or cancel
- EF: Payment gateway timeout → System retries N times → System notifies of failure
- EF: Duplicate transaction detected → System blocks → System notifies

### Postconditions to Always Include (Success)
- Transaction record created with reference ID
- Funds reserved or captured
- Confirmation sent to actor

---

## File & Document Management

### Upload Pattern
- **Alternate**: File type not supported → System rejects → System lists accepted types
- **Alternate**: File exceeds size limit → System rejects → System states limit
- **Exception**: Upload interrupted → System discards partial → System notifies

### Download Pattern
- **Exception**: File no longer available → System notifies → Suggests alternatives

---

## Reporting & Export

### Pattern Notes
- Always include: data range selection, format selection (PDF/CSV/Excel)
- Alternate: No data for selected range → System displays empty report with guidance
- Exception: Report generation times out → System notifies → System offers async delivery
