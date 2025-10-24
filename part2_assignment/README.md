# Assignment Design Notes

## A) Factory — Steady State with Cycles

### Modeling
- Variables: crafts/min per recipe (x_r ≥ 0).
- Conservation: For each item i, sum_r out_eff(i,r) x_r − sum_r in(i,r) x_r = b[i].
  - Target item t: b[t] = target rate (exact).
  - Intermediates: b[i] = 0.
  - Raw-only items: enforce net consumption (≤ cap) via two inequalities:
    (∑(out−in) x) ≤ 0 and −(∑(out−in) x) ≤ cap.
- Modules per machine type:
  - Speed factor affects crafts/min: eff_crafts_per_min(r) = machines[m].cpm * (1 + speed) * 60 / time_s(r).
  - Productivity multiplies outputs only: out_eff = out * (1 + prod_m).
- Machine caps:
  - ∑_{r uses m} x_r / eff_crafts_per_min(r) ≤ max_machines[m].
- Objective:
  - Phase-1 feasibility at requested target.
  - Phase-2 minimize total machines: ∑ x_r / eff_crafts_per_min(r).
  - Deterministic tie-break: fixed recipe ordering (lexicographic).

### Infeasibility Handling
- If exact target infeasible, binary-search max feasible target (36 iters → ~1e-10 accuracy).
- Hints: machine caps at limit; raw supplies at limit.

### Numeric
- Tolerances: 1e-9 on constraints.
- Solver: `scipy.optimize.linprog(method="highs")`. Deterministic given fixed inputs.

## B) Belts — Lower Bounds, Node Caps

### Transform
1. **Node caps**: split v → v_in → v_out, capacity cap(v).
2. **Lower bounds**: for each edge (u→v, lo, hi), add capacity (hi−lo) on (u_out→v_in), and set demand[v] += lo, demand[u] −= lo.
3. **Feasibility**: connect super-source to +demands and −demands to super-sink; run max-flow. If not saturated → infeasible; report min-cut (reachable set from super-source in residual).
4. **Main flow**: connect each source s_out → sink_in with its supply; run max-flow, then add back lower bounds to recover original flows.

### Determinism
- Dinic with lexicographic ordering of nodes and edge visits.

### Numeric
- Tolerance 1e-9 on bounds and conservation.

## Performance
- Factory LP: tiny-to-moderate problems finish in <2s with HiGHS.
- Belts: Dinic is O(E·min(V^{2/3}, E^{1/2})) typical; fine for typical contest sizes.

## Failure Modes & Edges
- Cycles/byproducts handled by equalities (b[i]=0).
- Redundant/degenerate recipes okay: solver sets x_r = 0.
- Disconnected conveyor components: feasible solver ignores them.

