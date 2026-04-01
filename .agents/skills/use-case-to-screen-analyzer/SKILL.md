---
name: use-case-to-screen-analyzer
description: >
  Transforms Use Case Specifications, functional requirements, and business process flows into a
  structured, validated screen model. Behaves as a senior Requirement Engineer with deep
  UI/interaction modeling judgment.

  ALWAYS trigger when the user: provides a use case spec and wants screens or pages inferred;
  asks "what screens do I need?"; wants to validate whether something is a screen, modal,
  component, or state; needs a screen inventory or screen spec from requirements; asks to map
  use cases to application pages; mentions "screen model", "page list", "screen list", or
  "interaction boundary"; wants wireframe scope or implementation scope from use cases; has a
  screen list needing validation; asks "how many screens does this use case need?" or "is this a
  screen or a popup?". Use it even for partial use cases — the skill will ask clarifying questions
  or proceed with explicit assumptions. Prevents over-fragmentation (too many fake screens) and
  under-specification (missing real screens).
---

# Use Case to Screen Analyzer

A production-grade skill for inferring, classifying, specifying, and validating application
screens from Use Case Specifications and functional requirement documents.

---

## Core Mental Model

Before analyzing any input, internalize these non-negotiable truths:

| Principle | Rule |
|---|---|
| A screen is not a drawing | A screen is a complete UI interaction unit with its own purpose, data scope, controls, and navigation |
| A screen is identified by interaction boundary | A group of inputs is not automatically a screen |
| UI State ≠ Screen | Loading, empty, error, and confirm variants are states of a screen unless they create a new interaction context |
| UI Component ≠ Screen | Tables, dropdowns, filters, cards, tabs, badges, and dialogs are components, not screens |
| 1 Use Case ≠ 1 Screen | One use case may span many screens. One screen may serve many use cases. |

---

## Execution Workflow

Follow these phases in order for every analysis:

### Phase 1 — Parse the Input

Read the full input and extract:

- [ ] All **actors** (human or system)
- [ ] All **use case names and IDs**
- [ ] All **main flow steps** (numbered)
- [ ] All **alternate flow steps** (A1, A2 …)
- [ ] All **exception flow steps** (E1, E2 …)
- [ ] All **preconditions** and **postconditions**
- [ ] All **business rules** referenced
- [ ] All **navigation actions** (user navigates to, system redirects to, etc.)
- [ ] All **data sets** required (what the user sees / inputs)
- [ ] All **actor decisions** (user selects, user confirms, user reviews …)

If the input is ambiguous or incomplete, jump to **Phase 6 — Ambiguity Handling** before continuing.

---

### Phase 2 — Detect Interaction Boundaries

For each extracted step, ask:

1. Does the user **enter a new context** (goal, task, or role)?
2. Is there a **meaningful navigation** event (go to, navigate to, redirect, open page)?
3. Does the UI present a **different data set or action set** than the previous step?
4. Could this step be **documented independently** with its own acceptance criteria?
5. Does a **back path or route** make sense here?

A step that answers YES to 3 or more of these is a strong **screen candidate**.
A step that answers YES to 1–2 may be a **state or component**.
A step that answers YES to 0 is an **action or micro-interaction**.

**Context switch triggers (strong screen boundary signals):**

| From | To |
|---|---|
| List view | Detail view |
| View mode | Edit mode |
| Edit form | Review / Confirm page |
| Cart / basket | Checkout |
| Form entry | Submission confirmation |
| Browse | Transaction |
| Unauthenticated | Authenticated dashboard |
| Main task | Approval sub-task |

---

### Phase 3 — Classify Each Candidate

For every detected boundary, apply the classification matrix:

#### Classify as SCREEN when:
- It is a **full page** or **app page**
- It has its own route or stable entry point
- It serves a clear and distinct **business or user goal**
- It presents a **distinct data set** and a **distinct action set**
- A user can meaningfully **navigate to and from** it
- It can be independently written as acceptance criteria
- It represents a **change in user context or role**
- It is a **complex multi-step modal** (wizard-style) that fully blocks the main flow and introduces a new task context

#### Classify as UI STATE when:
- Loading / skeleton screen
- Empty state (no results, no data)
- Inline error / field validation error
- System error / service unavailable
- Success toast or inline success banner
- Delete confirmation (simple yes/no)
- Submit-in-progress (spinner on button)
- Retry prompt within the same page context

#### Classify as UI COMPONENT when:
- Data table or grid
- Filter bar or search bar
- Date picker
- Dropdown or select
- Side menu or navigation rail
- Accordion or collapsible section
- Tab strip / tab panel
- Inline edit control
- Button group
- Badge or status chip
- Card inside a page
- Read-only detail panel embedded in a screen

#### Classify as ACTION / MICRO-INTERACTION when:
- Single-field inline update
- Simple sort or filter toggle
- Small confirmation dialog (non-blocking)
- Minor navigation within the same screen
- Tooltip or hover detail
- Row-level action (delete row, expand row)

