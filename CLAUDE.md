# CLAUDE.md — trainsift

Read first. This file gives the context, the non-negotiable rules and the reading order.

## 1. The project in one sentence

A deterministic, measured robot-learning training platform: a library and a command line that
audit a dataset, train a policy reproducibly, evaluate it with confidence intervals, export it
for deployment, and record every run as a **bundle** that any diagnostic method can be scored
against, including on a **fault suite** whose answers are known.

## 2. Context

- **demosift sifts the demonstrations before training; trainsift sifts the training after.**
  The two are siblings with the same shape: a public, deterministic package that computes the
  facts; whatever reasons over them with a model lives outside and is measured against them
  (`docs/03` D17). Nothing here calls a model.
- **This repository is the phase 4 deliverable of the author's roadmap**
  (`~/Roadmaps/roadmap-robotics/roadmaps/01-roadmap-core.md`): "a robot-learning training
  platform as a standalone repo — data ingestion and versioning, configured and reproducible
  distributed training, an evaluation harness, and a deployment path". It is not a third
  project next to that one; it *is* that one.
- **This repository is also the research vehicle of the author's Research Engineer roadmap**
  (`~/Roadmaps/roadmap-research-engineer`, since 2026-10-16): its modules R0.2 (reproduction),
  R0.4 (writing) and R1.4 (the LLM adapter) are milestones 1, 2 and 2B of this repository seen
  from the learner's side (D18, D19). That roadmap re-teaches nothing from here and points at
  these documents instead.
- **The fault suite is the method of tinct's Culprit applied to training runs**
  (`~/Project/tinct/docs/06-level2-faults-agent.md`): a fault is a versioned scenario with
  explicit parameters, applied without the rest of the pipeline knowing, verified as applied,
  and its answer is sealed. Any diagnostic method, an agent, a set of rules, a human, is scored
  the same way, against the same answers, with intervals.
- **Public repository, real users.** The users are the LeRobot community and, later, anyone
  training a policy in simulation. The tool must be useful to them for free, and honest about
  what it cannot see.
- **The author learns by reading.** Claude Code writes the code and, for each milestone, the
  guide that explains it (`docs/guides/`). A milestone without its guide and its numbers is not
  done.
- **Everything is in English**: code, comments, docs, commit messages, issues.
- **Bought conventions.** Quality rules, CI shape and release pipeline are inherited from
  demosift (`~/Project/demosift`), which inherited them from `cgp-ai`; the domain is new.

## 3. Reading order

demosift is checked out at `~/Project/demosift`; its `CLAUDE.md` and `docs/01` are read before
touching the data module, because demosift is this platform's first dependency and its gate.

1. `docs/01-architecture.md` — what the platform is and is not, its modules, its public surface, the run bundle, how a caller uses it
2. `docs/02-milestones.md` — milestones, how the platform proves itself, what is excluded
3. `docs/03-decisions.md` — decision log
4. `docs/04-run-bundle.md` — the bundle contract: what a run records, its bounds, its redaction
5. `docs/05-fault-suite.md` — the taxonomy, the scenarios, how a fault is applied, verified and sealed
6. `docs/06-scoring-protocol.md` — how any diagnostic method is scored, the baselines, the statistics, the frozen protocol

**Session resume:** read `CLAUDE.md`, `STATE.md` (current milestone, numbers, pending
decisions) and `JOURNAL.md` (dated history) before answering.

## 4. Non-negotiable rules

1. **No model, no agent, no cloud account.** The library calls no language or vision model,
   provisions nothing, and reaches nothing but `huggingface.co` and the machine it runs on. A
   GPU is rented by the author and used through the command line; the repository never holds
   a credential for it. A diagnostician that calls a model lives outside this repository: the
   program's private lab notebook at milestone 2 (D17), or a third party's submission.
2. **Audit before training.** A run starts by inspecting its dataset with the pinned demosift
   release; the inspection is stored in the bundle. A failing check blocks the run unless the
   override flag is passed, and the override is recorded in the bundle. Silent training on
   broken data is the first fault the platform refuses to commit itself.
3. **The bundle is the contract.** Everything a run produces that anyone downstream needs is in
   the bundle (`docs/04`), self-contained, local, bounded and redacted. An experiment tracker
   is an exporter, never the source of truth.
4. **Analyzers report facts, never causes.** An analyzer is a pure function of a bundle that
   returns findings with evidence. Naming the root cause is a diagnostic method's job, and the
   only one this repository ships is the deterministic `rules` baseline (`docs/06`).
