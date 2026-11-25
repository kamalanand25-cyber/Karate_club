# Modularity on the Karate Club Graph

**DSC212: Graph Theory | Research Assignment**

Recursive spectral modularity partitioning implemented from scratch on Zachary's Karate Club network, with per-split visualisations, node-metric tracking, and additional validation and robustness analysis.

> Method reference: M. E. J. Newman (2006), *Modularity and community structure in networks*, PNAS 103(23):8577–8582.

---

## Repository contents

| File | Description |
|---|---|
| `karate_modularity.ipynb` | Main deliverable. Runs top to bottom with no manual edits; contains code, outputs and all figures. |
| `README.md` | This file. |

## How to run

```bash
pip install numpy scipy pandas matplotlib scikit-learn networkx
jupyter notebook karate_modularity.ipynb      # then Kernel → Restart & Run All
```

No dataset download is needed: the graph ships with NetworkX (`nx.karate_club_graph()`), as do the other graphs used in the benchmark. The notebook uses only basic scientific Python plus NetworkX, and the spectral algorithm itself uses just `numpy.linalg.eigh`.

## Background

Zachary (1977) observed 34 members of a university karate club for two years. A dispute between the instructor ("Mr. Hi") and the administrator split the club in two. The friendship network (34 nodes, 78 edges) was recorded *before* the split, so we can test whether a graph-only method recovers the real fission.

**Modularity** compares observed within-group edges with a degree-preserving random baseline:

- Modularity matrix: `B = A − k kᵀ / 2m`
- Modularity of a split `s ∈ {−1,+1}ⁿ`: `Q = sᵀ B s / 4m`
- Relaxing `s` to a unit vector, the maximiser is the leading eigenvector of `B` (Rayleigh–Ritz). Its sign pattern gives the bipartition.
- Recursive bisection repeats this on each community and stops when no split helps.

## What the notebook does

**Assignment tasks**

1. Implements recursive spectral modularity partitioning (`spectral_modularity_partition`).
2. Draws the graph after every split (fixed spring layout, one colour per community, nodes labelled by ID).
3. After each split, computes degree, betweenness and closeness centrality and clustering coefficient with NetworkX.
4. Plots how each metric evolves per node across iterations (line plots, heatmaps).
5. Written discussion of which nodes stay central and how community structure affects the metrics.

**Extra analysis (Section 6)**

| Section | Analysis |
|---|---|
| 6.1 | Recovery of the true factions (ARI / NMI, misclassified nodes, contingency table) |
| 6.2 | Eigenvalue spectrum of `B`, leading-eigenvector soft assignments, 2-D spectral embedding |
| 6.3 | Comparison with greedy modularity, Louvain, label propagation, Girvan–Newman |
| 6.4 | Robustness of the split under random edge deletion (5–30%) |
| 6.5 | Degree-matched random-graph null model (500 graphs), z-score and p-value |
| 6.6 | Planted-partition (stochastic block model) sweep vs. theoretical detectability limit |
| 6.7 | Benchmark on Les Misérables, Davis Southern Women, Florentine Families, caveman, Barabási–Albert and Erdős–Rényi graphs |
| 6.8 | Relationship between node brokerage (betweenness) and community boundaries |

### Design choices worth knowing

- **Unweighted graph.** NetworkX's karate graph carries edge weights (total 231). The notebook strips them to match the handout's 78-edge unweighted graph.
- **Metrics per iteration.** Centralities are computed on the graph keeping only *intra-community* edges at each iteration (iteration 0 = original graph). On the full graph they would never change.
- **Three variants of the recursion:**
  - `assignment`: the handout's rule (plain restricted block `B^(C)`, stop when `λ₁ ≤ 0`).
  - `newman`: Newman's generalised matrix `B^(g)` (adds the diagonal correction).
  - `newman+dq`: as above, but also rejects any split whose exact `ΔQ ≤ 0`.

## Key results

| Result | Value |
|---|---|
| First split sizes | 16 + 18 |
| Modularity after first split | Q = 0.372 (λ₁ = 4.98) |
| Agreement with true factions | 33 / 34 nodes (ARI = 0.88); only node 8 misclassified |
| Final partition (handout rule) | 5 communities, Q = 0.409 |
| Final partition (Newman + ΔQ check) | 4 communities, Q = 0.393 |
| Best Q found by Louvain | 0.420 |
| Q of the true two-faction split | 0.358 |
| Significance vs. degree-matched null | z ≈ 3.1, p ≈ 0.006 |
| Most consistently central nodes | 33, 32 and 0 (top-5 for degree and betweenness at every iteration) |
| Rank stability vs. iteration 0 (Spearman, after 4 splits) | degree 0.84, betweenness 0.63, closeness 0.34 |

**Findings about the handout's method**

- The restricted block `B^(C)` is not Newman's generalised matrix, so `sᵀB^(C)s / 4m` is not the exact `ΔQ`, and the recursion tends to stop on one-sided splits rather than on `λ₁ ≤ 0`.
- `λ₁ > 0` is necessary but not sufficient for an improving split; checking `ΔQ > 0` explicitly fixes this.

See Section 7 of the notebook for the full discussion.

## References

- Newman, M. E. J. (2006). Modularity and community structure in networks. *PNAS*, 103(23), 8577–8582.
- Zachary, W. W. (1977). An information flow model for conflict and fission in small groups. *Journal of Anthropological Research*, 33(4), 452–473.
- NetworkX documentation: https://networkx.org

