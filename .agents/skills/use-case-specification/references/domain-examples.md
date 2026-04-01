# Full Worked Examples by Domain

## Table of Contents

- [Full Worked Examples by Domain](#full-worked-examples-by-domain)
  - [Table of Contents](#table-of-contents)
  - [E-Commerce: Place Order](#e-commerce-place-order)
    - [UC-01: Place Order](#uc-01-place-order)
  - [Banking: Transfer Funds](#banking-transfer-funds)
    - [UC-02: Transfer Funds Between Accounts](#uc-02-transfer-funds-between-accounts)
  - [SaaS: Manage Team Members](#saas-manage-team-members)
    - [UC-03: Invite Team Member](#uc-03-invite-team-member)
  - [Healthcare: Book Appointment](#healthcare-book-appointment)
    - [UC-04: Book Appointment](#uc-04-book-appointment)

---

## E-Commerce: Place Order

### UC-01: Place Order

**Brief Description**
The customer selects items, provides shipping and payment details, and submits an order for fulfillment.

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Customer | Authenticated user initiating the purchase |
| Secondary | Payment Gateway | Processes payment authorization |
| Secondary | Inventory Service | Validates and reserves stock |

**Preconditions**

- Customer is authenticated
- At least one item is in the shopping cart
- Items are in stock

**Trigger**
Customer proceeds to checkout from the shopping cart.

**Main Flow**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Customer | Reviews cart contents and selects "Proceed to Checkout" |
| 2 | System | Displays checkout form with shipping address and payment fields |
| 3 | Customer | Enters or confirms shipping address |
| 4 | System | Calculates shipping options and costs based on address |
| 5 | Customer | Selects preferred shipping method |
| 6 | Customer | Enters payment details |
| 7 | System | Validates payment details format and completeness |
| 8 | Customer | Reviews order summary and confirms submission |
| 9 | System | Reserves inventory for all items in cart |
| 10 | System | Submits payment authorization request |
| 11 | System | Creates order with status "Confirmed" |
| 12 | System | Sends order confirmation to customer's registered email |

**Alternate Flows**

> AF-01.A: Customer Uses Saved Address (triggered at Step 3)

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Customer | Selects a previously saved shipping address |
| 3b | System | Pre-fills address fields with saved data |
| 3c | — | Resumes at Step 4 |

> AF-01.B: Apply Discount Code (triggered at Step 8)

| Step | Actor / System | Action |
|------|----------------|--------|
| 8a | Customer | Enters a discount code before confirming |
| 8b | System | Validates the discount code |
| 8c | System | Applies discount and updates order total |
| 8d | — | Resumes at Step 8 |

**Exception Flows**

> EF-01.1: Item Out of Stock (triggered at Step 9)

| Step | Actor / System | Action |
|------|----------------|--------|
| 9a | System | Detects that one or more items are no longer available |
| 9b | System | Notifies customer of unavailable items |
| 9c | Customer | Removes unavailable items or cancels checkout |
| 9d | System | If cancelled, releases any partial reservations and ends use case |

> EF-01.2: Payment Declined (triggered at Step 10)

| Step | Actor / System | Action |
|------|----------------|--------|
| 10a | System | Receives payment decline response from gateway |
| 10b | System | Releases reserved inventory |
| 10c | System | Notifies customer that payment was unsuccessful |
| 10d | Customer | May re-enter payment details or cancel the order |

**Postconditions**

*Success:*

- Order record exists with status "Confirmed" and a unique order ID
- Inventory decremented for each ordered item
- Customer receives email confirmation

*Failure:*

- No order is created
- No charge is applied to the customer
- Cart contents are preserved

**Business Rules**

- BR-1: Discount codes cannot be combined unless explicitly flagged as stackable
- BR-2: Inventory must be reserved before payment is processed
- BR-3: Orders cannot be placed for items with zero stock

---

## Banking: Transfer Funds

### UC-02: Transfer Funds Between Accounts

**Brief Description**
An authenticated account holder initiates a transfer of a specified amount from one of their accounts to another account, either internal or external.

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Account Holder | Authenticated user initiating the transfer |
| Secondary | Core Banking System | Executes the debit and credit transactions |

**Preconditions**

- Account holder is authenticated and session is active
- Source account is active and in good standing
- Account holder has at least one eligible source account

**Trigger**
Account holder selects "Transfer Funds" from the account dashboard.

**Main Flow**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Account Holder | Selects source account from list of eligible accounts |
| 2 | System | Displays current balance and daily transfer limit for selected account |
| 3 | Account Holder | Specifies destination account and transfer amount |
| 4 | System | Validates that amount does not exceed available balance and daily limit |
| 5 | Account Holder | Selects transfer date (immediate or scheduled) |
| 6 | Account Holder | Reviews transfer summary and confirms |
| 7 | System | Applies applicable security verification (see <<include>> Verify Identity) |
| 8 | System | Debits source account and credits destination account atomically |
| 9 | System | Records transaction with unique reference number |
| 10 | System | Sends transfer confirmation to account holder |

**Alternate Flows**

> AF-02.A: Scheduled Future Transfer (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Account Holder | Selects a future date for the transfer |
| 5b | System | Creates a scheduled transfer record |
| 5c | System | Confirms scheduled transfer and displays the scheduled date |
| 5d | — | Use case ends; transfer executes on scheduled date |

**Exception Flows**

> EF-02.1: Insufficient Funds (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Detects that requested amount exceeds available balance |
| 4b | System | Notifies account holder of insufficient funds |
| 4c | Account Holder | May enter a lower amount or cancel the transfer |

> EF-02.2: Daily Limit Exceeded (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Detects that transfer would exceed daily limit |
| 4b | System | Displays remaining available limit for today |
| 4c | Account Holder | May adjust amount or cancel |

**Postconditions**

*Success:*

- Source account balance reduced by transfer amount
- Destination account balance increased by transfer amount
- Transaction record created with reference ID and timestamp

*Failure:*

- Account balances remain unchanged
- No transaction record is created

---

## SaaS: Manage Team Members

### UC-03: Invite Team Member

**Brief Description**
A workspace administrator invites a new member to join the workspace by sending an invitation to their email address and assigning an initial role.

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Administrator | Has permission to manage team membership |
| Secondary | Email Service | Delivers the invitation |

**Preconditions**

- Administrator is authenticated
- Workspace has not reached its member seat limit
- Administrator has "Manage Members" permission

**Trigger**
Administrator selects "Invite Member" from the Team Settings page.

**Main Flow**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Administrator | Enters the email address of the person to invite |
| 2 | System | Validates email format and checks if address is already a member |
| 3 | Administrator | Selects a role to assign to the new member |
| 4 | System | Displays invitation preview with role and access summary |
| 5 | Administrator | Confirms and sends the invitation |
| 6 | System | Creates a pending invitation record with expiry of 7 days |
| 7 | System | Sends invitation email with a secure join link |
| 8 | System | Displays confirmation and the pending invitee in the member list |

**Exception Flows**

> EF-03.1: Email Already a Member (triggered at Step 2)

| Step | Actor / System | Action |
|------|----------------|--------|
| 2a | System | Detects that email is already associated with an active member |
| 2b | System | Notifies administrator that this person is already a member |
| 2c | — | Use case ends without sending invitation |

> EF-03.2: Seat Limit Reached (triggered at Step 5)

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | System | Detects that workspace is at maximum seat capacity |
| 5b | System | Notifies administrator and presents upgrade options |
| 5c | — | Use case ends |

**Postconditions**

*Success:*

- Pending invitation record exists, expiring in 7 days
- Invitee receives an email with a join link
- Administrator sees invitee in "Pending" state in the member list

*Failure:*

- No invitation is created or sent
- Workspace membership is unchanged

---

## Healthcare: Book Appointment

### UC-04: Book Appointment

**Brief Description**
A patient books a medical appointment with a specific provider by selecting an available time slot.

**Actors**

| Role | Actor | Notes |
|------|-------|-------|
| Primary | Patient | Authenticated individual seeking care |
| Secondary | Provider | The clinician whose schedule is queried |
| Secondary | Notification Service | Sends confirmations and reminders |

**Preconditions**

- Patient has an active account and is authenticated
- Patient has selected or has an assigned provider

**Trigger**
Patient selects "Book Appointment" from the patient portal.

**Main Flow**

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | Patient | Selects appointment type (e.g., routine, follow-up, urgent) |
| 2 | System | Displays available providers eligible for the selected appointment type |
| 3 | Patient | Selects preferred provider |
| 4 | System | Retrieves and displays available time slots for the next 30 days |
| 5 | Patient | Selects a preferred date and time slot |
| 6 | System | Displays appointment summary for confirmation |
| 7 | Patient | Confirms the booking |
| 8 | System | Reserves the selected time slot in the provider's schedule |
| 9 | System | Creates appointment record with confirmation number |
| 10 | System | Sends confirmation to patient via email and in-app notification |

**Alternate Flows**

> AF-04.A: No Slots Available with Preferred Provider (triggered at Step 4)

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Detects no available slots within 30 days for selected provider |
| 4b | System | Notifies patient and offers to show alternative providers |
| 4c | Patient | Accepts suggestion or cancels booking |

**Exception Flows**

> EF-04.1: Slot Taken During Booking (triggered at Step 8)

| Step | Actor / System | Action |
|------|----------------|--------|
| 8a | System | Detects selected slot was booked by another patient concurrently |
| 8b | System | Notifies patient that slot is no longer available |
| 8c | System | Returns patient to available slots view (Step 4) |

**Postconditions**

*Success:*

- Appointment record exists with unique confirmation number
- Provider's slot is marked as reserved
- Patient receives confirmation via email and notification

*Failure:*

- No appointment is created
- Provider's schedule is unchanged
