# Competitive Programming Pattern Recognition Playbook

> No list is *truly* complete — there are hundreds of named algorithms. But roughly
> **55–65 triggers** below cover the overwhelming majority of AtCoder / Codeforces / ICPC
> problems (the original 35–45 core set, plus the specialized patterns that show up once you
> go past Div 2 D / rating 1600+). The goal isn't memorizing solutions — it's memorizing
> **triggers**: the moment a phrase or constraint appears, a technique should fire in your head.

---

## 1. The Constraint-First Mental Checklist

Run through these in order, every time, before writing code.

| # | Question | If the answer is... | Then think... |
|---|----------|---------------------|----------------|
| 1 | **What are the constraints (N, Q)?** | N ≤ 20 | Bitmask / brute force / meet-in-the-middle |
| | | N ≤ 500–1000 | O(N²) or O(N² log N) DP/graph |
| | | N ≤ 10⁵–10⁶ | O(N log N) or O(N) |
| | | N ≤ 10⁹+ | O(log N) / O(√N) / closed form / matrix power |
| 2 | **Count or optimize?** | Count | DP or combinatorics |
| | | Optimize (min/max) | Greedy or DP |
| 3 | **Is a graph hiding in the wording?** | cities, roads, flights, friends, dependencies, "can reach" | Graph traversal |
| 4 | **Does order/sequence matter?** | Yes | Sequence DP | 
| | | No | Combinatorics / sets |
| 5 | **Many queries on the same data?** | Yes | Preprocessing: prefix sums, sparse table, Fenwick/segment tree |
| 6 | **Is there a monotonic yes/no property as X increases?** | Yes (No No No Yes Yes Yes) | Binary search on the answer |
| 7 | **Does state evolve over steps/time?** | Yes | DP |
| 8 | **Building numbers digit by digit under a bound?** | Yes | Digit DP |
| 9 | **Is it a tree?** | Yes | Tree DFS / Tree DP / rerooting |
| 10 | **Repeated "are these connected / merge these" queries?** | Yes | DSU (Union-Find) |
| 11 | **Need the k-th something, or "at most/exactly K"?** | Yes | Binary search / DP with K as a dimension |
| 12 | **Contiguous subarray/substring condition?** | Yes | Prefix sum / two pointers / sliding window |
| 13 | **Asking for extremes over a moving window?** | Yes | Monotonic deque |

### Worked Examples — Checklist

**1a. N ≤ 20 → Bitmask/meet-in-the-middle**
*Problem:* N ≤ 20 tasks, each with a value, some pairs conflict — pick a max-value conflict-free subset.
*Approach:* `dp[mask]` = best value achievable using exactly the subset `mask`. Transition by trying to add each unset bit if it doesn't conflict with anything already in `mask`. 2²⁰ states is ~1M, fine at this N.

**1b. N ≤ 500–1000 → O(N²)**
*Problem:* Longest Common Subsequence of two strings, each length ≤ 1000.
*Approach:* `dp[i][j]` = LCS length using first `i` chars of A and first `j` chars of B. `dp[i][j] = dp[i-1][j-1]+1` if `A[i]==B[j]`, else `max(dp[i-1][j], dp[i][j-1])`. That's 10⁶ states — comfortably O(N²).

**1c. N ≤ 10⁵–10⁶ → O(N log N)/O(N)**
*Problem:* Given N ≤ 10⁵ jobs with deadlines and profits, maximize profit scheduling one job per day.
*Approach:* Sort jobs by profit descending (O(N log N)), then greedily place each job in the latest free slot ≤ its deadline using a DSU that jumps to the next free slot — the sort dominates the complexity.

**1d. N ≤ 10⁹+ → O(log N)/matrix power**
*Problem:* Compute the N-th Fibonacci number mod 1e9+7, N up to 10¹⁸.
*Approach:* Fibonacci recurrence is linear, so represent the transition as a 2×2 matrix and use fast matrix exponentiation — O(log N) matrix multiplications instead of O(N) additions.

**2a. Count → DP/combinatorics**
*Problem:* Count the number of ways to climb N stairs, taking 1 or 2 steps at a time.
*Approach:* `dp[i] = dp[i-1] + dp[i-2]` — this is literally Fibonacci in disguise. Counting problems almost always decompose into "ways to reach state i = sum of ways to reach states that can transition into i."

