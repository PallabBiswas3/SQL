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
