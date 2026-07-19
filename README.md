# DoubleShot

> **When the retry works. Twice.**

An interactive simulator showing what happens when an AI agent's retry logic fires on a non-idempotent operation. The agent reports success. The customer is charged twice.

**Agent Failure Series #6** · [Live demo →](https://rlasaf12.github.io/doubleshot/)

---

## What It Shows

You're watching two views of the same operation:

| AGENT VIEW | REALITY |
|---|---|
| 1 API call (the retry "corrected" the first) | 2 API calls (both executed) |
| HTTP 200 — succeeded | $198 charged / 2 emails / 2 orders |
| 0 errors in logs | 1 support ticket |

The agent sees clean logs. The backend sees duplicates.

---

## The Three Scenarios

### 💳 Payment Processing
A Stripe charge call times out at 1,300ms. Retry fires. Both calls reach the processor. Neither has an idempotency key. Customer is charged $99 twice.

### 📧 Email Dispatch
A password reset email is queued. SMTP relay doesn't acknowledge in time. Retry generates a new token and sends again. Customer receives two emails with two different reset links — first link is now invalid.

### 📦 Order Placement
An e-commerce order write times out mid-transaction. Retry creates a second order. Warehouse fulfillment queue sees two orders for the same customer. $224 in goods shipped twice.

---

## Why Your Evals Miss This

Test environments use fast, reliable networks. Retries never trigger. Coverage shows 100%. Production has packet loss, cold-start latency, and overloaded dependencies.

The bug only appears at real-world p99 latencies — exactly when you can't afford it.

---

## The Fix

Every non-idempotent tool must accept an `idempotency_key`. Generate it **once**, before the first attempt. Pass it on every retry.

```python
# Wrong: each retry generates a new request object
def charge_customer(amount):
    for attempt in range(3):
        try:
            return stripe.create_charge(amount=amount)  # no idempotency key
        except TimeoutError:
            continue

# Right: idempotency key generated once, shared across all retries
def charge_customer(amount, order_id):
    idempotency_key = f"charge_{order_id}_{amount}"
    for attempt in range(3):
        try:
            return stripe.create_charge(
                amount=amount,
                idempotency_key=idempotency_key  # same key every time
            )
        except TimeoutError:
            continue
```

Stripe, Braintree, Twilio, and modern email APIs all support this natively. If your internal API doesn't — that's the first thing to fix.

---

## Agent Failure Series

| # | Name | Failure Mode | Demo |
|---|------|-------------|------|
| 3 | [RaceFloor](https://rlasaf12.github.io/racefloor/) | Race conditions | Two agents, one resource, no coordination |
| 4 | [ConfidenceGap](https://rlasaf12.github.io/confidencegap/) | Silent HTTP 200 failures | The API says OK. The data wasn't saved. |
| 5 | [PromptJack](https://rlasaf12.github.io/promptjack/) | Prompt injection | Malicious input hijacks agent behavior |
| **6** | **DoubleShot** | **Retry amplification** | **The retry worked. Twice.** |

---

## Stack

Pure static HTML/CSS/JS — no build step, no framework, no dependencies. Self-contained single file. Open `index.html` in any browser.

---

*Built by [Harel Asaf](https://harelasaf.com)*
