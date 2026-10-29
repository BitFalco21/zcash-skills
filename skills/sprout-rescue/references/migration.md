# Sprout→Sapling migration runbook

Every statement here traces to the zcashd docs page *Sprout-to-Sapling Migration* (`zcash.readthedocs.io/en/master/rtd_pages/sapling_turnstile.html`, checked 2026-10-28). The page is the original tool documentation and predates later zcashd releases — `zcash-cli help z_setmigration` / `help z_getmigrationstatus` on the running node is the oracle for anything it does not pin down.

## Why a tool exists

The migration has two levels per the doc:

- **User level** — the standard zcashd payment RPCs (`z_sendmany`, `z_mergetoaddress`) prohibit any transaction directly between Sprout and Sapling addresses in either direction. Sending between them requires a **transparent address as an intermediary**, which publicly exposes the balance: shielded→transparent reveals the value and ties it to that t-addr, and transparent→shielded reshields it.
- **Consensus level** — a direct Sprout→Sapling transaction is allowed, but the balance is *passed through the transparent value pool* (no transparent address involved), which still exposes the value. Because users would not necessarily know that, it was deliberately left out of the standard RPC — the migration tool is the supported way to use it.

So "send Sprout → Sapling" is never private: the value crosses the transparent pool either way. The tool's privacy claim is narrower — it hides each user's migration transactions among the migrations of all other users happening around the same time, with amounts drawn from a random distribution. The doc **highly recommends** the tool over manual migration.

Turnstiles exist to monitor value entering and exiting each value pool — relevant context when a user asks whether the migration is visible to the network.

## What the tool does

Available since zcashd **2.0.5-2**. It migrates funds from **any** Sprout address in the wallet to a **single** destination Sapling address:

- up to 5 transactions **whenever the blockchain reaches a 500-block-height interval**
- amounts picked from a random distribution
- ends (and auto-disables) once the wallet's Sprout balance is below **0.01 ZEC**
- full design: ZIP 308 (the doc links the spec; treat the spec's parameters the same way — verify against the running node)

Destination: defaults to the **Sapling account 0 address** — "the first address derived from the wallet's master seed". Override by setting `-migrationdestaddress=<SAPLINGADDR>` in `zcash.conf`.

## Lifecycle and restart behavior

```bash
zcash-cli z_setmigration true      # enable
zcash-cli z_setmigration false     # disable
zcash-cli z_getmigrationstatus     # progress
```

- The node must stay running until all funds are transferred. **If the node is shut down before completion, the tool must be reactivated after restart** — `z_setmigration` does not persist across restarts on its own.
- `-migration` in `zcash.conf` starts migration automatically on every restart — and per the doc, a restart **re-enables the tool even after it was deactivated via RPC** while that flag is set. Users who want it off must remove the flag, not just call `z_setmigration false`.
- When the total in Sprout drops below 0.01 ZEC the tool disables itself.

## `z_getmigrationstatus` fields

Verbatim from the doc's example output:

| Field | Type | Meaning |
| --- | --- | --- |
| `enabled` | boolean | whether migration is enabled |
| `destination_address` | string | Sapling address receiving the funds |
| `unmigrated_amount` | numeric | total unmigrated ZEC |
| `unfinalized_migrated_amount` | numeric | sent but not yet finalized |
| `finalized_migrated_amount` | numeric | finalized total |
| `finalized_migration_transactions` | numeric | count of this wallet's migration txs |
| `time_started` | numeric, optional | block time of the first migration tx (Unix timestamp) |
| `migration_txids` | array | all migration txids involving this wallet |

## What this doc does not answer

State these as things to verify, not facts: whether later zcashd versions changed the cadence or amount distribution; what happens to an *encrypted* wallet mid-run; how to check per-note Sprout balances; whether any of these RPCs are deprecated in current builds; and anything about deadlines or future network upgrades. Each needs a live source — node `help`, release notes, or the ZIPs — before being repeated to a user.
