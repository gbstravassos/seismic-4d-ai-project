# Seismic 4D Time-Shift Estimation — AI-Assisted Scientific Development

> **Project status:** Planning / specification. The algorithms, tests, and numerical results described below are **planned**, not yet implemented or validated.

## Overview

This project develops a reproducible Python workflow for estimating time shifts between baseline and monitor seismic signals. It begins with controlled synthetic experiments, then compares windowed cross-correlation (CC) and dynamic time warping (DTW) under increasingly realistic conditions.

The project has **two complementary objectives**:

1. **Scientific:** understand, implement, test, and compare time-shift estimation methods relevant to time-lapse (4D) seismic analysis.
2. **Methodological:** learn how to use generative AI productively throughout scientific software development—from specification and task decomposition to coding, testing, review, and documentation—without treating AI output as independently verified evidence.

The initial release **does not require field seismic data**. A true 4D field validation would require comparable surveys acquired at different times; generating a synthetic monitor from a single survey does not constitute such validation.

## Research questions

- How accurately can CC and DTW recover known constant and time-varying time shifts?
- How do window length, sampling interval, interpolation, maximum lag, and warping constraints affect estimates?
- How robust are the methods to noise, amplitude scaling, and wavelet mismatch?
- Under what conditions do low residual misfit and apparently plausible shifts conceal an incorrect solution?
- Which tasks can be delegated to AI, and which require explicit scientific judgment or independent verification?

## Scientific background

A simplified baseline trace is constructed as

\[
d_b(t) = (r * w)(t),
\]

where \(r(t)\) is reflectivity, \(w(t)\) is a source wavelet, and \(*\) denotes convolution.

For the initial synthetic tests, the monitor is defined by

\[
d_m(t) = a(t)\,d_b\bigl(t-u(t)\bigr)+n(t),
\]

where \(u(t)\) is the prescribed time shift, \(a(t)\) is an amplitude factor, and \(n(t)\) is additive noise. This is a **controlled signal model**, not a full physical simulation of reservoir production or wave propagation. Later tests may introduce wavelet changes and other non-repeatability effects.

**Sign convention:** for positive constant \(u\), an event in the monitor appears at a **later** time than the corresponding baseline event. All implementations, plots, and tests must follow this convention. Time shifts are reported in milliseconds; time strain is dimensionless, \(\epsilon_t(t)=du(t)/dt\), using consistent time units. Time-strain estimates require particular care because numerical differentiation amplifies noise.

Time shifts can reflect propagation changes as well as acquisition and processing differences. They should **not** automatically be interpreted as reservoir changes. Any geological or reservoir interpretation is outside the initial scope.

## Scope

### In scope — initial project

- Reproducible generation of synthetic reflectivity and Ricker-wavelet traces.
- Baseline/monitor pairs with known constant and smoothly varying shifts.
- Controlled tests of additive noise, amplitude scaling, and wavelet mismatch.
- Global and windowed CC-based shift estimation.
- DTW-based shift estimation with explicit lag and strain/smoothness constraints.
- Subsample estimation where applicable, with numerical tests.
- Quantitative accuracy, robustness, and runtime comparisons.
- Automated unit and regression tests; figures and notebooks.
- Documentation of AI-assisted development decisions and verification.

### Out of scope — initial project

- Processing or interpretation of proprietary seismic datasets.
- Full 3D/4D seismic volumes, survey matching, and production workflows.
- Reservoir simulation, rock physics inversion, and quantitative saturation/pressure inference.
- Claims of field performance based solely on synthetic experiments.
- Fully autonomous AI decisions about scientific validity.

## Methods

### 1. Synthetic signal generation

Generate sparse or layered reflectivity sequences and convolve them with a configurable Ricker wavelet. Create monitors by applying known shifts using documented interpolation, with fixed random seeds for reproducibility. Separate clean reference cases from noisy and non-repeatable cases.

### 2. Cross-correlation (CC)

