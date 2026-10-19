# Milestones

Each milestone ships something usable on its own, as a release of the library. Work happens on
one milestone at a time (`CLAUDE.md` §4.10); ideas outside it become issues.

| # | Milestone | Content | Status |
|---|---|---|---|
| 0 | **Foundation** | The bundle contract, the first two analyzer families, the `rules` baseline, the scorer, the CLI, CI, the release workflow; synthetic fixtures; the learning guide. No GPU | specified |
| 1 | **Train and evaluate** | The LeRobot adapter producing real bundles (ACT and Diffusion Policy on `lerobot/pusht` and `lerobot/aloha_sim_insertion_human`, small configs); the demosift gate; the evaluation harness with intervals; reproducibility measured; recorded bundles replace the synthetic fixtures; ONNX export with parity test; the headline numbers first, then **the papers' own ablations at equal compute** (D20, 2026-10-18); **a discrepancy log** of the reproduction (paper claim, our result with its interval, the gap, suspected cause, what was tried), kept during the runs and not reconstructed after; **a reproducibility-paper draft** written from the recorded bundles in TMLR format (D18, 2026-10-16) | planned |
| 2 | **The fault suite v1.0** | The scenarios, the injection hooks, the verification, the campaign with sealed answers; the gold set published on the Hub; the `rules` baseline scored; the protocol frozen; the table; the D17 comparison (`rules` / `model-alone` / graph of agents, N runs, bootstrap intervals) written up as the **ablation section of an evaluation-benchmark paper**, whose draft is a deliverable of the milestone (D18, 2026-10-16) | planned |
| 2B | **LLM post-training adapter and the `TRAINSIFT-LLM-*` fault family** | *Skeleton written 2026-10-16 (D19); detailed when the milestone opens.* **Reference runs first** (D22, 2026-10-18): the GRPO and DPO runs of R1.1 recorded as clean bundles and scored on a standard verifiable benchmark with an interval over seeds — the first LLM numbers, and the campaign's clean controls. A second framework adapter recording a post-training run of a ≤ 1.5B open model (SFT/LoRA, DPO or GRPO; TRL or a from-scratch loop) as a bundle, exactly as the LeRobot adapter records a policy run; the fault family on post-training boundaries — reward hacking (a gameable reward), KL collapse or blow-up, chat-template bug, data contamination (evaluation items in the training data), tokenizer mismatch, learning-rate and warm-up faults — each a versioned scenario with injection hook, verification and sealed answer, under a `v2.0` protocol extension (`docs/05` gains the family then; `v1.0` is never changed); its campaign, the `rules` baseline scored first, its table. The module that specifies it in detail is R1.4 of `~/Roadmaps/roadmap-research-engineer/roadmaps/01-roadmap-research-engineer.md` | planned (skeleton) |
| 3 | **Scale and deploy** | A large public dataset (a subset of OXE or DROID, hundreds of gigabytes) ingested into a sharded format with throughput measured; data versioning of derived artefacts; multi-GPU training with DDP on rented machines; the training step profiled and MFU recorded, the largest wins applied with before and after numbers (D21, 2026-10-18); TensorRT with latency before and after; the phase 4 evidence. Beyond DDP and multi-node: Research Engineer modules R2.1–R2.2, not here (D21) | planned |
| 4 | **A second framework** | PPO locomotion in Isaac Lab through `rsl_rl`, its adapter, its fault family (reward scale, observation noise, domain-randomization ranges, actuator model), a `v2.0` protocol | later |
| 5 | **Curation loop** | Train on raw versus demosift-curated data and measure the difference; closes demosift's milestone 6 from this side | later |

## How the platform proves itself

A platform that is not itself measured is a wrapper. These are the measurements the platform
needs to be trusted, each introduced at the milestone where it first matters.

1. **Determinism of the diagnosis flow (milestone 0).** The same bundle produces the same
   findings and the same score, byte for byte; the tests assert it on fixtures and on
   deliberate corruptions of them.
2. **Hostile input is bounded (milestone 0).** Every reader has a size bound and an explicit
   error; a malformed, oversized or hand-crafted bundle becomes a named failure, never a crash
   or a silently wrong finding.
3. **Reproducibility of training (milestone 1).** The same config and seed, run twice on the
   same hardware, produce metrics curves within a tolerance the guide states and success rates
   whose intervals overlap. The tolerance is measured, then enforced as a test on the small
   configs.
4. **No conclusion from a single run (milestone 1).** Every published success rate is N
   episodes over S seeds with a Wilson interval; every comparison of two policies is a
   bootstrap difference with its interval; overlapping intervals are "not significant".
5. **A fault is verified as applied (milestone 2).** Each scenario ships a check that reads the
   raw artefacts of a faulted run and asserts the fault is present, and a check that the
   bundle does not leak the answer. A campaign is valid only if both pass on every bundle.
6. **The baseline is scored before any method (milestone 2).** The `rules` baseline's accuracy
   on the frozen campaign is the first line of the table. A method that does not beat it with
   non-overlapping intervals has not shown anything yet.
7. **Replay before change (milestone 2 onward).** Before any change to an analyzer or to the
   rule set, the new version is scored on the frozen campaign and compared with the current
   one; the go/no-go and the numbers go into `JOURNAL.md`.
8. **Throughput and cost are numbers (milestone 3).** Samples per second per GPU, MFU, wall
   clock per epoch, cost per run, before and after each optimization, each win traced to the
   profile that justified it (D21); export latency before and
   after TensorRT. Each in the README, each from a journaled run.

## Definition of done of the first pass (milestones 0–2)

| Someone checks | The pass delivers |
|---|---|
| It is installable and honest | `pip install trainsift`, `trainsift analyze <bundle>`, findings that say what they are evidence of and nothing more |
| It trains and measures | `trainsift train`, `trainsift evaluate`: a bundle with curves, a success rate with an interval, a parity-tested export |
| It is a benchmark | A frozen protocol, a published gold set, a scorer, a baseline line in the table, a submission format |
| It is measured | The README table: analyzer latency, reproducibility tolerance, evaluation intervals, the campaign's cost, the baseline's accuracy with intervals, coverage |
| It is serious engineering | CI, tests on recorded bundles, a one-page architecture, a decisions log, a versioned public surface |
| It is safe to point at anything | Bounded readers, explicit errors, redacted bundles, no credential but the caller's Hub token |
| Someone else uses it | Installs from the release, submits diagnoses to be scored; the program's private lab notebook is the first submitter (`docs/03` D17) |
| It is research | The papers' own ablations reproduced at equal compute (D20); a discrepancy log kept during the milestone-1 reproduction; a reproducibility-paper draft (milestone 1) and an evaluation-benchmark-paper draft with the D17 comparison as its ablation section (milestone 2); every number in either traces to `JOURNAL.md` or to a bundle; submission is the author's act (D18) |

## Excluded from the first pass

No multi-GPU, no large dataset, no TensorRT, no second framework, no curation loop: milestones
2B to 5. And, permanently: no model client, no agent, no server, no deployment.
