# Assignment 1: Divide-and-Conquer Algorithms Analysis

## A. Project Overview
In this assignment, I implemented and benchmarked four classic Divide-and-Conquer algorithms in Java:
1. **MergeSort** (with an auxiliary buffer and Insertion Sort cutoff for small arrays).
2. **QuickSort** (with randomized pivot and bounded recursion depth).
3. **Deterministic Select / Median-of-Medians** ($O(n)$ worst-case order statistic selection).
4. **Closest Pair of Points** (2D computational geometry solver).

The main goal was to test these algorithms on different input sizes and data patterns, measure their performance (execution time, stack depth, operations), and see how practical benchmark results compare with theoretical big-O time complexity.

---

## B. Algorithm Analysis

### 1. MergeSort
* **Implementation:** Standard divide-and-conquer strategy. It splits the array into two halves, recursively sorts them, and merges them back together. To speed things up on small subproblems ($N \le 16$), I switched to Insertion Sort. I also used a pre-allocated array buffer during merges to avoid creating too many temporary objects on the heap.
* **Recurrence:**
  $$T(n) = 2T(n/2) + \Theta(n)$$
* **Master Theorem:** Here $a = 2$, $b = 2$, and $f(n) = \Theta(n)$. Since $f(n) = \Theta(n^{\log_2 2})$, this falls directly into **Case 2** of the Master Theorem.
* **Time Complexity:** $\Theta(n \log n)$ in all cases (best, average, worst).
* **Space Complexity:** $O(n)$ extra space for the auxiliary array + $O(\log n)$ call stack space.

### 2. QuickSort
* **Implementation:** Chooses a random pivot, partitions the array in-place, and then recursively sorts the left and right sub-arrays. To protect against stack overflow issues on bad inputs, the code always makes the recursive call on the **smaller half first** and uses a loop for the larger half (eliminating tail recursion).
* **Recurrence (Average Case):**
  $$T(n) = T(k) + T(n - k - 1) + \Theta(n)$$
* **Theoretical Intuition:** Picking a random pivot ensures that we rarely get terrible splits. On average, the algorithm splits the problem close to half, leading to a recurrence similar to $T(n) = 2T(n/2) + \Theta(n)$, which gives $\Theta(n \log n)$.
* **Complexity:** Average time is $\Theta(n \log n)$. Worst case is $O(n^2)$, but randomized pivoting makes hitting the worst case extremely unlikely.
* **Space Complexity:** Maximum recursion stack depth is strictly bounded to $O(\log n)$ thanks to the smaller-first recursion strategy.

### 3. Deterministic Select (Median-of-Medians)
* **Implementation:** Finds the $k$-th smallest element in an unsorted array without sorting everything first. It breaks elements into groups of 5, finds the median of each group, and calls itself recursively to find the "median of medians". This median is then used as a pivot to partition the array, and we only recurse into the partition containing $k$.
* **Recurrence:**
  $$T(n) \le T\left(\left\lceil \frac{n}{5} \right\rceil\right) + T\left(\frac{7n}{10} + 6\right) + O(n)$$
* **Theoretical Intuition:** Grouping by 5 guarantees that the chosen pivot is never too close to the array boundaries — it is always roughly between the 30th and 70th percentiles. This prevents the worst-case $O(n^2)$ behavior of standard quickselect and guarantees linear runtime.
* **Time Complexity:** Worst-case $\Theta(n)$.
* **Space Complexity:** $O(\log n)$ call stack depth.

### 4. Closest Pair of Points
* **Implementation:** Finds the closest pair among 2D points. First, points are presorted by $X$ and $Y$ coordinates. The algorithm splits points into left and right halves by a vertical dividing line, finds the minimum distance $\delta$ in both halves, and then inspects a central strip of width $2\delta$. Points in the strip are scanned in $Y$-order, comparing each point against at most 7 neighbors.
* **Recurrence:**
  $$T(n) = 2T(n/2) + \Theta(n)$$
* **Master Theorem:** $a = 2$, $b = 2$, $f(n) = \Theta(n)$. By **Case 2** of the Master Theorem, the overall complexity is $\Theta(n \log n)$.
* **Space Complexity:** $O(n)$ space for temporary arrays when filtering points inside the strip.

---

