# JOURNAL

Dated history. Every number quoted elsewhere points to an entry here.

## 2026-09-29 — Repository created and specified

- Repository created at `~/Sites/trainsift` (since moved to `~/Project/trainsift`), specified
  before any code: README, working rules, and the six documents of the reading order.
- The idea was chosen over a multi-agent financial-markets assistant and over a ROS 2
  troubleshooting assistant, both examined in the same conversation. The criterion that
  decided: coherence with the author's three existing projects and no diversion from the
  roadmap. This repository is the roadmap's phase 4 deliverable; the fault suite applies the
  method of tinct's Culprit to training runs; demosift is the first dependency.
- **The name.** Forty-five candidates were checked on `.io`, `.com`, `.dev` and `.app` with
  `dig` and `whois`; every plain English word (bellwether, sextant, tessera, synoptic, oikos,
  panoptes, spyglass, whetstone, gauntlet, anneal, kiln, plateau, hindsight) was registered,
  mostly by resellers. `runsift`, the first choice, was dropped: `runsift.com` is a Y
  Combinator log-analytics company. `trainsift`: `.io` and `.com` free, `.dev` and `.app`
  without name servers (to confirm at the registrar), PyPI free, no GitHub repository of that
  name; the only homonym on the web is a script file in an unrelated student repository.
  `faultsift` and `policysift` were also free and kept as fallbacks. Recorded as D1.
- Nothing installed, nothing measured, no GPU rented.

## 2026-10-15 — The agent comparison placed in the program's private lab (D17)

- The comparison `rules` against `model-alone` against a graph of agents on the frozen
  campaign is one notebook of the program's private lab at milestone 2, N = 10 runs, bootstrap
  intervals, scored with `score_diagnoses` like a third party's submission. Recorded as D17.
  What the program has no instrument for: a corpus, a calibrated judge of explanations, a
  faithfulness check — properties of a product, not of the benchmark (`docs/06` §6).
- Touched: `CLAUDE.md` §2, §3 and rule 1; `README.md`; `STATE.md`; `docs/01` §1, §2 and §6;
  `docs/02` definition of done; `docs/04` §1 and the redaction paragraph; `docs/05` §8;
  `docs/06` §4 and §6. No code, nothing measured, no GPU rented.

## 2026-10-16 — Papers as deliverables, a milestone 2B skeleton, a path fixed (D18, D19)

- Recorded from a session in `~/Roadmaps/roadmap-research-engineer`, at the author's request,
  the day after that repository was specified. This repository is that roadmap's research
  vehicle: its module R0.2 is milestone 1 written as a reproducibility paper, its R0.4 is
  milestone 2 written as an evaluation-benchmark paper with the D17 comparison as the ablation
  section, its R1.4 is a new milestone 2B, an LLM post-training adapter with a
  `TRAINSIFT-LLM-*` fault family, written here as a skeleton. The reason the edits are made
  now rather than when that roadmap opens: trainsift opens first, and a discrepancy log is kept
  during a reproduction, not reconstructed after it.
- Touched: `docs/02` milestones 1, 2, new 2B, the definition of done, the exclusions;
  `docs/03` D18 and D19; `CLAUDE.md` §2 (one bullet) and §3 (path); `STATE.md` (one section);
  this file. `~/Sites/demosift` → `~/Project/demosift` wherever it appeared: both packages
  live under `~/Project/`. No code, nothing measured, no GPU rented, nothing open.

## 2026-10-16 — The documents consolidated: the benchmark, the lab notebook, nothing else

- At the author's request, from a session in `~/Roadmaps/roadmap-research-engineer`: the
  documents of this repository no longer refer to any hosted product over the platform. The
  diagnostician question lives in the program's private lab notebook (D17) and nowhere else;
  D2, D3, D6, D11, D14 and D17 are reworded to their outcome, `CLAUDE.md` §2, `README.md`,
  `STATE.md`, `docs/01` §6, `docs/05` §8, `docs/06` §6 and the earlier entries of this file
  follow. The program's production project is `~/Project/sherpapier`, unrelated to this
  platform (program board, `~/Roadmaps/program/docs/08`). No code, nothing measured,
  no GPU rented, nothing open.

## 2026-10-16 — Program pointers

- The program's steering repository exists since today (`~/Roadmaps/program`): the private lab
  D17 refers to is described in its `docs/03` §5, the board is its `docs/08`. D17's pointer and
  the entry above updated accordingly; nothing else changed.

## 2026-10-18 — Ablations, the training step under the profiler, LLM reference runs (D20–D23)

- Recorded from a session in `~/Roadmaps/program`, at the author's request ("oui fait tout
  ça"), after a review of this repository against its role as the proof of the Research
  Engineer craft (program PD-12, PD-15). The review found the measurement craft complete —
  reproduction with a discrepancy log, sealed answers, frozen protocol, baseline first,
  bootstrap — and three gaps: no experiment on a model's own design; training performance
  measured without a profile; the LLM milestone using post-training only as a fault substrate.
  The Research Engineer roadmap already held the matching work (R0.3, R1.1, R2.1–R2.3) outside
  this repository; the amendments link it here rather than duplicate it.
- Touched: `docs/03` D20–D23; `docs/02` milestones 1, 2B and 3, proof 8, the definition of
  done; `docs/04` §3 (`throughput/mfu`); `STATE.md` (the demosift pin is tagged; one section);
  this file. No code, nothing measured, no GPU rented, nothing open.

