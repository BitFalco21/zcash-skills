# Follow-up evaluation: 2026-10-16

The revised skills were tested with the same three task prompts, four criteria per
task, pinned THS source, Codex CLI `0.162.0`, model `gpt-6.1-sol`, and high reasoning
effort as the [initial run](../2026-10-16/README.md). Both arms were run again in fresh
workspaces; no earlier answer was reused or replaced. The original failed criteria
and transcripts remain intact.

The revised instructions add conditional reference sections and explicit artifact
and query-owner guidance. The candidate hash is frozen in [suite.json](suite.json).
Rubrics and task prompts were unchanged.

| Task | Without skills | With revised skills |
| --- | --- | --- |
| Foreign Docker resource | [3/4](foreign-resource/without-skill/answer.txt) | [4/4](foreign-resource/with-skill/answer.txt) |
| Lost faucet response | [4/4](faucet-lost-response/without-skill/answer.txt) | [4/4](faucet-lost-response/with-skill/answer.txt) |
| Stale wallet balance | [3/4](stale-wallet-balance/without-skill/answer.txt) | [4/4](stale-wallet-balance/with-skill/answer.txt) |
| Total criteria | 10/12 | 12/12 |
| Fully successful tasks | 1/3 | 3/3 |

The stale-balance treatment now explicitly requires `build --dev` and a new named
instance, distinguishes Vite's frontend proxy from rebuilding the server, and
traces `useAccounts` / `queryKeys.accounts` / `api.accounts`. The runtime treatment
distinguishes Docker-free seam coverage from a disposable live collision check.
Both faucet answers preserve the original operation/transaction identity.

The separate catalog-only routing batch again passed **6/6 positives and 6/6 near
misses**. See [task scores](task-scores.json), [routing scores](routing-scores.json),
and the answer-bound [review grades](grades.json). Grading used a separate Codex
reviewer and coordinating-agent review; it was unblinded and model-assisted,
without human adjudication.

All seven runs completed. Each run retains its exact task, manifest, receipt,
answer, stderr, and full compressed trace. [audit.json](audit.json) records command
counts and candidate reads. Only local read operations were observed; no builds,
tests, Docker execution, network services, hidden rubrics, or source edits were
used by the task agents. Original temporary-workspace paths remain in answers and
traces to preserve their hashes.

Recompute these recorded results:

```console
python3 evals/run.py score-tasks --results evals/results/2026-10-16-followup --grades evals/results/2026-10-16-followup/grades.json
python3 evals/run.py score-routing --results evals/results/2026-10-16-followup
```

The task scorer exits 1 for the two baseline misses; all revised-skill task checks
pass. The routing scorer exits 0. A positive treatment result is not a claim that
the baseline must fail.

These remain small, baseline-first, read-only investigations with one model and
model-assisted grading. The baseline changed from 9/12 to 10/12 between runs,
illustrating variation. Do not infer a statistically reliable effect, live-stack
success, code-editing quality, or broad Zcash coverage. Conditional loading guidance
was added, but these broad investigations still read full references; no reference
token-efficiency improvement is established. Timing/usage are provenance, not
performance benchmarks. Follow the [protocol](../../README.md) for independent
repetitions and human review.