**2b. Optimize → Greedy/DP**
*Problem:* Minimum number of coins to make amount V, given arbitrary coin denominations.
*Approach:* Greedy (always take the largest coin) fails for non-canonical coin systems (e.g. {1,3,4} to make 6 — greedy gives 4+1+1=3 coins, optimal is 3+3=2). Use DP instead: `dp[v] = min(dp[v-c]+1)` over all coins `c`.

**3. Graph hiding in wording**
*Problem:* N people, some pairs are friends — count the number of friend groups.
*Approach:* Model people as nodes, friendships as edges. Either run DFS/BFS from every unvisited node and count components, or process the friend pairs with DSU and count distinct roots at the end.

**4a. Order matters → Sequence DP**
*Problem:* Longest strictly increasing subsequence (LIS) of an array.
*Approach:* `dp[i]` = length of LIS ending at index `i`, computed as `1 + max(dp[j])` for all `j<i` with `A[j]<A[i]` (O(N²)), or maintain a "tails" array with binary search for O(N log N).

**4b. Order doesn't matter → Combinatorics**
*Problem:* Count the number of ways to choose K items from N distinct items.
*Approach:* Direct formula `C(N,K) = N!/(K!(N-K)!)`, computed with precomputed factorials and modular inverses if a modulus is involved — no need to enumerate actual orderings.

**5. Many queries on the same data**
*Problem:* Array of 10⁵ elements, Q=10⁵ queries asking for the sum of range [l, r], no updates.
*Approach:* Build a prefix-sum array once in O(N); each query then answers in O(1) as `prefix[r]-prefix[l-1]`. If updates were also required, upgrade to a Fenwick tree for O(log N) per operation.

**6. Monotonic yes/no property**
*Problem:* Distribute N packages across K trucks; minimize the maximum load any single truck carries.
*Approach:* Binary search on the answer X ("can every truck carry ≤ X?"). The feasibility check is monotonic — if X works, anything larger also works — so binary search the smallest feasible X, checking each candidate with a linear greedy simulation.

**7. State evolves over time → DP**
*Problem:* Minimum cost to climb a staircase where each step has an associated cost, and you can move 1 or 2 steps at a time.
*Approach:* `dp[i] = cost[i] + min(dp[i-1], dp[i-2])` — the state (current step) evolves, and the optimal decision at each step only depends on the immediately preceding states.

**8. Digit DP**
*Problem:* Count numbers in [1, N] whose digit sum is divisible by 3.
*Approach:* DP over digit positions with state `(position, tight, digit_sum_mod_3)`, where `tight` tracks whether the prefix built so far is still equal to N's prefix (constraining which digits can be placed next).

**9. Tree**
*Problem:* Find the diameter (longest path between any two nodes) of a tree.
*Approach:* Run DFS/BFS from any node to find the farthest node `u`, then DFS/BFS from `u` to find the farthest node `v` — the distance `u` to `v` is the diameter. (Alternative: single DFS computing, per node, the two longest downward paths through it.)

**10. Connected/merge queries**
*Problem:* Process a stream of edges being added one at a time; after each edge, answer whether two given nodes are connected.
*Approach:* DSU (Union-Find) with union by rank/size and path compression — near-O(1) amortized per union or find, which a fresh BFS/DFS per query couldn't match.

**11. K-th something / at most K**
*Problem:* Find the K-th smallest element in an unsorted array.
*Approach:* Quickselect (partition like quicksort, recurse only into the side containing the K-th index) for O(N) average time, or a min-heap of size K for O(N log K).

**12. Contiguous subarray/substring condition**
*Problem:* Length of the longest substring without repeating characters.
*Approach:* Sliding window with a hash map of last-seen index per character. Expand the right pointer; when a repeat is found, jump the left pointer just past its previous occurrence.

**13. Extremes over a moving window**
*Problem:* Given an array and window size K, output the maximum of every window of size K.
*Approach:* Monotonic deque storing indices with strictly decreasing values. Pop from the back while the new element is larger (they can never be the answer again), pop from the front when it falls outside the window.

---

## 2. Master Trigger Table (by category)

### Arrays & Sequences

| If you see... | Think... | Technique |
|---|---|---|
| Sum/count over all subarrays | Precompute cumulative sums | Prefix Sum |
| Range update, point query (or vice versa) | Defer the update | Difference Array |
| Longest/shortest subarray satisfying a condition, monotonic window | Shrink/grow a window | Two Pointers / Sliding Window |
| Next greater/smaller element, largest rectangle in histogram | Maintain monotonic order | Monotonic Stack |
| Max/min in every sliding window | Evict useless ends | Monotonic Deque |
| Values compressed but only relative order matters | Sort + re-map | Coordinate Compression |
| Need to search a sorted or "sorted-like" (unimodal) space | Halve the space | Binary Search |
| Function first decreases then increases (or vice versa) | Search the peak | Ternary Search |
| Subset-sum-like with N ≤ ~40 | Split in half | Meet in the Middle |

