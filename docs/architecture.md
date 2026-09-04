# Architecture

## One object

The public surface is deliberately narrow: the package exports a single class,
`TEA`, and the enum that types a study. Everything else is reached through
namespaces on the case object.

```
TEA
├── meta, goal, scope, inventory   study definition (TEA Guidelines B.1 / B.4)
├── capex, opex, profitability     cost and profitability services
├── equipment                      item level: sizing, costs, specifications
├── heat_integration               pinch, hen, pool
├── simulation                     simulator bridges (lazy)
├── interpretation                 sensitivity studies
├── results                        every computed result, persisted
├── plots                          figure service (lazy — matplotlib loads here)
├── reports                        report generation
└── backup, info                   backups, terminology
```

A case is created, loaded, validated, computed and saved through this one
object; there is no separate runner and no configuration script.

---

## Calculation pipeline

`update()` executes the whole assessment in ordered phases and returns a run
report. The order is a contract, not an implementation detail: heat integration
must resolve utility demand before operating cost can be computed, and operating
cost must exist before working capital and discounted cash flow.

```
0    validation
1    inventory resolution
1.3  no-recovery baseline          ┐
1.5  pinch analysis                │ heat integration
1.7  heat exchanger network        │
1.8  utility pool evaluation       ┘
2    equipment sizing and costing
3    capital cost
3.5  total annualised cost stamp
4    operating cost
5    revenue
5b   —
6    profitability (DCF)
```

---

## Registries as the extension point

Every family of methods sits behind a registry that maps a typed key to an
implementation class. Adding a costing method, a sizing method, a network
synthesiser or a figure means registering a class — not editing the caller.

| Registry | Keyed by | Currently registered |
|---|---|---|
| CAPEX estimators | method × variant | 3 factored variants |
| OPEX estimators | method × variant | 3 estimator families |
| Profitability estimators | method | NPV/IRR |
| Sizing methods | equipment type × method | 4 |
| HEN synthesisers | synthesiser id | 3 (MILP, MINLP-SWS, MINLP-AP) |
| Plot recipes | plot kind | 6 |

The consequence that matters for an assessment framework: a methodological
choice is a registry key stored in the case, so a study records which method
produced each number, and a comparison across cases can be checked for method
consistency rather than assumed.

---

## Results and staleness

Results are not returned and discarded — they are persisted in a typed container
alongside the inputs that produced them. Each result collection carries a hash
of its inputs and an upstream dependency, so the case can report which of its
results are stale after an input changes, rather than silently mixing results
computed under different assumptions.

This is what makes a multi-case study auditable: a number in a report is
traceable to the inputs, the method, the index base and the currency base that
produced it.

---

## Layering

```
tea/
├── meta/                 study definition — goal, scope, inventory
├── equipment/            item level
│   ├── specs/            equipment specifications and KPI evaluation
│   └── sizing/           sizing dispatch, mechanical layer
├── costs/
│   ├── capex/            service, aggregator, estimators
│   └── opex/             service, aggregator, estimators
├── profitability/        DCF, total annualised cost
├── heat_integration/
│   ├── pinch/            cascade, curves, identification, LP targeting
│   ├── hen/              synthesis, realisation, costing, selection
│   └── autobase/         no-recovery baseline
├── interpretation/       sensitivity studies
├── results/              typed result container, staleness
├── simulation/           transport-agnostic bridge, DWSIM backend
├── visualization/        recipes, primitives, export
├── reports/              report generation
├── units/                pint registry, unit and currency types
├── io/                   case index, repository, serialisation
└── utilities/            mixins, types, display

correlations/             cost-correlation library (separate package)
```

The dependency direction is one-way: `utilities` and `units` are depended on and
depend on nothing; `visualization` and `reports` read results and are read by
nothing. Numerical primitives — pinch cascade, network constraint systems,
correlation evaluation — hold no reference to the case object and are testable
in isolation.

---

## Design decisions worth naming

**Documentation-driven.** The specification is written to implementation
readiness before code, and the code follows it. Where the two diverge, the
specification is authoritative and the code is corrected — not the other way
round.

**No commercial dependencies.** Seven runtime dependencies, all open source.
MILP and LP run on HiGHS as bundled with SciPy. The package can be installed and
run without a licence server.

**Units are typed, not conventional.** Every physical quantity carries its unit
through a pint registry, and unit consistency is enforced at the field level
rather than trusted. Currency is a unit like any other, with index escalation
and conversion as explicit, audited operations.

**Separation of building and persisting.** Figures are constructed and returned;
writing them to disk is a separate call. The same holds for results: computing
and saving are distinct, so a case can be explored without being mutated.