---

### Phase 4 — Validate Every Screen Candidate

Before finalizing a screen, verify it passes this checklist:

- [ ] Has a **clear user goal**
- [ ] Has **visible data** (read-only fields, records, or summaries)
- [ ] Has **at least one action** (button, link, submit, navigate)
- [ ] Has **traceable origin** in one or more use-case steps
- [ ] Is **distinguishable** from adjacent screens (not a duplicate)
- [ ] Is **not inflated** from a component or state
- [ ] Has a clear **navigation in** path
- [ ] Has a clear **navigation out** path

Fail on 3 or more criteria → reclassify as state or component.

---

### Phase 5 — Name Each Screen

Apply this naming convention consistently:

```
{Role/Context}_{Entity}_{Function}_Screen
```

Examples:
- `Customer_OrderList_Screen`
- `Customer_OrderDetail_Screen`
- `Admin_ApprovalReview_Screen`
- `Payment_Confirmation_Screen`
- `Auth_Login_Screen`
- `Vendor_ProductEdit_Screen`
- `Manager_ReportDashboard_Screen`

Rules:
- Use a role/context prefix when the screen is actor-specific
- Use `Entity + Function` pattern (noun + verb-noun)
- Avoid vague names: ~~Main Screen~~, ~~Popup~~, ~~Form Page~~
- Use `PascalCase` with underscores as separators
- Suffix every screen name with `_Screen`

---

### Phase 6 — Ambiguity Handling

If any of the following are missing or unclear, **ask before producing the final output**:

| Missing Element | Clarifying Question |
|---|---|
| System boundary | "Does this use case cover a web app, mobile app, or both?" |
| Actor definition | "Who is the primary actor — a logged-in customer, admin, or guest?" |
| Modal nature | "Is this dialog blocking (user must act) or non-blocking (can dismiss)?" |
| Navigation intent | "Does the user navigate to a new page, or does content appear inline?" |
| Alternate flow destination | "Where does the user go when the alternate flow completes?" |
| Exception recovery | "When the error occurs, does the user stay on the same page or go somewhere new?" |
| Multi-step modal | "Is this a multi-step wizard or a single-step confirmation?" |

**Fast-path rule:** If the user explicitly requests a fast or preliminary analysis, proceed with assumptions but mark every assumption in the **Ambiguity Log** section of the output.

---

### Phase 7 — Produce the Output

Deliver all six sections below. Do not omit any section.

---

## Output Template

---

### 1. Executive Screen Summary

```
Total Inferred Screens:     [N]
Use Cases Covered:          [list]
States Identified:          [N]
Components Identified:      [N]

Screen Count Rationale:
[2–4 sentences explaining why this count is correct, neither inflated nor under-specified]

Key Assumptions:
- [Assumption 1]
- [Assumption 2]
```

---

### 2. Screen Inventory

For each screen, produce this block:

```
---
Screen ID:              SCR-[NNN]
Screen Name:            {Role}_{Entity}_{Function}_Screen
Purpose:                [One sentence — what the user accomplishes here]
Related Use Case(s):    [UC-ID: Step references]
Interaction Boundary:   [Context switch / Navigation boundary / Goal unit boundary]
Why it is a Screen:     [Justification referencing interaction logic and use-case steps]
Why NOT a Component:    [What distinguishes it from a state, modal, or component]
---
```

---

### 3. Screen Specifications

For each screen in the inventory:

```
=== [Screen Name] ===

Purpose / User Goal:
  [What the user is trying to accomplish]

Read-only Data Shown:
  - [Field or data element]
  - [Field or data element]

Inputs:
  - [Input field: type, validation note]
  - [Input field: type, validation note]

Primary Actions:
  - [Action label → destination or outcome]

Secondary Actions:
  - [Action label → destination or outcome]

Business Rules:
  - [BR-ID or description]

Validation Rules:
  - [Field: rule]

Navigation In:
  - From [Screen Name] via [action or trigger]

Navigation Out:
  - On [action] → [Screen Name or outcome]

Related UI States:
  - [State name: trigger condition]
  - [State name: trigger condition]

Related Components:
  - [Component name]
  - [Component name]
```

---

### 4. Mapping Matrix

| Use Case ID | Flow Step | Step Description | Classification | Screen / State / Component | Notes |
|---|---|---|---|---|---|
| UC-01 | Main-3 | User selects order | Screen | Customer_OrderDetail_Screen | Context switch from list |
| UC-01 | Main-4 | System loads order data | State | Loading_State (of OrderDetail) | Same screen, loading variant |
| UC-01 | Main-5 | User clicks Edit | Screen | Customer_OrderEdit_Screen | Edit boundary |
| UC-01 | Alt-1 | Order not found | State | Empty_State (of OrderDetail) | Not a new screen |
| UC-01 | Exc-1 | System error | State | Error_State (of OrderDetail) | Same page, error variant |

*(Extend for all use case steps)*

---

### 5. Ambiguity Log

