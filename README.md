# Singularity — Verified Decision Ledger

A public, append-only, hash-chained record of every decision an AI-assisted
portfolio system makes: probabilistic forecasts, buy/sell actions, exits, and
— unusually — **rejections**. Each record is timestamped when the decision is
made and graded mechanically against market outcomes afterwards. Nothing is
edited, nothing is deleted, and the misses stay on the page.

**Live site:** https://mericdonmezer-ops.github.io/singularity-ledger/

## Why this exists

Financial social media is full of claims and empty of receipts: winning calls
get pinned, losing calls get deleted, and nobody publishes what they *didn't*
buy. This ledger inverts that. The interesting question is not "what did you
pick" but "how calibrated are you, verifiably, over time" — including the
rejections that were wrong.

## The honesty contract — two tiers

| Tier | Meaning |
|---|---|
| **ARCHIVE** | Backfilled internal history (Mar 2026 → Jul 2026) published retroactively. Real and unedited, but it cannot claim pre-registration — outcomes were already knowable when it was published. |
| **LIVE** | Entered the chain **while still unresolved**. This is the verifiable tier: the block hash proves the claim predates the outcome. |

The genesis block states this distinction permanently. Labeling beats pretending.

## Verify it yourself

- `ledger.json` — full record set. Every record carries a `claimHash`
  (sha256 over its immutable claim fields).
- `chain.jsonl` — one block per change: sha256 of the full record set +
  the previous block's hash. Editing any published claim breaks the chain.
- Git history of this repo is the independent second witness: every block
  is committed shortly after it is minted.

```
# recompute the chain yourself — any tampering surfaces immediately
python -c "
import json, hashlib
blocks=[json.loads(l) for l in open('chain.jsonl')]
prev='0'*64
for b in blocks:
    payload='|'.join(str(b[k]) for k in ['seq','date','recordsHash','prevHash','totalRecords','newRecords','updatedOutcomes'])
    assert hashlib.sha256(payload.encode()).hexdigest()==b['blockHash'], f'block {b[\"seq\"]} forged'
    assert b['prevHash']==prev, f'block {b[\"seq\"]} chain broken'
    prev=b['blockHash']
print(f'{len(blocks)} blocks OK')
"
```

## What is (and isn't) published

Published: tickers, decision types, claim statements, reference prices,
stated probabilities, forward returns, Brier contributions.
Not published: position sizes, account values, broker order details.

## Disclaimer

This is the decision record of a private system, published for
transparency and calibration research. Nothing here is investment advice or
a recommendation to buy or sell any security.