Start with constant shifts and global correlation, then implement localized, windowed estimates. Document lag conventions, normalization, window overlap, boundary handling, and subsample peak interpolation. Compare the impact of window size on temporal resolution and stability.

### 3. Dynamic time warping (DTW)

Implement an explicit sample-alignment cost and dynamic-programming solution, with backtracking and transparent boundary conditions. Introduce maximum-lag and physically motivated strain/smoothness constraints. Verify the meaning of the returned shift field, including sign and time coordinates. Compare unconstrained and constrained variants only where both are well defined.

### 4. Time strain

Estimate \(du/dt\) from recovered shifts using a documented differentiation and smoothing strategy. Compare against the analytic derivative of the prescribed synthetic shift where available. Avoid treating a smooth-looking strain curve as proof of accuracy.

## Experimental design

| Experiment | Input condition | Primary purpose |
| --- | --- | --- |
| E01 | Identical baseline and monitor | Verify near-zero estimated shift |
| E02 | Known positive and negative constant shifts | Verify sign convention and accuracy |
| E03 | Known smooth time-varying shift | Test local recovery and strain |
| E04 | Several noise levels with fixed seeds | Quantify robustness |
| E05 | Constant and variable amplitude scaling | Test amplitude sensitivity |
| E06 | Wavelet mismatch | Test non-repeatability and bias |
| E07 | Window/lag/strain parameter sweep | Study sensitivity and failure modes |
| E08 | Repeated runs and runtime measurements | Assess reproducibility and cost |

Experiments must include zero-shift controls, positive and negative shifts, edge cases, and known-answer tests. Parameter ranges and quantitative pass/fail tolerances will be set **before** evaluating the methods; no accuracy threshold is claimed here without preliminary calibration.

## Evaluation metrics

- **Shift RMSE (ms):** root-mean-square difference between estimated and prescribed shifts on a common valid time support.
- **Shift bias (ms):** mean signed estimation error.
- **Maximum absolute error (ms):** worst-case deviation on valid support.
- **Time-strain RMSE (dimensionless):** error against the known derivative when available.
- **Residual NRMS (%):** normalized residual misfit before and after alignment, using one documented definition and the same valid support.
- **Runtime and memory:** recorded under specified hardware and software conditions.
- **Constraint violations:** count and magnitude of any lag or strain violations.

Report valid sample counts, masking, boundary exclusion, and interpolation choices alongside every metric. A lower post-alignment NRMS does **not** necessarily imply a more accurate time-shift estimate.

## Planned repository structure

```text
seismic-4d-ai-project/
├── README.md
├── PROJECT.md                 # Milestones and acceptance criteria
├── requirements.txt           # To be defined and pinned
├── .gitignore
├── docs/
│   ├── methodology.md
│   ├── decisions.md           # Technical decision log
│   ├── ai_workflow.md         # AI delegation and verification log
│   └── references.md
├── src/
│   └── seismic4d/
│       ├── __init__.py
│       ├── synthetic.py
│       ├── cross_correlation.py
│       ├── dtw.py
│       └── metrics.py
├── notebooks/
│   ├── 01_synthetic_data.ipynb
│   ├── 02_cross_correlation.ipynb
│   ├── 03_dtw.ipynb
│   └── 04_comparison.ipynb
├── tests/
│   ├── test_synthetic.py
│   ├── test_cc.py
│   ├── test_dtw.py
│   └── test_metrics.py
└── outputs/
    └── figures/
```

**Note:** this tree is a target layout, not a claim that all files currently exist. Generated outputs should normally be reproducible rather than committed in bulk. Large datasets and private or licensed files must not be committed without appropriate rights.

## Development roadmap

| Phase | Scientific deliverable | AI-development skill | Completion evidence |
| --- | --- | --- | --- |
| 1. Specification | Models, conventions, test plan | Convert goals into requirements | Reviewed `PROJECT.md` |
| 2. Synthetic data | Reproducible baseline/monitor generator | Delegate implementation with constraints | Known-answer tests pass |
| 3. Cross-correlation | Global/windowed CC estimator | Test-driven iteration and debugging | Signed shifts recovered on controls |
| 4. DTW | Constrained DTW estimator | Decompose complex algorithms | Path and constraint tests pass |
| 5. Benchmarking | Comparative tables and figures | Automate experiments, critique results | Reproducible metrics and failure cases |
| 6. Documentation | Final notebooks and report | Generate and audit documentation | Clean-environment reproduction |

