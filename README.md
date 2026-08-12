# tea — Techno-Economic Assessment for Process Concepts in the chemical industry

**Modular package for comparative techno-economic assesment studies of process concepts in the chemical industry. Rapidly set up and branch cases in an isolated SSOT input layer. Specify CAPEX, OPEX and profitability estimators, grouped pinch and hen design studies for streams from a specified heat-integration pool, and much more. Use the command tea.update() to trigger the calculation pipeline in a stateless service layer, including selection of the most cost-effective heat integration. Inspect and compare techno-economics in the results layer. Import flowsheet data into the case from external simulators. Use the interpretation layer to orchestrate sensitivity external sweeps. Persist and keep track of cases through serialization in the .json format. and write your frontend to make them inspectable and comparable on websites **

<!-- HERO FIGURE: insert the single most informative figure here, full width.
     Suggested: HEN grid or the HEN economics comparison, since it shows both
     the process-integration result and the cost consequence in one image.
     Two sentences of caption underneath: what it shows, what is notable. -->

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

## Capabilities

Status reflects the code, not the specification. `Partial` names the gap.

| Area | What it does | Status |
|---|---|---|
| **Capital cost** | Factored estimation with three published factor schemes (Lang refined 2003, Lang detailed 2003, Towler & Sinnott 2022); ISBL/OSBL separation where the scheme supports it; working capital by five methods | Implemented |
| **Operating cost** | Direct and indirect operating cost with three published estimator families (Towler & Sinnott 2022; Peters, Timmerhaus & West 2003; Seider, Lewin & Lewis 2017) | Implemented |
| **Profitability** | Discounted cash flow — NPV and IRR, MACRS depreciation, tax lag, working-capital recovery | Implemented |
| **Heat integration — targeting** | Pinch analysis: problem-table cascade, composite and grand composite curves, LP transshipment targeting | Implemented |
| **Heat integration — synthesis** | Three network synthesisers: MILP transshipment; MINLP stage-wise superstructure (Yee & Grossmann); MINLP with augmented penalty / outer approximation (Viswanathan & Grossmann 1990) | Implemented |
| **Utility selection** | Utility pool evaluation against the pinch; no-recovery baseline for comparison | Implemented |
| **Equipment sizing** | Compressor (direct specification), heat exchanger (LMTD shortcut), pressure vessel (Souders–Brown) | Partial — 3 of 6 methods |
| **Sensitivity analysis** | Multi-dimensional parameter sweeps over persisted scenarios, results retained for comparison | Partial — Monte Carlo not implemented |
| **Correlation library** | Editable cost-correlation library with correction factors and LaTeX rendering of the stored formula | Implemented |
| **Units and currency** | pint-backed unit system, eight currencies, cost-index escalation (CEPCI as default index) | Implemented |
| **Figures** | Five plot recipes with a notebook mode and a print mode; PNG / PDF / SVG export at fixed physical size | Implemented |
| **Reporting** | Word report at guideline `shall` level | Partial — one profile |
| **Process simulator link** | DWSIM flowsheet import — streams, unit operations, compounds | Partial — import only |

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

## Example

<!-- REPLACE THIS BLOCK with a screenshot of a real notebook session:
     one input cell, and underneath it the rendered show() table or figure.
     A screenshot of the actual API is worth more than a code listing, because
     it shows what using the package feels like. Keep it to one screen. -->

```
[ notebook screenshot: case setup → tea.update() → results table → figure ]
```

---

## Methodology and architecture

- [Methodology](docs/methodology.md) — assessment framework, cost basis,
  literature sources, what the numbers mean and what they do not.
- [Architecture](docs/architecture.md) — layering, calculation pipeline,
  extension points.

---

## Example case

<!-- One worked case. Give it a name, a system boundary, a capacity, a base
     year and a currency, then the headline result and one sensitivity.
     Two figures and a link to the full report PDF. Without this section the
     capability table above is a claim; with it, it is evidence. -->

---

## Scale

| | |
|---|---|
| Implementation | 171 modules, ~39,000 lines of code (excluding comments and docstrings) |
| Test suite | 324 test modules |
| Runtime dependencies | pint, pandas, pydantic, python-docx, SciPy, SymPy, matplotlib |
| Python | 3.11+ |

---

## Publication

<!-- Full citation of Wunderlich, Kretzschmar & Schomäcker (2024),
     Frontiers in Sustainability — with DOI link. -->

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
