# tea — Techno-Economic Assessment for Process Concepts in the chemical industry

**One methodology, applied identically across many process concepts — so that the
results are comparable, not merely available.**

> [!IMPORTANT]
> ## PRELIMINARY SHOWCASE — UPLOAD OF FULL SHOWCASE PENDING
>
> This repository currently presents **one worked case**, published to make the
> core workflow visible: case setup, flowsheet import, a single `update()` call,
> and the results layer.
>
> The full showcase is in preparation and will add further assessed cases, the
> sensitivity analysis with its results, and the generated assessment report.
> Until then, the capability table below states what the code does; the worked
> case below it is the evidence for the part that is already shown.

![Heat exchanger network synthesised for the CO₂ pressurization case](assets/hen_grid.png)

*A heat exchanger network synthesised by the package for the worked case below.
Each match carries its area, duty and annualised cost; the network recovers
1,613 kW of a 1,672.5 kW maximum-recovery target. The figure is generated from
the case object — it is not drawn by hand.*

---

## What it is

A Python package that takes a process concept — streams, equipment, utilities,
scenario assumptions — and produces a costed, discounted-cash-flow assessment,
with heat integration and sensitivity analysis inside the same model rather than
alongside it.

The difficulty in assessing process concepts is rarely the individual
calculation. It is that a comparison across twenty concepts is only meaningful
if all twenty were assessed under the same system boundary, the same cost
correlations, the same index and currency base, and the same discounting
assumptions. `tea` fixes those choices once, at case level, and applies them
identically to every case built on it.

The assessment framework follows the TEA Guidelines for CO₂ utilisation
(Zimmermann et al.). Goal and scope are not documentation — they are structural
elements of the case object, and the report is generated at the guideline
`shall` level.

---

## A worked case

CO₂ pressurization by three-stage compression with intercooling to 308 K. The
flowsheet is solved in DWSIM and imported; everything downstream of the import
is the package.

| | |
|---|---|
| System boundary | Gate to gate, functional unit kt·a⁻¹, TRL 9 |
| Cost basis | EUR, reference year 2026, CEPCI, plant type fluid, Germany |
| Financial assumptions | Interest 20 %, tax 30 %, 1 a investment phase, 10 a production, linear on-ramp |
| Operation | 90 % annual availability, ΔT_min 10 K |
| Equipment | 4 compressors (three compression stages plus an auxiliary), 3 pressure vessels, heat exchangers from the synthesised network |
| Utilities | Cooling water (20 → 25 °C), grid electricity (25 EUR·MWh⁻¹), LP, MP and HP steam |

### 1 · Setup — assumptions are structure, not prose

Every choice below is an attribute of the case object: typed, unit-carrying,
persisted and reproducible. Nothing here is a comment that a later calculation
could contradict.

![Case setup: scope, cost context and financial assumptions](assets/example_setup.png)

### 2 · What the import puts into the case

This is the source. Three compression stages `C-1`, `C-2`, `C-3` with
intercoolers `HX-C1` to `HX-C3` back to 308.15 K, knockout vessels `B-1` to
`B-3` on the condensate, and the water-recovery exchanger `HX-H2`.

![DWSIM flowsheet: three-stage CO₂ compression with intercooling](assets/dwsim_flowsheet.png)

The flowsheet is read once. From that point the case carries the streams, the
unit operations and the compound set as its own inventory — the assessment does
not reach back into the simulator for them. The import states what it did and
what it defaulted:

![Flowsheet import summary](assets/example_import.png)

Seven equipment items, materials defaulted to SS316, and a warning that the
compressors carry no power utility yet — wired up in the next cell. A default is
recorded as a default, not applied silently.

**Stream inventory, thermal section.** Every quantity the pinch analysis and the
network synthesis use later is visible here: heat capacity, inlet and outlet
temperature, phase, and whether the stream belongs to the heat-integration pool.
A result without its inventory is a number without a basis.

![Stream inventory, thermal section](assets/example_streams_thermal.png)

**Equipment inventory.** The sizing inputs per item, and the specification
choices the import made — orientation, design pressure, material. The sizing
outputs are still empty at this point; `update()` fills them, and section 4
costs them.

![Equipment inventory: pressure vessels and compressors](assets/example_equipment.png)

### 3 · One call runs the pipeline

`validate()` checks the case for completeness before anything is computed.
`update()` then runs equipment sizing, cost estimation, pinch targeting, network
synthesis and the cash-flow model. Neither returns a bare number: `validate()`
reports 0 errors against 16 warnings, `update()` a run report over 11 entries —
and one result it declined to compute rather than compute wrongly.

![validate() and update() with their result reports](assets/example_run.png)

### 4 · Results carry their own derivation

`breakdown=True` decomposes a number down to the line items and the applied
factors, so an estimate can be traced back to its basis rather than taken on
trust.

![CAPEX result with full breakdown](assets/example_results_capex.png)

Delivered equipment cost of 6.83 M EUR — dominated by the compressor train at
6.23 M EUR — becomes 34.17 M EUR of total capital investment under the refined
Lang factor of 5.00.

![Operating cost with variable and fixed breakdown](assets/example_results_opex.png)

### 5 · What heat integration is worth here

Four syntheses are run against the same pinch scenario by MINLP with augmented
penalty, varied over the exchanger minimum approach temperature and the
admission of intermediate utilities.

![HEN comparison across four synthesised realizations](assets/example_results_hen.png)

