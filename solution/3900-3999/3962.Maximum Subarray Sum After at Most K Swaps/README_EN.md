---
comments: true
difficulty: Hard
rating: 2672
source: Weekly Contest 506 Q4
tags:
    - Greedy
    - Binary Indexed Tree
    - Array
    - Hash Table
    - Ordered Set
    - Sorting
    - Heap (Priority Queue)
---

<!-- problem:start -->

# [3962. Maximum Subarray Sum After at Most K Swaps](https://leetcode.com/problems/maximum-subarray-sum-after-at-most-k-swaps)

[中文文档](/solution/3900-3999/3962.Maximum%20Subarray%20Sum%20After%20at%20Most%20K%20Swaps/README.md)

## Description

<!-- description:start -->

<p>You are given an integer array <code>nums</code> and an integer <code>k</code>.</p>

<p>You are allowed to perform <strong>at most</strong> <code>k</code> swap operations on the array.</p>

<p>In one swap operation, you may choose any two indices <code>i</code> and <code>j</code> and swap <code>nums[i]</code> and <code>nums[j]</code>.</p>

<p>Return an integer denoting the <strong>maximum possible <span data-keyword="subarray-nonempty">subarray</span> sum</strong> after performing the swaps.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<div class="example-block">
<p><strong>Input:</strong> <span class="example-io">nums = [1,-1,0,2], k = 1</span></p>

<p><strong>Output:</strong> <span class="example-io">3</span></p>

<p><strong>Explanation:</strong></p>

<ul>
	<li>We can swap on indices 1 and 3, resulting in the array <code>[1, 2, 0, -1]</code>.</li>
	<li>The subarray <code>[1, 2]</code> has a sum of 3, which is the maximum possible subarray sum after at most <code>k = 1</code>​​​​​​​ swap.</li>
</ul>
</div>

<p><strong class="example">Example 2:</strong></p>

<div class="example-block">
<p><strong>Input:</strong> <span class="example-io">nums = [4,3,2,4], k = 2</span></p>

<p><strong>Output:</strong> <span class="example-io">13</span></p>

<p><strong>Explanation:</strong></p>

<p>The maximum possible subarray sum after at most <code>k = 2</code> swaps is the sum of the entire array, which is 13.</p>
</div>

<p><strong class="example">Example 3:</strong></p>

<div class="example-block">
<p><strong>Input:</strong> <span class="example-io">nums = [-1,-2], k = 0</span></p>

<p><strong>Output:</strong> <span class="example-io">-1</span></p>

<p><strong>Explanation:</strong></p>

<ul>
	<li><code>k = 0</code> swaps are allowed.</li>
	<li>The possible subarrays are <code>[-1]</code>, <code>[-2]</code>, and <code>[-1, -2]</code>, with sums -1, -2, and -3 respectively.</li>
	<li>Among these sums, the maximum is -1.</li>
</ul>
</div>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= nums.length &lt;= 1500</code></li>
	<li><code>-10<sup>5</sup> &lt;= nums[i] &lt;= 10<sup>5</sup></code></li>
	<li><code>0 &lt;= k &lt;= nums.length</code></li>
</ul>

<!-- description:end -->

## Solutions

<!-- solution:start -->

### Solution 1

<!-- thinking:start -->

> **Thinking**
>
> At most $k$ arbitrary swaps means we may replace the smallest entries of some subarray by the largest entries outside it. $n\le 1500$ lets us enumerate $[l,r]$ and swap up to $k$ inside-minima with outside-maxima.
>
> For each segment take those two $k$-sets, sort, and apply the improving prefix. A heap can maintain them while the right end grows.
>
> This directory has no implemented solution yet; the walkthrough stops at “enumerate a segment plus a top-$k$ exchange”.

<!-- thinking:end -->

<!-- tabs:start -->

#### Python3

```python

```

#### Java

```java
class Solution {
    static class Fenwick {
        int n;
        int[] count;
        long[] sum;
        Fenwick(int n) {
            this.n = n;
            count = new int[n + 1];
            sum = new long[n + 1];
        }
        void add(int idx, int cnt, long val) {
            idx++;
            while (idx <= n) {
                count[idx] += cnt;
                sum[idx] += val;
                idx += idx & -idx;
            }
        }
        int count(int idx) {
            int res = 0;
            while (idx > 0) {
                res += count[idx];
                idx -= idx & -idx;
            }
            return res;
        }
        long sum(int idx) {
            long res = 0;
            while (idx > 0) {
                res += sum[idx];
                idx -= idx & -idx;
            }
            return res;
        }
        int kth(int k) {
            int idx = 0;
            for (int bit = Integer.highestOneBit(n); bit != 0; bit >>= 1) {
                int next = idx + bit;
                if (next <= n && count[next] < k) {
                    idx = next;
                    k -= count[next];
                }
            }
            return idx;
        }
        long firstK(int k, int[] values) {
            if (k <= 0)
                return 0;

            int pos = kth(k);

            int cntBefore = count(pos);
            long sumBefore = sum(pos);

            return sumBefore + (long)(k - cntBefore) * values[pos];
        }
        long lastK(int k, int[] values) {
            int total = count(n);

            if (k <= 0)
                return 0;

            if (k >= total)
                return sum(n);

            return sum(n) - firstK(total - k, values);
        }
        int totalCount() {
            return count(n);
        }
    }
    public long maxSum(int[] nums, int k) {
        int n = nums.length;
        int[] values = nums.clone();
        Arrays.sort(values);

        int m = 0;

        for (int x : values) {
            if (m == 0 || values[m - 1] != x) {
                values[m++] = x;
            }
        }
        values = Arrays.copyOf(values, m);
        Fenwick inside = new Fenwick(m);
        Fenwick outside = new Fenwick(m);
        for (int x : nums) {
            int idx = Arrays.binarySearch(values, x);
            outside.add(idx, 1, x);
        }
        long answer = Long.MIN_VALUE;
        for (int l = 0; l < n; l++) {
            inside = new Fenwick(m);
            outside = new Fenwick(m);
            for (int x : nums) {
                int idx = Arrays.binarySearch(values, x);
                outside.add(idx, 1, x);
            }
            long windowSum = 0;
            for (int r = l; r < n; r++) {
                int idx = Arrays.binarySearch(values, nums[r]);
                outside.add(idx, -1, -nums[r]);
                inside.add(idx, 1, nums[r]);
                windowSum += nums[r];
                int insideCount = r - l + 1;
                int outsideCount = n - insideCount;
                int maxSwaps = Math.min(
                        k,
                        Math.min(insideCount, outsideCount)
                );
                if (maxSwaps == 0) {
                    answer = Math.max(answer, windowSum);
                    continue;
                }
                int lo = 1;
                int hi = maxSwaps;
                int best = 0;
                while (lo <= hi) {
                    int mid = (lo + hi) >>> 1;
                    int smallestPos = inside.kth(mid);
                    int largestPos =
                            outside.kth(outsideCount - mid + 1);
                    if (values[smallestPos] < values[largestPos]) {
                        best = mid;
                        lo = mid + 1;
                    } else {
                        hi = mid - 1;
                    }
                }
                if (best > 0) {
                    long smallest =
                            inside.firstK(best, values);
                    long largest =
                            outside.lastK(best, values);
                    long candidate =
                            windowSum + largest - smallest;
                    answer = Math.max(answer, candidate);
                } else {
                    answer = Math.max(answer, windowSum);
                }
            }
        }
        return answer;
    }
}
```

#### C++

```cpp

```

#### Go

```go

```

<!-- tabs:end -->

<!-- solution:end -->

<!-- problem:end -->
