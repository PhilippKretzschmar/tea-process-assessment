# Methodology

What the numbers mean, where they come from, and where they stop being valid.

---

## Assessment framework

The package follows the **TEA Guidelines for CO₂ utilisation and Power-to-X**
(Zimmermann et al.). The choice is not decorative: the guidelines exist because
techno-economic results published under undeclared boundaries are not
comparable, and comparability is the whole purpose here.

Two consequences are structural rather than editorial:

- **Goal and scope are model objects**, not a section in a report. A case that
  has not declared its purpose, its functional unit and its system boundary is
  incomplete, and validation says so before anything is computed.
- **The report is generated at the guideline `shall` level.** Completeness
  against the mandatory reporting items is checked, not assumed.

---

## Cost estimation

### Capital cost

Estimation is **factored**: equipment purchase cost from correlations, then
installation, indirect and contingency factors applied by a published scheme.
Three schemes are implemented, selectable per case and recorded with the result:

| Scheme | Source |
|---|---|
| Lang refined | Peters, Timmerhaus & West (2003) |
| Lang detailed | Peters, Timmerhaus & West (2003) |
| Towler & Sinnott | Towler & Sinnott (2022) — ISBL/OSBL separated structurally |

**Validity: TRL 4–6.** Factored estimation is appropriate for concept studies
where equipment is identified and sized but not quoted. It is not a substitute
for a vendor quote, and the package does not present it as one. Estimator
families for earlier and later maturity are specified but not implemented.

Working capital is available by five methods (fraction of fixed capital,
fraction of total capital, months of operating cost, purchased-equipment
fraction, fixed amount), because the choice materially moves the discounted cash
flow and should be an explicit, recorded decision.

### Operating cost

Direct and indirect operating cost, with three published estimator families:

| Family | Source |
|---|---|
| Towler & Sinnott | Towler & Sinnott (2022) |
| Peters, Timmerhaus & West | Peters, Timmerhaus & West (2003) |
| Seider, Lewin & Lewis | Seider, Lewin & Lewis (2017) |

Utility cost is not an input assumption but a computed consequence: it follows
from the heat-integration result, so a network change propagates into operating
cost without manual reconciliation.

### Index and currency

Every cost carries a base year and a currency. Escalation to the reference year
uses a cost index — **CEPCI** is the package default, as the established index
for chemical plant capital cost. Currency conversion is a separate, explicit
step. Both are audited: the result records which index value and which exchange
rate produced it.

This is the mechanism that makes literature-sourced cost data usable. Values
taken from different publications in different years and currencies are
normalised to one basis before they are compared, and the normalisation is
visible in the output.

---

## Profitability

Discounted cash flow over the project life: **net present value and internal
rate of return**, with linear or MACRS depreciation, tax lag, and recovery of working
capital and salvage at end of life. The internal rate of return is solved
numerically rather than approximated.

**Total annualised cost** is computed separately as a capital-charge-rate
annualisation, because network and utility comparisons need a single annual
figure and a full DCF is the wrong instrument for that question.

---

## Heat integration

Process integration is treated as part of the assessment, not as a downstream
optimisation, because utility demand determines a large share of operating cost
in electrified and Power-to-X processes.

**Targeting.** Pinch analysis by problem-table cascade: minimum utility demand,
pinch location, composite and grand composite curves, and LP transshipment
targeting. This gives the thermodynamic bound before any network exists.

**Synthesis.** Three network synthesisers, so that a result can be checked
against an independent formulation rather than trusted:

| Synthesiser | Formulation | Source |
|---|---|---|
| MILP transshipment | Sequential — match selection at targeted energy, minimum units at MER; area and cost per match afterwards | Papoulias & Grossmann |
| MINLP stage-wise superstructure | Simultaneous — utility, area and unit trade-off, solved by outer approximation | Yee & Grossmann (1990), in the formulation of Ponce-Ortega et al. (2008); OA after Duran & Grossmann (1986) |
| MINLP augmented penalty | The same superstructure, solved by combined penalty function with OA/ER decomposition | Viswanathan & Grossmann (1990) |

**The two MINLP synthesisers do not return a proven optimum.** The stage-wise
superstructure has a non-convex objective, so the solvers are deterministic
heuristics with an incumbent guarantee: they return the best feasible network
they found, and where a gap is reported at all it is informative rather than a
proof — the augmented-penalty loop suppresses it as meaningless. The
augmented-penalty variant in particular tends to a structurally leaner network
than the published solutions of its source — a network in the same family, not
the same network.

That trade is deliberate and belongs to the estimate class. At concept stage the
question is whether integration is worth roughly two million a year and which
network family answers it, not which of two networks within one percent of each
other is formally optimal. The synthesis runs in seconds, which is what makes it
usable inside a parameter sweep at all. For a design decision downstream of
screening, the result is a starting point for a rigorous synthesis, not a
substitute for one.

The published test problems of the MINLP sources are used as input anchors, and
the verified solutions are frozen as regression fixpoints — one set deliberately
held out — so a change in the solver that moves them is caught. This matters
more than it may appear: a heat-exchanger-network synthesiser that has not been
checked against published problems produces plausible networks that cannot be
defended.

**Utility selection** evaluates a pool of candidate utilities against the pinch,
with a no-recovery baseline retained for comparison, so the value of integration
is quantified rather than assumed.

---

## Uncertainty

Sensitivity is handled by parameter sweeps over persisted scenarios: a case is
recomputed across a parameter grid and the results are retained side by side for
comparison. This is the appropriate instrument for concept-stage assessment,
where the dominant uncertainties are a small number of identifiable assumptions
— feedstock price, electricity price, capacity factor, discount rate — rather
than a distribution over everything.

Probabilistic sampling (Monte Carlo) is specified but not implemented.

---

## What this method cannot tell you

- **Not an environmental assessment.** No life-cycle inventory, no impact
  characterisation, no allocation. The inventory layer would carry an LCA, but
  that extension does not exist.
- **Not a procurement estimate.** Factored estimation at TRL 4–6. Costs are
  correlation-derived, not quoted.
- **Not a detailed design.** Equipment is sized to the level that costing
  requires, by shortcut methods. Rigorous sizing methods are specified but not
  implemented.
- **Not a validated flowsheet.** Mass and energy balances are taken as given, by
  hand or imported from a simulator. The package assesses a flowsheet; it does
  not converge one.

---

## Sources

- Zimmermann et al. — *Techno-Economic Assessment & Life Cycle Assessment
  Guidelines for CO₂ Utilization*
- Towler, G. & Sinnott, R. — *Chemical Engineering Design*, 2022
- Peters, M., Timmerhaus, K. & West, R. — *Plant Design and Economics for
  Chemical Engineers*, 2003
- Seider, W., Lewin, D. & Lewis, J. — *Product and Process Design Principles*,
  2017
- Papoulias, S. & Grossmann, I. — A structural optimization approach in process
  synthesis, Part II (heat recovery network transshipment model)
- Yee, T. & Grossmann, I. — Simultaneous optimization models for heat
  integration, 1990
- Ponce-Ortega, J., Jiménez-Gutiérrez, A. & Grossmann, I. — Optimal synthesis of
  heat exchanger networks involving isothermal process streams, 2008
- Duran, M. & Grossmann, I. — An outer-approximation algorithm for a class of
  mixed-integer nonlinear programs, 1986
- Viswanathan, J. & Grossmann, I. — A combined penalty function and outer
  approximation method for MINLP optimization, 1990