5. **A fault is a versioned scenario.** Explicit parameters, an injection hook at one boundary,
   a verification that it was applied, a sealed answer kept outside the bundle. A bundle never
   carries its own fault id. An observed failure that was not injected is recorded in the
   failure taxonomy, never in the fault statistics.
6. **The protocol, once frozen at `v1.0`, never changes.** Extensions are new versions.
   Comparability of scores is worth more than any improvement.
7. **No conclusion from a single run.** Training is stochastic and so is any model-backed
   diagnostic method. A published number comes from N runs with a 95 % bootstrap interval;
   two methods whose intervals overlap are reported as "not significant".
8. **No number in a document without a run.** A figure in `README.md` or `docs/` cites the
   `JOURNAL.md` entry that produced it. "Not measured yet" is an acceptable value; a guess is
   not. Every GPU hour rented is journaled with its cost.
9. **No secrets in the repository.** The only credentials the library ever touches are the Hub
   token and, optionally, an experiment tracker's key, both read from the environment; the CI
   scans for secrets; bundles are redacted before they leave the machine.
10. **Scope guard.** Work only on the current milestone (`STATE.md`). An idea outside it becomes
    an issue, not a change.
11. **This repository deploys nothing.** It publishes a package and a command-line image;
    running them anywhere is somebody else's repository. No Terraform, no cloud resource, no
    account id.
12. **Production quality from the first commit**: typed (`mypy --strict`), linted (`ruff`),
    tested (`pytest`, real recorded fixtures), no `TODO`, explicit errors, bounded inputs.
13. **A public surface is a contract.** What `trainsift/__init__.py` exports is versioned;
    removing or renaming any of it is a major version, and the changelog says so.
14. **Hostile input is expected.** A bundle can come from anyone. Every reader has a size bound
    and an explicit error; a malformed file never becomes a crash or a silent wrong number.

## 5. Stack

Python 3.12, `uv`, `src/` layout, single package `trainsift`. `pydantic` models, `pandas` +
`pyarrow` for parquet, `huggingface_hub` for the Hub, `click` for the CLI, `demosift` pinned
by tag. Training and evaluation are optional extras: `lerobot` (ACT, Diffusion Policy; SmolVLA
later), `torch`, `gymnasium` with `gym-pusht` and `gym-aloha` (MuJoCo, CPU-capable), `hydra`
for configs; `onnx` and `onnxruntime` for export; TensorRT at milestone 3. `dvc` at milestone 3.
Nothing else at runtime; the core (bundle, analyzers, faults, scoring) installs without any
of the extras. Released as a wheel on a `v*` tag and as a command-line image on GHCR.

### Naming

| Usage | Name | Example |
|---|---|---|
| package | `trainsift` | `import trainsift` |
| fault suite | `trainsift.faults` | `from trainsift.faults import FaultRegistry` |
| CLI | `trainsift` | `trainsift analyze runs/2026-10-03-act-pusht/` |
| fault ids | `TRAINSIFT-<CAT>-NN` | `TRAINSIFT-DATA-01`, `TRAINSIFT-CFG-02`, `TRAINSIFT-COMB-01` |
| observed failure modes | `X-<CAT>-NN` | emergent, never injected; the failure taxonomy |
| frozen protocol | `TRAINSIFT-PROTOCOL-v1.0.md` | |
| the gold set | "the trainsift fault suite v1.0" | on the Hub, a dataset of bundles |
| in prose | *trainsift*, lowercase, like *demosift* | never "TrainSift" |

## 6. Conventions

- Google-style docstrings; a module docstring says what the module is for and what it does
  not do yet.
- Tests use recorded bundles of real runs under `tests/fixtures/`; corruptions are made on
  copies in `tmp_path`, never on the fixture. Milestone 0 starts with synthetic bundles,
  replaced by recorded ones as soon as milestone 1 produces them.
- Anything that needs a GPU is behind an interface with a CPU implementation small enough for
  CI; tests that need a GPU are marked `@gpu` and run by hand, their numbers journaled.
- Conventional commits (`feat:`, `fix:`, `docs:`, `chore:`), one topic per PR.
- Bumping `demosift` is one pull request: the tag in `pyproject.toml`, `uv lock`, the CI green,
  the changelog line.
- Each milestone closes with: numbers in `README.md`, a `JOURNAL.md` entry, the learning guide
  in `docs/guides/`, and a release tag.
