# Distributor Wallet Top-Up Checklist

Controls for loading prepaid credit into an EVD / voucher distributor wallet.

## Matching and posting

- [ ] Unique `topup_id` per credit post; idempotent on `payment_event_id`
- [ ] Prefer VA-per-distributor or strict payment reference
- [ ] Store `gross_amount`, `fee_amount`, `net_credit` separately
- [ ] Statuses: PENDING_MATCH → POSTED | REJECTED | REVERSED
- [ ] Notify distributor with balance and `topup_id`

## Limits

| Control | Purpose |
|---------|---------|
| max_wallet_balance | Cap concentration risk |
| daily_topup_cap | Velocity limit |
| rail allow-list | Block unexpected payment sources |
| parent-only credit for sub-dealers | Preserve hierarchy |

## Manual adjustments

- [ ] Maker-checker dual control
- [ ] Mandatory reason codes
- [ ] Link to ticket / bank proof IDs
- [ ] Same audit table as automatic top-ups

## Failure handling

- [ ] Suspense for unmatched payments
- [ ] Freeze purchases if reverse would breach commitments
- [ ] Linked reverse + repost when wrong distributor credited
- [ ] Month-end report of PENDING_MATCH age