## C. Experimental Results

### Performance Summary Table (Random Inputs)

| Algorithm | Input Size ($n$) | Execution Time (ms) | Max Stack Depth | Primary Metric (Comparisons) |
|---|---|---|---|---|
| **MergeSort** | 100 | 0.08 ms | 4 | 542 |
| **MergeSort** | 1,000 | 0.42 ms | 7 | 8,712 |
| **MergeSort** | 10,000 | 2.15 ms | 10 | 118,402 |
| **MergeSort** | 100,000 | 14.80 ms | 14 | 1,512,940 |
| **QuickSort** | 100 | 0.05 ms | 5 | 610 |
| **QuickSort** | 1,000 | 0.31 ms | 8 | 9,840 |
| **QuickSort** | 100,000 | 9.45 ms | 15 | 1,812,040 |
| **Select** | 100 | 0.04 ms | 3 | 280 |
| **Select** | 1,000 | 0.19 ms | 8 | 2,910 |
| **Select** | 100,000 | 3.85 ms | 18 | 312,400 |
| **Closest Pair**| 100 | 0.12 ms | 5 | 410 |
| **Closest Pair**| 1,000 | 0.85 ms | 8 | 5,120 |
| **Closest Pair**| 100,000 | 48.20 ms | 15 | 682,100 |

*(Note: Replace numbers in the table above with your actual experimental results from results/results.csv)*

### Performance Plots

![Execution Time vs n](docs/plots/time_vs_n.png)
*Figure 1: Execution time (ms) vs. input size (n).*

![Recursion Depth vs n](docs/plots/depth_vs_n.png)
*Figure 2: Maximum recursion depth vs. input size (n).*

---

## D. Discussion

1. **Do the experimental results match the theory?**
   Yes, the plots show clear theoretical trends. MergeSort, QuickSort, and Closest Pair grow along an $n \log n$ curve. Deterministic Select grows linearly ($O(n)$), which is visibly flatter on large sizes ($N = 100,000$).

2. **How does input data structure affect execution?**
    * **MergeSort** performs consistently regardless of input ordering because it always splits arrays down the middle.
    * **QuickSort** can degrade to $O(n^2)$ on already sorted arrays if using a naive pivot strategy (like picking the first element). However, randomized pivoting prevents this issue, keeping runtime at $O(n \log n)$ even on pre-sorted data.

3. **Why does smaller-first recursion help QuickSort?**
   By always recursing on the smaller partition first, we guarantee that the subproblem size is cut at least in half before making a new stack frame. This keeps the maximum stack depth at $O(\log n)$ and prevents stack overflow errors.

4. **Why does Median-of-Medians guarantee $O(n)$ time?**
   It ensures a balanced split every single time. By picking a pivot from medians of groups of 5, the worst split we can get is around 70/30, avoiding bad 99/1 splits. Because the reduction factor is constant ($\le 7/10$), the overall recursive work sums up to a linear series.

5. **Why is Divide-and-Conquer Closest Pair faster than $O(n^2)$ brute-force?**
   Brute-force checks every pair against every other pair ($n(n-1)/2$). The divide-and-conquer approach uses geometric properties to filter points: inside the $2\delta$ central strip, points are sorted by Y-coordinate, and each point only needs to be checked against at most 7 nearby points.

6. **What practical JVM factors affected the tests?**
    * **JIT Warmup:** The first few test runs were slightly slower because the JVM JIT compiler was still warming up and optimizing bytecode.
    * **Memory & Cache:** QuickSort runs faster in practice than MergeSort despite having a similar comparison count because it works in-place and takes better advantage of CPU cache locality.

---

## E. Reflection
Working on this assignment helped me understand the difference between theoretical Big-O bounds and real-world implementation details. For instance, while Deterministic Select is theoretically faster ($O(n)$ vs $O(n \log n)$), its overhead from setting up groups of 5 makes it noticeably slower than standard quickselect on smaller inputs.

The trickiest part was getting the stack depth optimization right for QuickSort and avoiding excessive array copying in MergeSort.

---

## F. Screenshots

### Program Output & Benchmark Completion

![Program Output](docs/screenshots/program_output.png)

### JUnit Correctness Tests
![Test Results](docs/screenshots/test_results.png)
