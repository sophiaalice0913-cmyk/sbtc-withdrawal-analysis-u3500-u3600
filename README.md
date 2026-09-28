# sBTC Withdrawal Analysis — request-id u3500 through u3600

## Method

I analyzed the `sbtc-registry` contract print events and joined
`withdrawal-create`, `withdrawal-accept`, and `withdrawal-reject`
events by `request-id`.

Window: request-id u3500 through u3600 inclusive (101 requests).

## 1. Created vs completed withdrawals

- Withdrawal requests created: 101
- Completed (`withdrawal-accept`): 97
- Rejected (`withdrawal-reject`): 4
- Pending without an accept/reject event: 0

The completion count was determined using the event-join method:
`withdrawal-create` events were joined to `withdrawal-accept`
events by `request-id`.

## 2. Completion latency

For the 97 completed withdrawals, using the requested fields:

`withdrawal-create.block-height -> withdrawal-accept.burn-height`

- Minimum: 7 blocks
- Median: 7 blocks
- Maximum: 10 blocks

## 3. Actual fee vs max-fee

Among the 97 completed withdrawals:

- Actual fee exactly equal to max-fee: 0 / 97
- Actual fee greater than max-fee: 0 / 97
- Largest gap: request-id u3549
  - max-fee: 10,000 sats
  - actual fee: 69 sats
  - difference: 9,931 sats below the cap
- Closest observed fee to its cap: request-id u3555
  - max-fee: 340 sats
  - actual fee: 136 sats
  - difference: 204 sats below the cap

## 4. Requests without a completed sweep

Four request-ids had no `withdrawal-accept`:

- u3591 — rejected
- u3592 — rejected
- u3593 — rejected
- u3594 — rejected

Each has an explicit `withdrawal-reject` event. Therefore none of
these four should be classified as merely pending in this event set.

## 5. Concrete interpretation hazard

A `withdrawal-create` event is not evidence by itself that sBTC was
successfully cashed out to Bitcoin.

Requests u3591 through u3594 demonstrate the hazard directly: all four
have `withdrawal-create` events, but each later has an explicit
`withdrawal-reject` and no completed sweep.

Therefore an analysis that counts create events as successful cash-outs
would incorrectly classify failed withdrawal attempts as completed
Bitcoin withdrawals. Joining events by `request-id` through
`withdrawal-accept` / `withdrawal-reject` is necessary to distinguish
successful withdrawals from rejected ones.

## Evidence

A row-level CSV covering all 101 request-ids was generated as:

`sbtc-withdrawals-u3500-u3600.csv`
