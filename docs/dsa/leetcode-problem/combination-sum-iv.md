# From Recursive Thinking to Bottom-Up DP: Understanding Combination Sum IV

When I first learned dynamic programming, I found recursion much easier to understand than tabulation. Recursion feels natural because it matches the decision process: at each step, I choose a number, reduce the target, and continue. DP, on the other hand, feels flat and linear. This post explains how to connect those two views using the classic **Combination Sum IV** problem.

## Problem Idea

Given an array `nums` of distinct positive integers and a target `target`, count how many different **ordered sequences** can sum to `target`.

For example, if:

- `nums = [1, 2, 3]`
- `target = 4`

then sequences like `(1, 3)` and `(3, 1)` are considered different.

------

## 1. Starting with Recursive Thinking

A natural recursive idea is:

- if I still need to make sum `t`,
- I can choose any number `x` in `nums`,
- then the remaining subproblem becomes `t - x`.

So we define:
$$
f(t) = \text{number of ordered sequences that sum to } t
$$
Then the recursion is:
$$
f(t) = \sum_{x \in nums} f(t - x)
$$
with base cases:
$$
f(0) = 1
$$

$$
f(t) = 0 \quad \text{for } t < 0
$$

These base cases are very important:

- `f(0) = 1` means we have found one valid way to complete the sequence.
- `f(t) = 0` for negative `t` means that path is invalid.

If `nums = [1, 2, 3]`, then:
$$
f(4) = f(3) + f(2) + f(1)
$$

$$
f(3) = f(2) + f(1) + f(0)
$$

$$
f(2) = f(1) + f(0) + f(-1)
$$

and so on.

This recursive definition is correct, but it does a lot of repeated work.

------

## 2. Where the Repeated Computation Comes From

The recursion forms a tree. For example:

- `f(4)` calls `f(3)`, `f(2)`, `f(1)`
- `f(3)` also calls `f(2)`, `f(1)`, `f(0)`
- `f(2)` again calls `f(1)`, `f(0)`, ...

So states like:

- `f(2)`
- `f(1)`
- `f(0)`

are computed many times.

This is the key insight of dynamic programming:

**even though the recursion tree looks large, the number of distinct states is actually small.**

For this problem, the only meaningful states are:
$$
f(0), f(1), f(2), \dots, f(target)
$$
So instead of recomputing the same state again and again, we can store each answer once.

------

## 3. How to Decide Memo Array Size

A very common beginner question is: how large should the memo array be?

The answer is:

**the memo size depends on the state space, not on the input array length.**

Here the recursive function is `f(t)`, and the changing parameter is only `t`.

So the memo array should cover all possible values of `t` from `0` to `target`. That is why the size is:
$$
target + 1
$$
not `nums.length`.

This is a good general rule:

- if the state is `f(i)`, memo size depends on the range of `i`
- if the state is `f(left)`, memo size depends on the range of `left`
- if the state is `f(i, left)`, then the memo usually needs to be 2D

------

## 4. What the Memo Actually Stores

Another important point is what we store in `memo[t]`.

We do **not** store all paths or all sequences.

Instead, we store just one integer:
$$
memo[t] = \text{number of ordered sequences that sum to } t
$$
For example, with `nums = [1, 2, 3]`:
$$
f(2) = 2
$$
because the valid ordered sequences are:

- `(1, 1)`
- `(2)`

So `memo[2] = 2`.

It does not store the actual sequences, only the total count.

------

## 5. Translating Recursion into Bottom-Up DP

This is the part that often feels hard.

Recursion feels like making decisions in a tree. DP feels like filling a line or an array. The bridge between them is this:

**recursion shows the process, DP stores the result of each state.**

The recursive formula is already:
$$
f(t) = \sum_{x \in nums} f(t-x)
$$
Bottom-up DP simply replaces recursive calls with table lookups.

Define:
$$
dp[t] = f(t)
$$
Then:
$$
dp[t] = \sum_{x \in nums, \ t-x \ge 0} dp[t-x]
$$
and the base case becomes:
$$
dp[0] = 1
$$
The negative-state rule `f(t) = 0` for `t < 0` is handled by skipping invalid transitions instead of using negative indices.

So the recursive call:

- “call `f(t-x)`”

becomes the DP transition:

- “read `dp[t-x]`”

That is the real meaning of “remove the recursion, keep the accumulation.”

------

## 6. Why the DP Uses Two Loops

The bottom-up version usually has two loops:

- outer loop: enumerate the current target sum `t`
- inner loop: enumerate which number `x` from `nums` is used last

