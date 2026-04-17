# Full RUP-Aligned Worked Examples by Domain

## Table of Contents
1. [E-Commerce: Place Order](#e-commerce-place-order)
2. [Banking: Transfer Funds](#banking-transfer-funds)
3. [SaaS: Invite Team Member](#saas-invite-team-member)
4. [Healthcare: Book Appointment](#healthcare-book-appointment)

---

## E-Commerce: Place Order

# Use-Case Specification: Place Order
**Version:** 1.0

### Revision History
| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 01/Jan/25 | 1.0 | Initial version | BA Team |

### Actors
| Role | Actor | Notes |
|------|-------|-------|
| Primary | Customer | Authenticated user completing a purchase |
| Secondary | Payment Gateway | Authorizes payment transactions |
| Secondary | Inventory Service | Reserves stock |
| Secondary | Notification Service | Sends order confirmation |

---

### 1. Brief Description
This use case describes how an authenticated customer reviews their shopping cart, provides shipping and payment details, and submits an order for fulfillment.

---

### 2. Basic Flow of Events

> The use case begins when the Customer selects "Proceed to Checkout" from the shopping cart.

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Customer | Reviews cart contents and selects "Proceed to Checkout" |
| 2 | System | Displays checkout form with shipping address and payment fields |
| 3 | Customer | Enters or confirms shipping address: street, city, postal code, and country |
| 4 | System | Validates address completeness and retrieves available shipping methods with estimated costs |
| 5 | Customer | Selects preferred shipping method |
| 6 | Customer | Enters payment details: card number, expiry date, and CVV |
| 7 | System | Validates payment detail format and completeness |
| 8 | Customer | Reviews order summary — items, shipping, taxes, total — and confirms submission |
| 9 | System | Reserves inventory for all items in cart *(See S1: Reserve Inventory)* |
| 10 | System | Submits payment authorization request to Payment Gateway |
| 11 | System | Creates order record with status "Confirmed" and a unique order ID |
| 12 | System | Sends order confirmation to customer's registered email address |

---

### 3. Alternative Flows

#### 3.1 Shipping Address

##### AF-01.1: Customer Uses Saved Address (triggered at Step 3)
> *Triggered at Step 3 when the customer selects a previously saved address.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 3a | Customer | Selects a previously saved shipping address from the list |
| 3b | System | Pre-fills address fields with saved address data |
| 3c | System | Resume at Step 4 |

#### 3.2 Discounts

##### AF-01.2: Customer Applies Discount Code (triggered at Step 8)
> *Triggered at Step 8 when the customer enters a discount code before confirming.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 8a | Customer | Enters a discount code in the provided field |
| 8b | System | Validates the discount code against active promotions |
| 8c | System | Applies the discount and updates the order total |
| 8d | System | Resume at Step 8 |

#### 3.3 Inventory

##### AF-01.3: Item Out of Stock (triggered at Step 9)
> *Triggered at Step 9 when one or more items are no longer available.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 9a | System | Detects that one or more items cannot be reserved |
| 9b | System | Notifies the customer of the unavailable items by name |
| 9c | Customer | Removes unavailable items and continues, or cancels checkout |
| 9d | System | If cancelled: releases any partial reservations. Use case ends. |

#### 3.4 Payment

##### AF-01.4: Payment Declined (triggered at Step 10)
> *Triggered at Step 10 when the Payment Gateway rejects the authorization.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 10a | System | Receives payment decline response from the Payment Gateway |
| 10b | System | Releases reserved inventory |
| 10c | System | Notifies the customer that the payment was unsuccessful |
| 10d | Customer | May re-enter payment details or cancel the order |
| 10e | System | If retrying: resume at Step 6. If cancelling: use case ends. |

---

### 4. Subflows

#### S1: Reserve Inventory

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | System | Requests inventory reservation for each line item (product ID, quantity) |
| 2 | System | Inventory Service confirms reservation for each item |
| 3 | System | Returns to calling flow at Step 10 |

---

### 5. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-01-01 | Successful Purchase | Customer pays on first attempt; order confirmed and email sent |
| SC-01-02 | Payment Declined, Retry | First card declined; customer enters a second card and succeeds |
| SC-01-03 | Item Out of Stock | Item unavailable at reservation; customer removes it and completes order |
| SC-01-04 | Discount Code Applied | Customer applies valid code; total reduced before payment |

---

### 6. Preconditions

#### 6.1 Authenticated Session
- The customer is authenticated and has an active session.

#### 6.2 Non-Empty Cart
- At least one item is present in the shopping cart.

#### 6.3 Stock Available
- All cart items are in stock at the time checkout begins.

---

### 7. Postconditions

#### 7.1 Success
- Order record exists with status "Confirmed" and a unique order ID.
- Inventory is decremented for each purchased item.
- Customer receives an email order confirmation.

#### 7.2 Failure
- No order record is created.
- No payment is charged.
- Cart contents are preserved for the customer.

---

### 8. Extension Points

#### 8.1 Apply Loyalty Discount
> *Location: After Step 8 (order review), when the customer has redeemable loyalty points.*
> The use case "Apply Loyalty Discount" <<extends>> this use case at this point.

---

### 9. Special Requirements

#### 9.1 Payment Authorization Latency
- Payment authorization (Step 10) must complete within 5 seconds under normal load.

#### 9.2 Data Security
- All payment details must be transmitted and processed over TLS 1.2 or higher.
- Card numbers must not be stored in application logs.

---

### 10. Additional Information

**Assumptions:**
- The payment gateway returns a synchronous authorization response.
- Inventory reservation is held for 15 minutes if payment fails.

**Related Use Cases:**
- `<<include>>` UC-00: Authenticate User — required precondition
- `<<extend>>` UC-05: Apply Loyalty Discount — optional at Extension Point 8.1

---

## Banking: Transfer Funds

# Use-Case Specification: Transfer Funds Between Accounts
**Version:** 1.0

### Actors
| Role | Actor | Notes |
|------|-------|-------|
| Primary | Account Holder | Authenticated individual initiating the transfer |
| Secondary | Core Banking System | Executes debit and credit operations |

---

### 1. Brief Description
This use case describes how an authenticated account holder transfers a specified amount from a source account to a destination account, either internal or external, immediately or on a scheduled date.

---

### 2. Basic Flow of Events

> The use case begins when the Account Holder selects "Transfer Funds" from the account dashboard.

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | Account Holder | Selects source account from list of eligible accounts |
| 2 | System | Displays current balance and daily transfer limit for selected account |
| 3 | Account Holder | Enters destination account number and transfer amount |
| 4 | System | Validates that the amount does not exceed available balance or daily transfer limit |
| 5 | Account Holder | Selects transfer date: immediate or a future scheduled date |
| 6 | Account Holder | Reviews transfer summary and confirms submission |
| 7 | System | Applies identity verification *(See S1: Verify Account Holder Identity)* |
| 8 | System | Debits source account and credits destination account atomically |
| 9 | System | Records transaction with a unique reference number and timestamp |
| 10 | System | Sends transfer confirmation to the account holder's registered contact |

---

### 3. Alternative Flows

#### 3.1 Transfer Timing

##### AF-02.1: Scheduled Future Transfer (triggered at Step 5)
> *Triggered at Step 5 when the account holder selects a future date.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 5a | Account Holder | Selects a future date for the transfer |
| 5b | System | Creates a scheduled transfer record with the specified date |
| 5c | System | Confirms the scheduled transfer date to the account holder. Use case ends; transfer executes on the scheduled date. |

#### 3.2 Validation Failures

##### AF-02.2: Insufficient Funds (triggered at Step 4)
> *Triggered at Step 4 when the requested amount exceeds available balance.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Detects that the requested amount exceeds the available balance |
| 4b | System | Notifies the account holder of the insufficient balance |
| 4c | Account Holder | May enter a lower amount or cancel |
| 4d | System | If retrying: resume at Step 3. If cancelling: use case ends. |

##### AF-02.3: Daily Transfer Limit Exceeded (triggered at Step 4)
> *Triggered at Step 4 when the transfer would exceed the daily limit.*

| Step | Actor / System | Action |
|------|----------------|--------|
| 4a | System | Detects that the transfer would exceed the account's daily limit |
| 4b | System | Displays the remaining available transfer limit for today |
| 4c | Account Holder | May adjust the amount or cancel |
| 4d | System | If retrying: resume at Step 3. If cancelling: use case ends. |

---

### 4. Subflows

#### S1: Verify Account Holder Identity

| Step | Actor / System | Action |
|------|----------------|--------|
| 1 | System | Prompts account holder for secondary verification (e.g., OTP sent to registered mobile) |
| 2 | Account Holder | Submits the verification code |
| 3 | System | Validates the code against the issued OTP |
| 4 | System | Returns to calling flow at Step 8 |

---

### 5. Key Scenarios

| Scenario ID | Name | Description |
|-------------|------|-------------|
| SC-02-01 | Immediate Transfer, Success | Transfer completes immediately; both accounts updated |
| SC-02-02 | Scheduled Transfer | Transfer is queued for a future date |
| SC-02-03 | Insufficient Funds | Account holder reduces amount and retries successfully |

---

### 6. Preconditions

#### 6.1 Authenticated Session
- Account holder is authenticated and the session is active.

#### 6.2 Eligible Source Account
- Account holder has at least one active account in good standing.

---

### 7. Postconditions

#### 7.1 Success
- Source account balance is reduced by the transfer amount.
- Destination account balance is increased by the transfer amount.
- Transaction record exists with a unique reference ID and timestamp.

#### 7.2 Failure
- Account balances remain unchanged.
- No transaction record is created.

---

### 8. Extension Points

#### 8.1 Apply Transfer Fee Waiver
> *Location: After Step 4, when the account holder's plan includes fee waivers for external transfers.*

---

### 9. Special Requirements

#### 9.1 Atomicity
- Debit and credit operations (Step 8) must be executed as a single atomic transaction.

#### 9.2 Audit Trail
- All transfer attempts — successful or failed — must be logged with actor ID, timestamp, and amount.

---

### 10. Additional Information

**Assumptions:**
- OTP delivery is handled by the Core Banking System's messaging service.
- Scheduled transfers are executed by a batch process at 00:01 on the scheduled date.

**Related Use Cases:**
- `<<include>>` UC-00: Authenticate Session
- `<<include>>` S1: Verify Account Holder Identity
