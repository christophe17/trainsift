# trainsift

**Train a robot policy, measure it, and know why it failed.**

trainsift is a robot-learning training platform that is **deterministic and measured**: it
audits a dataset before training, trains a policy reproducibly, evaluates it with confidence
intervals, exports it for deployment, and records every run as a self-contained **bundle**.
Over those bundles it runs **analyzers** that report facts, and it ships a **fault suite**:
versioned scenarios injected into a training pipeline, with sealed answers, so that any
diagnostic method, an agent, a set of rules, a human, can be scored on how often it finds the
culprit. No model, no agent, no cloud account.

demosift sifts the demonstrations before you train; trainsift sifts the training after.
[demosift](https://github.com/christophe17/demosift) is this platform's first dependency and its
gate. One measured question sits beside the benchmark: whether agents name a run's cause
better than a rule set and than a single model call. It is answered in the program's private
lab notebook, scored on this benchmark like any other submission (`docs/03` D17).

## Measured numbers

No number in this table is estimated: each comes from a recorded run in
[JOURNAL.md](JOURNAL.md), and the table is updated when the numbers change.

| Metric | Value | Recorded |
|---|---|---|
| Analyzer wall clock on a recorded bundle | not measured yet | |
| Training reproducibility (same seed, twice) | not measured yet | |
| Evaluation: success rate and interval, ACT on pusht | not measured yet | |
| Cost of one training run on a rented GPU | not measured yet | |
| Fault suite v1.0: scenarios, bundles, GPU hours, cost | not measured yet | |
| `rules` baseline: diagnostic accuracy on the fault suite, with interval | not measured yet | |
| Dependencies of a bare install | not measured yet | |
| Test suite | not measured yet | |

## Quick start (target, milestone 0)

Nothing is released yet. This is the surface milestone 0 builds, written first so that the code
has something to be true to.

```bash
pip install trainsift
trainsift analyze runs/2026-10-03-act-pusht/     # findings over a run bundle, text or JSON
trainsift faults list                            # the fault suite and its versions
trainsift score --method rules campaign/v1.0/    # the deterministic baseline, with intervals
```

With the `lerobot` extra, from milestone 1:

```bash
trainsift audit lerobot/pusht                    # demosift's inspection, stored in the bundle
trainsift train configs/act_pusht.yaml           # a bundle under runs/, reproducible by seed
trainsift evaluate runs/<bundle>/ --episodes 50  # success rate with a Wilson interval
trainsift export runs/<bundle>/ --onnx           # the checkpoint for deployment, parity-tested
```

## The run bundle in one paragraph

A bundle is a directory: a manifest (versions, git SHA, dataset id and revision, the demosift
inspection, the resolved config and its hash, the seed, the hardware), a metrics table in
parquet, an events log, evaluation results per checkpoint, references to the checkpoints, the
frozen environment, and redacted logs. It is what the analyzers read, what the fault suite
produces, what a diagnostic method reads, and what a third party submits to be scored. The
contract is in [docs/04-run-bundle.md](docs/04-run-bundle.md).

## What this repository does not do

It calls no model, runs no agent, serves no HTTP, provisions no GPU, and deploys nothing.
Anything that reasons over a run with a model, one call or a graph of agents, imports this
package from outside and is scored by it like any third party ([docs/06](docs/06-scoring-protocol.md)).
Keeping that out is what makes the platform testable to the last digit, and its fault suite
an honest benchmark for any method, including the author's own.

## Repository layout (target)

```
src/trainsift/   bundle, data (demosift gate), train (adapters), evaluate, deploy, analyze, faults, score, cli
tests/           pytest suite on recorded bundles (tests/fixtures/)
docs/            architecture, milestones, decisions, bundle contract, fault suite, scoring protocol, guides
Dockerfile       the command line in a box
CLAUDE.md        working rules of the repository; STATE.md and JOURNAL.md track progress
```

## License

Apache-2.0. See [LICENSE](LICENSE).
