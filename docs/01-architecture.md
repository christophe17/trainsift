# Architecture

What the platform is, what it is not, and how its pieces fit. Where a choice is still open it
is marked and carries its decision reference.

## 1. What the library is

A **training platform** for robot-learning policies whose every output is reproducible and
measured, and a **benchmark for diagnostic methods** built on top of it. Everything here is
deterministic code or batch computation: no model call, no agent, no server. Where a model's
judgement enters, naming a cause from the findings, it enters outside this repository: in the
program's private lab notebook at milestone 2 (`docs/03` D17), or in a third party's method,
each scored here like any other.

The reason the two halves live in one package: the fault suite needs the platform to produce
faulted runs, and the platform needs the analyzers to be honest about its own runs. They share
the bundle. Splitting them would mean two packages that agree on one format, which is one
package.

## 2. The picture

```
  a dataset on the Hub ──▶ data: resolve, demosift gate ──▶ train: framework adapter ──▶ bundle
                                                                  │  (LeRobot first; Isaac Lab later)
                                                                  ▼
                                                        evaluate: rollouts, intervals ──▶ bundle
                                                                  │
                                                                  ▼
                                                        deploy: ONNX export, parity, latency ──▶ bundle

  a bundle ──▶ analyze: pure functions ──▶ findings (facts with evidence, never a cause)
                                              │
  the fault suite ──▶ faulted runs ──▶ bundles + sealed answers ──▶ score: any method's diagnoses ──▶ accuracy with intervals
                                                                              ▲
                                     `rules` (here) · `model-alone`, a graph of agents (the lab notebook, D17) · third parties
```

Two flows over one artefact. The **run flow** turns a dataset and a config into a bundle, then
enriches it with evaluation and export results. The **diagnosis flow** reads bundles, computes
findings, and scores methods that name causes. The bundle is the only thing that crosses.

## 3. Modules

| Module | Does | Does not |
|---|---|---|
| `bundle` | The `RunBundle` models, reader, writer, validation, redaction, bounds, renderings (`docs/04`) | Interpret anything |
| `data` | Resolve a dataset id and revision on the Hub; run the pinned demosift inspection and store it; block on failure unless overridden; at milestone 3, sharded formats and versioning of derived artefacts | Compute a fact about a dataset: that is demosift's, and every number it stores comes from demosift |
| `train` | Framework adapters. An adapter takes a resolved config, propagates the seed, captures the environment, writes metrics and events into the bundle as they happen, and exposes the fault hooks. `lerobot` first (ACT, Diffusion Policy) | Implement a policy or a training loop; the framework does, the adapter records |
| `evaluate` | The harness: N episodes over seeds in a simulator, success rate with a Wilson interval, bootstrap for comparisons, a sample of videos, results per checkpoint into the bundle | Decide that a policy is good; it reports the rate and the interval |
| `deploy` | ONNX export, a parity test (same observations, same actions within a tolerance), a latency benchmark before and after optimization; TensorRT at milestone 3 | Serve anything |
| `analyze` | Analyzers: `(RunBundle) -> list[Finding]`, each pure, bounded, tested on recorded bundles | Name a cause (rule 4) |
| `faults` | The registry of versioned scenarios, the injection hooks, the verification that a fault was applied, campaign generation with sealed answers (`docs/05`) | Know who is being scored |
| `score` | Read a campaign and a method's diagnoses, compute accuracy per category and scenario, false positives on clean controls, calibration, cost, with bootstrap intervals; the `rules` baseline (`docs/06`) | Run a model |
| `cli` | `audit`, `train`, `evaluate`, `export`, `analyze`, `faults`, `score` | |

Every module has an interface with a small implementation that runs on CPU in CI. The `train`,
`evaluate` and `deploy` modules are extras: the core installs without torch.

## 4. The analyzers

An analyzer looks at one aspect of a bundle and returns findings. A finding has a stable id,
a category (`data`, `config`, `dynamics`, `evaluation`, `environment`), a severity (`fail`,
`warn`, `info`), a one-line statement, and evidence: the numbers, keys or events that support
it, with their location in the bundle. It never says "therefore the cause is". Two families
open milestone 0:

- **Consistency of config against dataset.** fps in the config against fps in the inspection;
  camera keys and resolutions the policy expects against the ones the dataset has; action and
  state dimensions; the chunk or horizon length against the episode length distribution;
  normalization statistics against the dataset's revision.
- **Metrics curves.** Loss not decreasing over a window; loss diverging; a loss step change
  at an event (learning-rate change, resume); gradient-norm spikes; validation diverging from
  training; a checkpoint evaluated that is not the last or the best; evaluation with fewer
  episodes than the claimed precision needs.

Later families: environment (seed propagation, mixed-precision overflow flags, version drift
between the config's pin and the frozen environment) and evaluation protocol (train and eval
environments differing on image size, max steps or seed set).

Analyzers are the platform's honesty about its own runs, and the raw material of every
diagnostic method. They are exact, and cheap: a bundle is analysed in seconds, on a laptop,
which is a number the README publishes.

## 5. The public surface

`trainsift/__init__.py` re-exports what a caller needs, so that nothing deeper is imported:

| Name | What it is |
|---|---|
| `RunBundle`, `Finding`, `Diagnosis`, `Campaign`, `Score` | The types a caller stores, branches on, or submits |
| `read_bundle(path) -> RunBundle` | Bounded, validated, explicit errors |
| `analyze_bundle(bundle) -> list[Finding]` | Every analyzer of the installed version |
| `list_faults(version=None) -> list[FaultScenario]` | The suite, read-only |
| `score_diagnoses(campaign, diagnoses, *, resamples=2000) -> Score` | Accuracy with intervals |
| `rules_diagnose(findings) -> Diagnosis` | The deterministic baseline |
| `to_json(obj) -> str`, `to_markdown(obj) -> str` | Renderings for a tool and for a report |
| `UnsupportedBundleError`, `BundleTooLargeError`, `FaultNotAppliedError` | The errors a caller catches |
| `__version__` | |

With the extras: `train_run(config) -> Path`, `evaluate_checkpoint(bundle, checkpoint, *,
episodes, seeds) -> EvalResult`, `export_onnx(bundle, checkpoint) -> Path`, `apply_fault(config,
scenario) -> config`. Every model is Pydantic; `model_dump(mode="json")` is the storable form.
This surface is a contract (`CLAUDE.md` §4.13).

## 6. How a caller uses it

Any caller, the program's private lab notebook or a third party's method, imports the package
pinned by tag. It calls `read_bundle` and
`analyze_bundle` with no model; whatever reasons over the findings, a rule set, one model
call, a graph of agents, receives them as they are and adds nothing to them; its diagnoses
are scored with `score_diagnoses` on the published campaign, exactly as a third party's. If a
caller ever needs a new fact about a run, the fact is added here, tested here, released
here, and the pin is bumped there. That rule is the same as demosift's D12, and it is what
keeps the benchmark honest: the method being scored cannot move the scorer. Since D17 the
first such caller is the lab notebook of milestone 2, which produces the `model-alone` and
graph lines of the table (`docs/06` §4).

## 7. Where the library goes next

Milestone 1 makes the bundles real (LeRobot adapter, harness). Milestone 2 freezes the fault
suite and publishes the gold set. Milestone 3 is scale and deployment: a large public dataset
ingested in a sharded format with throughput measured, data versioning, multi-GPU training,
ONNX and TensorRT with latency numbers. A second framework adapter, PPO locomotion in Isaac
Lab with `rsl_rl`, arrives when the author's phase 2 policy exists to be recorded, and brings
its own fault family. See `docs/02`.