The meaning is:

for each sum `t`, try all possible last choices.

If the last chosen number is `x`, then before that we must already have formed `t - x`. So we add `dp[t-x]` into `dp[t]`.

This gives:
$$
dp[t] += dp[t-x]
$$
for every valid `x`.

This loop order matters. Since this problem counts **ordered sequences**, we must process it in a way that preserves order. Using the outer loop over `t` and the inner loop over `nums` correctly counts permutations rather than combinations.

------

## 7. A Small Example

Let `nums = [1, 2, 3]` and `target = 4`.

We fill the DP array from small to large.

### Base case

$$
dp[0] = 1
$$

### For `t = 1`

- choose `1`: use `dp[0]`
- choose `2` or `3`: invalid

So:
$$
dp[1] = dp[0] = 1
$$

### For `t = 2`

- choose `1`: use `dp[1]`
- choose `2`: use `dp[0]`
- choose `3`: invalid

So:
$$
dp[2] = dp[1] + dp[0] = 1 + 1 = 2
$$

### For `t = 3`

- choose `1`: use `dp[2]`
- choose `2`: use `dp[1]`
- choose `3`: use `dp[0]`

So:
$$
dp[3] = dp[2] + dp[1] + dp[0] = 2 + 1 + 1 = 4
$$

### For `t = 4`

- choose `1`: use `dp[3]`
- choose `2`: use `dp[2]`
- choose `3`: use `dp[1]`

So:
$$
dp[4] = dp[3] + dp[2] + dp[1] = 4 + 2 + 1 = 7
$$
Thus the answer is:
$$
dp[4] = 7
$$

------

## 8. Why DP Feels Linear While Recursion Feels Like a Tree

This is one of the most important conceptual points.

Recursion emphasizes:

**how one problem branches into smaller subproblems.**

DP emphasizes:

**what the answer of each distinct state is.**

The recursion tree may look complicated, but many nodes represent the same state. DP compresses all identical states into one table entry.

So DP is not “losing the decision process.” It is simply merging all repeated states.

A good way to think about it is:

- recursion tree = process graph
- DP table = result table

------

## 9. Can We Do Rolling Array Optimization?

A natural next question is whether we can optimize the space.

Usually, rolling array optimization works when:
$$
dp[i]
$$
depends only on a fixed number of previous states, such as:

- `dp[i-1]`
- `dp[i-2]`

For example, Fibonacci can be optimized to `O(1)` space because:
$$
f(i) = f(i-1) + f(i-2)
$$
uses only two previous values.

But this problem is different:
$$
dp[i] = \sum_{x \in nums, \ i-x \ge 0} dp[i-x]
$$
The current state may depend on many previous positions, depending on the values in `nums`. It is not limited to a fixed constant number of predecessors.

So this problem usually cannot be optimized down to a few scalar variables in the same simple way as Fibonacci.

The key reason is **dependency range**, not merely that it is “doing a sum.”

------

## 10. Final Bottom-Up Java Code

```java
class Solution {
    public int combinationSum4(int[] nums, int target) {
        int[] dp = new int[target + 1];
        dp[0] = 1;

        for (int i = 1; i < dp.length; i++) {
            for (int j = 0; j < nums.length; j++) {
                if (i - nums[j] >= 0) {
                    dp[i] += dp[i - nums[j]];
                }
            }
        }

        return dp[target];
    }
}
```

------

## 11. What I Learned from This Problem

This problem taught me several important DP ideas.

First, the size of the memo or DP array depends on the **state space**, not on the input length.

Second, memoization and tabulation both store the answer for each state, not the actual paths.

Third, converting recursion to DP is not about copying the shape of the recursion tree. It is about identifying:

1. what the state means
2. what smaller states it depends on
3. in what order those states should be computed

Finally, recursion and DP are not two unrelated techniques. They are two different views of the same recurrence:

- recursion explores the state transitions dynamically
- DP precomputes the same transitions in a controlled order

Once I stopped staring at the tree shape and started focusing on **state definitions** and **dependencies**, the bottom-up approach became much easier to understand.

------

## Conclusion

For Combination Sum IV, recursive thinking provides the most intuitive starting point:

- choose a number
- reduce the target
- continue

But the real power of dynamic programming comes from recognizing that many branches of the recursion tree lead to the same subproblem.

By defining:
$$
f(t) = \text{number of ordered sequences that sum to } t
$$
and then turning that recurrence into a DP table, we move from exponential brute force to efficient bottom-up computation.

That is the heart of dynamic programming:
**identify repeated states, store their answers once, and reuse them.**

