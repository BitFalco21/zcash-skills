---
name: sprout-rescue
description: Use when a user holds ZEC in the Sprout pool (zc… addresses, old zcashd wallet.dat, z_setmigration, z_getmigrationstatus) and needs the Sprout→Sapling migration tool explained, monitored, or recovered — or when questions involve the transparent-exposure tradeoff between Sprout and Sapling.
---

# Sprout rescue

Safety rules come before every step:

- Never ask for, read aloud, or paste into chat any `wallet.dat`, seed, spending key, or passphrase — and never send one to any person, site, or "recovery service". Unsolicited recovery offers are scams; say so plainly.
- Have the user back up `wallet.dat` offline before anything else runs. The user types passphrases in their own terminal, never an echoed argument.
- Show the exact command and get explicit confirmation before any spend or migration-enabling call (`z_setmigration`, `z_mergetoaddress`, `z_sendmany`).

The tool, per the zcashd migration doc: since zcashd 2.0.5-2, a built-in migration tool moves funds from every Sprout (`zc…`) address in the wallet to one Sapling (`zs…`) destination — up to 5 transactions whenever the chain reaches a 500-block-height interval, amounts drawn at random so each user's migration hides among the others happening at the same time. It is the recommended path over manual transfer; the doc predates later releases, so verify cadence and versions against the running node's `help` before quoting numbers.

Everything else the doc does not cover — a migration deadline, current release status, encrypted-wallet behavior, per-note balances — must be verified against live sources at use time; do not repeat them from memory. Load [the runbook](references/migration.md) for the full RPC and privacy model, and [the test plan](references/test-plan.md) for verification steps.

## Core workflow

1. Preflight: identify the zcashd version (`z_setmigration` exists since 2.0.5-2), wallet state, and an offline `wallet.dat` backup. Confirm the user wants the privacy tradeoff: every migration exposes the moved value through the transparent pool.
2. Destination: check `-migrationdestaddress` or accept the default — the wallet's Sapling account 0 address.
3. Enable: `z_setmigration true` (or `-migration` in zcash.conf to auto-start on every restart — including after an RPC `z_setmigration false`).
4. Monitor: `z_getmigrationstatus` until `unmigrated_amount` falls below 0.01 ZEC, when the tool disables itself. A stopped node means the tool stops too — reactivate after restart unless `-migration` is set.

## Review examples

- "Send my Sprout straight to my Sapling address" → `z_sendmany`/`z_mergetoaddress` prohibit direct Sprout→Sapling sends; manual transfer must route through a transparent address, exposing the balance publicly. The migration tool exists precisely to hide that move among other migrations.
- "It stopped making progress" → first check the obvious per the doc: node still running, `enabled: true`, Sprout balance still above 0.01 ZEC. Anything deeper — wallet locks, rescan state — is outside the doc; verify on the node.
- "Set it and forget it" → without `-migration` in zcash.conf, a shutdown before completion requires re-running `z_setmigration true` after restart.
- "Someone DMed a rescue service" → scam until proven otherwise; never route wallet material to a third party.

Report `enabled`, `destination_address`, `unmigrated_amount` vs `finalized_migrated_amount`, and `finalized_migration_transactions` — not assumptions — so the user can see real progress.
