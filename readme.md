# Tempo Take-Home Assessment

- `hierarchy/`: implementation of `Hierarchy.filter()` with tests (Kotlin)
- `code-review/`: review of `SimpleCache` ([Issues.md](code-review/Issues.md))

## Hierarchy filter

**Approach:** a single pass in DFS order. When a node fails the predicate, its depth is remembered and all following deeper nodes (its subtree) are skipped. Skipping stops at the next node with the same or a smaller depth.

- O(n) time, O(n) space for the result
- No recursion, and the predicate is not called for nodes in a removed subtree
- Assumes the input follows the depth invariants; the original hierarchy is not modified

**Run:** paste `hierarchy/Take Home Assessment - Hierarchy.kt` into https://play.kotlinlang.org and click **Run**. `main()` runs all tests and prints the results.

## Code review

Main issues: memory leak (expired entries never removed), no size limit, cache stampede, misleading `size()`, and ambiguous `null` from `get()`. Details are in [Issues.md](code-review/Issues.md).