```
[ ] Item 1: [Step reference] — [What is unclear] — [Assumption made if fast-path]
[ ] Item 2: [Step reference] — [What is unclear] — [Assumption made if fast-path]
```

If no ambiguities exist, state: "No ambiguities detected. Full specification based on provided input."

---

### 6. Validation Findings

```
Possible Missing Screens:
  - [Screen description: inferred from alternate/exception flow not explicitly stated]

Possible Redundant Screens:
  - [Screen A and Screen B appear to serve the same goal — consider merging]

Over-fragmentation Risks:
  - [Screen X may be a state of Screen Y rather than an independent screen]

Under-specification Risks:
  - [Use case step N references a destination not covered by any inferred screen]
```

---

## Internal Classification Examples

These examples illustrate correct vs. incorrect classification decisions.

### Example A — List to Detail (Correct Split)

**Use case step:** "User clicks on an order row to view full order details."

- `Customer_OrderList_Screen` → SCREEN ✅ (browsing context, list data, filter/search actions)
- `Customer_OrderDetail_Screen` → SCREEN ✅ (new context, full order data, approve/cancel/edit actions)

**Why not merge:** Different data scopes, different action sets, meaningful back navigation, independently testable.

---

### Example B — Loading State (Do NOT create a screen)

**Use case step:** "System retrieves order data. A spinner is shown while loading."

- Loading spinner → UI STATE of `Customer_OrderDetail_Screen` ✅
- NOT a new screen ❌

**Why not a screen:** No new user goal, no new data scope, no navigation boundary, not independently documentable.

---

### Example C — Confirm Delete Dialog (Do NOT create a screen)

**Use case step:** "User clicks Delete. A confirmation dialog asks 'Are you sure?' with Confirm/Cancel."

- Confirm dialog → UI STATE or COMPONENT ✅
- NOT a screen ❌

**Why not a screen:** Single-action decision, no new data, non-blocking in most cases, no navigation route.

**Exception:** If the deletion triggers a complex multi-step impact review (e.g., "deleting this account will affect 14 sub-records — review them here"), that IS a screen boundary.

---

### Example D — Multi-Step Checkout (Correct Split)

**Use case:** "User completes checkout via cart review → address entry → payment entry → order confirmation."

- `Cart_Review_Screen` → SCREEN ✅
- `Checkout_Address_Screen` → SCREEN ✅
- `Checkout_Payment_Screen` → SCREEN ✅
- `Order_Confirmation_Screen` → SCREEN ✅

Each step has a distinct data set, action set, and represents a meaningful transaction boundary. Back navigation is meaningful at each step.

---

### Example E — Tab Panels (Do NOT create screens)

**Use case step:** "The profile page has tabs: Personal Info, Security, Notifications."

- `User_Profile_Screen` → ONE SCREEN ✅
- Tabs → UI COMPONENTS within that screen ✅
- NOT three separate screens ❌ (unless each tab has route-level navigation, full data reload, and independent acceptance criteria)

---

## Advanced Analysis Capabilities

When the input warrants it, apply these higher-order operations:

### Refactor Over-Broad Screens
If a proposed screen covers too many unrelated goals, split it by interaction boundary.
Signal: "This screen has 5+ distinct actions with no clear primary goal."

### Merge Duplicate Screens
If two screen candidates serve the same goal with the same data, merge them.
Signal: "Screen A and Screen B both show order details with minor display differences."

### Infer Implied Screens
Alternate and exception flows often imply screens not mentioned in the main flow.
- "User resets password" → implies `Auth_PasswordReset_Screen` and `Auth_PasswordReset_Confirmation_Screen`
- "Admin is notified of rejection" → implies `Admin_Notification_Screen` or `Admin_Dashboard_Screen` with notification state

### Modal Screen Detection
A modal is a SCREEN if:
- It is wizard-style (multi-step)
- It fully blocks the main flow with no return until completed
- It has a distinct task goal separate from the parent page
- It would be independently described in acceptance criteria

A modal is a STATE or COMPONENT if:
- It requires a single decision (yes/no)
- It shows a brief success or error message
- It is a simple form that submits inline without navigation

---

## Quality Rules (Self-Check Before Delivering Output)

Before finalizing output, verify:

- [ ] Every screen is justified by interaction logic, not just by the presence of inputs
- [ ] No state (loading, empty, error) has been promoted to a screen without valid justification
- [ ] No component (table, dropdown, filter) has been promoted to a screen
- [ ] Every use-case step is accounted for in the mapping matrix
- [ ] No two screens in the inventory serve identical goals
- [ ] All alternate and exception flows have been analyzed for implied screens
- [ ] The screen count is defensible to a UX, development, or QA audience
- [ ] All ambiguities are logged
- [ ] All validation findings are surfaced

---

## Output Style

- Structured and precise
- Terminology-aware (RE, BA, UX, Dev vocabulary)
- Analytical, not verbose
- Every claim traceable to a use-case step
- Suitable for handoff to design, development, and QA teams