### Worked Examples — Arrays & Sequences

**Prefix Sum**
*Problem:* Answer Q queries of "sum of elements from index l to r" on a static array.
*Approach:* Build `prefix[i] = A[0]+...+A[i-1]` once, O(N). Each query answers in O(1) via `prefix[r+1]-prefix[l]`.

**Difference Array**
*Problem:* Apply M range updates ("add v to every element in [l,r]"), then output the final array.
*Approach:* Instead of updating each range directly (O(N) per update), do `diff[l]+=v; diff[r+1]-=v` (O(1) per update). Take the prefix sum of `diff` once at the end to recover the final array in O(N).

**Two Pointers / Sliding Window**
*Problem:* Find the length of the smallest contiguous subarray with sum ≥ S.
*Approach:* Expand the right pointer, adding to a running sum. Once the sum ≥ S, shrink from the left as much as possible while the condition still holds, recording the minimum window length seen. Each pointer moves forward only, so it's O(N) total, not O(N²).

**Monotonic Stack**
*Problem:* Largest rectangle area in a histogram of bar heights.
*Approach:* Maintain a stack of indices with increasing bar heights. When a shorter bar appears, pop taller bars off the stack — each pop computes a candidate rectangle using the popped height and a width stretching back to the new stack top. Every bar is pushed and popped once → O(N).

**Monotonic Deque**
*Problem:* Maximum of every sliding window of size K in an array.
*Approach:* Same core idea as the checklist example above — keep a deque of indices with strictly decreasing values; the front is always the current window's max. O(N) overall since each index enters/leaves the deque once.

**Coordinate Compression**
*Problem:* Values up to 10⁹, but you need to index them into a Fenwick tree (which needs small, dense indices).
*Approach:* Collect all distinct values, sort them, and map each original value to its rank (1..N) in that sorted order. Use ranks wherever the original value would've been used as an index.

**Binary Search**
*Problem:* Find the position of a target value in a sorted array of 10⁵ elements.
*Approach:* Classic O(log N): compare target to the middle element, discard the half that can't contain it, repeat.

