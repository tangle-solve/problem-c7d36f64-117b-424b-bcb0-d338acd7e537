<!-- solve-problem:c7d36f64-117b-424b-bcb0-d338acd7e537 -->
# Linear-time directed nonnegative shortest paths

A [Tangle Solve problem](https://solve.tangle.tools/problems/c7d36f64-117b-424b-bcb0-d338acd7e537).

## What is the problem?

Design a deterministic algorithm for the **single-source shortest path (SSSP)** problem: given a directed graph \(G=(V,E)\) with \(n\) vertices, \(m\) edges, and nonnegative real edge weights, and a source \(s\), compute \(\mathrm{dist}(s,v)\) for all \(v\in V\), in \(O(m+n)\) time in the **comparison-addition model** (edge weights are real numbers; the only operations allowed on weights are additions and comparisons; integer/word-RAM tricks are not available).

This is Dijkstra's problem: the classical algorithm runs in \(O(m + n\log n)\) (Fibonacci heaps), and the "sorting barrier" intuition says that any algorithm must essentially sort the vertices by distance, hence cost \(\Omega(m + n\log n)\) — unless one can exploit combinatorial structure to avoid fully sorting. The goal: remove the logarithmic factor entirely while remaining deterministic and comparison-based. Ambiguity note: some literature allows randomization or integer weights; the census target here is explicitly deterministic, real weights, worst case \(O(m+n)\). The output is distance values (and implicitly a shortest-path tree); explicit path reconstruction in \(O(m+n)\) should also hold.

## Why does it matter?

SSSP with nonnegative weights is among the most-used primitives in computing (network routing, road/transportation navigation, scheduling, flow algorithms). Beating the classical bound removes a logarithmic overhead from a ubiquitous operation, and would likely transfer to many downstream algorithms (matching, flow, centrality). It is also a flagship question in the theory of "sorting lower bounds don't apply to graph problems": linear time would complete a picture of shortest paths as strictly easier than sorting in directed graphs. Practitioners at scale (large-scale routing, as the census notes) care because near-linear exact algorithms are increasingly the gold standard.

## What's already known?

- **Dijkstra (1959)** + Fibonacci heaps (Fredman & Tarjan 1987): \(O(m+n\log n)\), optimal if the algorithm must sort the distances.
- **Thorup (1999):** linear-time SSSP for **undirected** graphs with nonnegative **integer/real** weights in the word-RAM / pointer machine setting — showing the barrier is beatable, but only undirected and not comparison-addition.
- **Haeupler et al.** gave a (verify) comparison-based undirected SSSP breaking the sorting barrier.
- **Duan, Jiayi Mao, Xiao Mao, Shu, and Yin (2025)** broke the sorting barrier for **directed** graphs: \(O(m\log^{2/3} n)\)-type deterministic comparison-addition time (up to lower-order terms; verify exact exponent/dependencies). Key idea: hierarchical clustering/components that allow partial rather than full ordering, recursively handling small-diameter components and certifying approximate orderings cheaply. This is the state of the art and the direct starting point.
- Known lower bound: only the trivial \(\Omega(m+n)\); no super-linear comparison-addition lower bound is known, so linear time is not ruled out.

Why current approaches fall short: the Duan et al. recursion incurs logarithmic overhead through repeated approximate-sorting/clustering passes; nobody has shown the hierarchy can be flattened into a constant number of linear phases.

## What would count as solving it?

1. **Full solve (upper bound):** a proof that a deterministic comparison-addition algorithm computes exact SSSP on arbitrary directed nonnegative-weight graphs in worst-case \(O(m+n)\) time. Referee criterion: full correctness proof plus running-time analysis matching Dijkstra's output.
2. **Partial credit variants:**
   - \(O(m + n\,\mathrm{polylog}\,n)\) improvements below the 2025 bound, e.g. \(O(m\log\log n)\) or \(O(m + n\log\log n)\) — meaningful steps.
   - A super-linear lower bound \(\Omega(m + n f(n))\) in the comparison-addition model for directed SSSP — resolves the question in the negative direction.
   - Randomized linear time (Las Vegas) would also be a major, clearly publishable milestone.
3. **Implementation is not required** — this is a theory target — though experiments demonstrating a practical hierarchy-flattening heuristic would be of interest.

## How to start

1. **Read arXiv:2504.17033 first, in full**, especially the sections on the hierarchical decomposition and how the \(\log^{2/3} n\) arises from the recursion; identify exactly where each log factor is paid (approximate sorting, cluster refinement, recomputation across levels).
2. Study **Thorup's undirected linear-time algorithm** to understand which undirected tricks (component-based shortcutting) fail for directed graphs; the gap between directed and undirected is the crux.
3. Natural approach: try to make the 2025 hierarchy **level-reducing** — e.g., replace recursive approximate sorting with a single pass that certifies enough order information using the graph structure (small-diameter components can be finalized without global order). Consider whether a two-level scheme or incremental "hint" reuse across levels can shave the exponent toward 0.
4. Main obstacle: without sorting, one cannot cheaply extract the minimum from the frontier; any progress must show the graph supplies combinatorial substitutes for priority-queue ordering. A lower-bound attempt (reductions from sorting in adversarial graph gadgets) is the dual entry point.

## Key references

- R. Duan, Jiayi Mao, Xiao Mao, X. Shu, L. Yin. *Breaking the Sorting Barrier for Directed Single-Source Shortest Paths.* arXiv:2504.17033, DOI: 10.1145/3717823.3718179 (STOC 2025; verify venue). Gives the first deterministic comparison-addition \(O(m\log^{2/3} n)\)-type (verify exact form) directed SSSP algorithm beating \(O(m+n\log n)\). https://arxiv.org/abs/2504.17033
- E. W. Dijkstra (1959). *A note on two problems in connexion with graphs.* Classical \(O(n^2)\)/priority-queue algorithm; the baseline being improved.
- M. L. Fredman, R. E. Tarjan (1987). *Fibonacci heaps and their uses in improved network optimization algorithms.* \(O(m+n\log n)\) Dijkstra via Fibonacci heaps; the "sorting barrier" benchmark.
- M. Thorup (1999). *Undirected single-source shortest paths with positive integer weights in linear time.* Linear-time SSSP for undirected graphs in the word-RAM/pointer-machine model; the existence proof that the barrier is beatable (different model and direction).

## Warnings

- Do not confuse models: Thorup's linear-time result uses word parallelism/integer tricks and **undirected** graphs; the census target is the **comparison-addition** model with **real** weights on **directed** graphs. Any claimed solution must honestly account for every weight operation.
- The 2025 bound's exact form (\(\log^{2/3}n\) vs. variants, additive vs. multiplicative \(n\)-terms) should be verified from the paper before building on it.
- Randomized or integer-weight linear-time results (if any exist; verify) do not solve this problem.
- A common dead end: attempting to fix Dijkstra with a cleverer heap — the sorting barrier argument shows pure priority-queue approaches cannot beat \(n\log n\); the algorithm must avoid ever fully ordering the distances.
- Explicit path reconstruction must not silently cost extra; the shortest-path tree should be a by-product.

---
*`ALGO-000119` · unverified*

## Acceptance guidance

1. **Full solve (upper bound):** a proof that a deterministic comparison-addition algorithm computes exact SSSP on arbitrary directed nonnegative-weight graphs in worst-case \(O(m+n)\) time. Referee criterion: full correctness proof plus running-time analysis matching Dijkstra's output.
2. **Partial credit variants:**
   - \(O(m + n\,\mathrm{polylog}\,n)\) improvements below the 2025 bound, e.g. \(O(m\log\log n)\) or \(O(m + n\log\log n)\) — meaningful steps.
   - A super-linear lower bound \(\Omega(m + n f(n))\) in the comparison-addition model for directed SSSP — resolves the question in the negative direction.
   - Randomized linear time (Las Vegas) would also be a major, clearly publishable milestone.
3. **Implementation is not required** — this is a theory target — though experiments demonstrating a practical hierarchy-flattening heuristic would be of interest.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/c7d36f64-117b-424b-bcb0-d338acd7e537?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: ALGO-000119.

- [Public source catalog](https://drewstone.github.io/problems/#/p/ALGO-000119)
