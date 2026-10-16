# Scoring protocol

How any diagnostic method is scored on the fault suite, what the baselines are, and what the
statistics require. The protocol is frozen at `v1.0` at the end of milestone 2 as
`TRAINSIFT-PROTOCOL-v1.0.md`, published with the campaign, and never edited after. This
document is its draft.

## 1. What a method submits

For every bundle of a campaign, one `Diagnosis`:

| Field | Holds |
|---|---|
| `bundle_id` | The sha256 of the bundle's manifest |
| `primary.category` | One of `DATA`, `CFG`, `ENV`, `EVAL`, `COMB`, `CLEAN` |
| `primary.scenario_id` | A scenario id, or `null` when the method names only a category |
| `secondary` | Optional: a second candidate, same shape |
| `confidence` | In `[0, 1]`, the method's own estimate that `primary` is right |
| `evidence` | Finding ids and bundle locations the diagnosis rests on |
| `cost` | Seconds of wall clock; tokens and currency when a model was used, `0` otherwise |
| `method` | Name, version, and the sha256 of its configuration (prompts, model id, rule set) |

A submission is the list of diagnoses, the method's description, and, for a stochastic
method, the number of independent runs it was produced from.

## 2. Metrics

| Metric | Definition |
|---|---|
| **Category accuracy** | `primary.category` equals the sealed category |
| **Scenario accuracy** | `primary.scenario_id` equals the sealed scenario; counted only on the bundles where the method named one |
| **Clean-control false positives** | Diagnoses other than `CLEAN` on control bundles |
| **Per-category accuracy** | Category accuracy stratified by sealed category; no global mean is published without this breakdown |
| **Calibration** | Reliability curve of `confidence` against correctness, expected calibration error |
| **Cost** | Median and total, per diagnosis, in seconds and in currency |

Calibration is published because a method that knows when it does not know is the one that
can escalate; the same reason as tinct's.

## 3. Statistics

- 95 % intervals by **bootstrap** over bundles, at least 2 000 resamples.
- A stochastic method (anything with a model) is run **N = 3 times in CI and N = 10 for a
  published number**; the published accuracy is the mean over runs and the interval covers
  both sources of variance, bundles and runs.
- Two methods are compared by the bootstrap difference of their accuracies; the word
  "better" is used only when the interval of the difference excludes zero. Otherwise the
  result is reported as "not significant", in the table, in those words.
- Every score carries the campaign's protocol version and the method's configuration hash.

## 4. The baselines

| Baseline | Where | What it is |
|---|---|---|
| `rules` | This repository, `score.rules` | A deterministic rule set over findings: fixed precedence of finding ids to categories and scenarios, no model. The first line of every table |
| `model-alone` | The program's private lab notebook (`docs/03` D17), outside this repository | One model given the bundle's Markdown rendering and the finding list as text, no tools, no corpus, one call. The answer to "why not just ask a model" |
| A graph of agents | The same notebook | Agents over the findings as tool results, adding nothing to them. The question D17 keeps, whether agents name the cause better than `rules` and than `model-alone`, answered with N = 10 runs and the comparison rule of §3, and scored exactly as a third party's |

The `rules` baseline is versioned like any method; a change to it is replayed on the frozen
campaign and journaled before release (`docs/02`, "Replay before change").

## 5. Submissions by third parties

`trainsift score --method <name> --diagnoses <file> campaign/` computes the table locally;
`trainsift score validate-submission` checks the format, the configuration hash and the run
count. A leaderboard, if one ever exists, is a file in this repository updated by pull
request, with the submission's artefacts linked; nothing more is promised at `v1.0`.

## 6. What is not scored

The quality of an explanation, the usefulness of a proposed fix, the faithfulness of a report
to its evidence: these are properties of a product. Measuring them would take a judge
calibrated against human grades and a faithfulness check, and the program has no such
instrument (`docs/03` D17). The protocol scores one thing, whether the
cause was named, so that its number means one thing.
