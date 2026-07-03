# 🧩 Data Structures & Algorithms

[← Back to main README](../README.md)

> DS/ML interviews at larger tech companies often include a lighter DSA round than pure SWE roles, but it's still common. These focus on patterns that show up most often for data roles.

---

### Q: What is the time complexity of common operations on a Python dict/hash map, and why?

**Answer:**
Average case for **lookup, insert, and delete is O(1)**, thanks to hashing — the key's hash determines a bucket index directly, avoiding a linear scan. Worst case is **O(n)** if many keys collide into the same bucket (rare with a good hash function and load-factor-based resizing, but theoretically possible). This is why hash maps are the default choice for "have I seen this before" / frequency-counting problems instead of scanning a list repeatedly (O(n) per lookup).

---

### Q: Given an array of integers, find two numbers that add up to a target (Two Sum). What's the most efficient approach?

**Answer:**
Brute force nested loop is O(n²). The efficient approach uses a hash map to store values seen so far, achieving **O(n) time, O(n) space**:

```python
def two_sum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []
```
As you iterate, check if the complement needed to reach the target has already been seen — this avoids re-scanning the array for each element.

---

### Q: Explain the difference between BFS and DFS, and when you'd prefer one over the other.

**Answer:**
**BFS (Breadth-First Search)** explores level by level using a queue — ideal for finding the **shortest path** in an unweighted graph, since it guarantees the first time you reach a node is via the shortest route. **DFS (Depth-First Search)** explores as far as possible down one branch before backtracking, using a stack (or recursion) — better suited for exploring all paths, detecting cycles, topological sorting, or when memory is a concern (DFS can use less memory than BFS on wide graphs, since it doesn't need to hold an entire frontier level).

---

### Q: What is the time complexity of quicksort, both average and worst case, and why does the worst case happen?

**Answer:**
Average case: **O(n log n)** — each partition step is O(n), and with a good pivot choice the recursion depth is O(log n). Worst case: **O(n²)** — this happens when the pivot chosen is consistently the smallest or largest element (e.g., always picking the first element as pivot on an already-sorted array), causing maximally unbalanced partitions and O(n) recursion depth. Mitigation: **randomized pivot selection** or **median-of-three** pivot selection makes worst-case behavior extremely unlikely in practice.

---

### Q: How would you find the top-K most frequent elements in a large list efficiently?

**Answer:**
Naive approach: count frequencies (O(n)), then fully sort by frequency (O(n log n)). More efficient: use a **min-heap of size k** — iterate through frequency counts, push into the heap, and pop the smallest whenever the heap exceeds size k. This gives **O(n log k)**, better than O(n log n) when k is much smaller than n.

```python
from collections import Counter
import heapq

def top_k_frequent(nums, k):
    counts = Counter(nums)
    return heapq.nlargest(k, counts.keys(), key=counts.get)
```

---

### Q: What is dynamic programming, and how do you recognize when a problem can be solved with it?

**Answer:**
Dynamic programming solves problems by breaking them into overlapping subproblems, solving each subproblem once, and **caching (memoizing)** the result to avoid redundant recomputation — trading space for time. Recognize DP applicability when a problem has: (1) **optimal substructure** (the optimal solution can be built from optimal solutions to subproblems), and (2) **overlapping subproblems** (a naive recursive solution would recompute the same subproblem many times, as in naive recursive Fibonacci). If subproblems don't overlap, plain divide-and-conquer (no memoization needed) is more appropriate.

---

### Q: Explain what a heap is and why it's the right structure for a "streaming top-K" problem.

**Answer:**
A heap is a tree-based structure maintaining the min (or max) element at the root, supporting **O(log n)** insertion and removal of the min/max, and **O(1)** peek at the min/max. For a streaming top-K problem (continuously arriving data, need the K largest seen so far at any point), a **min-heap of size K** is ideal: for each new element, compare against the heap's minimum — if the new element is larger, pop the min and push the new element (O(log k) per update), keeping memory bounded to O(k) regardless of stream length, unlike sorting the entire stream each time.

---

### Q: What is the difference between an array and a linked list, in terms of time complexity tradeoffs?

**Answer:**
**Arrays** offer O(1) random access (index-based) but O(n) insertion/deletion in the middle (requires shifting elements), and contiguous memory gives better cache locality. **Linked lists** offer O(1) insertion/deletion at a known position (just pointer updates) but O(n) access to an arbitrary element (must traverse from the head), and worse cache performance due to non-contiguous memory. Choose arrays when you need frequent random access and know the size roughly in advance; choose linked lists when you need frequent insertions/deletions and don't need random access.

---

### Q: Given a large dataset that doesn't fit in memory, how would you find the median?

**Answer:**
If data is on disk/streaming and doesn't fit in memory, options include: (1) **external merge sort** then read the middle element(s) directly from the sorted file, or (2) maintain **two heaps** (a max-heap for the lower half, a min-heap for the upper half) while streaming through data once, keeping them balanced in size so the median is always derivable from the heap tops in O(log n) per insertion — useful if you need a running median as data streams in, not just a final answer. For a one-time approximate answer on massive data, **reservoir sampling** or **t-digest/quantile sketches** (used in systems like Spark) trade exactness for the ability to compute approximate quantiles in a single pass with bounded memory.

---

### Q: Explain Big-O notation and why worst-case analysis matters even if average case is usually better.

**Answer:**
Big-O describes how an algorithm's resource usage (time/space) scales as input size grows, focusing on the dominant term as n → ∞ (ignoring constants and lower-order terms). Worst-case matters because average-case performance can hide **pathological inputs** that cause severe slowdowns in production — e.g., quicksort's O(n²) worst case on adversarial/sorted input, or a hash map's O(n) worst case under intentional hash-collision attacks (a real security concern for public-facing APIs). Systems that need reliability guarantees (real-time systems, security-sensitive code) often specifically require worst-case bounded algorithms, not just good average-case behavior.

---

### Q: How would you detect a cycle in a linked list, and what's the space-efficient solution?

**Answer:**
The space-efficient solution is **Floyd's Cycle Detection (tortoise and hare)**: use two pointers, one moving one step at a time (slow) and one moving two steps at a time (fast). If there's a cycle, the fast pointer will eventually "lap" the slow pointer and they'll meet; if there's no cycle, the fast pointer reaches the end (`None`) first. This runs in **O(n) time, O(1) space** — versus a hash-set approach that also works in O(n) time but uses **O(n) space** to track visited nodes.

---

### Q: What's the difference between memoization and tabulation in dynamic programming?

**Answer:**
**Memoization** is top-down: write the natural recursive solution, but cache results (e.g., in a dict or array) so repeated calls with the same arguments return instantly instead of recomputing. **Tabulation** is bottom-up: iteratively build up a table of solutions starting from the base cases, without recursion. Memoization is often more intuitive to write and only computes subproblems that are actually needed; tabulation avoids recursion overhead/stack depth limits and can sometimes be optimized to use less space (e.g., only keeping the last 2 rows of a DP table instead of the whole table).

---
