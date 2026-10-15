# Initial skill evaluation: 2026-10-16

Three paired, read-only investigations completed against THS commit
[`97ae98d5e62ef5044d73890868a90a230303461f`](https://github.com/zcashlabs/thus-spoke-zakura/tree/97ae98d5e62ef5044d73890868a90a230303461f).
Each arm used Codex CLI `0.162.0`, model `gpt-6.1-sol`, and high reasoning effort.
The candidate tree includes the preceding THS source-map refresh. Its hash and the
unchanged source/prompt hashes are recorded in [suite.json](suite.json) and the
per-run manifests and receipts.

## Task results

Each task has four predeclared criteria. All four are required for task success.
The review was performed by a separate Codex reviewer and checked by the
coordinating agent. It is model-assisted, unblinded grading, without human
adjudication. [grades.json](grades.json) records every decision, answer hash, and
supporting passage or omission; [task-scores.json](task-scores.json) is machine checked.

| Task | Without skills | With skills |
| --- | --- | --- |
| Foreign Docker resource | [3/4](foreign-resource/without-skill/answer.txt) | [4/4](foreign-resource/with-skill/answer.txt) |
| Faucet response lost across restart | [4/4](faucet-lost-response/without-skill/answer.txt) | [4/4](faucet-lost-response/with-skill/answer.txt) |
| Stale wallet balance | [2/4](stale-wallet-balance/without-skill/answer.txt) | [3/4](stale-wallet-balance/with-skill/answer.txt) |
| Total criteria | 9/12 | 11/12 |
| Fully successful tasks | 1/3 | 2/3 |

The runtime baseline did not distinguish seam coverage from the required disposable
Docker collision check. The treatment explicitly made that distinction. Both faucet
answers preserved the operation/transaction identity and traced production recovery.

Both stale-balance answers missed the rubric's Vite-only frontend versus server
artifact distinction. The baseline also omitted the documented `build --dev`
workflow and the concrete `useAccounts`/query-key ownership trace. The treatment
supplied those latter details but still missed one criterion. No rubric was relaxed
after seeing these omissions. This is a strict answer-completeness assessment,
not a demonstration of broken runtime code.

## Routing results

The separate catalog-only batch passed **6/6 positive requests and 6/6 near misses**.
See the [response](routing/answer.txt), [score](routing-scores.json), and frozen
request/catalog input in [TASK.txt](routing/TASK.txt). Neutral case IDs concealed
which requests were positive. Ground truth and skill bodies were withheld.

This measures selection from names/descriptions in one batch. It does not establish
native skill discovery or automatic loading during task execution.

## Audit and reproduction

All seven runs completed; none was discarded or replaced. The six task runs used
only local source-reading commands. The [audit](audit.json) records command counts
and actual candidate skill/reference reads; the routing run used no tools. No
out-of-workspace reads, builds, tests, Docker execution, edits, or network access
were observed in these agent traces. Proposed validation in an answer was not
counted as an executed test.

Each run directory retains its exact `TASK.txt`, `manifest.json`, `receipt.json`,
`answer.txt`, empty `stderr.txt`, and full trace compressed as `trace.jsonl.gz`.
The traces contain public source excerpts and can be read with `gzip -dc`.
Original absolute temporary-workspace paths in answers and traces are preserved
to keep their hashes valid. Early captures' prompt/manifest copies were recovered
from the prepared workspace only after matching their hashes to the receipts.

Recompute the scores from these checked-in artifacts:

```console
python3 evals/run.py score-tasks --results evals/results/2026-10-16 --grades evals/results/2026-10-16/grades.json
python3 evals/run.py score-routing --results evals/results/2026-10-16
```

The task command exits 1 because three arms miss at least one criterion; the routing
command exits 0. Both print the recorded results. For fresh model runs, follow the
[evaluation protocol](../../README.md).

One baseline-first pair per task, a single model, three THS investigations, and
unblinded model-assisted review cannot establish a reliable skill effect or broad
Zcash skill coverage. The tasks evaluate investigation answers rather than code
patches or live-stack success. Timing and token counts are provenance, not performance
benchmarks; pairs overlapped in wall-clock time. Repeat with counterbalanced order,
independent human review, and coding tasks before making stronger quality claims.
