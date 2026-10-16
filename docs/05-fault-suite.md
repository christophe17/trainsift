# The fault suite

> This is the part of the platform that makes it a benchmark. The method is tinct's Culprit
> (`~/Project/tinct/docs/06-level2-faults-agent.md`), moved from a physical loop to a training
> pipeline: a fault is a versioned scenario with explicit parameters, applied at one boundary
> without the rest of the pipeline knowing, verified as applied, and its answer is sealed.

## 1. The question posed

When a policy underperforms, several causes produce the same observation, a loss that
plateaus, a success rate that disappoints: the data is not what the config believes, the
config contradicts the data or itself, the software environment is not the one assumed, or
the evaluation protocol lies. An engineer staring at a loss curve cannot tell them apart; a
diagnostic method that can is worth something, and the only way to know whether it can is to
inject each cause deliberately and count.

The measure of the suite: **diagnostic accuracy per category, with intervals, for any method,
against a baseline that uses no model.**

## 2. Categories

Categories are defined by **where the fault is injected**, not by what is observed. Observed
symptoms are the analyzers' business; a fault is a cause.

| Category | The fault lives in | Injection hook |
|---|---|---|
| `DATA` | The dataset as the run sees it | The data module, between the Hub and the adapter: a transform on metadata or on the loaded frames |
| `CFG` | The resolved configuration | The config resolution step, after the user's file is read and before the framework receives it |
| `ENV` | The software environment | The adapter's environment setup: seeding, precision, version pins |
| `EVAL` | The evaluation protocol | The harness, between the checkpoint and the simulator |
| `COMB` | Two of the above at once | Both hooks |
| `CLEAN` | Nothing; the control | None; the run is nominal |

## 3. The scenarios of `v1.0` (candidates)

Every scenario below is a candidate until milestone 2 shows that it produces a bundle whose
findings are reliably distinguishable from the clean controls' across seeds. A scenario that
does not is dropped from `v1.0`, not softened. The suite needs **at least eight applied
scenarios, at least two per category among `DATA`, `CFG`, `EVAL`, at least one `ENV`, one
`COMB`, and clean controls**.

| ID | Fault | Parameters | Expected symptoms (what analyzers should see) |
|---|---|---|---|
| `TRAINSIFT-DATA-01` | fps metadata wrong | factor 0.5 on `info.json` fps | inspection passes or warns; chunk length against episode duration inconsistent; policy timing off at eval |
| `TRAINSIFT-DATA-02` | task labels permuted | fraction 0.3 of episodes | inspection passes; loss plateaus higher than control; success rate low on the affected task |
| `TRAINSIFT-DATA-03` | episodes truncated | last 20 % of frames dropped on 30 % of episodes | inspection: video span check fails per camera; episode length outliers; success rate low |
| `TRAINSIFT-DATA-04` | stale normalization statistics | stats from another dataset's revision | manifest: stats revision differs from dataset revision; loss starts high; actions saturate at eval |
| `TRAINSIFT-DATA-05` | two camera streams swapped | keys exchanged | inspection passes; loss normal; success rate collapses at eval |
| `TRAINSIFT-DATA-06` | actions shifted against observations | offset of k frames | loss floor higher; success rate low |
| `TRAINSIFT-CFG-01` | learning rate too high | factor 10 | loss diverges or oscillates; gradient-norm spikes |
| `TRAINSIFT-CFG-02` | chunk length inconsistent with fps | chunk covering several seconds at the dataset's fps | config-versus-inspection finding; loss normal; eval jerky, low success |
| `TRAINSIFT-CFG-03` | image size at train differs from eval | resize mismatch | train normal; eval finding on observation shape; success collapses |
| `TRAINSIFT-CFG-04` | a camera dropped from the policy input | one key removed | config-versus-inspection finding; loss plateaus; success low |
| `TRAINSIFT-CFG-05` | under-trained checkpoint | steps factor 0.1 | loss still decreasing at the end; checkpoint not at plateau |
| `TRAINSIFT-ENV-01` | seed not propagated to data workers | worker seeding disabled | two runs with the same seed diverge beyond the reproducibility tolerance |
| `TRAINSIFT-ENV-02` | mixed precision without loss scaling | fp16, scaler off | NaN or overflow flags in metrics; candidate, may not fail reliably |
| `TRAINSIFT-EVAL-01` | wrong checkpoint evaluated | an early step | eval finding: evaluated step is neither last nor best |
| `TRAINSIFT-EVAL-02` | normalization not applied at inference | inference wrapper bypassed | train normal; parity test fails; success collapses |
| `TRAINSIFT-EVAL-03` | eval environment differs from train | max steps or seed set changed | env config finding; success differs from a nominal re-evaluation |
| `TRAINSIFT-EVAL-04` | too few episodes for the claimed precision | episodes 5 | harness finding on interval width; a protocol fault, not a policy fault |
| `TRAINSIFT-COMB-01` | `DATA-04` + `CFG-01` | | both symptom sets |
| `TRAINSIFT-CLEAN-01`, `-02` | nominal, two seeds | | nothing beyond the control's own findings |

Each scenario is a file `src/trainsift/faults/scenarios/TRAINSIFT-<CAT>-NN.yaml`: id, version,
category, parameters, the hook it uses, the expected symptoms as finding ids, the verification
procedure, and a cost class (`cheap`: a data or eval transform on an existing run; `train`: a
new training run).

## 4. How a fault is applied

The adapter exposes **hooks at four boundaries** (data, config, environment, evaluation). A
hook is a function that takes the boundary's object and returns a modified one. The fault
registry installs the scenario's hook before the run; nothing else in the pipeline is told.
The hooks write nothing to the bundle: no event, no metric, no manifest field (rule 5). The
bundle of a faulted run is indistinguishable in form from a clean one.

Faults on `DATA` are applied on a **copy** of the dataset's metadata or frames in the run's
cache, never on the Hub cache shared with other runs.

## 5. Verification

Each scenario ships two checks, run by `trainsift faults verify` on every bundle of a campaign:

1. **Applied.** Reads the raw artefacts of the run (the mutated metadata copy, the resolved
   config, the environment, the evaluation invocation) and asserts the fault is present, with
   the expected parameters. A campaign with one unverified bundle is invalid.
2. **Sealed.** Greps the bundle for the scenario id, the category name and the parameter
   values in any form, and asserts none appears. The answer lives only in the campaign's
   `answers.json`, which is not part of any bundle.

## 6. The campaign

A campaign is `N` seeds × the scenarios of a protocol version, plus the clean controls,
produced by `trainsift faults campaign --protocol v1.0 --seeds N`. It is a directory of
bundles and one sealed `answers.json` mapping bundle identity to scenario id. Its cost in GPU
hours and currency is journaled and published; it is the campaign's number, and the bound of
D13.

The `v1.0` campaign is published on the Hub as a dataset of bundles (D10). The answers file
is released after a first scoring window, its length set in the protocol.

## 7. Injected versus emergent

A failure that was not injected and appears anyway, a crash, a divergence on a clean control,
a simulator error, is recorded in `docs/failure-taxonomy.md` with an `X-<CAT>-NN` id, its
frequency, its cause when found, and its mitigation. It is never counted in the fault
statistics, and a recurrent one may be promoted to an injectable scenario in a later protocol
version.

## 8. What the suite does not test

Whether a policy is good. Whether a method's explanation is well written. Whether a fix works.
The first is the harness's number. The last two are properties of a product; no instrument in
the program measures them, and no number about them is published (`docs/03` D17). The suite answers one question only: given a run with a known
injected cause, does the method name it.
