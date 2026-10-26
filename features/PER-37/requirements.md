# Requirements — PER-37

**Ticket:** Show a retry action when a pending payment stays pending  
**Type:** Feature  
**Repo:** `app` (Python)

---

## Product Scope

Users cannot distinguish a pending payment from a failed one. When a payment remains pending for more than 2 minutes, the system must provide a retry mechanism with clear product-defined limits and feedback.

**In scope:**
- UI changes to expose retry action for pending payments
- Detection of pending state exceeding 2-minute threshold
- Retry orchestration within product-defined limits

**Out of scope:**
- Payment API contract changes
- Payment calculation logic changes
- Multi-repo work

---

## Acceptance Criteria

### AC1: Detect pending payments exceeding 2-minute threshold

**Given** a payment has been in pending state  
**When** the system checks its timestamp  
**Then** the payment is marked retry-eligible if pending for more than 2 minutes

---

### AC2: Display retry action for retry-eligible payments

**Given** a payment is retry-eligible (pending > 2 minutes)  
**When** the user views the payment status UI  
**Then** a retry action is visible at `[PRODUCT_DECISION: button location]`

---

### AC3: Enforce maximum retry limit

**Given** a payment has been retried `[PRODUCT_DECISION: max_retries]` times  
**When** the user attempts another retry  
**Then** the retry action is disabled  
**And** `[PRODUCT_DECISION: post-limit feedback]` is displayed

---

### AC4: Provide feedback after each retry attempt

**Given** a user triggers retry on a pending payment  
**When** the retry is submitted  
**Then** the UI shows `[PRODUCT_DECISION: loading/processing state]`  
**And** updates to reflect the new payment state when the retry completes

---

## Open Product Decisions

The following decisions must be resolved before breakdown:

1. **Retry button location** — Where in the payment status UI does the retry action appear? (e.g., inline with status text, below payment summary, in action menu)
2. **Maximum retries allowed** — How many retry attempts are permitted per payment? (e.g., 3, 5, unlimited)
3. **Post-limit user feedback** — What does the user see after exhausting retry attempts? (e.g., "Contact support", "Payment failed—refund initiated", disable with no message)
4. **Loading/processing state** — What visual feedback accompanies a retry in progress? (e.g., spinner, disabled button, status text change)

---

## Notes

- No changes to payment calculation logic or payment API contracts.
- Single-repo scope (`app`).
- Edge case: if a payment transitions from pending to failed/succeeded naturally, the retry action must reflect the updated state.
