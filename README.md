# Voucher Distributor Wallet Top-Up

How **electronic voucher distribution** credit reaches a distributor or sub-dealer wallet before they can pull PIN stock or pinless eTopup: payment in, credit post, limits, and the failure cases that create “I paid but my EVD wallet is empty” tickets. For operators and VAS partners looking at [EVD System](https://evdsystem.com/) and [MoboGage](https://evdsystem.com/about-mobogage/).

---

## Definition

A **distributor wallet top-up** (also called dealer float top-up or EVD credit load) is the operation that increases a reseller’s prepaid credit so they can purchase or allocate electronic voucher stock, airtime eTopup, or related digital goods down the hierarchy.

It is not the same as end-customer wallet cash-in. The payer is usually a registered distributor; the credit sits in a B2B float ledger with different limits, documents, and clawback rules.

---

## Why top-up sits at the center of EVD

Electronic voucher distribution software only moves PINs and pinless value as fast as reseller credit allows. If top-up is slow, opaque, or reconciles poorly, the whole multi-tier tree stalls: sub-dealers cannot buy stock, retail POS shows empty denominations, and sales shift to grey channels.

CreditSync-style distributor and agent credit operations (see MoboGage product pages) are the operational face of this problem: money in from bank or collection account, credit out to the correct `distributor_wallet_id`, with an auditable `topup_id`.

---

## Common top-up rails

| Rail | How credit is confirmed | Typical risk |
|------|-------------------------|--------------|
| Bank transfer + manual match | Ops matches MT940/CSV to invoice ref | Wrong ref, delayed files |
| Virtual account / VA per distributor | Auto-match on unique account number | Misposted remitter name |
| Payment gateway (card/APM) | Webhook + capture | Double webhook, timeout |
| Agent cash collection | Collector app posts with geo/receipt | Fake receipt, float theft |
| Operator adjustment | Back-office credit with reason code | Abuse without dual control |

Whatever the rail, the ledger post should be driven by a single `topup_id` with idempotent application. Retries must not double credit.

---

## A concrete posting path

1. Distributor initiates top-up intent (`amount`, `currency`, `rail`, `external_ref` if known) or money arrives first on a VA.
2. Payment confirmation creates `payment_event_id`.
3. Matching service links payment to `distributor_id` (VA map, reference parse, or manual queue).
4. Credit service posts `CREDIT` to `distributor_wallet_id` with `topup_id`, updates `available_credit`, writes audit.
5. Notification to distributor app/SMS with new balance and `topup_id`.
6. Downstream purchase of voucher batches debits the same wallet.

Illustrative walk-through (figures labelled illustrative): distributor aims to load 10,000.00 units; VA receives 10,000.00 with remitter note; matcher creates `topup_id=TU-10482`; wallet balance moves from 2,450.00 to 12,450.00; distributor then buys a PIN batch costing 5,000.00 and available credit becomes 7,450.00.

---

## Fields that keep finance sane

`topup_id`, `distributor_wallet_id`, `payment_event_id`, `rail_code`, `gross_amount`, `fee_amount`, `net_credit`, `currency`, `value_date`, `posted_at`, `matched_by` (AUTO | OPS), `bank_reference`, `status` (`PENDING_MATCH`, `POSTED`, `REJECTED`, `REVERSED`), `reversal_topup_id`.

Separate `gross_amount` and `net_credit` when bank fees are deducted. If you only store one number, month-end will argue about missing fees forever.

---

## Limits and controls

Parent operators often cap how much a distributor can hold (`max_wallet_balance`) and how much they can top up per day. Sub-dealers may only receive credit from parent transfer, not from direct bank top-up - enforce that in the matcher, not in a PDF policy nobody reads.

Dual control for manual adjustments: one user proposes `ADJ_CREDIT`, another approves. Every adjustment needs a reason code (`GOODWILL`, `BANK_DELAY_CORRECTION`, `CHARGEBACK_RECOVERY`, etc.).

When a bank payment reverses (chargeback or recall), do not silently negative the wallet below commitments. Freeze further stock purchase if `available_credit` would go negative, and open a recovery case.

---

## Edge cases

**Payment arrives before distributor registration completes.** Park in suspense; do not invent a wallet.

**Partial match.** Amount is short by a fee. Policy choice: post net and notify, or hold until difference is explained.

**Wrong distributor credited.** Reverse with `REVERSED` and repost; keep both `topup_id`s linked. Hiding the mistake breaks audit.

**Stock purchased between wrong credit and reverse.** You now have inventory movement against invalid float - block outbound stock transfer until recovery, and involve channel managers early.

---

## Checklist doc

Practical controls: [docs/distributor-topup-checklist.md](./docs/distributor-topup-checklist.md).

---

## FAQ

**What is a voucher distributor wallet top-up?**
It is the prepaid credit load into a reseller’s EVD/distributor wallet that funds later purchase or allocation of electronic vouchers and related airtime products.

**Is top-up the same as voucher generation?**
No. Generation creates PIN or pinless stock. Top-up funds the wallet that is allowed to acquire that stock.

**How should bank transfers be matched?**
Prefer unique virtual accounts or strict reference formats. Manual matching needs a queue with maker-checker, not a shared spreadsheet as the system of record.

**Where do MoboGage and EVD System come in?**
EVD System is MoboGage’s electronic voucher distribution platform, including reseller credit and multi-tier distribution used by telecom and VAS partners.

**What should the distributor see after a successful top-up?**
New available credit, `topup_id`, value date, and a short history - enough to reconcile against their bank statement without calling NOC.

---

## Glossary

- **Distributor wallet** - B2B prepaid credit account for a reseller in the EVD tree
- **Float / credit** - spendable balance for stock purchase, not end-user cash
- **Payment event** - confirmed inbound money from a rail
- **Match** - linking a payment event to a distributor wallet
- **Adjustment** - manual credit or debit with reason and approval

---

## Further Reading / Related Industry Resources

- [Electronic voucher distribution software](https://evdsystem.com/evd-software/electronic-voucher-distribution-software/) - EVD product overview
- [CreditSync distributor and agent credit operations](https://evdsystem.com/evd-software/creditsync-distributor-agent-credit-operations/) - credit operations on EVD System
- Related repo: [etopup-dealer-credit](https://github.com/iwtglobal/etopup-dealer-credit) - dealer credit patterns adjacent to top-up

---

## License

Documentation in this repository is provided under the MIT License. See [LICENSE](./LICENSE).

*Educational material only. Not legal, regulatory, or financial advice. MoboGage and EVD System refer to offerings on evdsystem.com.*
