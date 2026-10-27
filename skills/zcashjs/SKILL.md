---
name: zcashjs
description: Use when building, reviewing, or debugging JavaScript or TypeScript code that uses the @zcashlabs/zcash.js SDK — chain queries, local wallets, proposals, sending, shielding, PCZT signing, sync, or payment recovery on Node or in browsers.
---

# Zcash JS

Identify the installed package version and platform (Node vs browser) before asserting behavior. The public surface is the release's `src/index.ts`, and the docs under `docs/api/` describe it — cite the tag, not memory. The package is an experimental prerelease; qualifying a claim means finding it in the current release, not in plans under `docs/planning` or `docs/research`.

Choose the client first: `createPublicClient` for JSON-RPC reads, `createLightClient` for lightwallet queries/streams, `createWalletClient` for local accounts, scanning, proposals, signing, and submission. Public and light clients never open a wallet; a wallet's queries read its own scanned state, not the chain.

Load [the SDK map](references/zcashjs.md) for the API surface and defaults, [storage and platforms](references/zcashjs.md#storage-and-platforms) before opening a wallet anywhere but Linux, [the payment flow](references/zcashjs.md#propose-send-shield-and-recover) for propose/send/shield/recovery, and [limits and security](references/zcashjs.md#limits-and-security) before promising a capability or handling secrets.

For payment work, treat `propose` → review → `send` → `pending.wait` as the unit, and the operations journal as the recovery surface. A timed-out send is `unknown`, not failed: inspect `error.operationId`/`error.paymentState` and resume the recorded operation rather than building a replacement. An attached signer is required for spending; importing a mnemonic does not make the wallet ready to spend.

## Core workflow

1. Verify the SDK version, target network, endpoint protocol (JSON-RPC vs gRPC vs gRPC-Web), and storage kind against the installed package.
2. Follow one full call path — client construction, request, observation — in the SDK's own docs before writing code.
3. For wallet flows, trace signer attachment, sync, proposal review, dispatch, and restart recovery through `accounts`, `operations`, and `pczt` before claiming a payment behavior.
4. Confirm platform limits (Node filesystem storage is Linux-only today, browser assets/CORS, unqualified paths) before telling the user something works.

## Review examples

- `await wallet.send(...)` timed out: the submission may have landed. Reopen the same database, list `wallet.operations`, and resume the recorded `operationId` — a fresh `send` with a new request ID can pay twice.
- `wallet.accounts.import({ mnemonic })` succeeded but `send` fails with `SIGNER_REQUIRED`: the returned signer starts unattached. `accounts.attachSigner` must return a binding whose `state` is `ready`.
- A browser wallet "works in dev" but can't reach the endpoint in production: browser light clients need a gRPC-Web endpoint with CORS, and the bundler must emit the lazy WASM/worker/proving assets alongside the app.
- `getBalance` returned a total but `propose` rejects: a displayed total is not spendable input selection; the proposal applies the transaction policy, confirmations, and freshness rules.
- A "custom network" that silently talks to mainnet: presets are validated, but discovery via `readGenesisHash` does not authenticate the endpoint — for a known chain, supply the configured genesis hash.

Report the package version, network, storage kind, endpoint protocol, and the exact error `code`/`operationId` observed — not "it failed".
