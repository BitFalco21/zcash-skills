# Test plan

What was run vs documented for this skill: **verification was doc reading only** — every claim traces to the zcashd *Sprout-to-Sapling Migration* docs page. No node was executed here; the procedures below are for an operator to run, not reported as done.

## Regtest plan (operator-run)

1. Environment: a zcashd ≥ 2.0.5-2 node (the version the doc says introduced the tool), a wallet containing Sprout funds, proving parameters present.
   - Current-network constraint to verify on the operator's setup: creating new Sprout notes may be disabled — the doc does not say; a wallet that already holds Sprout, or imported `zc…` keys, is the reliable fixture.
2. Baseline: `z_getmigrationstatus` before enabling — note `enabled` and `unmigrated_amount`.
3. `z_setmigration true`, then mine/advance blocks and watch `migration_txids` appear at the documented interval (the doc says batches land when the chain reaches a 500-block-height interval — on regtest that means a lot of blocks; confirm the actual interval on the build in use before promising it).
4. Restart cases:
   - Stop the node mid-run without `-migration` in `zcash.conf` → restart → confirm the tool stays off until `z_setmigration true` is re-issued (the doc says it must be reactivated).
   - With `-migration` set → `z_setmigration false` → restart → confirm the doc's claim that the flag re-enables it anyway.
5. Completion: keep running until `unmigrated_amount` < 0.01 ZEC → confirm the tool self-disables and `finalized_migrated_amount` reflects the moved funds.

## Read-only checklist (real wallet)

Nothing below spends or changes state — safe to run on a live wallet:

```bash
zcash-cli z_getmigrationstatus    # enabled? unmigrated_amount? destination_address?
```

The doc provides no balance-listing RPC beyond that; whatever the operator uses to inspect notes must be verified against their build's `help` output.

## Repo-side checks (run for this skill)

- `python3 docs/check-facts.py` and `--upstream` — the cadence/version oracles read the docs page itself.
- `python3 docs/gen-manifest.py` — idempotent manifest.
- `python3 docs/verify-claims.py --base <base>` — every added line re-checked.
- `python3 -m unittest discover -s tests`.
- Secret/address scan — only `zc…`/`zs…` placeholders appear in the skill.
