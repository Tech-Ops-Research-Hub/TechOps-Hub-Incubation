**Duration:** 2 hours (120 minutes)

**Total Marks:** 100

**Allowed languages/tools:** Any mainstream language (Python, Java, C++, JavaScript/TypeScript, Go). Provide runnable code snippets and brief test cases. Pseudo-code allowed only when implementation is impractical within time.

**Environment:** Local development environment or online judge. Write code with clear function signatures, include time/space complexity analysis, and at least one test case per coding problem.

**Submission:** Single repository or a single markdown file containing answers, code files, and readme. Name files clearly (e.g., `partA_q1.py`, `partC_design.md`). Include instructions to run code.

---

## Instructions (hard rules)

* Complete all questions.
* For coding problems: include implementation + complexity analysis + 1 example run.
* For design problems: include architecture outline (ASCII or bullet), data model, API endpoints, chosen data structures, bottlenecks, scaling plan, and trade-offs.
* Show reasoning. Marks awarded for correctness, clarity, and justification.

---

## Marking breakdown

* Part A — Coding (40 marks)
* Part B — Algorithmic reasoning (20 marks)
* Part C — System Design (30 marks)
* Part D — Integration (10 marks)

---

# Part A — Coding (40 marks) — 4 problems, 10 marks each

**Requirements for each problem:**

* Correctness and edge cases (6 marks)
* Time/space complexity analysis (2 marks)
* One test case showing input and output (2 marks)

### A1 (10 marks) — Kth largest in a stream

Implement a class `KthLargest(k)` supporting:

* `add(num)` — add integer to stream
* `query()` — return current k-th largest element among all seen so far

Expected: maintain efficient structure (min-heap or equivalent). Provide O(log k) update.

Example:

```
S = KthLargest(3)
S.add(3)
S.add(1)
S.add(5)
S.query() -> 1  # 3rd largest among [3,1,5]
```

---

### A2 (10 marks) — Shortest path in weighted graph

Given a directed graph with positive weights, implement shortest-path from source `s` to all nodes using Dijkstra’s algorithm. Provide complexity analysis and demonstration on a small graph (≤6 nodes).

---

### A3 (10 marks) — Interval overlap queries

Implement a data structure to:

* `insert(interval)`
* `query(point)` -> list of intervals containing `point`

Balanced interval handling can be approximate; segment tree or sorted list with binary search is acceptable. Provide complexity analysis.

---

### A4 (10 marks) — String edit distance

Given strings `s` and `t`, compute minimal operations to transform `s` to `t` using insert, delete, and replace. Implement dynamic programming algorithm and explain complexity.

---

# Part B — Algorithmic reasoning (20 marks)

Each question 5 marks: 3 marks for correct conclusion, 2 marks for justification.

### B1 (5 marks)

Given a hash table using chaining and load factor α = n/m. Under what conditions will average lookup be O(1)?

### B2 (5 marks)

Explain why union-find with path compression + union by rank is nearly O(α(n)) per operation.

### B3 (5 marks)

For a balanced B-tree of order `t`, derive complexity for search, insert, and delete. Explain why B-trees are used in databases.

### B4 (5 marks)

Solve the recurrence `T(n)=2T(n/2)+n`. State asymptotic bound.

---

# Part C — System Design (30 marks)

Answer both questions. For each: 10 marks for architecture & data model, 5 marks for scalability & trade-offs, 5 marks for mapping to data structures.

### C1 (15 marks) — Design a notification system

Requirements:

* Deliver notifications to thousands of users with low latency.
* Support push and pull.
* Basic deduplication.

Provide: high-level architecture, core data model, delivery pipeline, and scaling strategy.

### C2 (15 marks) — Design search autocomplete

Requirements:

* Provide prefix suggestions (≤50ms p95) for ≤1M terms.
* Rank by frequency.
* Handle typo tolerance (basic edit distance).

Provide: data structure choice (trie or hash), caching approach, and simple ranking strategy.

---

# Part D — Integration (10 marks)

### D1 (10 marks) — Implement core function for autocomplete

Implement function `top_k_suggestions(prefix, K)` that given a dictionary-based structure returns top-K suggestions ranked by frequency.

Provide:

* Implementation (5 marks)
* Complexity analysis (3 marks)
* Discussion (2 marks) on how to scale beyond 1M terms.

---

# Grading rubric (explicit)

* Exact correctness: 50% of coding marks.
* Edge cases and robustness: 20% of coding marks.
* Complexity and justification: 20% of each question's marks.
* Quality of explanations and design trade-offs: proportionally applied in Parts B, C, D.

---

# Examiner notes

* Deduct points for missing complexity analysis.
* For design questions, diagrams may be ASCII boxes or bulleted outlines.
* Partial credit awarded for sound reasoning and trade-offs even if code is incomplete.

---

End of test