Suggested duration: approximately two weeks of focused work, adjustable according to validation needs. Milestones are evidence-based rather than calendar-based.

## How AI is used

Generative AI may assist with:

- Converting research objectives into technical specifications and acceptance tests.
- Explaining unfamiliar methods and proposing alternatives.
- Drafting Python modules, tests, notebooks, and documentation.
- Reviewing code for numerical, logical, and reproducibility issues.
- Suggesting diagnostic plots, stress tests, and failure scenarios.

**Human responsibilities:** approve scientific assumptions; verify references and formulas; inspect sign and unit conventions; define acceptance criteria; interpret evidence; and decide whether a result is credible.

AI-generated explanations, code, citations, and numerical claims are **not accepted as validated solely because the model produced or reviewed them**. Use independent known-answer tests, analytic controls, reproducible runs, and source checks. Record significant AI contributions, corrections, and verification steps in `docs/ai_workflow.md`. Do not fabricate citations or experimental results.

## Reproducibility and quality standards

Before a result is considered accepted:

1. The exact input generation procedure, random seed, sampling interval, and parameter values are documented.
2. A clean environment can install pinned dependencies and execute the required tests.
3. Time units, shift signs, interpolation, and valid-support masks are explicit.
4. Metrics compare methods on equivalent valid support.
5. Figures identify synthetic ground truth and estimated values clearly.
6. Failed tests and known limitations are documented, not silently omitted.
7. AI-produced code and analysis receive appropriate independent checks.

## Installation and execution

The repository is **not yet executable**: the implementation and dependency file will be added during development. Once available, the intended workflow will be:

```bash
git clone <REPOSITORY_URL>
cd seismic-4d-ai-project
python -m venv .venv
source .venv/bin/activate        # Linux/macOS
python -m pip install -r requirements.txt
python -m pytest
jupyter lab
```

Replace `<REPOSITORY_URL>` with the actual GitHub URL. On Windows, use the appropriate virtual-environment activation command. These commands describe the **planned** workflow and are not evidence of a working release.

## Data policy

Phase 1 uses generated synthetic data only. Future work may explore openly licensed field seismic datasets, subject to source attribution, usage rights, and data suitability. A single seismic survey with an artificially warped monitor will be identified as a **pseudo-4D experiment**, not a true repeat-survey 4D validation.

## Limitations

- The initial forward model is simplified and does not reproduce full elastic wave propagation or acquisition geometry.
- Similar waveform shapes can yield ambiguous alignments; DTW may overfit noise or waveform differences.
- CC trades temporal localization against stability; DTW depends strongly on cost function and constraints.
- Interpolation and differentiation can introduce artifacts and apparent precision.
- A good synthetic benchmark does not establish field reliability.
- AI assistance can accelerate development but may introduce subtle numerical and scientific errors.

## References and attribution

A curated, verified bibliography on time-lapse seismic, cross-correlation, DTW, and time-strain estimation will be maintained in `docs/references.md`. References should be checked against primary publications before being added; no unverified bibliography is asserted in this planning README.

## Project status and next action

**Current milestone:** Phase 1 — specification.

**Next action:** create and review `PROJECT.md`, including parameter ranges, sign conventions, exact metric definitions, baseline tests, and acceptance thresholds. Only then begin implementing the synthetic generator.

## License and contributions

A software license has **not yet been selected**. Until one is added, public visibility alone does not grant general reuse rights. Contribution guidelines and an explicit license may be added when the initial code is ready.

---

*This is a learning and research-development project. Its primary outputs are reproducible experiments, scientifically justified comparisons, and a documented workflow for responsible AI-assisted scientific programming.*