**Ternary Search**
*Problem:* A cost function f(x) strictly decreases then strictly increases (convex) — find the x that minimizes it.
*Approach:* Pick two interior points m1 < m2 in the current range; compare f(m1) and f(m2). Whichever is larger, its side of the range can be discarded (the minimum can't be there), shrinking the search space by ~1/3 each iteration.

**Meet in the Middle**
*Problem:* N ≤ 40 items with weights — does any subset sum to exactly target T?
*Approach:* Split the items into two halves of ~20 each. Enumerate all 2²⁰ subset sums of each half. Sort one half's sums, then for each sum in the other half, binary search for the complementary value (T − sum). Turns an infeasible 2⁴⁰ brute force into two feasible 2²⁰ passes.

### Graphs

| If you see... | Think... | Technique |
|---|---|---|
| Shortest path, unweighted | Level-by-level | BFS |
| Shortest path, weighted, non-negative | Greedy relax | Dijkstra |
| Shortest path, negative edges | Relax N−1 times | Bellman-Ford |
| All-pairs shortest path, small N | O(N³) DP | Floyd-Warshall |
| Connected components / merge groups | Union by rank/size | DSU (Union-Find) |
| Minimum cost to connect everything | Pick cheapest edges without cycles | Kruskal / Prim (MST) |
| Task ordering with prerequisites | No cycles allowed | Topological Sort (DAG) |
| Is the graph 2-colorable? | Check odd cycles | Bipartite Check (BFS/DFS coloring) |
| Assign pairs optimally, bipartite | Augmenting paths | Hungarian / Hopcroft-Karp Matching |
| Max amount that can flow through a network | Push flow along augmenting paths | Max Flow (Ford-Fulkerson/Dinic) |
| Flow with costs attached | Cheapest augmenting path | Min-Cost Max-Flow |
| "Either A or not-B" style boolean constraints | Implication graph | 2-SAT |
| Cut vertices/bridges, biconnected components | DFS with low-link values | Tarjan's Algorithm |
| Strongly connected components | Two DFS passes or low-link | Kosaraju / Tarjan SCC |

### Trees

| If you see... | Think... | Technique |
|---|---|---|
| Any general "tree problem" | Root it, then recurse | Tree DFS / Tree DP |
| Answer depends on both subtree and "rest of tree" | Two-pass DFS | Rerooting Technique |
| Repeated "lowest common ancestor" queries | Precompute jumps | Binary Lifting (LCA) |
| Path queries, subtree queries on a tree | Linearize the tree | Euler Tour + Segment/Fenwick Tree |
| Path queries with updates, need O(log²N) | Break into chains | Heavy-Light Decomposition |
| Repeated "distance between two nodes" on a static tree, or divide-and-conquer over tree paths | Decompose around centroids | Centroid Decomposition |
| Persist old versions of a tree/array across updates | Reuse unchanged nodes | Persistent Segment Tree |

### Dynamic Programming

| If you see... | Think... | Technique |
|---|---|---|
| Count the number of ways | Sum over choices | Counting DP |
| Multiple decisions made in sequence with overlapping subproblems | Memoize state | Classic DP |
| N ≤ ~20, subsets of items matter | State = bitmask | Bitmask DP |
| Count numbers ≤ N with a digit property | Build digit by digit with a "tight" flag | Digit DP |
| Two sequences compared (edit distance, LCS, subsequence) | 2D grid DP | Sequence-Pair DP |
| Knapsack-flavored ("choose items under a budget") | Weight/value table | Knapsack DP |
| DP transition is a linear recurrence and N is huge (10⁹) | Represent transition as a matrix | Matrix Exponentiation |
| Optimal split points, and cost function is "well-behaved" (quadrangle inequality) | Optimal split is monotonic | Divide & Conquer / Knuth Optimization |
| DP with a sliding-window-like transition (min/max of a range of prior states) | Maintain monotonic candidates | Monotonic Deque Optimization |
| Convex-hull-shaped transitions | Maintain lower/upper envelope of lines | Convex Hull Trick / Li Chao Tree |
| Randomness or expectation involved | E[X] recurrence | Probability / Expected Value DP |
| Game where two players alternate optimally | Win/lose states | Game Theory DP |

### Strings

| If you see... | Think... | Technique |
|---|---|---|
| Find one pattern inside a text | Avoid re-scanning | KMP |
| Compare a string to all its own prefixes/suffixes | Z-function | Z-Algorithm |
| Search many patterns at once inside one text | Build a trie with failure links | Aho-Corasick |
| Compare/hash substrings quickly | Precompute rolling hash | Polynomial/Rolling Hash |
| Palindrome-related | Expand from center, or DP, or linear-time for all palindromes | Manacher's Algorithm |
| Lexicographic ordering of all suffixes, repeated substring problems | Sort suffixes | Suffix Array (+ LCP array) |
| Prefix-based storage/search of many strings | Shared prefix tree | Trie |

### Range Data Structures

| If you see... | Think... | Technique |
|---|---|---|
| Static array, repeated min/max/gcd queries, no updates | Precompute powers of 2 | Sparse Table |
| Point updates + range sum/min queries | Binary indexed structure | Fenwick Tree (BIT) |
| Range updates + range queries, more general ops | Tree over the array | Segment Tree |
| Range update should be lazy, not applied immediately to every leaf | Defer and push down | Lazy Propagation |
| 2D grid, point/range updates and queries | Extend BIT to 2 dimensions | 2D Fenwick Tree |
| Queries can be answered offline, sorted cleverly | Reorder queries | Offline Processing + Sorting |
| Many range queries, updates are rare/absent, offline is fine | Block size ≈ √N | Mo's Algorithm |
| Data too large for one structure, updates are batched | Split into blocks | Sqrt Decomposition |
| Repeated "merge the smaller set into the larger" | Amortized merging | Small-to-Large (DSU on Tree) |

### Number Theory & Combinatorics

| If you see... | Think... | Technique |
|---|---|---|
| Answers required modulo a prime | Careful mod arithmetic | ModInt / Fast Exponentiation |
| nCr / nPr modulo a prime, many queries | Precompute once | Factorials + Modular Inverse |
| GCD/LCM-heavy problems | Euclid's algorithm | GCD/Extended Euclid |
| Solve x ≡ a (mod m) systems | Combine moduli | Chinese Remainder Theorem |
| Count via overlapping bad cases | Add/subtract overlaps | Inclusion-Exclusion |
| Counting coprime pairs / multiplicative functions | Sieve-based function | Möbius Function / Sieve of Eratosthenes |
| Multiply two large polynomials fast | Transform to point-value form | FFT / NTT |
| Two players remove objects optimally (Nim-like) | XOR of pile sizes | Sprague-Grundy / Nim Theory |

### Geometry

| If you see... | Think... | Technique |
|---|---|---|
| Orientation of 3 points, turning left/right | Sign of cross product | Cross Product |
| Smallest polygon containing all points | Sort by angle, sweep | Convex Hull |
| Do two segments intersect? | Orientation tests | Segment Intersection |
| Sweep over events left to right | Sorted event processing | Sweep Line |

---

## 3. Phrase → Pattern Quick Recognition

| Problem phrase | Pattern that should fire |
|---|---|
| "Count numbers ≤ N divisible by / containing digit X" | Digit DP |
| "Maximum profit, buy/sell once or K times" | Greedy / DP |
| "Shortest path from A to B" | Dijkstra / BFS |
| "Can we split into K parts satisfying condition X?" | Binary Search + Greedy/DP check |
| "Q updates and queries, Q up to 10⁵" | Segment Tree / Fenwick Tree |
| "Exactly 15–20 items, choose a subset" | Bitmask DP |
| "Largest rectangle / span" | Monotonic Stack |
| "Maximum of every window of size K" | Monotonic Deque |
| "How many connected components after each edge?" | DSU |
| "Distance between two nodes in a tree, many queries" | LCA (Binary Lifting) or Euler Tour + RMQ |
| "First occurrence of pattern P in text T" | KMP / Z-function |
| "Is this string a permutation of a palindrome?" | Frequency count / bitmask parity |
| "Minimum spanning network" | Kruskal / Prim |
| "Two players play optimally, who wins?" | Game theory / Grundy numbers |
| "N up to 10⁹, but recurrence is linear" | Matrix Exponentiation |
| "Assign workers to tasks minimizing cost" | Hungarian Algorithm / Min-Cost Flow |

---

## 4. What Experts Actually Memorize (not solutions — triggers)

- Sum over every subarray → **Prefix Sum**
- Exactly K operations allowed → **DP with K as a state dimension**
- Minimum cost to go from A to B → **Graph shortest path**
- Parent-child relationships → **Tree**
- Count numbers ≤ N with a property → **Digit DP**
- Online range updates → **Lazy Segment Tree**
- Repeated merging of groups → **Union-Find**
- "Smallest X such that condition holds" and condition is monotonic → **Binary search on the answer**
- Two players, optimal play → **Game DP / Grundy theory**
- "Nearest greater element to the left/right" → **Monotonic Stack**

---

## 5. Learning Roadmap (order matters)

**Foundation (do these cold):**
1. Time Complexity & Big-O
2. Prefix Sum & Difference Array
3. Two Pointers & Sliding Window
4. Binary Search (on array + on answer)
5. Sorting & Greedy
6. Stacks, Queues, Monotonic Stack/Deque
7. Hash Maps & Sets
8. Recursion & Backtracking

**Core (this is where most rating gains happen):**
9. Graph Traversals (DFS/BFS)
10. Trees & Tree DP
11. Dynamic Programming (1D → 2D → bitmask)
12. Digit DP
13. Union-Find (DSU)
14. Shortest Paths (Dijkstra, Bellman-Ford, Floyd-Warshall)
15. Minimum Spanning Trees
16. Segment Trees & Fenwick Trees (+ lazy propagation)
17. String Algorithms (KMP, Z, Trie, hashing)
18. Number Theory & Modular Arithmetic

**Advanced (push past 1900–2000+ rating):**
19. Binary Lifting / LCA, Euler Tour techniques
20. Heavy-Light Decomposition, Centroid Decomposition
21. Sqrt Decomposition, Mo's Algorithm, Small-to-Large
22. Flows (Max Flow, Min-Cost Max-Flow), Bipartite Matching
23. FFT/NTT, Convex Hull Trick
24. Game Theory (Grundy numbers), 2-SAT
25. Persistent Data Structures, Suffix Arrays/Automata

---

## 6. Practical Habit

After solving any problem, don't just save the code. Write **one line**: the exact phrase or
constraint that should have triggered the technique. Over months this becomes your own
searchable trigger index — and it's a genuinely good thing to keep versioned alongside a
DS/ML-style reference repo, since the format (trigger → technique) is identical.
