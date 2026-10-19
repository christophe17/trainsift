# STATE

**Current milestone: 0 — Foundation.** Specified 2026-09-29; no code yet.

## Where this stands

The repository is specified before any code: `README.md`, `CLAUDE.md`, the six documents of
the reading order. The name was checked the same day (`docs/03` D1). demosift `v0.3.0` is the
release to pin; it was tagged on 2026-09-30 (`~/Project/demosift/STATE.md`). Nothing is installed,
nothing is measured.

The question whether agents diagnose a run better than `rules` and than `model-alone` is one
notebook of the program's private lab, run at milestone 2 on the frozen campaign (`docs/03`
D17); the `model-alone` and graph lines of the table (`docs/06` §4) come from that notebook.
Milestone 2 waits for none of it, and nothing in milestone 0 changes.

## Specification amendments of 2026-10-16

Recorded from a session in `~/Roadmaps/roadmap-research-engineer` at the author's request
(`JOURNAL.md`, same date): milestones 1 and 2 gain a discrepancy log and two paper drafts as
deliverables (D18); a milestone 2B, the LLM post-training adapter and its fault family, is
written as a skeleton (D19); the stale `~/Sites/demosift` path is fixed to `~/Project/demosift`.
Nothing in milestone 0 changes; nothing is open.

## Specification amendments of 2026-10-18

Recorded from a session in `~/Roadmaps/program` at the author's request (`JOURNAL.md`, same
date), to make the repository a fuller proof of the Research Engineer craft: milestone 1
reproduces the papers' own ablations after the headline numbers, and module R0.3's
pre-registered study runs on this harness (D20); milestone 3 profiles the training step and
records MFU, parallelism beyond DDP staying in the Research Engineer roadmap's R2 (D21);
milestone 2B starts with clean post-training runs scored on a standard benchmark (D22); D16's
last sentence retired (D23). Nothing in milestone 0 changes; nothing is open.

## Next steps, in order

1. **Bootstrap the package** in the shape of demosift: `pyproject.toml` with the core
   dependencies and the `lerobot`, `eval`, `deploy` extras; `uv`, `ruff`, `mypy --strict`,
   `pytest` with coverage; the `ci` workflow (lint, types, tests, image build, dependency audit,
   secret scan); the `release` workflow (wheel, GitHub release, GHCR image). Public repository
   under the author's account (D15).
2. **The bundle contract** (`docs/04`): the `RunBundle` Pydantic models, the reader with its
   bounds and explicit errors, the writer, the redaction pass, `to_json` and `to_markdown`
   renderings; synthetic fixtures under `tests/fixtures/synthetic/`.
3. **The first analyzers** (`docs/01` §4): the config-versus-dataset consistency set and the
   metrics-curve set, each a pure function returning findings with evidence; tested on the
   synthetic fixtures and on deliberate corruptions of them.
4. **The `rules` baseline and the scorer** (`docs/06`): `Diagnosis` model, the rule set over
   findings, campaign reading with sealed answers, accuracy per category, clean-control false
   positives, bootstrap intervals, the comparison rule.
5. **The CLI**: `analyze`, `faults list`, `score`; text on a terminal, JSON when asked.
6. **Numbers**: analyzer wall clock on the synthetic fixtures, dependency count, coverage;
   into `JOURNAL.md` and the README table.
7. **Learning guide** for milestone 0 (`docs/guides/00-foundation.md`): the bundle, the
   analyzers, the scoring, the release pipeline, written against what exists.
8. **Milestone 1 opens**: the LeRobot adapter, the first real bundles, which replace the
   synthetic fixtures.

Milestone 0 closes when steps 1 to 7 are done and `v0.1.0` is tagged.

## Numbers

None yet. Nothing has run.

## Pending decisions

- **GitHub organization** (D15): the repository starts under the author's account, as
  demosift did; moving later keeps history and redirects.
- **Hugging Face organization for the published fault suite** (D10): decided when milestone 2
  publishes it; the handle `trainsift` is to be checked on the Hub before then.
- **The exact LeRobot version to pin at milestone 1**, and whether the `lerobot` package
  exposes a stable training entry point or the adapter drives its scripts.
- **Feasibility of the `ENV` scenarios** (`docs/05`): each is a candidate until a campaign
  shows it produces a reliably distinguishable bundle; a scenario that does not is dropped
  from `v1.0`, not softened.
- **The bundle size bound and the campaign's GPU budget**: configuration values set at
  milestones 0 and 2 respectively, journaled when set.