Two numbers come out of this, and they answer different questions.

**What integration is worth at all.** Three of the four syntheses produce a
heat-recovering network at 1.56 to 1.92 M EUR·a⁻¹ total annualised cost. The
third, at EMAT 5, recovers nothing and lands on 3.98 M EUR·a⁻¹ — which is
exactly the un-integrated baseline the utility pool computes independently. The
gap between the best network and that baseline is **2.42 M EUR·a⁻¹**.

**What the choice of network is worth.** Among the three that do recover heat,
0.36 M EUR·a⁻¹ separates best from worst — and the difference is structural, not
marginal. Realizations 1 and 2 share the same approach temperature and recover
the same duty to within 1 %; admitting intermediate utilities in realization 2
cuts the exchanger area from 1,615 to 659 m² and the network capital cost from
666 to 373 k EUR. That is the kind of thing a single synthesis run cannot tell
you.

![Total annualised cost, utility usage and capital cost across the four networks](assets/hen_compare.png)

![Composite curves for the base pinch scenario](assets/composite_curves.png)

### The full session

**[`examples/co2_pressurization.ipynb`](examples/co2_pressurization.ipynb)** —
case setup, flowsheet import, estimators, heat integration, results and figures,
with the stored outputs. GitHub renders it directly.

The notebook is evidence, not a tutorial: it is not executable outside its
original environment, because the package is pre-release and not distributed
here and the import step needs a local DWSIM file.

---

## Capabilities

Status reflects the code, not the specification. `Partial` names the gap.

| Area | What it does | Status |
|---|---|---|
| **Capital cost** | Factored estimation with three published factor schemes (Lang refined 2003, Lang detailed 2003, Towler & Sinnott 2022); ISBL/OSBL separation where the scheme supports it; working capital by five methods | Implemented — [worked example](#4--results-carry-their-own-derivation) |
| **Operating cost** | Direct and indirect operating cost with three published estimator families (Towler & Sinnott 2022; Peters, Timmerhaus & West 2003; Seider, Lewin & Lewis 2017) | Implemented — [worked example](#4--results-carry-their-own-derivation) |
| **Profitability** | Discounted cash flow — NPV and IRR, MACRS depreciation, tax lag, working-capital recovery | Implemented |
| **Heat integration — targeting** | Pinch analysis: problem-table cascade, composite and grand composite curves, LP transshipment targeting | Implemented — [worked example](#5--what-heat-integration-is-worth-here) |
| **Heat integration — synthesis** | Three network synthesisers: MILP transshipment; MINLP stage-wise superstructure (Yee & Grossmann); MINLP with augmented penalty / outer approximation (Viswanathan & Grossmann 1990) | Implemented — [worked example](#5--what-heat-integration-is-worth-here) |
| **Utility selection** | Utility pool evaluation against the pinch; no-recovery baseline for comparison | Implemented |
| **Equipment sizing** | Compressor (direct specification), heat exchanger (LMTD shortcut), pressure vessel (Souders–Brown) | Partial — 3 of 6 methods |
| **Sensitivity analysis** | Multi-dimensional parameter sweeps over persisted scenarios, results retained for comparison | Partial — Monte Carlo not implemented |
| **Correlation library** | Editable cost-correlation library with correction factors and LaTeX rendering of the stored formula | Implemented |
| **Units and currency** | pint-backed unit system, eight currencies, cost-index escalation (CEPCI as default index) | Implemented |
| **Figures** | Five plot recipes with a notebook mode and a print mode; PNG / PDF / SVG export at fixed physical size | Implemented |
| **Reporting** | Word report at guideline `shall` level | Partial — one profile |
| **Process simulator link** | DWSIM flowsheet import — streams, unit operations, compounds | Partial — import only; [worked example](#2--what-the-import-puts-into-the-case) |

---

## Scope and limitations

Stated deliberately, because an assessment method is only useful if its range of
validity is known.

- **Estimate class.** The implemented estimator family is factored estimation,
  valid over TRL 4–6. The package does not attempt vendor-quote accuracy and
  does not carry an equipment price database beyond its correlation library.
- **No life-cycle assessment.** `tea` covers the techno-economic half only. The
  inventory layer — system boundary, mass and energy balances, utility demand —
  is the layer an LCA would build on, but that extension does not exist.
- **Simulator coupling is import-only.** A flowsheet can be read; writing
  parameters back and re-solving in a loop is specified but not implemented.
- **Solver.** MILP and LP run on HiGHS as bundled with SciPy; NLP subproblems on
  SciPy. No commercial solver licence is required.

---

## Methodology and architecture

- [Methodology](docs/methodology.md) — assessment framework, cost basis,
  literature sources, what the numbers mean and what they do not.
- [Architecture](docs/architecture.md) — layering, calculation pipeline,
  extension points.

---

## Scale

| | |
|---|---|
| Implementation | 171 modules, ~39,000 lines of code (excluding comments and docstrings) |
| Test suite | 324 test modules |
| Runtime dependencies | pint, pandas, pydantic, python-docx, SciPy, SymPy, matplotlib |
| Python | 3.11+ |

---

## Status and availability

`tea` is in pre-release. The source repository is private while the first
release is being completed, and this repository presents the package rather than
distributing it.

**Source access is available on request** — get in touch and I will add you as a
reader on the private repository.

---

## Contact

<!-- Name, e-mail, ORCID, LinkedIn. -->
