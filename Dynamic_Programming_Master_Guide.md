# The Definitive Dynamic Programming Master Guide

### For Tech Interviews and Competitive Programming

---

## Welcome

If Dynamic Programming feels like magic that other people can do and you cannot, this guide is for you. The truth is simple: DP is not magic. It is a small set of repeatable thinking patterns. Once you see the patterns, you stop memorizing solutions and start deriving them.

This guide covers **10 categories** and **24 carefully chosen LeetCode problems**. Every single problem is explained with the same five-step structure, so your brain learns one process and applies it 24 times. Repetition of structure is what builds real skill.

Work through it slowly. Write the code yourself before reading mine. You will be fine.

---

## Table of Contents

**Part I — Foundations**
- [What Dynamic Programming Actually Is](#what-dynamic-programming-actually-is)
- [The Two Signals: When To Use DP](#the-two-signals-when-to-use-dp)
- [Memoization vs Tabulation](#memoization-vs-tabulation)
- [The Universal 5-Question Framework](#the-universal-5-question-framework)
- [How To Read Every Problem In This Guide](#how-to-read-every-problem-in-this-guide)

**Part II — The 10 Categories**
1. [1D DP (Linear DP)](#category-1--1d-dp-linear-dp)
2. [2D DP (Grid / Matrix DP)](#category-2--2d-dp-grid--matrix-dp)
3. [String DP](#category-3--string-dp)
4. [Tree DP](#category-4--tree-dp)
5. [Knapsack DP](#category-5--knapsack-dp)
6. [Interval DP](#category-6--interval-dp)
7. [Digit DP](#category-7--digit-dp)
8. [Bitmask DP](#category-8--bitmask-dp)
9. [Game Theory DP](#category-9--game-theory-dp)
10. [Probability / Expectation DP](#category-10--probability--expectation-dp)

**Part III — Mastery**
- [Pattern Recognition Cheat Sheet](#pattern-recognition-cheat-sheet)
- [Common Bugs and How To Avoid Them](#common-bugs-and-how-to-avoid-them)
- [A 6-Week Study Plan](#a-6-week-study-plan)

---
---

# Part I — Foundations

## What Dynamic Programming Actually Is

Here is the whole idea in one sentence:

> **Dynamic Programming is recursion that refuses to solve the same problem twice.**

That is it. Nothing more.

Think about computing Fibonacci numbers with plain recursion. To get `fib(50)`, you compute `fib(49)` and `fib(48)`. But `fib(49)` also computes `fib(48)`. So you computed `fib(48)` twice. And inside those, `fib(47)` gets computed three times. The waste explodes. Plain recursion does about 2^50 operations.

DP says: *write down the answer to `fib(48)` the first time you compute it, and look it up after that.* Now you do 50 operations instead of 2^50.

Everything else in this guide — grids, strings, trees, bitmasks — is just a different shape of "write down the answer so you never redo work."

### Three words you must understand

| Word | Plain-English Meaning |
|---|---|
| **State** | A snapshot of "where am I in the problem right now". For Fibonacci the state is just the number `n`. For a grid it is `(row, col)`. For a knapsack it is `(item index, remaining capacity)`. |
| **Transition** | The rule that says: *"to get the answer for this state, combine the answers of these smaller states."* This is your recurrence relation. |
| **Base Case** | The smallest states whose answers you simply know, with no computation. The recursion must land here or it never stops. |

If you can name the **state**, the **transition**, and the **base case**, you have solved the problem. The code is just typing after that.

---

## The Two Signals: When To Use DP

A problem is a DP problem if **both** of these are true.

### Signal 1 — Overlapping Subproblems

The same smaller problem gets asked again and again. If you drew the recursion tree, you would see the same node repeated in different branches.

If every subproblem is unique and never repeats, you have **Divide and Conquer** (like Merge Sort), not DP.

### Signal 2 — Optimal Substructure

The best answer to the big problem is built from the best answers to its smaller parts.

Example: the shortest path from A to C through B equals (shortest path A→B) + (shortest path B→C). The big optimum is made of small optima.

### Practical interview triggers

When you see these phrases, your DP alarm should ring:

- "Find the **maximum / minimum** ..."
- "**Count the number of ways** to ..."
- "Is it **possible** to reach / make / partition ..."
- "Find the **longest / shortest** subsequence / substring / path ..."
- The problem asks for an answer **modulo 10^9 + 7** (a strong counting-DP hint)
- Constraints are small in a suspicious way: `n <= 20` (think bitmask), `n <= 500` (think O(n^2) or O(n^3) interval DP)

### A quick warning about Greedy

Many beginners try Greedy first and get burned. Greedy means: *"make the choice that looks best right now, never look back."* Greedy only works when a local best choice is provably also globally best.

DP exists precisely because that is often false. DP tries **all** choices at each step and keeps the best — but efficiently, by caching. Whenever you cannot prove a greedy rule, switch to DP.

---

## Memoization vs Tabulation

There are exactly two ways to write any DP. Learn both. Interviewers love when you can convert one into the other.

### Top-Down (Memoization) — "Recursion + a notebook"

You write the natural recursive solution, then add a cache (array or hash map). Before computing, check the cache. After computing, store in the cache.

```
function solve(state):
    if state is a base case: return known answer
    if cache has state: return cache[state]
    answer = combine(solve(smaller states))
    cache[state] = answer
    return answer
```

**Good for:** problems where the state space is weird or sparse, trees, digit DP, and any time the recursion is easier to see than the iteration order.
**Risk:** stack overflow on deep recursion; slightly slower due to function call overhead.

### Bottom-Up (Tabulation) — "Fill a table in the right order"

You start at the base cases and build forward in a loop until you reach the answer. No recursion at all.

```
dp[base cases] = known answers
for each state in dependency order:
    dp[state] = combine(dp[smaller states])
return dp[final state]
```

**Good for:** everything with a regular shape (arrays, grids, strings). It is faster, has no stack limit, and allows **space optimization**.
**Risk:** you must figure out the correct loop order so dependencies are already computed.

### The rule of thumb

> Think in **Top-Down** (it matches how your brain reasons). Ship in **Bottom-Up** (it is faster and cleaner).

In this guide, the primary code is always Bottom-Up, with Top-Down or space-optimized variants added whenever they teach something important.

### Space Optimization

If `dp[i]` only depends on `dp[i-1]` and `dp[i-2]`, you do not need an array of size `n`. You need two variables. This drops memory from O(n) to O(1).

If `dp[i][j]` only depends on row `i-1`, you only need two rows, or sometimes one row updated carefully. This drops O(n·m) to O(m). You will see this technique many times below.

---

## The Universal 5-Question Framework

Before writing a single line of code for **any** DP problem, answer these five questions out loud. This is the habit that separates people who solve new problems from people who only recall old ones.

1. **What is my state?**
   *"`dp[i]` means ____."* Write this sentence down literally. If you cannot finish the sentence, you do not understand the problem yet. This is the #1 cause of failure.

2. **What are my choices at this state?**
   Usually: take it or skip it. Go right or go down. Match the characters or do not. Enumerate every legal move.

3. **What is the transition (recurrence)?**
   Combine the choices with `max`, `min`, `+` (for counting), or `or` (for feasibility).

4. **What are the base cases?**
   The smallest inputs: empty string, zero items, capacity zero, a leaf node.

5. **What order do I fill the table in, and where is the final answer?**
   Dependencies must be computed before they are used. The answer is not always `dp[n]` — sometimes it is `max(dp)` over everything.

---

## How To Read Every Problem In This Guide

Each of the 24 problems follows the same five sections:

1. **THE BLUEPRINT** — the problem in plain words, with exact input/output and a tiny example.
2. **THE THOUGHT PROCESS** — why brute force or greedy fails, the state, the transition, the base cases.
3. **THE FRAMEWORK** — time and space complexity, each justified in one sentence.
4. **THE CODE** — production-quality C++ and Python, bottom-up first, variants where useful.
5. **CODE WALKTHROUGH** — a chronological tour of the code, mapped back to the thinking.

**How to use it:** read sections 1 and 2. Close the guide. Try to write the code yourself. Then come back and compare. Reading code you did not struggle with teaches almost nothing.

---
---

# Part II — The 10 Categories

---

# Category 1 — 1D DP (Linear DP)

> **Core Definition:** A linear sequence of states where the current state depends on a constant number of previous states.

This is the front door to Dynamic Programming. The state is a single index `i` walking along an array or a number line, and `dp[i]` depends on `dp[i-1]`, `dp[i-2]`, and so on — a fixed, small number of neighbours behind it.

**How to recognise it:** you are walking left to right through a sequence (days, stairs, houses, coins) and at each step you make a small local decision.

**The signature move:** because each state looks back only a constant distance, you can almost always drop the array and keep two or three rolling variables. O(n) space becomes O(1) space.

**Problems in this category:** LeetCode 70, 198, 121.

---

## Problem 1.1 — LeetCode 70: Climbing Stairs

### 1. THE BLUEPRINT (Input/Output Expectation)

You are standing at the bottom of a staircase with `n` steps. Each time you move, you may climb either **1 step** or **2 steps**. The question: how many distinct ways can you reach the top?

- **Input:** one integer `n` (the number of steps).
- **Output:** one integer — the count of distinct ordered ways to reach step `n`.

**Tiny concrete example:** `n = 3`.

The distinct ways are:
1. 1 + 1 + 1
2. 1 + 2
3. 2 + 1

Note that `1 + 2` and `2 + 1` count as **different** ways, because the order of moves differs. So the answer is **3**.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why brute force fails.** The obvious approach is recursion: `ways(n) = ways(n-1) + ways(n-2)`. If you write that with no cache, computing `ways(45)` re-derives `ways(40)` thousands of times. The recursion tree is a binary tree of depth `n`, so the cost is roughly O(2^n). At `n = 45` your program hangs. The subproblems overlap massively — that is Signal 1, so DP applies.

**Why greedy fails.** There is nothing to be greedy about. We are not optimising anything; we are *counting*. Greedy has no meaning here.

**Finding the state.** Ask the key question: *"If I am standing on step `i`, where could I have come from?"*

There are exactly two possibilities, and they do not overlap:
- I was on step `i-1` and took a single step.
- I was on step `i-2` and took a double step.

Every path to step `i` passes through exactly one of these two. So the counts simply add.

> **State definition:** `dp[i]` = the number of distinct ways to reach step `i`.

**The transition:**

```
dp[i] = dp[i-1] + dp[i-2]
```

This is the Fibonacci recurrence. Climbing Stairs *is* Fibonacci wearing a costume.

**The base cases.**
- `dp[0] = 1`. There is exactly one way to be at the ground: do nothing. (This "empty way" convention is important — it makes the formula work for `i = 2`.)
- `dp[1] = 1`. One way to reach step 1: a single 1-step move.

Check `dp[2] = dp[1] + dp[0] = 1 + 1 = 2`. Correct: `1+1` and `2`. The base cases are consistent.

**The space insight.** `dp[i]` looks back only two slots. We never need the far history. Two variables are enough.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(n)** — we compute each of the `n` states exactly once, doing constant work per state.
- **Space Complexity: O(1)** — the space-optimized version stores only the previous two answers, regardless of how large `n` is. (The plain table version is O(n).)

### 4. THE CODE

**C++ — Bottom-Up Tabulation (clear version)**

```cpp
class Solution {
public:
    int climbStairs(int n) {
        // dp[i] = number of distinct ways to reach step i.
        vector<int> dp(n + 1, 0);

        // Base cases.
        dp[0] = 1;              // one way to stand at the ground: do nothing
        if (n >= 1) dp[1] = 1;  // one way to reach step 1

        // Build forward: every step is reached from the two steps behind it.
        for (int i = 2; i <= n; ++i) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }

        return dp[n];
    }
};
```

**C++ — Space-Optimized O(1) (the version to show in an interview)**

```cpp
class Solution {
public:
    int climbStairs(int n) {
        if (n <= 2) return n;   // n=1 -> 1 way, n=2 -> 2 ways

        int twoBack = 1;        // dp[i-2], starts as dp[0]
        int oneBack = 2;        // dp[i-1], starts as dp[2]... see note below

        // We start the loop at i = 3 because dp[1] and dp[2] are already known.
        for (int i = 3; i <= n; ++i) {
            int current = oneBack + twoBack;  // dp[i] = dp[i-1] + dp[i-2]
            twoBack = oneBack;                // slide the window forward
            oneBack = current;
        }

        return oneBack;         // after the loop, oneBack holds dp[n]
    }
};
```

*(Note: here `twoBack` is initialised to `dp[1] = 1` and `oneBack` to `dp[2] = 2`, so the first iteration correctly produces `dp[3]`.)*

**Python — Bottom-Up Tabulation**

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        # dp[i] = number of distinct ways to reach step i
        dp = [0] * (n + 1)

        dp[0] = 1                 # base case: one way to do nothing
        if n >= 1:
            dp[1] = 1             # base case: one single step

        for i in range(2, n + 1):
            dp[i] = dp[i - 1] + dp[i - 2]

        return dp[n]
```

**Python — Space-Optimized O(1)**

```python
class Solution:
    def climbStairs(self, n: int) -> int:
        if n <= 2:
            return n

        two_back, one_back = 1, 2     # dp[1], dp[2]

        for _ in range(3, n + 1):
            # Python tuple assignment slides the window in one clean line.
            two_back, one_back = one_back, one_back + two_back

        return one_back
```

**Python — Top-Down Memoization (for comparison)**

```python
from functools import lru_cache

class Solution:
    def climbStairs(self, n: int) -> int:
        @lru_cache(maxsize=None)      # this decorator IS the memoization
        def ways(i: int) -> int:
            if i <= 1:                # base cases: dp[0] = dp[1] = 1
                return 1
            return ways(i - 1) + ways(i - 2)

        return ways(n)
```

### 5. CODE WALKTHROUGH

1. **Allocate the table.** `dp` has size `n + 1` because we want a valid index for every step from `0` to `n` inclusive. Off-by-one errors here are the most common bug in all of DP — always size the array to the largest index you will touch, plus one.
2. **Seed the base cases.** `dp[0] = 1` encodes "the empty path is a valid way to stand at the start." The guard `if (n >= 1)` prevents writing to `dp[1]` when the array is too small.
3. **Loop forward from `i = 2`.** We start at 2 because that is the first index whose answer is *not* already known. The direction matters: by the time we read `dp[i-1]` and `dp[i-2]`, the loop has already written them. This is the "dependency order" from question 5 of the framework.
4. **Apply the transition verbatim.** `dp[i] = dp[i-1] + dp[i-2]` is the recurrence from our thought process, typed directly. Nothing clever is added.
5. **Return `dp[n]`.** The final state is exactly the question that was asked.
6. **In the optimized version**, the array disappears and the two rolling variables take its place. The three-line slide — compute `current`, shift `twoBack`, shift `oneBack` — is a pattern you will reuse in dozens of problems. Make it muscle memory.

---

## Problem 1.2 — LeetCode 198: House Robber

### 1. THE BLUEPRINT (Input/Output Expectation)

A street has houses in a row, each holding some amount of money, given as array `nums`. You want to rob as much as possible. The one rule: **you cannot rob two adjacent houses** — doing so triggers the alarm.

- **Input:** an integer array `nums`, where `nums[i]` is the money in house `i`.
- **Output:** a single integer — the maximum money you can rob without ever picking two neighbours.

**Tiny concrete example:** `nums = [2, 7, 9, 3, 1]`.

- Robbing houses 0, 2, 4 gives `2 + 9 + 1 = 12`.
- Robbing houses 1, 3 gives `7 + 3 = 10`.
- Robbing houses 0, 2 gives `2 + 9 = 11`.

The best is **12**.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why greedy fails.** The tempting greedy rule is "always rob the biggest house available." Try it on `[2, 7, 9, 3, 1]`: you grab 9, which blocks 7 and 3; then you grab 2 and 1, giving `9 + 2 + 1 = 12`. Lucky — it worked here. Now try `[2, 1, 1, 2]`: greedy grabs a 2, blocks its neighbour, grabs the other 2, total 4. Also fine. But try `[5, 1, 1, 5, 1, 1, 5]` versus a case like `[3, 10, 3, 1, 2]`: greedy takes 10, blocking 3 and 3, then takes 2 → total 12. The optimum is also 12 here. Greedy *often* works and then fails on a case you did not test — which is exactly why it is dangerous. The real reason it breaks is that taking a large house permanently destroys two other options, and no local rule can see far enough to price that cost. DP prices it exactly by trying both branches.

**Why brute force fails.** Each house is a yes/no decision, so there are up to 2^n subsets to check. At `n = 100` that is impossible. But notice: once you have decided about houses `0..i-1`, the only thing that matters going forward is *how much you have* and *whether house `i-1` was robbed*. The history beyond that is irrelevant — overlapping subproblems confirmed.

**Finding the state.** Ask: *"What is the most money I can get from the first `i` houses?"*

> **State definition:** `dp[i]` = the maximum money obtainable considering houses `0` through `i-1` (that is, the first `i` houses).

Using "first `i` houses" rather than "ending at house `i`" is deliberate — it gives us a clean `dp[0] = 0` for the empty street and avoids special cases.

**The choices at house `i-1`** (the last house in our window) are exactly two:
- **Skip it.** Then the answer is whatever was best for the first `i-1` houses: `dp[i-1]`.
- **Rob it.** Then I collect `nums[i-1]`, but house `i-2` is now forbidden, so I must fall back to the best over the first `i-2` houses: `dp[i-2] + nums[i-1]`.

Take whichever is larger.

**The transition:**

```
dp[i] = max(dp[i-1], dp[i-2] + nums[i-1])
```

**The base cases.**
- `dp[0] = 0` — no houses, no money.
- `dp[1] = nums[0]` — one house, rob it (all values are non-negative).

**The answer** is `dp[n]`.

Notice the shape: like Climbing Stairs, each state depends only on the two before it. Same O(1) space trick applies.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(n)** — one pass over the houses, with a constant-time `max` comparison at each one.
- **Space Complexity: O(1)** — the optimized version keeps only two rolling values since `dp[i]` never looks further back than `dp[i-2]`.

### 4. THE CODE

**C++ — Bottom-Up Tabulation**

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();

        // dp[i] = max money obtainable from the FIRST i houses (indices 0..i-1).
        vector<int> dp(n + 1, 0);

        dp[0] = 0;                 // base case: no houses available
        dp[1] = nums[0];           // base case: only one house, so take it

        for (int i = 2; i <= n; ++i) {
            int skipCurrent = dp[i - 1];                  // do not rob house i-1
            int robCurrent  = dp[i - 2] + nums[i - 1];    // rob it, skip house i-2
            dp[i] = max(skipCurrent, robCurrent);
        }

        return dp[n];
    }
};
```

*(Safe for `n >= 1`, which the constraints guarantee. If you want full safety, guard `dp[1]` with `if (n >= 1)`.)*

**C++ — Space-Optimized O(1)**

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        int twoBack = 0;   // dp[i-2]
        int oneBack = 0;   // dp[i-1]

        for (int money : nums) {
            // Decide for the current house, then slide the window forward.
            int current = max(oneBack, twoBack + money);
            twoBack = oneBack;
            oneBack = current;
        }

        return oneBack;    // holds dp[n] after the final house
    }
};
```

**Python — Bottom-Up Tabulation**

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        n = len(nums)

        # dp[i] = max money from the first i houses (indices 0 .. i-1)
        dp = [0] * (n + 1)
        dp[1] = nums[0]                      # base case: a single house

        for i in range(2, n + 1):
            skip_current = dp[i - 1]                  # leave house i-1 alone
            rob_current = dp[i - 2] + nums[i - 1]     # take it, skip its neighbour
            dp[i] = max(skip_current, rob_current)

        return dp[n]
```

**Python — Space-Optimized O(1)**

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        two_back, one_back = 0, 0            # dp[i-2], dp[i-1]

        for money in nums:
            two_back, one_back = one_back, max(one_back, two_back + money)

        return one_back
```

### 5. CODE WALKTHROUGH

1. **Size the table to `n + 1`, not `n`.** The extra slot holds `dp[0]`, the empty-street case. This single design choice removes a whole class of index bugs, because `dp[i-2]` is always legal once `i >= 2`.
2. **Mind the index shift.** `dp[i]` talks about the first `i` houses, so the *actual house* being decided is `nums[i-1]`, not `nums[i]`. Getting this wrong is the most frequent mistake on this problem. Say it to yourself while coding: *"table index `i`, array index `i-1`."*
3. **Write both branches as named variables.** `skipCurrent` and `robCurrent` are not required by the compiler, but they make the code self-documenting and make the two choices from the thought process visible on screen. Interviewers notice this.
4. **`max` performs the decision.** We never commit to a rule like "always take the big one." We evaluate both futures and keep the better one. That is the essence of DP replacing greedy.
5. **Return `dp[n]`** — the best over all `n` houses.
6. **The optimized loop** iterates over values rather than indices, which removes the index shift entirely. `two_back` and `one_back` are the same two table cells, just renamed as variables. The single-line Python swap performs "compute then slide" atomically, which prevents the classic bug of overwriting `one_back` before you have read it.

---

## Problem 1.3 — LeetCode 121: Best Time to Buy and Sell Stock

### 1. THE BLUEPRINT (Input/Output Expectation)

You are given the price of a stock on each day. You may complete **at most one transaction**: buy on one day, sell on a strictly later day. Find the maximum profit. If no profit is possible, return 0.

- **Input:** an integer array `prices`, where `prices[i]` is the price on day `i`.
- **Output:** a single integer — the maximum achievable profit, or `0` if every option loses money.

**Tiny concrete example:** `prices = [7, 1, 5, 3, 6, 4]`.

Buy on day 1 (price 1), sell on day 4 (price 6). Profit = `6 - 1 = 5`. No better pair exists, so the answer is **5**.

Counter-example: `prices = [7, 6, 4, 3, 1]` is strictly falling, so every transaction loses money. Answer is **0** (we simply do not trade).

### 2. THE THOUGHT PROCESS (Mental Model)

**Why brute force fails.** Check every pair `(buy day i, sell day j)` with `i < j` and keep the best difference. That is two nested loops — O(n^2). With `n` up to 10^5, that is 10^10 operations. Far too slow.

**The key realisation.** Look at the brute force again. For a fixed **sell day** `j`, the best profit is `prices[j] - (the smallest price seen on any day before j)`. You do not need to re-scan the history every time — you can carry the minimum forward as you walk.

That carried value *is* the DP state. This problem is DP in its purest, leanest form.

**Finding the state.** There are two equivalent ways to frame it. Both are worth knowing.

*Framing A (the classic):*
> `minPriceSoFar` = the lowest price on any day from `0` to `i-1`.
> `maxProfit` = the best profit achievable using any sell day up to `i`.

*Framing B (the "state machine", which generalises to the harder stock problems):*
> `hold[i]` = the maximum cash balance on day `i` if I am currently **holding** a share.
> `free[i]` = the maximum cash balance on day `i` if I am currently **not holding** a share.

Framing B is more powerful because LeetCode 122, 123, 188, and 309 are all the same machine with extra states bolted on. Learn it now and those problems become easy later.

**The transitions for Framing B:**

```
hold[i] = max(hold[i-1],  -prices[i])        // keep holding, or buy today
free[i] = max(free[i-1],  hold[i-1] + prices[i])   // stay out, or sell today
```

Read `hold[i] = max(hold[i-1], -prices[i])` carefully. Because only **one** transaction is allowed, buying today means we started from zero cash, so the balance is `-prices[i]` — we do **not** add a previous profit. (In the multi-transaction version LC 122, this term becomes `free[i-1] - prices[i]`. That one character is the entire difference between the two problems.)

**The base cases.**
- `hold[0] = -prices[0]` — on day 0, the only way to hold a share is to have bought it today.
- `free[0] = 0` — on day 0, doing nothing leaves you with zero.

**The answer** is `free[n-1]`: at the end you must have sold, because holding a share you never sold earns nothing.

**The transitions for Framing A** are even simpler:

```
maxProfit = max(maxProfit, prices[i] - minPriceSoFar)
minPriceSoFar = min(minPriceSoFar, prices[i])
```

Both run in one pass.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(n)** — a single sweep through the prices, with constant-time comparisons on each day.
- **Space Complexity: O(1)** — we carry only two scalars (`minPrice` and `maxProfit`, or `hold` and `free`), never an array.

### 4. THE CODE

**C++ — One-Pass DP (Framing A)**

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int minPrice  = INT_MAX;   // cheapest price seen so far (our best buy day)
        int maxProfit = 0;         // best profit found so far; 0 means "do not trade"

        for (int price : prices) {
            // Step 1: could selling TODAY beat our record?
            // minPrice is guaranteed to come from an earlier day.
            maxProfit = max(maxProfit, price - minPrice);

            // Step 2: update the best buy price for all FUTURE days.
            minPrice = min(minPrice, price);
        }

        return maxProfit;
    }
};
```

*(Guarding against overflow: on the very first day `minPrice` is `INT_MAX`, so `price - minPrice` is negative and `max` keeps 0. To be fully safe against signed overflow, initialise `minPrice = prices[0]` and start the loop at index 1.)*

**C++ — State Machine DP (Framing B, generalises to harder variants)**

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        // hold = best cash balance while OWNING a share (always negative here)
        // free = best cash balance while owning NOTHING
        int hold = -prices[0];
        int free = 0;

        for (int i = 1; i < (int)prices.size(); ++i) {
            // Order matters conceptually: compute both from the PREVIOUS day's values.
            int prevHold = hold;
            int prevFree = free;

            hold = max(prevHold, -prices[i]);              // keep holding, or buy today
            free = max(prevFree, prevHold + prices[i]);    // stay out, or sell today
        }

        return free;   // we must end holding nothing
    }
};
```

**Python — One-Pass DP (Framing A)**

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        min_price = float('inf')     # cheapest price seen on any earlier day
        max_profit = 0               # 0 is the floor: we can always refuse to trade

        for price in prices:
            # Sell today against the cheapest earlier day.
            max_profit = max(max_profit, price - min_price)
            # Then record today as a candidate buy day for the future.
            min_price = min(min_price, price)

        return max_profit
```

**Python — State Machine DP (Framing B)**

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        hold = -prices[0]            # balance if we own a share
        free = 0                     # balance if we own nothing

        for price in prices[1:]:
            # Compute both from the previous day's values simultaneously.
            hold, free = max(hold, -price), max(free, hold + price)

        return free
```

### 5. CODE WALKTHROUGH

1. **Initialise the two carried values.** `minPrice` starts at infinity so that the first real price always replaces it. `maxProfit` starts at 0, which bakes in the rule "if everything loses money, do not trade." That single initialisation handles the entire falling-prices edge case with no special code.
2. **Order inside the loop is critical.** We compute `maxProfit` **before** updating `minPrice`. Why? Because selling requires buying on a *strictly earlier* day. If we updated `minPrice` first, then on a day with a new low we would compute `price - price = 0`, effectively allowing a same-day buy and sell. It would not break this particular problem (the profit is 0), but it is logically wrong, and in variants with fees or cooldowns that mistake produces wrong answers. Build the correct habit now.
3. **`max` and `min` replace branching.** No `if` statements, no nesting — just two comparisons per day. This is what a clean O(n) sweep looks like.
4. **The state-machine version names what the DP is really tracking:** two mutually exclusive situations, and on each day you either stay in your situation or transition out of it. In the Python one-liner, the right-hand side is fully evaluated before either name is rebound, so `hold + price` safely uses *yesterday's* `hold`. In C++ we make that explicit with `prevHold` and `prevFree` — if you assigned `hold` first and then used it for `free`, you would be allowing a buy and a sell on the same day. This is the single most common bug in all stock DP problems.
5. **Return `free`, not `max(hold, free)`.** A share you are still holding has not been converted to profit, and `hold` is negative anyway. Ending empty-handed is always at least as good.

---

# Category 2 — 2D DP (Grid / Matrix DP)

> **Core Definition:** Two-dimensional state spaces where the state depends on multiple layout dimensions (for example, row and column).

The state is now a pair `(i, j)` instead of a single index. You are standing on a cell of a grid, and the answer for that cell is built from its neighbours — usually the cell above and the cell to the left, because movement is restricted to "right and down".

**How to recognise it:** the input is a matrix, or two sequences being compared, or any situation needing two independent coordinates to describe "where you are".

**The signature move:** fill the table row by row, left to right. Because row `i` depends only on row `i-1` and on earlier cells in row `i`, you can usually compress to **one or two rows** of memory.

**The classic trap:** the first row and first column have no neighbour above or to the left. Handle them with a padding row and column, or with explicit boundary logic. We will show both.

**Problems in this category:** LeetCode 62, 64, 221.

---

## Problem 2.1 — LeetCode 62: Unique Paths

### 1. THE BLUEPRINT (Input/Output Expectation)

A robot sits in the top-left corner of an `m x n` grid. It can only move **right** or **down**, one cell at a time. How many distinct paths lead to the bottom-right corner?

- **Input:** two integers `m` (number of rows) and `n` (number of columns).
- **Output:** a single integer — the number of distinct paths from `(0, 0)` to `(m-1, n-1)`.

**Tiny concrete example:** `m = 3, n = 2` (3 rows, 2 columns).

The paths are:
1. Down → Down → Right
2. Down → Right → Down
3. Right → Down → Down

Answer: **3**.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why brute force fails.** Recursively explore both moves from every cell. Every path has length `m + n - 2`, and at each step there are up to 2 choices, so the recursion tree holds roughly 2^(m+n) nodes. For a 18x18 grid that is already billions of calls. But the cell `(5, 5)` is reached through a huge number of different prefixes, and its answer is the same every time — textbook overlapping subproblems.

**Finding the state.** Reverse the question. Instead of "where can I go from here?", ask *"how could I have arrived here?"*

To stand on cell `(i, j)`, the robot's previous position was either:
- the cell directly **above**, `(i-1, j)`, arriving by moving down, or
- the cell directly **to the left**, `(i, j-1)`, arriving by moving right.

There is no third option, and the two sets of paths are disjoint. So the counts add.

> **State definition:** `dp[i][j]` = the number of distinct paths from the start `(0, 0)` to cell `(i, j)`.

**The transition:**

```
dp[i][j] = dp[i-1][j] + dp[i][j-1]
```

**The base cases.** The entire first row and the entire first column equal 1. Reasoning: to reach any cell in row 0 you must move right the whole way — exactly one path. Same logic, vertically, for column 0.

```
dp[0][j] = 1   for all j
dp[i][0] = 1   for all i
```

**The answer** is `dp[m-1][n-1]`.

**The space insight.** Row `i` reads only from row `i-1` (directly above) and from the cell just left of it in row `i`. So a single 1D array of length `n`, updated in place left to right, is sufficient. When we compute `curr[j]`, the value sitting in `curr[j]` is still row `i-1`'s value (the cell above) and `curr[j-1]` has already been updated to row `i` (the cell left). Both neighbours are available in one array. Beautiful.

**Bonus — the math shortcut.** Every path is a sequence of exactly `m-1` downs and `n-1` rights in some order, so the answer is the binomial coefficient `C(m+n-2, m-1)`. Mention this in an interview to show depth, but code the DP, because the DP generalises to obstacles (LC 63) and the formula does not.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(m · n)** — we visit each of the `m · n` cells exactly once, doing one addition per cell.
- **Space Complexity: O(n)** — the optimized version keeps a single row of length `n`, because each row only ever reads from the row immediately above it. (The full table version is O(m · n).)

### 4. THE CODE

**C++ — Bottom-Up Tabulation (full table, clearest to read)**

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        // dp[i][j] = number of distinct paths from (0,0) to (i,j).
        // Initialising everything to 1 handles row 0 and column 0 for free.
        vector<vector<int>> dp(m, vector<int>(n, 1));

        // Start at (1,1): row 0 and column 0 are already correct.
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                // Paths arriving from above  + paths arriving from the left.
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }

        return dp[m - 1][n - 1];
    }
};
```

**C++ — Space-Optimized to a Single Row**

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        // row[j] holds the path count for column j of the row we are building.
        // Row 0 is all 1s.
        vector<int> row(n, 1);

        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                // Before this write:
                //   row[j]     still holds the value from the row ABOVE
                //   row[j - 1] already holds the value from the CURRENT row (left)
                row[j] = row[j] + row[j - 1];
            }
        }

        return row[n - 1];
    }
};
```

**Python — Bottom-Up Tabulation**

```python
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        # dp[i][j] = number of distinct paths from (0,0) to (i,j).
        # Pre-filling with 1 covers the first row and first column.
        dp = [[1] * n for _ in range(m)]

        for i in range(1, m):
            for j in range(1, n):
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1]   # from above + from left

        return dp[m - 1][n - 1]
```

**Python — Space-Optimized to a Single Row**

```python
class Solution:
    def uniquePaths(self, m: int, n: int) -> int:
        row = [1] * n                 # represents row 0

        for _ in range(1, m):
            for j in range(1, n):
                # row[j] is still the cell above; row[j-1] is already the cell left.
                row[j] += row[j - 1]

        return row[-1]
```

### 5. CODE WALKTHROUGH

1. **Initialise the whole table to 1.** This is a small trick that pays off: instead of writing two extra loops to fill row 0 and column 0, we make 1 the default. Every base case is now handled before the main loop starts.
2. **Start both loops at index 1.** Cell `(0, 0)` and the entire border are already correct. Starting at 1 also guarantees that `i-1` and `j-1` are never negative, so no boundary checks are needed inside the hot loop.
3. **The transition is a single addition** — the recurrence typed literally. Because we iterate rows top to bottom and columns left to right, `dp[i-1][j]` (previous row) and `dp[i][j-1]` (earlier this row) are both already final when we read them. That is the dependency order working for us.
4. **Return the bottom-right cell**, which is the destination the question asked about.
5. **The 1D optimization deserves a slow read.** The statement `row[j] += row[j-1]` is doing double duty. At the moment of execution, `row[j]` has not yet been touched this iteration, so it still contains the answer for `(i-1, j)` — the cell above. Meanwhile `row[j-1]` was updated one iteration ago, so it contains `(i, j-1)` — the cell to the left. The single array simultaneously holds part of the old row and part of the new row, and the split point moves right as we go. Understanding this in-place trick unlocks space optimization for nearly every grid DP you will meet.

---

## Problem 2.2 — LeetCode 64: Minimum Path Sum

### 1. THE BLUEPRINT (Input/Output Expectation)

You are given an `m x n` grid filled with non-negative numbers. Starting at the top-left and moving only **right** or **down**, reach the bottom-right. Find the path whose sum of visited numbers is **smallest**, and return that sum.

- **Input:** a 2D integer array `grid`.
- **Output:** a single integer — the minimum possible sum along a valid path.

**Tiny concrete example:**

```
grid = [[1, 3, 1],
        [1, 5, 1],
        [4, 2, 1]]
```

The path `1 → 3 → 1 → 1 → 1` (right, right, down, down) sums to **7**. No path does better, so the answer is 7. Notice the path through the middle (`1 → 1 → 5 → ...`) is worse because of the 5.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why greedy fails.** The natural greedy rule is "at each cell, step onto the smaller of the two neighbours." Consider:

```
[[1,  2, 100],
 [1, 100,  1],
 [1,   1,  1]]
```

From the start, greedy must pick between right (2) and down (1). It picks down. That is actually fine here. But flip it:

```
[[1,   1, 1],
 [2, 100, 1],
 [1,   1, 1]]
```

Greedy at the start compares right (1) and down (2), picks right — and in this grid that happens to work. The point is that *you cannot know*. A cheap first step can lead into a region where every continuation is expensive, and a locally expensive step can open a cheap corridor. Greedy sees one cell ahead; the cost of a decision is only revealed many cells later. DP evaluates the full consequence of both choices and keeps the better one.

**Why brute force fails.** Same as Unique Paths — exponentially many paths, with the same cells recomputed over and over.

**Finding the state.** This is Unique Paths with the operation changed. Instead of *counting* ways in, we take the *cheapest* way in and add the toll for standing on the current cell.

> **State definition:** `dp[i][j]` = the minimum sum of any path from `(0, 0)` to `(i, j)`, including the value of `grid[i][j]` itself.

**The transition:**

```
dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])
```

In words: pay the toll for this cell, then add the cheaper of the two ways to arrive here.

**The base cases.**
- `dp[0][0] = grid[0][0]` — the start cell costs its own value.
- **First row:** the only way in is from the left, so `dp[0][j] = dp[0][j-1] + grid[0][j]` — a running prefix sum.
- **First column:** the only way in is from above, so `dp[i][0] = dp[i-1][0] + grid[i][0]`.

These borders cannot use `min` because one of the two neighbours does not exist. Taking `min` with a non-existent cell is the classic bug here; treating it as 0 would wrongly allow entering from outside the grid.

**The answer** is `dp[m-1][n-1]`.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(m · n)** — each cell is processed exactly once with constant work.
- **Space Complexity: O(n)** — the optimized version carries a single row, since each row depends only on the row directly above. (O(1) extra if you are allowed to overwrite the input grid in place.)

### 4. THE CODE

**C++ — Bottom-Up Tabulation (explicit borders)**

```cpp
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();

        // dp[i][j] = cheapest path cost from (0,0) to (i,j), inclusive.
        vector<vector<int>> dp(m, vector<int>(n, 0));

        dp[0][0] = grid[0][0];                    // base: the start cell

        // First column: can only be entered from above.
        for (int i = 1; i < m; ++i)
            dp[i][0] = dp[i - 1][0] + grid[i][0];

        // First row: can only be entered from the left.
        for (int j = 1; j < n; ++j)
            dp[0][j] = dp[0][j - 1] + grid[0][j];

        // Interior: both entries exist, so pick the cheaper one.
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                dp[i][j] = grid[i][j] + min(dp[i - 1][j], dp[i][j - 1]);
            }
        }

        return dp[m - 1][n - 1];
    }
};
```

**C++ — Space-Optimized to a Single Row**

```cpp
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();

        vector<int> row(n, 0);

        // Build row 0 as a running prefix sum.
        row[0] = grid[0][0];
        for (int j = 1; j < n; ++j) row[j] = row[j - 1] + grid[0][j];

        for (int i = 1; i < m; ++i) {
            // Column 0 of this row: only reachable from directly above,
            // and row[0] currently holds the row above. So just add.
            row[0] += grid[i][0];

            for (int j = 1; j < n; ++j) {
                // row[j]     = cell ABOVE (not yet overwritten)
                // row[j - 1] = cell LEFT  (already overwritten this row)
                row[j] = grid[i][j] + min(row[j], row[j - 1]);
            }
        }

        return row[n - 1];
    }
};
```

**Python — Bottom-Up Tabulation**

```python
class Solution:
    def minPathSum(self, grid: List[List[int]]) -> int:
        m, n = len(grid), len(grid[0])

        # dp[i][j] = cheapest cost to reach (i,j), including grid[i][j]
        dp = [[0] * n for _ in range(m)]
        dp[0][0] = grid[0][0]

        # First column: forced to come from above.
        for i in range(1, m):
            dp[i][0] = dp[i - 1][0] + grid[i][0]

        # First row: forced to come from the left.
        for j in range(1, n):
            dp[0][j] = dp[0][j - 1] + grid[0][j]

        # Interior cells have a real choice.
        for i in range(1, m):
            for j in range(1, n):
                dp[i][j] = grid[i][j] + min(dp[i - 1][j], dp[i][j - 1])

        return dp[m - 1][n - 1]
```

**Python — Space-Optimized to a Single Row**

```python
class Solution:
    def minPathSum(self, grid: List[List[int]]) -> int:
        m, n = len(grid), len(grid[0])

        # Row 0 is a simple prefix sum: there is only one way across it.
        row = [0] * n
        row[0] = grid[0][0]
        for j in range(1, n):
            row[j] = row[j - 1] + grid[0][j]

        for i in range(1, m):
            row[0] += grid[i][0]                 # column 0: only from above
            for j in range(1, n):
                row[j] = grid[i][j] + min(row[j], row[j - 1])

        return row[-1]
```

### 5. CODE WALKTHROUGH

1. **Seed `dp[0][0]` first.** Everything else is built relative to it. Forgetting to add `grid[0][0]` and leaving it at 0 is a silent off-by-one-cell error that produces an answer that is *almost* right.
2. **Handle the two borders in separate loops, before the main nested loop.** This is the opposite choice from Unique Paths, where a blanket initialisation of 1 worked. Here each border cell needs a running sum, so it needs its own pass. Doing it up front means the interior loop never needs an `if` to check whether a neighbour exists — and a hot loop without branches is both faster and easier to read.
3. **The interior transition reads as English:** the cost of standing here, plus the cheaper of the two ways to get here. Compare it to Unique Paths: the structure is identical, `+` became `min`, and we added the cell's own toll. Recognising that two problems share a skeleton is exactly the pattern-matching skill you are training.
4. **Loop order guarantees correctness.** Top to bottom, left to right means both `dp[i-1][j]` and `dp[i][j-1]` are finalised before we read them.
5. **In the 1D version, `row[0] += grid[i][0]` is the subtle line.** At that instant `row[0]` holds the cost of reaching `(i-1, 0)`. Adding the current cell's value promotes it to the cost of reaching `(i, 0)`. It must happen *before* the inner `j` loop begins, because `row[0]` is the "left neighbour" that `j = 1` will read.
6. **Return the last cell of the final row** — the destination.

---

## Problem 2.3 — LeetCode 221: Maximal Square

### 1. THE BLUEPRINT (Input/Output Expectation)

Given a 2D binary matrix (each cell is the character `'0'` or `'1'`), find the largest **square** made entirely of `'1'`s, and return its **area**.

- **Input:** a 2D array `matrix` of characters `'0'` and `'1'`.
- **Output:** a single integer — the area (side × side) of the largest all-ones square. Return 0 if there are no ones.

**Tiny concrete example:**

```
matrix = [['1','0','1','0','0'],
          ['1','0','1','1','1'],
          ['1','1','1','1','1'],
          ['1','0','0','1','0']]
```

There is a 2x2 block of ones whose bottom-right corner is at row 2, column 3. No 3x3 all-ones square exists. So the largest side is 2 and the answer is **4**.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why brute force fails.** Try every cell as a top-left corner, then try every possible side length, then verify all cells inside that square. That is O(m · n · min(m,n)^2) — far too slow, and it re-checks the same interior cells endlessly.

**The crucial reframing.** Do not think about squares as whole objects. Think about **one specific corner**.

> Fix the **bottom-right corner** at cell `(i, j)`. Ask: *what is the largest all-ones square whose bottom-right corner is exactly here?*

If you can answer that for every cell, the global answer is just the maximum over all cells. This "anchor the shape at one cell" trick is the heart of many matrix DP problems.

**Finding the transition.** Suppose `matrix[i][j] == '1'`. Can a square of side 3 end here? It can, if and only if three overlapping squares of side 2 also exist, ending at the three neighbouring corners:

- `(i-1, j)` — the square directly above,
- `(i, j-1)` — the square directly to the left,
- `(i-1, j-1)` — the square diagonally up-left.

Picture it. To build a `k x k` square ending at `(i, j)`, you need the top part (a `(k-1) x (k-1)` square ending just above), the left part (one ending just to the left), and the corner filling (one ending diagonally). If **any** of the three is smaller than `k-1`, there is a hole, and the `k x k` square cannot exist.

Since all three must be at least `k-1`, the limiting factor is the **smallest** of the three.

> **State definition:** `dp[i][j]` = the side length of the largest all-ones square whose **bottom-right corner** is at `(i, j)`.

**The transition:**

```
if matrix[i][j] == '1':
    dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
else:
    dp[i][j] = 0
```

The `min` is the whole insight. Many people guess `max` or `+1` from a single neighbour; those are wrong because a square needs *every* supporting region to be large enough, not just one.

A zero cell forces `dp[i][j] = 0` — no square can end on a zero.

**The base cases.** For cells in row 0 or column 0, a square ending there can have side at most 1. So `dp[i][j] = 1` if the cell is `'1'`, else 0. The padding trick below handles this automatically.

**The answer.** Track the maximum `dp` value seen, call it `best`, and return `best * best` — the question wants area, not side length. Forgetting to square is an extremely common slip.

**The padding trick.** Allocate the table with dimensions `(m+1) x (n+1)` and shift all indices by one. Row 0 and column 0 of the table become a border of zeros that is never part of the matrix. Now `dp[i-1][j-1]` is always a legal read, the base cases become automatic (a cell in matrix row 0 reads three zeros and gets `1`), and the main loop needs no boundary checks at all. Learn this technique — it simplifies a huge number of 2D DP problems.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(m · n)** — each cell is visited once and performs a constant-time three-way minimum.
- **Space Complexity: O(n)** — the optimized version keeps one row plus a single saved diagonal value, since each cell reads only from the row above and its immediate left. (The padded table version is O(m · n).)

### 4. THE CODE

**C++ — Bottom-Up Tabulation (with padding)**

```cpp
class Solution {
public:
    int maximalSquare(vector<vector<char>>& matrix) {
        int m = matrix.size(), n = matrix[0].size();

        // Padded by one row and one column of zeros.
        // dp[i][j] corresponds to matrix[i-1][j-1].
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

        int bestSide = 0;

        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                if (matrix[i - 1][j - 1] == '1') {
                    // A square ending here is limited by its three supports:
                    // above, left, and diagonal. The weakest one decides.
                    dp[i][j] = 1 + min({ dp[i - 1][j],
                                         dp[i][j - 1],
                                         dp[i - 1][j - 1] });

                    bestSide = max(bestSide, dp[i][j]);
                }
                // If the cell is '0', dp[i][j] stays 0 - no square can end here.
            }
        }

        return bestSide * bestSide;   // the question asks for AREA
    }
};
```

**C++ — Space-Optimized to a Single Row**

```cpp
class Solution {
public:
    int maximalSquare(vector<vector<char>>& matrix) {
        int m = matrix.size(), n = matrix[0].size();

        vector<int> dp(n + 1, 0);   // one padded row
        int bestSide = 0;
        int diagonal = 0;           // holds dp[i-1][j-1] before it is overwritten

        for (int i = 1; i <= m; ++i) {
            diagonal = 0;           // reset: dp[i-1][0] is always 0
            for (int j = 1; j <= n; ++j) {
                int savedAbove = dp[j];   // dp[i-1][j], about to be overwritten

                if (matrix[i - 1][j - 1] == '1') {
                    dp[j] = 1 + min({ dp[j],        // above
                                      dp[j - 1],    // left (already this row)
                                      diagonal });  // up-left, saved last iteration
                    bestSide = max(bestSide, dp[j]);
                } else {
                    dp[j] = 0;
                }

                diagonal = savedAbove;    // becomes the diagonal for column j+1
            }
        }

        return bestSide * bestSide;
    }
};
```

**Python — Bottom-Up Tabulation (with padding)**

```python
class Solution:
    def maximalSquare(self, matrix: List[List[str]]) -> int:
        m, n = len(matrix), len(matrix[0])

        # Padded with a zero row and zero column so edge cells need no special case.
        # dp[i][j] describes matrix[i-1][j-1].
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        best_side = 0

        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if matrix[i - 1][j - 1] == '1':
                    # The weakest of the three supporting squares caps our size.
                    dp[i][j] = 1 + min(dp[i - 1][j],      # above
                                       dp[i][j - 1],      # left
                                       dp[i - 1][j - 1])  # diagonal
                    best_side = max(best_side, dp[i][j])
                # '0' cells stay at 0 by default.

        return best_side * best_side     # AREA, not side
```

**Python — Space-Optimized to a Single Row**

```python
class Solution:
    def maximalSquare(self, matrix: List[List[str]]) -> int:
        m, n = len(matrix), len(matrix[0])

        dp = [0] * (n + 1)      # one padded row
        best_side = 0

        for i in range(1, m + 1):
            diagonal = 0        # dp[i-1][j-1]; starts at 0 for the padded column
            for j in range(1, n + 1):
                saved_above = dp[j]          # this is dp[i-1][j]

                if matrix[i - 1][j - 1] == '1':
                    dp[j] = 1 + min(dp[j], dp[j - 1], diagonal)
                    best_side = max(best_side, dp[j])
                else:
                    dp[j] = 0

                diagonal = saved_above       # shift for the next column

        return best_side * best_side
```

### 5. CODE WALKTHROUGH

1. **Pad the table by one in each dimension.** The loops then run from 1 to `m` and 1 to `n`, and every access to `dp[i-1][j-1]` is safe. The zero border naturally encodes "there is nothing above or to the left," which is exactly the truth for cells on the matrix edge.
2. **Remember the index offset.** Table cell `dp[i][j]` describes matrix cell `matrix[i-1][j-1]`. Write that mapping in a comment — it is the single biggest source of confusion when people first use padding.
3. **Branch on the cell's character.** If it is `'0'`, we leave `dp[i][j]` at its initialised zero. No square can terminate on an empty cell, and writing 0 correctly blocks any larger square from forming through this position later.
4. **The three-way `min` is the algorithm.** Read it as: *"I can extend by one only as far as my weakest support allows."* If above is 3, left is 3, but the diagonal is only 1, then the biggest square ending here has side 2 — the hole in the diagonal region caps us. Test that case on paper until it is obvious.
5. **Track the running maximum inside the loop.** We never scan the finished table again; `bestSide` is updated at the moment each cell is computed.
6. **Square the answer at the very end.** `dp` stores side lengths, and the problem asks for area. This one line is a frequent source of wrong-answer submissions.
7. **In the 1D version, the `diagonal` variable is the trick.** As we move right along a row, `dp[j]` is about to be overwritten with the current row's value — but the *next* column will need that old value as *its* diagonal. So we stash it in `saved_above` before writing, then roll it into `diagonal` at the end of the iteration. Resetting `diagonal = 0` at the start of each row corresponds to the zero in the padded column. This save-the-diagonal pattern appears in every 1D-compressed DP where the recurrence reaches up-and-left; once you have written it once, you own it.

---

# Category 3 — String DP

> **Core Definition:** String manipulation, subsequence tracking, and structural pattern matching across one or more strings.

String DP is really 2D DP in disguise. When you compare two strings, your state is a pair of pointers — one into each string — and `dp[i][j]` describes "the answer for the first `i` characters of A against the first `j` characters of B."

**How to recognise it:** two strings being compared, or one string being split. Words like *subsequence*, *transform*, *match*, *interleave*, *palindrome*.

**The signature move at every cell:** ask one question — *do the current characters match?* If yes, consume both and move diagonally. If no, you must skip a character from one string or the other, and you try both.

**The golden convention:** always use **1-based** table indices with a padded row and column representing the empty prefix. So `dp[i][j]` refers to `A[0..i-1]` and `B[0..j-1]`, and the character being compared is `A[i-1]` vs `B[j-1]`. Every string DP in this guide uses this convention. Consistency here prevents hours of debugging.

**Problems in this category:** LeetCode 1143, 72, 115.

---

## Problem 3.1 — LeetCode 1143: Longest Common Subsequence

### 1. THE BLUEPRINT (Input/Output Expectation)

Given two strings, find the length of their longest common **subsequence**. A subsequence keeps the original relative order but may delete any characters; it does **not** need to be contiguous.

- **Input:** two strings `text1` and `text2`.
- **Output:** a single integer — the length of the longest subsequence present in both.

**Tiny concrete example:** `text1 = "abcde"`, `text2 = "ace"`.

The subsequence `"ace"` appears in both (in `"abcde"` we skip `b` and `d`). Its length is 3. Nothing longer is common, so the answer is **3**.

Contrast with *substring*: `"ace"` is not a substring of `"abcde"` because the letters are not adjacent. Subsequence rules are looser — that looseness is what makes DP necessary.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why brute force fails.** Generate every subsequence of `text1` (there are 2^n of them) and test each against `text2`. Exponential and hopeless for strings of length 1000.

**Why greedy fails.** A greedy rule like "match the first common character you find" breaks easily. Compare `"ab"` and `"ba"`: greedily matching `a` first commits you to the `a` in position 1 of `"ba"`, leaving nothing after it to match `b`. You get 1, which happens to be correct here, but now consider `"aab"` vs `"aba"` — greedy matching can lock you into an early character and lose a longer alignment later. You cannot know whether to use a match or save the character for a better one later. Only trying both branches is safe.

**Finding the state.** Walk both strings from the end (or equivalently, define the table over prefixes). Ask: *"what is the LCS of the first `i` characters of A and the first `j` characters of B?"*

> **State definition:** `dp[i][j]` = the length of the LCS of `text1[0..i-1]` and `text2[0..j-1]`.

**The two cases.** Look at the last characters of the two prefixes, `text1[i-1]` and `text2[j-1]`.

*Case 1 — they match.* Then it is always safe to pair them up. (Why always safe? Because any LCS that does not use this pair can be rewritten to use it without getting shorter — a standard exchange argument.) Having consumed one character from each string, we add 1 to the answer for the remaining prefixes:

```
dp[i][j] = 1 + dp[i-1][j-1]
```

*Case 2 — they do not match.* These two characters cannot both be the end of the common subsequence. At least one of them is useless, so we must discard one — but which? We do not know, so we try both and keep the better:

```
dp[i][j] = max(dp[i-1][j],    // throw away text1[i-1]
               dp[i][j-1])    // throw away text2[j-1]
```

**The base cases.** `dp[0][j] = 0` and `dp[i][0] = 0`. The LCS with an empty string is always empty. The padded row and column give us this for free.

**The answer** is `dp[m][n]`, where `m` and `n` are the full lengths.

**Why no `dp[i-1][j-1]` term in Case 2?** Because `max(dp[i-1][j], dp[i][j-1])` already covers it: `dp[i-1][j-1] <= dp[i-1][j]`, so including it would change nothing. Adding redundant terms is harmless but noisy; leave them out.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(m · n)** — we fill a table with `m · n` cells, each requiring one comparison and one constant-time combination.
- **Space Complexity: O(min(m, n))** — the optimized version keeps two rows (or one row plus a diagonal), because row `i` only ever reads row `i-1`. (The full table is O(m · n), which you need if you must reconstruct the actual subsequence.)

### 4. THE CODE

**C++ — Bottom-Up Tabulation (full table)**

```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int m = text1.size(), n = text2.size();

        // dp[i][j] = LCS length of text1[0..i-1] and text2[0..j-1].
        // Row 0 and column 0 represent empty prefixes and stay 0.
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                if (text1[i - 1] == text2[j - 1]) {
                    // Characters match: pair them and solve the smaller prefixes.
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    // No match: discard one character, try both, keep the best.
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        return dp[m][n];
    }
};
```

**C++ — Space-Optimized to Two Rows**

```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int m = text1.size(), n = text2.size();

        // Only the previous row is ever read, so two rows suffice.
        vector<int> prev(n + 1, 0), curr(n + 1, 0);

        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                if (text1[i - 1] == text2[j - 1])
                    curr[j] = 1 + prev[j - 1];     // diagonal
                else
                    curr[j] = max(prev[j],         // from above
                                  curr[j - 1]);    // from the left
            }
            swap(prev, curr);    // O(1) pointer swap; curr is reused next round
        }

        return prev[n];          // after the final swap, prev holds the last row
    }
};
```

**Python — Bottom-Up Tabulation**

```python
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        m, n = len(text1), len(text2)

        # dp[i][j] = LCS of text1[:i] and text2[:j]
        dp = [[0] * (n + 1) for _ in range(m + 1)]

        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if text1[i - 1] == text2[j - 1]:
                    dp[i][j] = 1 + dp[i - 1][j - 1]      # match: take the diagonal
                else:
                    dp[i][j] = max(dp[i - 1][j],         # skip a char of text1
                                   dp[i][j - 1])         # skip a char of text2

        return dp[m][n]
```

**Python — Space-Optimized to Two Rows**

```python
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        m, n = len(text1), len(text2)

        prev = [0] * (n + 1)

        for i in range(1, m + 1):
            curr = [0] * (n + 1)
            for j in range(1, n + 1):
                if text1[i - 1] == text2[j - 1]:
                    curr[j] = 1 + prev[j - 1]
                else:
                    curr[j] = max(prev[j], curr[j - 1])
            prev = curr          # the row we just built becomes the previous row

        return prev[n]
```

**Bonus — reconstructing the actual subsequence (Python)**

```python
def lcs_string(text1: str, text2: str) -> str:
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = 1 + dp[i - 1][j - 1]
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])

    # Walk backwards from the bottom-right, retracing the decisions.
    out, i, j = [], m, n
    while i > 0 and j > 0:
        if text1[i - 1] == text2[j - 1]:
            out.append(text1[i - 1])     # this character was part of a match
            i, j = i - 1, j - 1
        elif dp[i - 1][j] >= dp[i][j - 1]:
            i -= 1                        # we had moved up
        else:
            j -= 1                        # we had moved left

    return ''.join(reversed(out))
```

### 5. CODE WALKTHROUGH

1. **Build a `(m+1) x (n+1)` table.** The extra row and column are the empty-prefix base cases. They are initialised to 0, which is already the correct answer, so no base-case loop is needed.
2. **Loops run 1 to `m` and 1 to `n`,** so `i` and `j` are prefix *lengths*, not array positions. The character at the end of prefix `i` is `text1[i-1]`. Internalise this mapping.
3. **The `if` is the entire problem.** One comparison splits the world into two cases, and each case has exactly one line. When a problem reduces this cleanly, you know the state was chosen well.
4. **The match branch moves diagonally** (`i-1, j-1`) because consuming one character from *each* string shrinks both prefixes. The `+1` records that we found one more common character.
5. **The mismatch branch moves up or left** — each move discards one character from one string. We take `max` because we want the longest result, and we try both because there is no way to know in advance which discard is better.
6. **Return `dp[m][n]`,** the cell for both full strings.
7. **The two-row optimization** works because the recurrence only reaches back one row. In C++, `swap(prev, curr)` exchanges internal pointers in O(1) — it does not copy data. Be careful: `curr` now holds stale values from two rows ago, so every cell must be written before being read. Our loop writes `curr[j]` for every `j >= 1` and `curr[0]` is always 0, so we are safe. In Python, allocating a fresh `curr` each row is slightly slower but completely safe from stale data.
8. **Reconstruction needs the full table.** You cannot rebuild the path from two rows, because the backtrack walks through all of history. If an interviewer asks for the actual string and not just the length, say this out loud — it shows you understand the trade-off you made.

---

## Problem 3.2 — LeetCode 72: Edit Distance

### 1. THE BLUEPRINT (Input/Output Expectation)

Given two strings `word1` and `word2`, find the **minimum number of operations** needed to convert `word1` into `word2`. Three operations are allowed, each costing 1:

- **Insert** a character
- **Delete** a character
- **Replace** a character

- **Input:** two strings `word1` and `word2`.
- **Output:** a single integer — the minimum number of operations.

**Tiny concrete example:** `word1 = "horse"`, `word2 = "ros"`.

1. `"horse"` → `"rorse"` (replace `h` with `r`)
2. `"rorse"` → `"rose"` (delete `r`)
3. `"rose"` → `"ros"` (delete `e`)

Three operations, and no shorter sequence exists. Answer: **3**.

This measure is called *Levenshtein distance*, and it powers spell checkers, DNA sequence alignment, and `diff` tools.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why greedy fails.** A greedy rule such as "fix the first mismatched character immediately" cannot decide *which* operation to apply. Should you replace it, or delete it and hope the next character lines up, or insert something before it? The right choice depends entirely on what comes afterwards, which greedy cannot see.

**Why brute force fails.** At each mismatch you branch three ways, so the recursion tree has size roughly 3^(m+n). Yet different operation sequences constantly arrive at the *same* pair of remaining suffixes — massive overlap, so DP it is.

**Finding the state.** Same shape as LCS: two prefix pointers.

> **State definition:** `dp[i][j]` = the minimum number of operations to convert `word1[0..i-1]` into `word2[0..j-1]`.

**The two cases.** Compare the last characters of the two prefixes.

*Case 1 — `word1[i-1] == word2[j-1]`.* These characters already agree. Do nothing, cost 0, and move both pointers back:

```
dp[i][j] = dp[i-1][j-1]
```

*Case 2 — they differ.* We must spend exactly one operation. There are three ways, and we take the cheapest:

| Operation | Resulting subproblem | Why |
|---|---|---|
| **Replace** `word1[i-1]` with `word2[j-1]` | `dp[i-1][j-1]` | Both characters are now handled, so both pointers shrink. |
| **Delete** `word1[i-1]` | `dp[i-1][j]` | We removed a character from `word1`, so only `i` shrinks; `word2`'s character still needs matching. |
| **Insert** `word2[j-1]` at the end of `word1` | `dp[i][j-1]` | The inserted character matches and is consumed, so only `j` shrinks; all of `word1`'s prefix is still unhandled. |

```
dp[i][j] = 1 + min(dp[i-1][j-1],   // replace
                   dp[i-1][j],     // delete
                   dp[i][j-1])     // insert
```

Spend a moment on the insert case, which confuses almost everyone at first. When you insert the needed character onto the end of `word1`, that character immediately pairs with `word2[j-1]`. Both are then done. But `word1`'s original prefix of length `i` has not been touched at all, so the remaining work is `word1[0..i-1]` versus `word2[0..j-2]` — exactly `dp[i][j-1]`.

**The base cases.** These are *not* zero, unlike LCS.
- `dp[i][0] = i` — to turn a prefix of length `i` into the empty string, you must delete all `i` characters.
- `dp[0][j] = j` — to turn the empty string into a prefix of length `j`, you must insert all `j` characters.

These must be filled explicitly. Leaving them at 0 is the most common bug in this problem, and it produces answers that are too small.

**The answer** is `dp[m][n]`.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(m · n)** — every one of the `m · n` table cells is computed once using a constant-time three-way minimum.
- **Space Complexity: O(n)** — the optimized version carries one row plus a saved diagonal, because each row depends only on the row above it. (The full table is O(m · n).)

### 4. THE CODE

**C++ — Bottom-Up Tabulation (full table)**

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size(), n = word2.size();

        // dp[i][j] = min operations to turn word1[0..i-1] into word2[0..j-1].
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

        // Base case: deleting every character of word1 to reach "".
        for (int i = 0; i <= m; ++i) dp[i][0] = i;

        // Base case: inserting every character of word2 into "".
        for (int j = 0; j <= n; ++j) dp[0][j] = j;

        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                if (word1[i - 1] == word2[j - 1]) {
                    // Characters already agree: free move, shrink both.
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    // Spend one operation; pick the cheapest of three futures.
                    dp[i][j] = 1 + min({ dp[i - 1][j - 1],   // replace
                                         dp[i - 1][j],       // delete from word1
                                         dp[i][j - 1] });    // insert into word1
                }
            }
        }

        return dp[m][n];
    }
};
```

**C++ — Space-Optimized to a Single Row**

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size(), n = word2.size();

        // dp represents the previous row; it starts as row 0 (all inserts).
        vector<int> dp(n + 1);
        for (int j = 0; j <= n; ++j) dp[j] = j;

        for (int i = 1; i <= m; ++i) {
            int diagonal = dp[0];   // this is dp[i-1][0] before we overwrite it
            dp[0] = i;              // dp[i][0] = i deletions

            for (int j = 1; j <= n; ++j) {
                int savedAbove = dp[j];        // dp[i-1][j], needed as next diagonal

                if (word1[i - 1] == word2[j - 1]) {
                    dp[j] = diagonal;                               // free match
                } else {
                    dp[j] = 1 + min({ diagonal,      // replace (dp[i-1][j-1])
                                      savedAbove,    // delete  (dp[i-1][j])
                                      dp[j - 1] });  // insert  (dp[i][j-1])
                }

                diagonal = savedAbove;
            }
        }

        return dp[n];
    }
};
```

**Python — Bottom-Up Tabulation**

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        # dp[i][j] = min edits to turn word1[:i] into word2[:j]
        dp = [[0] * (n + 1) for _ in range(m + 1)]

        # Base cases are NOT zero here.
        for i in range(m + 1):
            dp[i][0] = i          # delete i characters
        for j in range(n + 1):
            dp[0][j] = j          # insert j characters

        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if word1[i - 1] == word2[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1]          # characters agree: free
                else:
                    dp[i][j] = 1 + min(dp[i - 1][j - 1],  # replace
                                       dp[i - 1][j],      # delete from word1
                                       dp[i][j - 1])      # insert into word1

        return dp[m][n]
```

**Python — Space-Optimized to a Single Row**

```python
class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)

        dp = list(range(n + 1))          # row 0: j insertions

        for i in range(1, m + 1):
            diagonal = dp[0]             # dp[i-1][0]
            dp[0] = i                    # dp[i][0] = i deletions

            for j in range(1, n + 1):
                saved_above = dp[j]      # dp[i-1][j]

                if word1[i - 1] == word2[j - 1]:
                    dp[j] = diagonal
                else:
                    dp[j] = 1 + min(diagonal, saved_above, dp[j - 1])

                diagonal = saved_above

        return dp[n]
```

**Python — Top-Down Memoization (closest to the raw thought process)**

```python
from functools import lru_cache

class Solution:
    def minDistance(self, word1: str, word2: str) -> int:
        @lru_cache(maxsize=None)
        def solve(i: int, j: int) -> int:
            # Base cases: one string is exhausted.
            if i == 0:
                return j          # insert the rest of word2
            if j == 0:
                return i          # delete the rest of word1

            if word1[i - 1] == word2[j - 1]:
                return solve(i - 1, j - 1)

            return 1 + min(solve(i - 1, j - 1),   # replace
                           solve(i - 1, j),       # delete
                           solve(i, j - 1))       # insert

        return solve(len(word1), len(word2))
```

### 5. CODE WALKTHROUGH

1. **Fill the borders first, and fill them with real values.** `dp[i][0] = i` and `dp[0][j] = j` are not decorations — they carry the cost of emptying or building a string from scratch. Unlike LCS, zeros here are wrong. Write these two loops before anything else.
2. **Iterate over prefix lengths, compare `word1[i-1]` to `word2[j-1]`.** The same 1-based convention as LCS keeps the two solutions visually parallel, which helps you remember both.
3. **The equal branch costs nothing.** We copy the diagonal unchanged. There is no `+1`, and there is no `min` — when the characters already agree, there is provably no reason to spend an operation on them.
4. **The unequal branch always costs exactly 1,** so the `1 +` sits outside the `min`. Inside, the three terms correspond one-to-one with replace, delete, insert. Writing them in that fixed order every time (diagonal, up, left) makes the code easy to verify at a glance.
5. **Return `dp[m][n]`,** the cost for the complete strings.
6. **In the 1D version, two saved values do the work of the missing row.** `diagonal` holds `dp[i-1][j-1]` and `saved_above` holds `dp[i-1][j]` just before it gets overwritten. The line `dp[0] = i` must come *after* reading `diagonal = dp[0]`, otherwise the first column's diagonal is corrupted — a subtle ordering bug worth remembering.
7. **The memoized version is the one to write first in an interview.** It is a direct transcription of the thought process, so you can produce it quickly and correctly under pressure, then offer to convert it to bottom-up and then to O(n) space. Showing that progression is far more impressive than jumping straight to the optimized code.

---

## Problem 3.3 — LeetCode 115: Distinct Subsequences

### 1. THE BLUEPRINT (Input/Output Expectation)

Given strings `s` and `t`, count how many **distinct subsequences of `s`** are equal to `t`. Two subsequences are considered different if they use different sets of positions in `s`, even if the resulting text is the same.

- **Input:** two strings `s` (the source) and `t` (the target).
- **Output:** a single integer — the number of ways to choose positions in `s` that spell out `t`.

**Tiny concrete example:** `s = "rabbbit"`, `t = "rabbit"`.

`s` contains three `b`s in a row, and `t` needs only two of them. We choose which two:

1. Use `b`s at positions 2 and 3 (skip position 4)
2. Use `b`s at positions 2 and 4 (skip position 3)
3. Use `b`s at positions 3 and 4 (skip position 2)

Answer: **3**.

Another: `s = "babgbag"`, `t = "bag"` gives **5**.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why this is harder than LCS.** LCS asks "how long?", which is an optimisation (`max`). This asks "how many?", which is counting (`+`). Counting problems are stricter: you must make sure every valid configuration is counted **exactly once** — no double counting, no missed cases. That discipline changes how you write the recurrence.

**Why brute force fails.** Enumerate all 2^|s| subsequences of `s` and compare each to `t`. Hopeless for `|s| = 1000`.

**Finding the state.** Keep the same two-pointer prefix shape.

> **State definition:** `dp[i][j]` = the number of distinct ways to form `t[0..j-1]` using only characters from `s[0..i-1]`.

Read that carefully: `i` indexes the **source** `s`, and `j` indexes the **target** `t`.

**The reasoning at each cell.** We are deciding what to do with the source character `s[i-1]`.

*Option A — ignore `s[i-1]` entirely.* This option is **always** available; we are never forced to use a character of the source. The count of ways is then whatever we could do with a shorter source:

```
dp[i][j] += dp[i-1][j]
```

*Option B — use `s[i-1]` to match `t[j-1]`.* Only available if `s[i-1] == t[j-1]`. If we do that, both characters are consumed and the remaining work is to form `t[0..j-2]` from `s[0..i-2]`:

```
if s[i-1] == t[j-1]:
    dp[i][j] += dp[i-1][j-1]
```

**The transition:**

```
dp[i][j] = dp[i-1][j] + (s[i-1] == t[j-1] ? dp[i-1][j-1] : 0)
```

**Why this counts correctly.** The two options are mutually exclusive — in any given solution, the source character at position `i-1` is either used or not used. So the counts add with no overlap and no gap. That is the hallmark of a well-formed counting recurrence: partition the solution space by one binary decision.

Notice the contrast with LCS. There, on a match, we *always* took the match and never considered skipping it. Here, on a match, we must count **both** using it and skipping it, because skipping it may lead to a different valid selection of positions — and different positions means a different subsequence by this problem's definition. This single difference is the whole problem.

**The base cases.**
- `dp[i][0] = 1` for all `i`. There is exactly **one** way to form the empty target: pick nothing. This is the crucial seed; if you set it to 0, your entire table stays 0.
- `dp[0][j] = 0` for `j >= 1`. An empty source cannot produce a non-empty target.
- Corner: `dp[0][0] = 1`, covered by the first rule.

**The answer** is `dp[m][n]` where `m = |s|`, `n = |t|`.

**A practical note on overflow.** The counts can exceed the range of a 32-bit integer on some inputs. LeetCode guarantees the final answer fits in a signed 32-bit integer, but intermediate sums can still overflow. In C++, use `unsigned long long` or `long long` for the table and cast at the end. In Python, integers are arbitrary precision, so there is nothing to worry about.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(m · n)** — one pass over a table of `m · n` cells with constant work per cell.
- **Space Complexity: O(n)** — the optimized version uses a single array of length `n+1` traversed **right to left**, because each row depends only on the row above.

### 4. THE CODE

**C++ — Bottom-Up Tabulation (full table)**

```cpp
class Solution {
public:
    int numDistinct(string s, string t) {
        int m = s.size(), n = t.size();

        // dp[i][j] = number of ways to build t[0..j-1] from s[0..i-1].
        // Use a 64-bit type: intermediate sums can overflow 32 bits.
        vector<vector<unsigned long long>> dp(m + 1,
                                              vector<unsigned long long>(n + 1, 0));

        // Base case: exactly one way to build the EMPTY target - choose nothing.
        for (int i = 0; i <= m; ++i) dp[i][0] = 1;
        // dp[0][j] for j >= 1 stays 0: empty source cannot build a real target.

        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                // Option A: skip s[i-1]. Always allowed.
                dp[i][j] = dp[i - 1][j];

                // Option B: use s[i-1] to cover t[j-1]. Only if they match.
                if (s[i - 1] == t[j - 1]) {
                    dp[i][j] += dp[i - 1][j - 1];
                }
            }
        }

        return (int)dp[m][n];
    }
};
```

**C++ — Space-Optimized to a Single Row (traversed backwards)**

```cpp
class Solution {
public:
    int numDistinct(string s, string t) {
        int m = s.size(), n = t.size();

        vector<unsigned long long> dp(n + 1, 0);
        dp[0] = 1;                       // one way to build the empty target

        for (int i = 1; i <= m; ++i) {
            // Go RIGHT TO LEFT so dp[j-1] still holds the PREVIOUS row's value.
            for (int j = n; j >= 1; --j) {
                if (s[i - 1] == t[j - 1]) {
                    dp[j] += dp[j - 1];  // skip-count already in dp[j]; add use-count
                }
                // If no match, dp[j] keeps the previous row's value - correct,
                // because the only option was to skip s[i-1].
            }
        }

        return (int)dp[n];
    }
};
```

**Python — Bottom-Up Tabulation**

```python
class Solution:
    def numDistinct(self, s: str, t: str) -> int:
        m, n = len(s), len(t)

        # dp[i][j] = ways to form t[:j] using s[:i]
        dp = [[0] * (n + 1) for _ in range(m + 1)]

        # Exactly one way to form the empty target from any prefix of s.
        for i in range(m + 1):
            dp[i][0] = 1

        for i in range(1, m + 1):
            for j in range(1, n + 1):
                dp[i][j] = dp[i - 1][j]                   # Option A: skip s[i-1]
                if s[i - 1] == t[j - 1]:
                    dp[i][j] += dp[i - 1][j - 1]          # Option B: use s[i-1]

        return dp[m][n]
```

**Python — Space-Optimized (single row, backwards)**

```python
class Solution:
    def numDistinct(self, s: str, t: str) -> int:
        n = len(t)

        dp = [0] * (n + 1)
        dp[0] = 1                        # the empty target

        for ch in s:
            # Right to left keeps dp[j-1] as the previous row's value.
            for j in range(n, 0, -1):
                if ch == t[j - 1]:
                    dp[j] += dp[j - 1]

        return dp[n]
```

### 5. CODE WALKTHROUGH

1. **Seed `dp[i][0] = 1` for every `i`.** This is the engine of the entire table. It says: *"no matter how much source you have, there is exactly one way to produce nothing — select nothing."* Every count in the table is ultimately built by adding up copies of these 1s. Set them to 0 and you get a table of zeros and a confusing debugging session.
2. **Leave `dp[0][j] = 0` for `j >= 1`.** An empty source can never produce a non-empty target. The default zero initialisation already says this, so no code is needed.
3. **Write the skip-option first, unconditionally.** `dp[i][j] = dp[i-1][j]` captures "ignore this source character," which is legal in every situation. Starting from this and then *adding* the match case makes the mutual exclusivity visible in the code's shape.
4. **Add, do not replace, on a match.** This is the defining line of the problem. LCS would overwrite with `1 + diagonal`; here we accumulate `dp[i-1][j] + dp[i-1][j-1]`. The accumulation is what generates 3 instead of 1 on `"rabbbit"`: each of the three `b`s contributes its own branch to the sum.
5. **Return the bottom-right cell** — full source, full target.
6. **The backwards inner loop in the 1D version is not optional.** With one array, `dp[j]` currently holds the previous row's value for column `j`. The recurrence needs `dp[i-1][j-1]`, which means we must read `dp[j-1]` *before* this row overwrites it. Iterating `j` from high to low guarantees that. If you loop forwards, `dp[j-1]` is already the current row's value and you will count many subsequences multiple times — producing a number that is too large. This "iterate backwards to preserve the previous row" rule will come back in the 0/1 Knapsack section; it is the same idea in a different costume.
7. **Use a 64-bit accumulator in C++.** The final answer fits in 32 bits by problem guarantee, but partial sums on adversarial inputs may not. Casting once at the return is cheap insurance.

---

# Category 4 — Tree DP

> **Core Definition:** Dynamic programming executed on tree structures, typically leveraging DFS traversal to aggregate states at parent nodes.

Here the "table" disappears. A tree already gives you the dependency order for free: a node's answer depends on its children's answers, and nothing else. So a **post-order DFS** — children first, parent last — is exactly a bottom-up DP. The recursion stack *is* the table.

**How to recognise it:** the input is a `TreeNode*` (or a graph that is a tree), and the question asks for a maximum, minimum, or count over the whole structure.

**The signature move:** write a helper function that returns **everything the parent needs to know about this subtree**, packed into a small bundle — often a pair of numbers. Then combine the children's bundles into the parent's bundle.

**The two-value pattern.** The most important Tree DP idea: when a node's answer depends on *whether the node itself was used*, return **two** numbers instead of one. One for "I used this node," one for "I did not." The parent then picks whichever it is allowed to use.

**The global-vs-returned distinction.** Often the answer that a parent needs (`returned`) is different from the answer to the overall question (`global`). You return one thing upward and update a separate global maximum on the side. Both problems below use this.

**Problems in this category:** LeetCode 337, 543.

---

## Problem 4.1 — LeetCode 337: House Robber III

### 1. THE BLUEPRINT (Input/Output Expectation)

The houses are now arranged as a **binary tree** instead of a row. Each node holds an amount of money. Same alarm rule: you cannot rob two **directly connected** houses — that is, a parent and its child cannot both be robbed. Find the maximum money you can rob.

- **Input:** the `root` of a binary tree; each node has an integer `val`.
- **Output:** a single integer — the maximum total money.

**Tiny concrete example:**

```
      3
     / \
    2   3
     \    \
      3    1
```

- Robbing the root (3) forbids both children (2 and 3), but allows the grandchildren (3 and 1): total `3 + 3 + 1 = 7`.
- Robbing the second level (2 and 3) gives `5`.

The answer is **7**.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why the naive recursion fails.** The obvious idea: for each node, compare "rob this node + best of all four grandchildren" against "skip this node + best of both children." Written directly, this recurses into grandchildren *and* into children (which themselves recurse into the grandchildren), so every subtree is solved multiple times. On a path-shaped tree that becomes exponential. Classic overlapping subproblems.

You could fix that with a hash map keyed by node pointer. That works, but there is a much cleaner fix.

**The key insight — return two answers.** The reason the naive version has to reach down to grandchildren is that it cannot tell whether a child was robbed. Fix that by making every subtree report both possibilities:

> For each node, return a pair:
> - `robbed` = the maximum money from this subtree **when this node IS robbed**
> - `skipped` = the maximum money from this subtree **when this node is NOT robbed**

Now the parent has complete information and never needs to look past its immediate children.

**The transitions.** Let the left child report `(Lr, Ls)` and the right child report `(Rr, Rs)`.

*If I rob this node:* I collect `node.val`, and both children are now forbidden. So each child must contribute its `skipped` value:

```
robbed = node.val + Ls + Rs
```

*If I skip this node:* each child is free to do whatever is best for itself — it may rob or skip, independently:

```
skipped = max(Lr, Ls) + max(Rr, Rs)
```

Read `skipped` carefully. It is **not** `Lr + Rr`. Skipping the parent does not force the children to be robbed; it merely permits it. Each child independently takes its own best. This is the most common mistake on this problem.

**The base case.** A null node returns `(0, 0)` — no money either way. This is the empty subtree, and it makes the combination formulas work with no special cases for missing children.

**The answer** is `max(robbed, skipped)` at the root.

**Why a post-order DFS?** Because `robbed` and `skipped` for a node are expressions in the children's values. The children must be solved first. Post-order (left, right, then self) guarantees exactly that. Tree DP is post-order DFS; that equivalence is worth saying out loud in an interview.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(n)** — each of the `n` nodes is visited exactly once and performs constant work combining two pairs.
- **Space Complexity: O(h)** — the recursion stack holds one frame per level, where `h` is the tree height; that is O(log n) for a balanced tree and O(n) in the worst case of a degenerate chain.

### 4. THE CODE

**C++ — Post-order DFS returning a pair**

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right)
 *         : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int rob(TreeNode* root) {
        pair<int, int> result = dfs(root);
        // .first = rob this node, .second = skip it. Take the better.
        return max(result.first, result.second);
    }

private:
    // Returns { maxMoneyIfThisNodeIsRobbed, maxMoneyIfThisNodeIsSkipped }.
    pair<int, int> dfs(TreeNode* node) {
        // Base case: an empty subtree yields nothing in either scenario.
        if (node == nullptr) return {0, 0};

        // Post-order: solve both children completely before deciding here.
        pair<int, int> left  = dfs(node->left);
        pair<int, int> right = dfs(node->right);

        // Rob this node -> both children MUST be skipped.
        int robbed = node->val + left.second + right.second;

        // Skip this node -> each child independently does whatever is best.
        int skipped = max(left.first, left.second)
                    + max(right.first, right.second);

        return {robbed, skipped};
    }
};
```

**C++ — Alternative: memoized naive recursion (useful to know, slower)**

```cpp
class Solution {
public:
    int rob(TreeNode* root) {
        unordered_map<TreeNode*, int> memo;   // cache keyed by node identity
        return solve(root, memo);
    }

private:
    int solve(TreeNode* node, unordered_map<TreeNode*, int>& memo) {
        if (node == nullptr) return 0;
        if (memo.count(node)) return memo[node];

        // Choice 1: rob this node, then jump to the four grandchildren.
        int robbed = node->val;
        if (node->left) {
            robbed += solve(node->left->left,  memo)
                    + solve(node->left->right, memo);
        }
        if (node->right) {
            robbed += solve(node->right->left,  memo)
                    + solve(node->right->right, memo);
        }

        // Choice 2: skip this node, take the best from both children.
        int skipped = solve(node->left, memo) + solve(node->right, memo);

        return memo[node] = max(robbed, skipped);
    }
};
```

**Python — Post-order DFS returning a tuple (the clean solution)**

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right

class Solution:
    def rob(self, root: Optional[TreeNode]) -> int:

        def dfs(node):
            """Return (money_if_robbed, money_if_skipped) for this subtree."""
            if not node:
                return (0, 0)                  # empty subtree: nothing either way

            left_robbed,  left_skipped  = dfs(node.left)    # post-order:
            right_robbed, right_skipped = dfs(node.right)   # children first

            # Rob here -> children are forbidden, so use their 'skipped' values.
            robbed = node.val + left_skipped + right_skipped

            # Skip here -> each child chooses freely and independently.
            skipped = max(left_robbed, left_skipped) + max(right_robbed, right_skipped)

            return (robbed, skipped)

        return max(dfs(root))                  # best of the two root scenarios
```

**Python — Memoized naive version (for comparison)**

```python
class Solution:
    def rob(self, root: Optional[TreeNode]) -> int:
        memo = {}

        def solve(node):
            if not node:
                return 0
            if node in memo:
                return memo[node]

            # Rob this node: skip to grandchildren.
            robbed = node.val
            if node.left:
                robbed += solve(node.left.left) + solve(node.left.right)
            if node.right:
                robbed += solve(node.right.left) + solve(node.right.right)

            # Skip this node: children are unrestricted.
            skipped = solve(node.left) + solve(node.right)

            memo[node] = max(robbed, skipped)
            return memo[node]

        return solve(root)
```

### 5. CODE WALKTHROUGH

1. **The public method is a thin wrapper.** It calls the DFS helper and resolves the final pair with one `max`. Keeping the recursive machinery in a private helper with a *different return type* than the public method is a clean design habit — the helper returns a pair; the problem wants an int.
2. **The base case returns a zero pair, not a single zero.** Returning `(0, 0)` for a null child means the combination formulas below need no `if (node->left)` guards. The empty subtree behaves like a perfectly ordinary subtree worth nothing. Eliminating special cases this way makes the code shorter and much harder to get wrong.
3. **Both recursive calls happen before any computation.** That is post-order traversal, and it is the tree's version of "fill the table in dependency order." By the time we compute this node's values, both children are final.
4. **`robbed` uses `.second` from both children** — their skipped values — because the alarm rule forbids a robbed parent from having robbed children. The constraint of the problem appears in the code as literally one field selection.
5. **`skipped` takes a `max` per child, then adds.** The two children are independent subproblems; nothing connects them except their shared parent, which we are not robbing. Taking `max` *inside* each child before adding is correct precisely because of that independence. If you wrote `max(left.first + right.first, left.second + right.second)` you would be wrongly forcing both children into the same decision.
6. **Return the pair upward.** The entire DP state travels in the return value — no global array, no hash map, no explicit memo. The call stack provides the storage.
7. **The memoized version shows the alternative,** and it is worth being able to write. It is slower (hash lookups on pointers, plus it touches grandchildren) and needs a `memo` to avoid exponential blowup. Mention it in an interview, then present the pair-returning version as the improvement. Demonstrating that you can see the redundancy and design it away is the point.

---

## Problem 4.2 — LeetCode 543: Diameter of Binary Tree

### 1. THE BLUEPRINT (Input/Output Expectation)

The **diameter** of a binary tree is the length of the longest path between any two nodes. The path does **not** have to pass through the root. Length is measured in **edges**, not nodes.

- **Input:** the `root` of a binary tree.
- **Output:** a single integer — the number of edges on the longest path.

**Tiny concrete example:**

```
      1
     / \
    2   3
   / \
  4   5
```

The longest path is `4 → 2 → 5`, or equally `4 → 2 → 1 → 3`. The second one visits 4 nodes, so it has **3 edges**. Answer: **3**.

Edge case: a single node has diameter **0** — there are no edges.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why the naive approach is wasteful.** You might write: "for every node, diameter through it = height(left) + height(right); take the max over all nodes." That is correct, but if `height()` is a separate recursive function called at every node, you recompute heights over and over — O(n^2) on a skewed tree. The fix is to compute heights and the diameter in a **single** traversal.

**The key insight — the highest point of the path.** Any path in a tree has a unique **topmost node**: the one closest to the root. Once you fix that node, the path is forced into a simple shape — it goes down into the left subtree as far as possible, through the node, and down into the right subtree as far as possible.

So, for every node `v`:

```
best path whose topmost point is v  =  depth(left subtree) + depth(right subtree)
```

where `depth` counts **edges** downward. The global diameter is the maximum of that quantity over all nodes.

**The critical distinction — return one thing, record another.**

This is the heart of Tree DP and it trips up nearly everyone the first time.

- **What the parent needs from me (the return value):** the longest *downward* path starting at me. A path continuing up into my parent can only use **one** of my two sides — it cannot go down-left, come back up, and go down-right; that would revisit me. So I return `1 + max(leftDepth, rightDepth)`.

- **What the answer needs from me (the global update):** the longest path *bending* at me, which uses **both** sides: `leftDepth + rightDepth`. This value is a candidate for the answer but is **not** returned upward, because my parent cannot extend a path that already bends at me.

Keep those two quantities separate in your head. One goes up. One goes into a global maximum.

**The base case.** A null node has depth `-1` in edge terms. Why `-1`? Because a leaf node should have depth 0 (no edges below it), and a leaf computes `1 + max(-1, -1) = 0`. The arithmetic works out. (If you prefer counting nodes instead of edges, use 0 for null and subtract 1 from the final answer. Both are fine — just pick one and be consistent. Mixing them is how people end up off by one.)

**The answer** is the global maximum, initialised to 0.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(n)** — a single post-order traversal visits each node once, doing constant work per node.
- **Space Complexity: O(h)** — only the recursion stack, proportional to the tree height; O(log n) when balanced, O(n) for a degenerate chain.

### 4. THE CODE

**C++ — Single-pass DFS with a global maximum**

```cpp
class Solution {
public:
    int diameterOfBinaryTree(TreeNode* root) {
        int best = 0;        // the answer; at least 0 for any tree
        depth(root, best);
        return best;
    }

private:
    // Returns the depth of this subtree in EDGES (a leaf has depth 0).
    // Side effect: updates `best` with the longest path bending at this node.
    int depth(TreeNode* node, int& best) {
        // Null is one edge "above" a leaf, so -1 makes a leaf come out as 0.
        if (node == nullptr) return -1;

        int leftDepth  = depth(node->left,  best);
        int rightDepth = depth(node->right, best);

        // Path that BENDS here uses both sides. It is a candidate answer
        // but cannot be extended upward, so we only record it.
        int bendingHere = (leftDepth + 1) + (rightDepth + 1);
        best = max(best, bendingHere);

        // What the PARENT can use: one side only, plus the edge up to me.
        return 1 + max(leftDepth, rightDepth);
    }
};
```

**C++ — Node-counting variant (some people find this clearer)**

```cpp
class Solution {
public:
    int diameterOfBinaryTree(TreeNode* root) {
        int best = 0;
        height(root, best);
        return best;
    }

private:
    // Returns height in NODES (a leaf has height 1, null has height 0).
    int height(TreeNode* node, int& best) {
        if (node == nullptr) return 0;

        int lh = height(node->left,  best);
        int rh = height(node->right, best);

        // Edges on a path bending here = (nodes left) + (nodes right).
        best = max(best, lh + rh);

        return 1 + max(lh, rh);
    }
};
```

**Python — Single-pass DFS (edge convention)**

```python
class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        self.best = 0          # longest path found anywhere, in edges

        def depth(node) -> int:
            """Return subtree depth in edges; a leaf is 0, null is -1."""
            if not node:
                return -1

            left_depth  = depth(node.left)
            right_depth = depth(node.right)

            # Path bending at this node uses BOTH sides.
            # (+1 for the edge down to each child.)
            bending_here = (left_depth + 1) + (right_depth + 1)
            self.best = max(self.best, bending_here)

            # Upward, only ONE side can be used.
            return 1 + max(left_depth, right_depth)

        depth(root)
        return self.best
```

**Python — Node-counting variant with a nonlocal variable**

```python
class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        best = 0

        def height(node) -> int:
            """Return height in nodes; null is 0, a leaf is 1."""
            nonlocal best
            if not node:
                return 0

            lh = height(node.left)
            rh = height(node.right)

            # Edge count of the path bending here.
            best = max(best, lh + rh)

            return 1 + max(lh, rh)

        height(root)
        return best
```

### 5. CODE WALKTHROUGH

1. **The public method owns the answer variable** and passes it by reference (C++) or captures it via `self` / `nonlocal` (Python). The helper's job is to compute depths; updating the answer is a side effect along the way. Separating "what I return" from "what I record" is the structural lesson of this problem.
2. **The null base case sets the whole numbering scheme.** With `-1` for null, a leaf computes `1 + max(-1, -1) = 0`, correctly reporting zero edges below it. Every `+1` elsewhere in the function is consistent with that choice. If you ever get an answer that is off by exactly one or two, this is the first place to look.
3. **Recurse into both children first.** Post-order again. Depths must be known before the node can do anything.
4. **Compute the bending path and record it.** `(leftDepth + 1) + (rightDepth + 1)` counts the edges down to each child plus everything below. This is the only place the answer is updated, and it happens once per node — which is why the whole algorithm is O(n) instead of O(n^2).
5. **Return the one-sided depth.** `1 + max(leftDepth, rightDepth)` is strictly less information than the bending value, and that is intentional: a parent extending a path through me may use only one of my branches. Returning `bendingHere` by mistake is the single most common bug on this problem, and it produces answers that are too large.
6. **Discard the final return value.** We call `depth(root)` and ignore what it gives back, because the root's one-sided depth is not what was asked. The answer lives entirely in `best`.
7. **This template generalises widely.** The same "return one-sided, record two-sided" shape solves LeetCode 124 (Binary Tree Maximum Path Sum) — swap depths for sums and clamp negatives to zero — and LeetCode 1245 (Tree Diameter) on general trees. Learn the shape, not the problem.

---

# Category 5 — Knapsack DP

> **Core Definition:** Resource optimization problems subject to strict capacity, weight, or choice constraints.

Knapsack is the most reused pattern in all of DP. If you master the skeleton here, you will recognise it inside problems that never mention bags or weights.

**The classic story.** You have a bag that holds at most `W` kilograms. You have `n` items, each with a weight and a value. Choose items to maximise total value without exceeding `W`.

**The universal state:** `dp[i][w]` = the best result using the first `i` items with capacity exactly/at most `w`. The capacity becomes the second dimension — this is the move that converts a seemingly exponential choice problem into a polynomial table.

### The three flavours you must distinguish

| Flavour | Rule | Each item can be used |
|---|---|---|
| **0/1 Knapsack** | Take it or leave it | at most **once** |
| **Unbounded Knapsack** | Unlimited supply | **any number** of times |
| **Bounded Knapsack** | Limited supply `k` | at most **k** times |

### The single most important rule in this whole category

When you compress the 2D table to a 1D array, **the direction of the capacity loop decides which flavour you get.**

```
0/1 Knapsack      ->  for w from W down to weight[i]      (BACKWARDS)
Unbounded Knapsack ->  for w from weight[i] up to W        (FORWARDS)
```

**Why?** With a single array, `dp[w - weight[i]]` is the state we read from.
- Going **backwards**, that cell has not yet been touched this round, so it still represents "before item `i` was available." The item therefore gets used at most once.
- Going **forwards**, that cell was already updated this round, so it may already include item `i`. Reading it lets the item be used again — which is exactly what unbounded means.

Write those two lines on a card. Nearly every knapsack bug is a wrong loop direction.

### The three question types

The skeleton is identical; only the combining operator changes.

| Question | Operator | Initialisation |
|---|---|---|
| *Is it possible?* | `or` | `dp[0] = true`, rest `false` |
| *How many ways?* | `+` | `dp[0] = 1`, rest `0` |
| *Best value?* | `max` / `min` | `dp[0] = 0`, rest `-inf` / `+inf` |

**Problems in this category:** LeetCode 416 (0/1 feasibility), 322 (unbounded minimisation), 494 (0/1 counting).

---

## Problem 5.1 — LeetCode 416: Partition Equal Subset Sum

### 1. THE BLUEPRINT (Input/Output Expectation)

Given an array of positive integers, decide whether it can be split into **two groups with equal sums**. Every element must go into exactly one group.

- **Input:** an integer array `nums` of positive integers.
- **Output:** a boolean — `true` if such a split exists, `false` otherwise.

**Tiny concrete example:** `nums = [1, 5, 11, 5]`.

Total sum is 22, so each half must sum to 11. The split `[11]` and `[1, 5, 5]` works. Answer: **true**.

Counter-example: `nums = [1, 2, 3, 5]`. Total is 11, which is odd, so no equal split is possible. Answer: **false**.

### 2. THE THOUGHT PROCESS (Mental Model)

**The reframing that solves the problem.** "Split into two equal halves" sounds like it needs two decisions per element. It does not. Let `total = sum(nums)`. If a valid split exists, each side sums to `total / 2`. And here is the trick: **you only need to find one side.** Whatever you do not pick automatically forms the other side, and its sum is automatically `total - total/2 = total/2`.

So the question collapses to:

> *Is there a subset of `nums` that sums to exactly `target = total / 2`?*

That is **Subset Sum**, which is 0/1 Knapsack with a feasibility question. Each number is an item whose weight *and* value are the number itself; the capacity is `target`.

**The instant rejection.** If `total` is odd, `total / 2` is not an integer and no split can exist. Check this first and return `false` — it is one line and it removes a whole class of edge cases.

**Why brute force fails.** Each element is in or out, giving 2^n subsets. With `n = 200` that is astronomically large. But notice the overlap: subsets `{1, 5}` and `{5, 1}` reach the same running sum of 6, and from that point onward the remaining problem is identical. All that matters is *which sum have I built so far*, not *how I built it*. That observation is the state.

**Finding the state.**

> **State definition:** `dp[i][w]` = `true` if some subset of the first `i` numbers sums to exactly `w`.

**The transition.** For number `nums[i-1]`, two choices:
- **Skip it:** reachable if `w` was already reachable with the first `i-1` numbers → `dp[i-1][w]`.
- **Take it:** reachable if `w - nums[i-1]` was reachable before → `dp[i-1][w - nums[i-1]]` (only valid when `w >= nums[i-1]`).

```
dp[i][w] = dp[i-1][w] OR dp[i-1][w - nums[i-1]]
```

**The base cases.**
- `dp[0][0] = true` — the empty subset sums to 0. This is the seed that makes everything else reachable.
- `dp[0][w] = false` for `w > 0` — with no numbers you cannot build a positive sum.

**The answer** is `dp[n][target]`.

**Space compression.** Each row reads only the row above, so one boolean array of length `target + 1` suffices. Because this is **0/1** knapsack, the capacity loop must run **backwards**. Running it forwards would let a single number be counted twice: after setting `dp[5] = true` using the number 5, a forward loop would then read `dp[5]` while computing `dp[10]` and conclude that 10 is reachable using the number 5 twice. Backwards prevents exactly this.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(n · target)** where `target = sum/2` — we consider each of the `n` numbers against each of the `target` possible sums once. (This is *pseudo-polynomial*: it depends on the numeric value of the sum, not just the input length.)
- **Space Complexity: O(target)** — a single boolean array over all reachable sums, since each number's pass reads only the previous pass's state.

### 4. THE CODE

**C++ — 1D Bottom-Up (the standard solution)**

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int total = accumulate(nums.begin(), nums.end(), 0);

        // An odd total can never split into two equal integer halves.
        if (total % 2 != 0) return false;

        int target = total / 2;

        // dp[w] = true if some subset seen so far sums to exactly w.
        vector<bool> dp(target + 1, false);
        dp[0] = true;                 // the empty subset always reaches 0

        for (int num : nums) {
            // BACKWARDS: guarantees each number is used at most once,
            // because dp[w - num] still describes the state before this number.
            for (int w = target; w >= num; --w) {
                dp[w] = dp[w] || dp[w - num];
            }

            // Optional early exit: nothing will un-reach the target later.
            if (dp[target]) return true;
        }

        return dp[target];
    }
};
```

**C++ — 2D Bottom-Up (easier to reason about the first time)**

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int total = accumulate(nums.begin(), nums.end(), 0);
        if (total % 2 != 0) return false;

        int n = nums.size(), target = total / 2;

        // dp[i][w] = can the first i numbers reach the sum w?
        vector<vector<bool>> dp(n + 1, vector<bool>(target + 1, false));

        // Base case: the empty subset reaches 0 regardless of how many
        // numbers are available.
        for (int i = 0; i <= n; ++i) dp[i][0] = true;

        for (int i = 1; i <= n; ++i) {
            for (int w = 1; w <= target; ++w) {
                dp[i][w] = dp[i - 1][w];                     // skip nums[i-1]

                if (w >= nums[i - 1] && dp[i - 1][w - nums[i - 1]]) {
                    dp[i][w] = true;                         // take nums[i-1]
                }
            }
        }

        return dp[n][target];
    }
};
```

**Python — 1D Bottom-Up**

```python
class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        total = sum(nums)

        if total % 2 != 0:
            return False                 # odd total: impossible immediately

        target = total // 2

        # dp[w] = True if some subset of the numbers processed so far sums to w
        dp = [False] * (target + 1)
        dp[0] = True                     # empty subset reaches sum 0

        for num in nums:
            # Backwards so dp[w - num] refers to the state BEFORE this number.
            for w in range(target, num - 1, -1):
                dp[w] = dp[w] or dp[w - num]

            if dp[target]:               # early exit, purely an optimisation
                return True

        return dp[target]
```

**Python — Bitset trick (very fast, worth knowing for contests)**

```python
class Solution:
    def canPartition(self, nums: List[int]) -> bool:
        total = sum(nums)
        if total % 2:
            return False
        target = total // 2

        # Each bit position w of `reachable` means "sum w is achievable".
        reachable = 1                    # only bit 0 is set: sum 0 is reachable

        for num in nums:
            # Shifting left by num adds `num` to every reachable sum at once.
            reachable |= reachable << num

        # Check whether bit `target` is set.
        return (reachable >> target) & 1 == 1
```

### 5. CODE WALKTHROUGH

1. **Compute the total and reject odd sums first.** This guard is both a correctness requirement and a free speedup. It also prevents an integer-division bug where `total / 2` silently rounds down.
2. **Create `dp` of size `target + 1`.** Index `w` ranges over every sum from 0 to the target inclusive — hence the `+1`.
3. **`dp[0] = true` is the seed.** Everything in this table is generated by starting from the empty subset and adding numbers. If you forget this line, the array is all `false` and stays all `false` forever.
4. **The outer loop walks the items, the inner loop walks the capacity.** This ordering is the knapsack skeleton. Keep it in that order; swapping the loops changes the meaning (and in the counting variants, it changes combinations into permutations — see Coin Change II).
5. **The inner loop runs backwards, from `target` down to `num`.** Say the reason out loud as you write it: *"backwards, so each number is used at most once."* Stopping at `num` rather than 0 is a small bonus — any `w < num` cannot possibly use this number, so there is nothing to do there, and it also keeps `w - num` non-negative without an explicit check.
6. **`dp[w] = dp[w] || dp[w - num]`** is the feasibility version of the knapsack transition. Compare it with the `max` version for value problems and the `+` version for counting problems — same skeleton, different operator. Recognising that family resemblance is the goal of this whole category.
7. **The early exit is optional but free.** Once the target is reachable, no later number can make it unreachable, so we can stop.
8. **The bitset trick is worth understanding.** A Python integer acts as an arbitrarily wide bit array. Bit `w` set means "sum `w` is reachable." Shifting the whole mask left by `num` adds `num` to every reachable sum simultaneously, and OR-ing merges "with the number" into "without it." This does the inner loop in a handful of machine words instead of `target` iterations. In C++ the same idea uses `std::bitset<20001>` with `bits |= bits << num`.

---

## Problem 5.2 — LeetCode 322: Coin Change

### 1. THE BLUEPRINT (Input/Output Expectation)

You have coins of given denominations, with an **unlimited supply** of each. Find the **fewest coins** needed to make up a given amount. If the amount cannot be made, return `-1`.

- **Input:** an integer array `coins` (the denominations) and an integer `amount`.
- **Output:** the minimum number of coins summing to `amount`, or `-1` if impossible.

**Tiny concrete example:** `coins = [1, 2, 5]`, `amount = 11`.

The best combination is `5 + 5 + 1`, using **3** coins. Answer: 3.

Impossible case: `coins = [2]`, `amount = 3`. Every combination of 2s is even, so 3 is unreachable. Answer: **-1**.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why greedy fails — and this is the famous example.** The natural greedy rule is "always take the largest coin that fits." With US currency it happens to work, which is why people trust it. Now try `coins = [1, 3, 4]`, `amount = 6`:

- Greedy takes 4, leaving 2. Then 1, leaving 1. Then 1. Total: **3 coins** (4 + 1 + 1).
- Optimal: `3 + 3`. Total: **2 coins**.

Greedy is wrong. There is no ordering of the coins that fixes it, because the failure depends on the arithmetic relationship between denominations, not on their order. This example is the cleanest proof that greedy and DP are different tools, and it is worth memorising for interviews.

**Why brute force fails.** Try every count of every coin: exponential. But many different coin sequences reach the same remaining amount, and from there the subproblem is identical. Overlapping subproblems again.

**Finding the state.** The only thing that matters after spending some coins is *how much is left to make*.

> **State definition:** `dp[a]` = the minimum number of coins needed to make exactly the amount `a`.

**The transition.** To make amount `a`, the last coin used must be one of the denominations. Try each in turn:

```
dp[a] = 1 + min( dp[a - c] )  over all coins c with c <= a
```

In words: *pick a coin `c` as the last one, then solve the smaller amount `a - c`, and add 1 for the coin you just used.*

**The base case.** `dp[0] = 0` — making the amount zero requires zero coins. Like the `true` in Subset Sum, this is the seed from which the whole table grows.

**The "impossible" marker.** Initialise every other entry to a sentinel meaning *unreachable*. The clean choice is `amount + 1`, because any real answer uses at most `amount` coins (all 1s), so `amount + 1` is larger than any valid answer and will never be chosen by `min`. At the end, if `dp[amount]` is still the sentinel, nothing ever reached it, so return `-1`.

Using `INT_MAX` instead is tempting but dangerous: `1 + INT_MAX` overflows. If you do use `INT_MAX`, guard with `if (dp[a - c] != INT_MAX)` before adding. The `amount + 1` sentinel sidesteps that entirely.

**Unbounded, so the loop goes forwards.** A coin may be reused, so when we compute `dp[a]` we are happy for `dp[a - c]` to already include coin `c`. In the 1D formulation the amount loop runs **upwards**, which is precisely the unbounded direction from the rule at the top of this category.

**The answer** is `dp[amount]`, or `-1` if it is still the sentinel.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(amount · number_of_coins)** — for each of the `amount` sub-amounts we try every denomination once.
- **Space Complexity: O(amount)** — one array indexed by amount; the state needs no second dimension because coins may be reused freely.

### 4. THE CODE

**C++ — Bottom-Up Tabulation (amount-outer form)**

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        // Sentinel: amount + 1 is strictly greater than any real answer,
        // since the worst valid answer uses `amount` coins of value 1.
        const int UNREACHABLE = amount + 1;

        // dp[a] = fewest coins to make exactly a.
        vector<int> dp(amount + 1, UNREACHABLE);
        dp[0] = 0;                       // zero coins make zero

        for (int a = 1; a <= amount; ++a) {
            for (int c : coins) {
                if (c <= a) {
                    // Use coin c as the LAST coin, then solve a - c.
                    dp[a] = min(dp[a], dp[a - c] + 1);
                }
            }
        }

        return dp[amount] == UNREACHABLE ? -1 : dp[amount];
    }
};
```

**C++ — Knapsack-style form (coin-outer, shows the unbounded direction)**

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        const int UNREACHABLE = amount + 1;
        vector<int> dp(amount + 1, UNREACHABLE);
        dp[0] = 0;

        for (int c : coins) {
            // FORWARDS: dp[a - c] may already use coin c, which is exactly
            // what we want for an unbounded supply.
            for (int a = c; a <= amount; ++a) {
                dp[a] = min(dp[a], dp[a - c] + 1);
            }
        }

        return dp[amount] == UNREACHABLE ? -1 : dp[amount];
    }
};
```

**Python — Bottom-Up Tabulation**

```python
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        UNREACHABLE = amount + 1            # larger than any valid answer

        # dp[a] = fewest coins that make exactly a
        dp = [UNREACHABLE] * (amount + 1)
        dp[0] = 0                           # base case: zero needs no coins

        for a in range(1, amount + 1):
            for c in coins:
                if c <= a:
                    dp[a] = min(dp[a], dp[a - c] + 1)

        return dp[amount] if dp[amount] != UNREACHABLE else -1
```

**Python — Knapsack-style form**

```python
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        dp = [float('inf')] * (amount + 1)
        dp[0] = 0

        for c in coins:
            # Forwards = unbounded: each coin may be reused.
            for a in range(c, amount + 1):
                dp[a] = min(dp[a], dp[a - c] + 1)

        return dp[amount] if dp[amount] != float('inf') else -1
```

**Python — Top-Down Memoization**

```python
from functools import lru_cache

class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        @lru_cache(maxsize=None)
        def fewest(rest: int) -> float:
            if rest == 0:
                return 0                     # base case
            if rest < 0:
                return float('inf')          # overshot: invalid branch

            # Try every coin as the last one; keep the cheapest.
            return min((fewest(rest - c) + 1 for c in coins), default=float('inf'))

        result = fewest(amount)
        return -1 if result == float('inf') else int(result)
```

### 5. CODE WALKTHROUGH

1. **Choose the sentinel deliberately.** `amount + 1` is not arbitrary. The largest conceivable valid answer is `amount` coins (all of value 1), so `amount + 1` is guaranteed to be bigger than every real answer, which makes `min` ignore it automatically — and crucially, `UNREACHABLE + 1` does not overflow. This one choice removes every special case from the inner loop.
2. **`dp[0] = 0` is the anchor.** Every chain of `min(dp[a - c] + 1)` eventually bottoms out here. Without it, the table never escapes the sentinel.
3. **The outer loop walks amounts from 1 upward.** By the time we compute `dp[a]`, every smaller amount is final — dependency order satisfied.
4. **The inner loop tries every coin as the final coin.** This is the "enumerate all choices, keep the best" principle that defines DP, written as plainly as possible. The guard `c <= a` keeps the index non-negative.
5. **Return `-1` when the sentinel survives.** If `dp[amount]` was never improved, no combination reached the target.
6. **Compare the two loop orders.** The amount-outer form and the coin-outer form both work for this problem because `min` is commutative and associative over the choices — the set of (coin, sub-amount) pairs considered is the same either way. But the coin-outer form makes the **unbounded** nature explicit: the forward `a` loop is literally the rule from the top of this section. Contrast it with Partition Equal Subset Sum, which ran backwards. Lining up those two inner loops side by side is the fastest way to lock the rule into memory.
7. **One warning about loop order in a sibling problem.** For LeetCode 518 (Coin Change II, which *counts* combinations), loop order is **not** interchangeable: coins must be the outer loop, otherwise you count permutations instead of combinations and get a much larger number. Minimisation forgives the swap; counting does not. Know which kind of problem you are in.

---

## Problem 5.3 — LeetCode 494: Target Sum

### 1. THE BLUEPRINT (Input/Output Expectation)

Given an array of non-negative integers and a target, put a `+` or a `-` in front of **every** number, then concatenate them into an expression. Count how many different sign assignments make the expression evaluate to the target.

- **Input:** an integer array `nums` and an integer `target`.
- **Output:** a single integer — the number of valid sign assignments.

**Tiny concrete example:** `nums = [1, 1, 1, 1, 1]`, `target = 3`.

Each `1` gets a sign. To total 3, we need four `+` and one `-` (`4 - 1 = 3`). There are 5 positions for the single minus sign, so there are **5** ways. Answer: 5.

### 2. THE THOUGHT PROCESS (Mental Model)

**The naive view.** Each number has 2 choices, so 2^n sign assignments. With `n = 20` that is about one million — technically brute-forceable, but the intended solution is far better and generalises.

**The transformation that turns this into Knapsack.** This step is the whole problem. Follow it slowly.

Split the numbers into two groups by the sign they receive:
- `P` = the set of numbers given a `+`
- `N` = the set of numbers given a `-`

Then by definition:

```
sum(P) - sum(N) = target
```

Also, every number lands in exactly one group, so:

```
sum(P) + sum(N) = total      (where total = sum of all nums)
```

Add the two equations. The `sum(N)` terms cancel:

```
2 * sum(P) = target + total

sum(P) = (target + total) / 2
```

Now look at what we have. The signs have vanished. The question has become:

> *How many subsets of `nums` sum to exactly `(target + total) / 2`?*

That is **0/1 Knapsack, counting flavour**. Choosing the subset `P` determines everything; whatever is left over is `N` automatically.

**The two guard conditions.** The transformation is only valid when the arithmetic produces a legal subset sum:

1. **`(target + total)` must be even.** Otherwise `sum(P)` is not an integer and no assignment can work → return 0.
2. **`abs(target)` must be at most `total`.** If the target is bigger in magnitude than everything combined, it is unreachable even with all plus or all minus signs → return 0. (Equivalently, check that `(target + total) / 2` is non-negative; a negative required subset sum is impossible. Note that `target` can be negative, so the absolute value check matters.)

Write both guards before allocating the table. They prevent negative array sizes and out-of-range indices.

**Finding the state.**

> **State definition:** `dp[s]` = the number of subsets (from the numbers processed so far) that sum to exactly `s`.

**The transition.** For each number `num`, every existing way to reach `s - num` becomes a new way to reach `s` by adding `num` to that subset:

```
dp[s] += dp[s - num]
```

**The base case.** `dp[0] = 1` — exactly one subset sums to zero: the empty one. Counting problems seed with 1, not 0. (Feasibility problems seed with `true`; value problems seed with 0. Same position in the skeleton, different value.)

**The loop direction.** Each number may be used at most once, so this is **0/1** knapsack: the sum loop runs **backwards**.

**One subtle edge case.** The array may contain zeros. A zero can take either sign without changing the total, so each zero doubles the number of valid assignments. Reassuringly, the DP handles this automatically: when `num = 0`, the backwards loop performs `dp[s] += dp[s - 0]`, which is `dp[s] += dp[s]` — doubling every count, exactly as required. You do not need special code, but you should be able to explain it if asked.

**The answer** is `dp[subsetTarget]`.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(n · S)** where `S = (target + total) / 2` — each of the `n` numbers is swept across all `S` possible subset sums once.
- **Space Complexity: O(S)** — a single counting array over the achievable sums, since each number's pass reads only the previous pass's counts.

### 4. THE CODE

**C++ — 1D Bottom-Up after the subset-sum transformation**

```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int total = accumulate(nums.begin(), nums.end(), 0);

        // Guard 1: target out of reach even with all signs aligned.
        if (abs(target) > total) return 0;

        // Guard 2: (target + total) must be even for sum(P) to be an integer.
        if ((target + total) % 2 != 0) return 0;

        int subsetTarget = (target + total) / 2;

        // dp[s] = number of subsets summing to exactly s.
        vector<int> dp(subsetTarget + 1, 0);
        dp[0] = 1;                   // the empty subset sums to 0: one way

        for (int num : nums) {
            // BACKWARDS: 0/1 knapsack, each number used at most once.
            for (int s = subsetTarget; s >= num; --s) {
                dp[s] += dp[s - num];
            }
        }

        return dp[subsetTarget];
    }
};
```

**C++ — 2D version (clearer the first time, handles negatives explicitly)**

```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int total = accumulate(nums.begin(), nums.end(), 0);
        if (abs(target) > total || (target + total) % 2 != 0) return 0;

        int n = nums.size(), S = (target + total) / 2;

        // dp[i][s] = ways to pick a subset of the first i numbers summing to s.
        vector<vector<int>> dp(n + 1, vector<int>(S + 1, 0));
        dp[0][0] = 1;                       // empty prefix, empty subset

        for (int i = 1; i <= n; ++i) {
            for (int s = 0; s <= S; ++s) {
                dp[i][s] = dp[i - 1][s];                   // exclude nums[i-1]
                if (s >= nums[i - 1]) {
                    dp[i][s] += dp[i - 1][s - nums[i - 1]];  // include it
                }
            }
        }

        return dp[n][S];
    }
};
```

**Python — 1D Bottom-Up**

```python
class Solution:
    def findTargetSumWays(self, nums: List[int], target: int) -> int:
        total = sum(nums)

        # Unreachable, or the required subset sum is not an integer.
        if abs(target) > total or (target + total) % 2 != 0:
            return 0

        subset_target = (target + total) // 2

        # dp[s] = number of subsets summing to exactly s
        dp = [0] * (subset_target + 1)
        dp[0] = 1                            # one way to sum to zero: pick nothing

        for num in nums:
            # Backwards: each number may be used at most once.
            for s in range(subset_target, num - 1, -1):
                dp[s] += dp[s - num]

        return dp[subset_target]
```

**Python — Top-Down Memoization on (index, running sum)**

```python
from functools import lru_cache

class Solution:
    def findTargetSumWays(self, nums: List[int], target: int) -> int:

        @lru_cache(maxsize=None)
        def count(i: int, running: int) -> int:
            # Base case: all numbers have a sign; did we land on target?
            if i == len(nums):
                return 1 if running == target else 0

            # Try both signs for nums[i] and add the two counts.
            return count(i + 1, running + nums[i]) + count(i + 1, running - nums[i])

        return count(0, 0)
```

**Python — Dictionary-based DP (no transformation needed, handles negatives naturally)**

```python
from collections import defaultdict

class Solution:
    def findTargetSumWays(self, nums: List[int], target: int) -> int:
        # Map each achievable running sum to the number of ways to reach it.
        ways = {0: 1}

        for num in nums:
            next_ways = defaultdict(int)
            for current_sum, count in ways.items():
                next_ways[current_sum + num] += count    # assign '+'
                next_ways[current_sum - num] += count    # assign '-'
            ways = next_ways

        return ways.get(target, 0)
```

### 5. CODE WALKTHROUGH

1. **Do the algebra before writing any loop.** The three lines computing `total`, checking the guards, and deriving `subsetTarget` are the actual solution; everything after them is the standard knapsack skeleton you already know. In an interview, derive `sum(P) = (target + total) / 2` on the whiteboard out loud — that derivation is what is being tested.
2. **Both guards are mandatory.** Without the parity check, integer division silently truncates and you compute a count for the wrong target. Without the magnitude check, `(target + total) / 2` can be negative, and `vector<int> dp(negative + 1)` is undefined behaviour in C++ and a crash or empty list in Python.
3. **`dp[0] = 1` seeds the counting.** Compare the three seeds across this category: Subset Sum used `dp[0] = true`, Coin Change used `dp[0] = 0`, and Target Sum uses `dp[0] = 1`. Same slot, three different question types. Internalising this table is more valuable than memorising any individual solution.
4. **Items outside, capacity inside, running backwards.** The identical three-line shape as Problem 5.1 — only the operator changed from `||` to `+=`. When you can see that two problems differ by one character, you have genuinely learned the pattern rather than the problems.
5. **`dp[s] += dp[s - num]` accumulates rather than overwrites,** because we are counting every way, not finding one way. The backwards direction guarantees `dp[s - num]` still holds counts from *before* this number was introduced, so no subset is counted with the same element twice.
6. **Return `dp[subsetTarget]`** — the number of valid `P` sets, which equals the number of valid sign assignments one-to-one.
7. **The dictionary version is the pragmatic fallback.** If you cannot recall the algebra under pressure, this version needs no transformation at all: it tracks every reachable running sum in a hash map, handles negative sums naturally, and is only slightly slower. Having a backup solution you can always write is worth a lot in a real interview.

---

# Category 6 — Interval DP

> **Core Definition:** Subproblem evaluation over arbitrary sub-ranges or intervals, merging smaller continuous blocks into larger optimal results.

The state is a **range** `[i, j]` rather than a single position. The answer for a long range is built by splitting it at some middle point `k`, solving both halves, and paying a cost to merge them.

**How to recognise it:** the problem involves an order of operations that you get to choose — which balloon to pop first, where to cut the rope, how to parenthesise a product, which matrices to multiply first. The *order* is the thing being optimised.

**The signature loop structure.** You cannot just loop `i` then `j`, because `dp[i][j]` depends on shorter ranges inside it. So you loop by **length**, shortest first:

```
for length = 2 to n:
    for i = 0 to n - length:
        j = i + length - 1
        for k = i+1 to j-1:
            dp[i][j] = best(dp[i][j], dp[i][k] + dp[k][j] + mergeCost(i, k, j))
```

Three nested loops → typically **O(n^3)** time and **O(n^2)** space. When you see `n <= 500` or smaller in the constraints, interval DP should come to mind.

**The crucial mental trick: think LAST, not FIRST.** In almost every interval DP, reasoning about which element you handle *first* fails, because that choice changes the structure of what remains. Reasoning about which element you handle **last** works, because at that moment the surroundings are known and fixed. Both problems below hinge on this reversal.

**Problems in this category:** LeetCode 312, 1039.

---

## Problem 6.1 — LeetCode 312: Burst Balloons

### 1. THE BLUEPRINT (Input/Output Expectation)

You have `n` balloons in a row, each painted with a number. When you burst balloon `i`, you earn `nums[left] * nums[i] * nums[right]` coins, where `left` and `right` are the balloons **currently adjacent** to `i`. After it bursts, its neighbours become adjacent to each other. If a neighbour is off the end of the row, treat it as a balloon with the value 1. Burst all the balloons; maximise your total coins.

- **Input:** an integer array `nums`.
- **Output:** a single integer — the maximum coins obtainable.

**Tiny concrete example:** `nums = [3, 1, 5, 8]`.

One optimal order:

| Burst | Row before | Coins earned |
|---|---|---|
| the `1` | `[3, 1, 5, 8]` | `3 * 1 * 5 = 15` |
| the `5` | `[3, 5, 8]` | `3 * 5 * 8 = 120` |
| the `3` | `[3, 8]` | `1 * 3 * 8 = 24` |
| the `8` | `[8]` | `1 * 8 * 1 = 8` |

Total: `15 + 120 + 24 + 8 = 167`. Answer: **167**.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why greedy fails.** "Burst the balloon that pays the most right now" is wrong because bursting changes the neighbourhood for everything that follows. A small payout now can set up a huge payout later by bringing two large numbers next to each other. Local payout tells you nothing about the total.

**Why the obvious DP formulation fails.** The natural first attempt is: *"let `dp[i][j]` be the best coins from the range `[i, j]`, and consider which balloon to burst **first**."* Try to write the recurrence and you hit a wall. If you burst balloon `k` first, the array joins up, and the two remaining sides are **no longer independent** — a balloon from the left half can later become adjacent to a balloon from the right half. The subproblems leak into each other, and leaking subproblems cannot be a DP.

Sit with that failure for a minute; it is the lesson of this problem. When subproblems are not independent, you have chosen the wrong decision to branch on.

**The insight — reverse the question. Ask which balloon is burst LAST.**

Suppose in the range `(i, j)` (open interval: balloons strictly between index `i` and index `j`), the very last balloon to be burst is `k`.

At the moment `k` bursts, every other balloon inside `(i, j)` is already gone. So `k`'s neighbours at that instant are exactly `i` and `j` — the untouched boundaries. Its payout is therefore **fixed and known**:

```
nums[i] * nums[k] * nums[j]
```

No ambiguity. And now the two sides really are independent: all balloons in `(i, k)` were burst before `k`, and all in `(k, j)` were too, and neither group could ever have been adjacent to the other, because `k` sat between them the whole time.

That independence is what makes the DP valid.

**Finding the state.**

> **State definition:** `dp[i][j]` = the maximum coins obtainable by bursting **all balloons strictly between** `i` and `j`, leaving `i` and `j` themselves intact.

Note the open interval. This is unusual and is the reason the problem feels slippery at first. Say it to yourself: *"`i` and `j` are walls, not balloons I burst."*

**The transition:**

```
dp[i][j] = max over k in (i, j) of:
           dp[i][k] + dp[k][j] + nums[i] * nums[k] * nums[j]
```

**The padding.** Add a `1` at the front and the back of the array. This turns the "off the end counts as 1" rule into ordinary array access and removes every boundary check. With padding, the array has length `n + 2` and the answer lives at `dp[0][n+1]`.

**The base case.** `dp[i][j] = 0` whenever `j <= i + 1`, because there are no balloons strictly between adjacent walls. Zero-initialisation gives this for free.

**The answer** is `dp[0][n+1]` on the padded array — all real balloons burst, both artificial walls untouched.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(n^3)** — there are O(n^2) intervals and each one tries up to `n` possible last-burst balloons.
- **Space Complexity: O(n^2)** — a table entry for every pair of boundary indices.

### 4. THE CODE

**C++ — Bottom-Up Tabulation (loop by interval length)**

```cpp
class Solution {
public:
    int maxCoins(vector<int>& nums) {
        int n = nums.size();

        // Pad with 1 on both ends so out-of-range neighbours are automatic.
        vector<int> balloons(n + 2, 1);
        for (int i = 0; i < n; ++i) balloons[i + 1] = nums[i];

        int m = n + 2;   // padded length

        // dp[i][j] = max coins from bursting everything STRICTLY between i and j.
        vector<vector<int>> dp(m, vector<int>(m, 0));

        // `length` is the distance between the two walls.
        // We need at least one balloon between them, so start at 2.
        for (int length = 2; length < m; ++length) {
            for (int i = 0; i + length < m; ++i) {
                int j = i + length;          // right wall

                // Try every balloon k inside (i, j) as the LAST one burst.
                for (int k = i + 1; k < j; ++k) {
                    int coins = dp[i][k]                              // left side
                              + dp[k][j]                              // right side
                              + balloons[i] * balloons[k] * balloons[j]; // k last

                    dp[i][j] = max(dp[i][j], coins);
                }
            }
        }

        return dp[0][m - 1];
    }
};
```

**C++ — Top-Down Memoization (mirrors the thought process most directly)**

```cpp
class Solution {
public:
    int maxCoins(vector<int>& nums) {
        int n = nums.size();
        balloons.assign(n + 2, 1);
        for (int i = 0; i < n; ++i) balloons[i + 1] = nums[i];

        memo.assign(n + 2, vector<int>(n + 2, -1));
        return solve(0, n + 1);
    }

private:
    vector<int> balloons;
    vector<vector<int>> memo;

    // Best coins from bursting everything strictly inside (i, j).
    int solve(int i, int j) {
        if (j <= i + 1) return 0;                 // no balloons between the walls
        if (memo[i][j] != -1) return memo[i][j];

        int best = 0;
        for (int k = i + 1; k < j; ++k) {
            best = max(best, solve(i, k) + solve(k, j)
                             + balloons[i] * balloons[k] * balloons[j]);
        }

        return memo[i][j] = best;
    }
};
```

**Python — Bottom-Up Tabulation**

```python
class Solution:
    def maxCoins(self, nums: List[int]) -> int:
        # Pad both ends with 1 so edge neighbours need no special handling.
        balloons = [1] + nums + [1]
        m = len(balloons)

        # dp[i][j] = max coins from bursting all balloons strictly between i and j
        dp = [[0] * m for _ in range(m)]

        # Grow the interval length; shorter intervals are already solved.
        for length in range(2, m):
            for i in range(m - length):
                j = i + length

                # Choose which balloon inside (i, j) is burst LAST.
                for k in range(i + 1, j):
                    coins = (dp[i][k]                                  # left part
                             + dp[k][j]                                # right part
                             + balloons[i] * balloons[k] * balloons[j])  # k last
                    dp[i][j] = max(dp[i][j], coins)

        return dp[0][m - 1]
```

**Python — Top-Down Memoization**

```python
from functools import lru_cache

class Solution:
    def maxCoins(self, nums: List[int]) -> int:
        balloons = [1] + nums + [1]

        @lru_cache(maxsize=None)
        def solve(i: int, j: int) -> int:
            """Max coins from bursting everything strictly inside (i, j)."""
            if j <= i + 1:
                return 0                      # nothing between adjacent walls

            return max(
                solve(i, k) + solve(k, j) + balloons[i] * balloons[k] * balloons[j]
                for k in range(i + 1, j)
            )

        return solve(0, len(balloons) - 1)
```

### 5. CODE WALKTHROUGH

1. **Pad first.** `[1] + nums + [1]` converts the special rule about the ends into plain arithmetic. Every subsequent index is shifted by one, so stay alert, but the payoff — zero boundary checks in a triple-nested loop — is well worth it.
2. **Loop by `length`, not by `i` and `j` directly.** `dp[i][j]` depends on `dp[i][k]` and `dp[k][j]`, both of which span **shorter** distances. Processing in increasing length order guarantees they are already computed. This is the interval-DP equivalent of "fill the table in dependency order," and getting it wrong produces confidently wrong answers rather than crashes — which makes it hard to debug. Be deliberate here.
3. **Start `length` at 2.** With `length = 1` the walls are adjacent and there is nothing between them; `dp` is already 0 there. The loop bound `i + length < m` keeps `j` inside the array.
4. **The inner `k` loop is "try every possible last balloon."** `k` ranges strictly between the walls, so `k` is always a real balloon and never a wall.
5. **The three-term sum is the recurrence verbatim.** Left side, right side, and the fixed payout for `k`. The reason `balloons[i]` and `balloons[j]` appear — rather than whatever `k`'s original neighbours were — is the entire insight: when `k` goes last, the walls *are* its neighbours. If you can explain that sentence clearly, you have understood interval DP.
6. **Return `dp[0][m-1]`** — the full padded range, meaning every real balloon has been burst.
7. **The memoized version is often easier to produce under pressure,** because it needs no reasoning about loop order; recursion discovers the order itself. Writing it first and then saying "and I can convert this to a bottom-up triple loop by iterating over interval length" is a strong interview move.

---

## Problem 6.2 — LeetCode 1039: Minimum Score Triangulation of Polygon

### 1. THE BLUEPRINT (Input/Output Expectation)

You have a convex polygon with `n` vertices, labelled in **clockwise order** with the values in `values`. Cut the polygon into `n - 2` triangles using non-crossing diagonals. Each triangle's score is the **product of its three vertex values**, and the total score is the sum over all triangles. Find the minimum possible total score.

- **Input:** an integer array `values`, where `values[i]` is the label of vertex `i`.
- **Output:** a single integer — the smallest achievable total score.

**Tiny concrete example:** `values = [1, 3, 1, 4, 1, 5]`.

The best triangulation gives a total of **13**.

Simpler example: `values = [1, 2, 3]`. A triangle is already a triangle — only one triangulation exists, scoring `1 * 2 * 3 = 6`. Answer: 6.

Another: `values = [3, 7, 4, 5]`. A quadrilateral has two triangulations.
- Cut along the diagonal 0–2: triangles `(3,7,4)` and `(3,4,5)` → `84 + 60 = 144`.
- Cut along the diagonal 1–3: triangles `(7,4,5)` and `(3,7,5)` → `140 + 105 = 245`.

The minimum is **144**.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why brute force fails.** The number of triangulations of a convex polygon is the Catalan number `C(n-2)`, which grows exponentially. You cannot enumerate them.

**Why greedy fails.** "Always cut off the triangle with the smallest product" is a reasonable-sounding rule that breaks, because removing a cheap triangle may leave behind a shape whose remaining triangles are all expensive. The cost of a cut only reveals itself later.

**The insight — fix an edge, then ask which vertex completes its triangle.** This is the same "think about one forced thing" move as Burst Balloons, in a geometric costume.

Consider the polygon spanning vertices `i` through `j` (going clockwise), which is closed off by the single edge from `j` back to `i`. In **any** triangulation of this sub-polygon, the edge `(i, j)` belongs to **exactly one** triangle. (Every edge of a polygon is on the boundary of exactly one triangle in a triangulation — that is a basic property.) The third vertex of that triangle is some `k` strictly between `i` and `j`.

Once you pick `k`, the picture splits cleanly into three independent pieces:

1. The triangle `(i, k, j)`, costing `values[i] * values[k] * values[j]`.
2. The sub-polygon on vertices `i` through `k`.
3. The sub-polygon on vertices `k` through `j`.

Because the triangle `(i, k, j)` physically separates the two sub-polygons, no diagonal can cross between them. The pieces are genuinely independent — the same condition that made Burst Balloons work.

**Finding the state.**

> **State definition:** `dp[i][j]` = the minimum triangulation score of the sub-polygon formed by vertices `i, i+1, ..., j`.

Note this is a **closed** interval, unlike Burst Balloons' open one — yet the recurrence comes out looking nearly identical. That is not a coincidence; both are "split a range at `k` and pay a merge cost of three multiplied values."

**The transition:**

```
dp[i][j] = min over k in (i, j) of:
           dp[i][k] + dp[k][j] + values[i] * values[k] * values[j]
```

**The base cases.** If `j - i < 2`, the vertices `i..j` form a line segment or a single point, not a polygon. There is nothing to triangulate, so the score is 0. Zero-initialisation handles it.

**The answer** is `dp[0][n-1]` — the whole polygon.

**The relationship to Burst Balloons.** Put the two recurrences side by side:

```
Burst Balloons:   dp[i][j] = max over k  of  dp[i][k] + dp[k][j] + a[i]*a[k]*a[j]
Triangulation:    dp[i][j] = min over k  of  dp[i][k] + dp[k][j] + v[i]*v[k]*v[j]
```

They are the same algorithm with `max` swapped for `min`. If you solved one, you have solved the other. Noticing this is exactly the pattern-recognition skill that makes contest programmers fast.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(n^3)** — O(n^2) sub-polygons, each trying up to `n` choices for the apex vertex `k`.
- **Space Complexity: O(n^2)** — one table entry per pair of endpoint vertices.

### 4. THE CODE

**C++ — Bottom-Up Tabulation**

```cpp
class Solution {
public:
    int minScoreTriangulation(vector<int>& values) {
        int n = values.size();

        // dp[i][j] = min score to triangulate the polygon on vertices i..j.
        vector<vector<int>> dp(n, vector<int>(n, 0));

        // A real sub-polygon needs at least 3 vertices, so length >= 2
        // (length is measured as j - i).
        for (int length = 2; length < n; ++length) {
            for (int i = 0; i + length < n; ++i) {
                int j = i + length;

                dp[i][j] = INT_MAX;

                // Edge (i, j) belongs to exactly one triangle.
                // Try every possible third vertex k.
                for (int k = i + 1; k < j; ++k) {
                    int score = dp[i][k]                                 // left part
                              + dp[k][j]                                 // right part
                              + values[i] * values[k] * values[j];       // the triangle

                    dp[i][j] = min(dp[i][j], score);
                }
            }
        }

        return dp[0][n - 1];
    }
};
```

**C++ — Top-Down Memoization**

```cpp
class Solution {
public:
    int minScoreTriangulation(vector<int>& values) {
        int n = values.size();
        vals = values;
        memo.assign(n, vector<int>(n, -1));
        return solve(0, n - 1);
    }

private:
    vector<int> vals;
    vector<vector<int>> memo;

    int solve(int i, int j) {
        // Fewer than three vertices: no triangle, no cost.
        if (j - i < 2) return 0;
        if (memo[i][j] != -1) return memo[i][j];

        int best = INT_MAX;
        for (int k = i + 1; k < j; ++k) {
            best = min(best, solve(i, k) + solve(k, j)
                             + vals[i] * vals[k] * vals[j]);
        }

        return memo[i][j] = best;
    }
};
```

**Python — Bottom-Up Tabulation**

```python
class Solution:
    def minScoreTriangulation(self, values: List[int]) -> int:
        n = len(values)

        # dp[i][j] = min triangulation score for the polygon on vertices i..j
        dp = [[0] * n for _ in range(n)]

        # Grow the span. length = j - i; we need at least 3 vertices, so >= 2.
        for length in range(2, n):
            for i in range(n - length):
                j = i + length

                # The edge (i, j) is in exactly one triangle. Pick its apex k.
                dp[i][j] = min(
                    dp[i][k] + dp[k][j] + values[i] * values[k] * values[j]
                    for k in range(i + 1, j)
                )

        return dp[0][n - 1]
```

**Python — Top-Down Memoization**

```python
from functools import lru_cache

class Solution:
    def minScoreTriangulation(self, values: List[int]) -> int:

        @lru_cache(maxsize=None)
        def solve(i: int, j: int) -> int:
            if j - i < 2:
                return 0            # a segment or a point has no triangle

            return min(
                solve(i, k) + solve(k, j) + values[i] * values[k] * values[j]
                for k in range(i + 1, j)
            )

        return solve(0, len(values) - 1)
```

### 5. CODE WALKTHROUGH

1. **No padding needed here,** unlike Burst Balloons. The vertices are real and the interval is closed, so indices map directly onto the input. One less thing to keep straight.
2. **Loop by `length = j - i`, starting at 2.** When `j - i` is 0 or 1 we have a point or an edge, with score 0 — already correct from initialisation. Starting at 2 means the first sub-polygons considered are triangles, whose score is a single product with both sub-parts empty.
3. **Reset `dp[i][j]` to `INT_MAX` before the `k` loop** in the C++ version, because we are minimising and the array was zero-initialised. Forgetting this leaves every answer at 0 — a silent, total failure. (The Python version sidesteps it by using a generator inside `min`, which has no starting value to get wrong. That is a small reason to prefer the comprehension form.)
4. **The `k` loop enumerates every legal apex.** `k` runs strictly between `i` and `j`, so the triangle `(i, k, j)` is always non-degenerate and the two sub-polygons are always well defined.
5. **The three-term sum mirrors the geometry exactly:** the cost of the triangle sitting on edge `(i, j)`, plus the two independent regions it carves off. Draw a hexagon on paper, pick an edge, pick an apex, and shade the two remaining regions. Once you see that picture, the recurrence writes itself.
6. **Return `dp[0][n-1]`** — the polygon on all vertices.
7. **The two interval problems in this category share one template.** Keep a mental snippet: *loop by length → loop by start → compute end → loop by split point → combine two halves plus a merge cost.* Matrix Chain Multiplication, LeetCode 1547 (Minimum Cost to Cut a Stick), LeetCode 1000 (Merge Stones), and Optimal Binary Search Tree are all this same snippet with a different merge cost. One template, an entire problem family.

---

# Category 7 — Digit DP

> **Core Definition:** Digit-by-digit processing used to count numbers or combinations within a range `[L, R]` satisfying specific string or mathematical properties.

This is the category that looks scariest and is actually the most mechanical. Once you learn the template, every digit DP problem is the same code with a different "is this number good?" rule.

**The problem shape:** *"Count the integers between L and R that have property P."* The range can be gigantic — up to 10^18 or beyond — so you obviously cannot loop over it. Instead you **build the numbers digit by digit** and count whole families of them at once.

### The three pieces of state

Every digit DP has these, and usually nothing else:

| Piece | Meaning | Why you need it |
|---|---|---|
| `pos` | which digit position we are filling, left to right | the main loop variable |
| `tight` | is the prefix so far **exactly equal** to the bound's prefix? | if yes, the next digit is capped by the bound; if no, it can be anything 0–9 |
| *(property state)* | whatever the question tracks: a running digit sum, the last digit placed, a remainder, a bitmask of used digits | this is the only part that changes between problems |

Some problems add a fourth flag, `started` (also called `leadingZero`), to distinguish "I have not placed a real digit yet" from "I placed a zero." You need it when the property is sensitive to the *length* of the number or to which digits are allowed.

### The `tight` flag explained properly

This is the one idea people get stuck on, so read it slowly.

Say the bound is `N = 538` and we are filling digits left to right.

- At position 0, we may place `0..5`. If we place `5`, the prefix still matches `N`'s prefix, so we are **tight**: at position 1, we are capped at `3`.
- If instead we place `4` (or anything below `5`), our number is already guaranteed smaller than `N` no matter what follows. We are **not tight**: all remaining positions are free to use `0..9`.

Once you lose tightness, you never get it back. And here is the payoff: **all non-tight states with the same remaining positions and the same property value are identical**, so they share one cached answer. That collapse from 10^18 numbers to a few thousand states is the entire reason digit DP works.

### The range trick

Counting over `[L, R]` is done with a prefix subtraction:

```
answer = count(0 .. R) - count(0 .. L-1)
```

If `L` is given as a huge string, decrementing it is fiddly. A cleaner equivalent:

```
answer = count(0 .. R) - count(0 .. L) + (1 if L itself satisfies the property else 0)
```

Use whichever you find easier to get right. We use the second form below.

### Why top-down is the standard here

For every other category in this guide, bottom-up is the production choice. Digit DP is the exception. The recursion naturally handles the `tight` branch (which prunes most of the tree), the state space is sparse, and the iterative version requires manually separating tight from non-tight prefixes — more code, more bugs, no real speed gain. **Memoized recursion is the professional solution for digit DP.** We show iterative alternatives where they are genuinely clean.

**Problems in this category:** LeetCode 2719, 902.

---

## Problem 7.1 — LeetCode 2719: Count of Integers

### 1. THE BLUEPRINT (Input/Output Expectation)

You are given two numeric **strings** `num1` and `num2`, and two integers `min_sum` and `max_sum`. Count the integers `x` such that:

- `num1 <= x <= num2` (as numbers), and
- `min_sum <= digit_sum(x) <= max_sum`.

Return the count modulo `10^9 + 7`.

- **Input:** strings `num1`, `num2` (each up to 22 digits long) and integers `min_sum`, `max_sum`.
- **Output:** a single integer — the count, taken modulo `10^9 + 7`.

**Tiny concrete example:** `num1 = "1"`, `num2 = "12"`, `min_sum = 1`, `max_sum = 8`.

Check each number from 1 to 12:
- 1 through 8 have digit sums 1 through 8 — all inside `[1, 8]`. That is 8 numbers.
- 9 has digit sum 9, which is above 8. Rejected.
- 10, 11, 12 have digit sums 1, 2, 3 — all valid. That is 3 more.

Total: **11**.

The numbers are given as strings because they can be far too large for a 64-bit integer. That alone tells you digit DP is intended.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why brute force fails.** With bounds up to 10^22, looping from `num1` to `num2` would take longer than the age of the universe. We must count in bulk.

**Step 1 — reduce two bounds to one.** Define a helper:

```
countUpTo(N) = how many integers in [0, N] have a digit sum in [min_sum, max_sum]
```

Then:

```
answer = countUpTo(num2) - countUpTo(num1) + (1 if num1 itself qualifies else 0)
```

Subtracting `countUpTo(num1)` removes everything strictly below `num1` **and** `num1` itself, so we add `num1` back if it qualifies. (Zero is never counted because the constraints guarantee `min_sum >= 1`, and zero's digit sum is 0.)

**Step 2 — build numbers digit by digit.** Walk the digits of `N` from left to right. At each position, choose a digit, respecting the cap if we are still tight. Track the digit sum accumulated so far.

**Finding the state.**

> **State definition:** `f(pos, total, tight)` = the number of ways to fill positions `pos, pos+1, ..., end` such that the final digit sum lands in `[min_sum, max_sum]`, given that the digits before `pos` already sum to `total` and `tight` tells us whether the prefix exactly matches `N`'s prefix.

Do we need a `started` flag here? **No** — and understanding why is worth a moment. Leading zeros contribute 0 to the digit sum, so a number like `"0042"` has the same digit sum as `"42"`. The property we are tracking is simply blind to leading zeros. Skipping an unnecessary state dimension keeps the code smaller and faster. Always ask whether your property actually cares.

**The transition:**

```
limit = (tight ? digit of N at pos : 9)

f(pos, total, tight) = sum over d = 0..limit of
                       f(pos + 1, total + d, tight AND (d == limit))
```

The expression `tight AND (d == limit)` is the heart of it: we stay tight only if we were tight *and* we chose the maximum allowed digit. Choose anything smaller and we are free forever after.

**The base case.** When `pos` reaches the end of the string, we have built a complete number:

```
return 1 if total >= min_sum else 0
```

We only check the lower bound here, because the upper bound is enforced by pruning (below).

**The pruning.** As soon as `total > max_sum`, no continuation can help — digits only add. Return 0 immediately. This both prunes the search and caps the `total` dimension of the memo table at `max_sum`, keeping memory small.

**The answer** is the subtraction formula from Step 1, with care taken over the modulus (see the code notes).

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(D · S · 10)** where `D` is the number of digits (at most 23) and `S` is `max_sum` (at most 400) — there are `D · S · 2` states and each one loops over 10 digit choices, which is roughly 184,000 operations per bound.
- **Space Complexity: O(D · S)** — the memo table has one entry per (position, running sum, tight) triple, plus O(D) recursion stack depth.

### 4. THE CODE

**C++ — Top-Down Memoization (the standard digit DP solution)**

```cpp
class Solution {
public:
    int count(string num1, string num2, int min_sum, int max_sum) {
        minS = min_sum;
        maxS = max_sum;

        long long hi = countUpTo(num2);    // numbers in [0, num2] that qualify
        long long lo = countUpTo(num1);    // numbers in [0, num1] that qualify

        // countUpTo(num1) removed num1 itself, so add it back if it qualifies.
        int ds = 0;
        for (char c : num1) ds += c - '0';
        long long addBack = (ds >= min_sum && ds <= max_sum) ? 1 : 0;

        // (hi - lo) can be negative after the modulus, so normalise.
        return (int)(((hi - lo + addBack) % MOD + MOD) % MOD);
    }

private:
    static const int MOD = 1000000007;

    string bound;
    int minS, maxS;
    // memo[pos][runningSum][tight]; -1 means "not computed yet".
    int memo[24][401][2];

    int countUpTo(const string& N) {
        bound = N;
        memset(memo, -1, sizeof(memo));
        return dfs(0, 0, 1);           // start: position 0, sum 0, tight = true
    }

    int dfs(int pos, int total, int tight) {
        // Prune: digits only add, so we can never come back under max_sum.
        if (total > maxS) return 0;

        // Base case: a complete number has been built.
        if (pos == (int)bound.size()) {
            return (total >= minS) ? 1 : 0;
        }

        if (memo[pos][total][tight] != -1) return memo[pos][total][tight];

        // If still tight, we cannot exceed the bound's digit at this position.
        int limit = tight ? (bound[pos] - '0') : 9;

        long long ways = 0;
        for (int d = 0; d <= limit; ++d) {
            // We remain tight only if we were tight AND picked the cap digit.
            ways += dfs(pos + 1, total + d, (tight && d == limit) ? 1 : 0);
        }

        return memo[pos][total][tight] = (int)(ways % MOD);
    }
};
```

**Python — Top-Down Memoization**

```python
from functools import lru_cache

class Solution:
    def count(self, num1: str, num2: str, min_sum: int, max_sum: int) -> int:
        MOD = 10 ** 9 + 7

        def count_up_to(bound: str) -> int:
            """Count integers in [0, bound] whose digit sum is in range."""
            L = len(bound)

            @lru_cache(maxsize=None)
            def dfs(pos: int, total: int, tight: bool) -> int:
                # Prune: the sum can only grow from here.
                if total > max_sum:
                    return 0

                # Base case: the number is complete.
                if pos == L:
                    return 1 if total >= min_sum else 0

                # Cap the digit choice only while the prefix still matches.
                limit = int(bound[pos]) if tight else 9

                ways = 0
                for d in range(limit + 1):
                    ways += dfs(pos + 1, total + d, tight and d == limit)

                return ways % MOD

            result = dfs(0, 0, True)
            dfs.cache_clear()          # the cache is bound-specific; reset it
            return result

        high = count_up_to(num2)
        low = count_up_to(num1)

        # count_up_to(num1) excluded nothing above num1 but did consume num1,
        # so add num1 back if it qualifies.
        add_back = 1 if min_sum <= sum(int(c) for c in num1) <= max_sum else 0

        return (high - low + add_back) % MOD
```

**Python — Iterative Tabulation (for readers who want the bottom-up form)**

```python
class Solution:
    def count(self, num1: str, num2: str, min_sum: int, max_sum: int) -> int:
        MOD = 10 ** 9 + 7

        def count_up_to(bound: str) -> int:
            L = len(bound)

            # free[pos][s] = ways to fill positions pos..L-1 freely (no cap)
            #                so that the total added from here is exactly s.
            # Built backwards from the end.
            free = [[0] * (max_sum + 1) for _ in range(L + 1)]
            free[L][0] = 1                       # nothing left to place

            for pos in range(L - 1, -1, -1):
                for s in range(max_sum + 1):
                    total = 0
                    for d in range(10):
                        if s - d >= 0:
                            total += free[pos + 1][s - d]
                    free[pos][s] = total % MOD

            # Walk the bound left to right. At each position, every digit
            # strictly below the bound's digit releases a free suffix.
            answer = 0
            prefix_sum = 0

            for pos, ch in enumerate(bound):
                cap = int(ch)
                for d in range(cap):
                    # Remaining digits must contribute between these limits.
                    lo = max(0, min_sum - prefix_sum - d)
                    hi = max_sum - prefix_sum - d
                    if hi < 0:
                        continue
                    for s in range(lo, hi + 1):
                        answer = (answer + free[pos + 1][s]) % MOD
                prefix_sum += cap
                if prefix_sum > max_sum:
                    break                        # bound's own prefix already too big
            else:
                # Loop finished without breaking: the bound itself is a candidate.
                if min_sum <= prefix_sum <= max_sum:
                    answer = (answer + 1) % MOD

            return answer

        high = count_up_to(num2)
        low = count_up_to(num1)
        add_back = 1 if min_sum <= sum(int(c) for c in num1) <= max_sum else 0

        return (high - low + add_back) % MOD
```

### 5. CODE WALKTHROUGH

1. **The public method does the range arithmetic and nothing else.** Two calls to the helper, one correction term, one modulus cleanup. Keeping the range logic separate from the digit logic is what makes digit DP solutions readable.
2. **Normalise the subtraction.** In C++, `hi - lo` can be negative once both have been reduced modulo `MOD`, and `%` on a negative number in C++ yields a negative result. The idiom `((x % MOD) + MOD) % MOD` fixes it. In Python, `%` already returns a non-negative result for a positive modulus, so the plain expression is safe — a real language difference worth knowing.
3. **Clear the memo between bounds.** The cached values depend on `bound`, so reusing them across `num1` and `num2` would give wrong answers. C++ does `memset` inside `countUpTo`; Python calls `cache_clear()`. Forgetting this is the number-one digit DP bug, and the symptom is maddening: correct on one test, wrong on another.
4. **Prune before anything else.** `if (total > maxS) return 0;` sits at the very top, before the base case and before the cache lookup. Placing it first means the `total` index can never exceed `maxS`, which is what lets us size the memo array at `401` instead of something enormous.
5. **`limit` encodes the entire bound constraint in one line.** Tight means capped; not tight means free. Everything else about the upper bound is handled by this single ternary.
6. **`tight && d == limit` propagates tightness correctly.** Both conditions are required. A very common bug is writing just `d == limit`, which wrongly re-enters the tight state after it was already lost, producing an undercount.
7. **The base case checks only the lower bound,** because the upper bound was already enforced by the pruning line. Splitting the two-sided condition this way is a small elegance that also shrinks the state space.
8. **About the iterative version:** notice how much more code it takes. It precomputes `free[pos][s]`, the count of free suffixes achieving each additional sum, then walks the bound once, releasing a batch of free suffixes at every position where it dips below the cap. It is a faithful bottom-up translation, and it is genuinely useful when recursion depth or function-call overhead matters in a contest. But for interviews, write the memoized version — it is shorter, clearer, and the standard.

---

## Problem 7.2 — LeetCode 902: Numbers At Most N Given Digit Set

### 1. THE BLUEPRINT (Input/Output Expectation)

You are given a sorted array `digits` of distinct digit characters (for example `["1","3","5","7"]`) and an integer `n`. Using **only** those digits, with unlimited repetition, how many positive integers can you write that are **less than or equal to `n`**?

- **Input:** a sorted array of single-character strings `digits`, and an integer `n`.
- **Output:** a single integer — the count of writable positive integers at most `n`.

**Tiny concrete example:** `digits = ["1", "3", "5", "7"]`, `n = 100`.

- **1-digit numbers:** 1, 3, 5, 7 → 4 numbers, all at most 100.
- **2-digit numbers:** any of 4 choices in each position → `4 * 4 = 16` numbers, from 11 up to 77 — all at most 100.
- **3-digit numbers:** the smallest is 111, which already exceeds 100 → 0 numbers.

Total: `4 + 16 = 20`. Answer: **20**.

Note that `digits` never contains `"0"` in this problem's constraints, so there is no leading-zero complication in the counting version.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why brute force fails.** `n` can be up to 10^9, so iterating is far too slow. We need to count families of numbers at once.

**Split the problem by length.** Let `L` be the number of digits in `n`, and `k = len(digits)`.

*Part A — numbers with fewer than `L` digits.* **Every** such number is automatically less than `n`, because it is shorter. So we just count freely: for a length of `i`, each of the `i` positions independently takes any of the `k` digits, giving `k^i` numbers.

```
Part A = k^1 + k^2 + ... + k^(L-1)
```

*Part B — numbers with exactly `L` digits that are at most `n`.* This is where the `tight` idea comes in, applied directly instead of through recursion.

Walk the positions of `n` from left to right, maintaining the invariant: *"so far, my number matches `n`'s digits exactly."* At position `i`:

- **Place a digit strictly smaller than `n[i]`.** Our number is now guaranteed below `n`, no matter what follows. All remaining `L - i - 1` positions are completely free, so this contributes `(count of digits < n[i]) * k^(L - i - 1)` numbers at once. This is a whole family counted in one multiplication.

- **Place a digit exactly equal to `n[i]`.** We stay tight and move to the next position. Possible only if `n[i]` is actually in our digit set.

- **If `n[i]` is not in our digit set,** we cannot continue matching. Every remaining tight branch dies. Stop immediately and return what we have.

- **If we survive all `L` positions,** we have reconstructed `n` itself using only allowed digits, and `n <= n` is true, so add 1.

**Finding the state.** In the iterative form, the state is just the position plus an implicit tight flag that is always true until we stop. In the general recursive form:

> **State definition:** `f(pos, tight, started)` = the count of valid completions from position `pos` onward, where `tight` says the prefix still matches `n`, and `started` says we have already placed a real (non-skipped) digit.

**Why `started` is needed in the recursive version.** Numbers shorter than `L` are modelled as "skip the first few positions." A skipped position is not a digit, it is an absence. Without the `started` flag you cannot tell "not yet begun" from "placed a zero," and you would either miscount the short numbers or count the empty number as valid.

**The base case** for the recursive version: at `pos == L`, return 1 if `started` is true (we built a real positive number), otherwise 0 (we skipped everything, which is not a number).

**The answer** is `Part A + Part B`, or equivalently `f(0, true, false)`.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(L · k)** — we walk each of the `L` digit positions of `n` once and scan the `k` allowed digits at each, where `L` is about 10 and `k` is at most 9.
- **Space Complexity: O(1)** for the iterative version (a few counters and powers), or **O(L)** for the memoized recursive version, which stores a constant number of states per position.

### 4. THE CODE

**C++ — Iterative Counting (the clean, standard solution)**

```cpp
class Solution {
public:
    int atMostNGivenDigitSet(vector<string>& digits, int n) {
        string s = to_string(n);
        int L = s.size();
        int k = digits.size();

        int answer = 0;

        // ---- Part A: every number with FEWER than L digits is automatically < n.
        // There are k^len such numbers for each length.
        int power = 1;
        for (int len = 1; len < L; ++len) {
            power *= k;            // k^len
            answer += power;
        }

        // ---- Part B: numbers with exactly L digits, built to stay <= n.
        for (int i = 0; i < L; ++i) {
            bool canMatch = false;     // is s[i] available in our digit set?

            // How many free suffixes follow if we drop below s[i] right here?
            int freeSuffixes = 1;
            for (int t = 0; t < L - i - 1; ++t) freeSuffixes *= k;

            for (const string& d : digits) {
                if (d[0] < s[i]) {
                    // Strictly smaller: the rest is unconstrained.
                    answer += freeSuffixes;
                } else if (d[0] == s[i]) {
                    canMatch = true;   // we can stay tight and continue
                    break;             // digits are sorted; nothing smaller remains
                } else {
                    break;             // sorted, so everything after is larger too
                }
            }

            // If s[i] is not writable, no tight branch survives past here.
            if (!canMatch) return answer;
        }

        // We matched every digit of n, so n itself is writable and counts.
        return answer + 1;
    }
};
```

**C++ — Generic Digit DP with Memoization (the transferable template)**

```cpp
class Solution {
public:
    int atMostNGivenDigitSet(vector<string>& digits, int n) {
        s = to_string(n);
        allowed.clear();
        for (const string& d : digits) allowed.push_back(d[0] - '0');

        memo.assign(s.size() + 1, vector<vector<int>>(2, vector<int>(2, -1)));
        return dfs(0, 1, 0);        // pos 0, tight = true, started = false
    }

private:
    string s;
    vector<int> allowed;
    vector<vector<vector<int>>> memo;   // memo[pos][tight][started]

    int dfs(int pos, int tight, int started) {
        // Base case: a complete number. It counts only if we actually
        // placed at least one digit.
        if (pos == (int)s.size()) return started ? 1 : 0;

        if (memo[pos][tight][started] != -1) return memo[pos][tight][started];

        int ways = 0;

        // Option 1: leave this position empty (still building a shorter number).
        // Skipping means our number is strictly shorter, hence smaller, so
        // tightness is released.
        if (!started) {
            ways += dfs(pos + 1, 0, 0);
        }

        // Option 2: place one of the allowed digits.
        int limit = tight ? (s[pos] - '0') : 9;
        for (int d : allowed) {
            if (d > limit) break;                       // allowed[] is sorted
            ways += dfs(pos + 1, (tight && d == limit) ? 1 : 0, 1);
        }

        return memo[pos][tight][started] = ways;
    }
};
```

**Python — Iterative Counting**

```python
class Solution:
    def atMostNGivenDigitSet(self, digits: List[str], n: int) -> int:
        s = str(n)
        L = len(s)
        k = len(digits)
        digit_set = set(digits)

        answer = 0

        # Part A: all numbers strictly shorter than n are automatically smaller.
        for length in range(1, L):
            answer += k ** length

        # Part B: numbers of exactly length L, kept <= n.
        for i, ch in enumerate(s):
            # Any allowed digit below s[i] frees every remaining position.
            smaller = sum(1 for d in digits if d < ch)
            answer += smaller * (k ** (L - i - 1))

            # To keep going we must be able to match s[i] exactly.
            if ch not in digit_set:
                return answer

        # Every digit of n was matchable, so n itself is writable.
        return answer + 1
```

**Python — Generic Digit DP with Memoization**

```python
from functools import lru_cache

class Solution:
    def atMostNGivenDigitSet(self, digits: List[str], n: int) -> int:
        s = str(n)
        L = len(s)
        allowed = [int(d) for d in digits]       # already sorted ascending

        @lru_cache(maxsize=None)
        def dfs(pos: int, tight: bool, started: bool) -> int:
            if pos == L:
                return 1 if started else 0       # the empty "number" does not count

            ways = 0

            # Skip this position: we are still building a shorter number,
            # which is automatically below n, so tightness is dropped.
            if not started:
                ways += dfs(pos + 1, False, False)

            limit = int(s[pos]) if tight else 9
            for d in allowed:
                if d > limit:
                    break                        # sorted, so the rest are larger
                ways += dfs(pos + 1, tight and d == limit, True)

            return ways

        return dfs(0, True, False)
```

### 5. CODE WALKTHROUGH

1. **Part A is pure combinatorics, no DP required.** Numbers with fewer digits than `n` can never exceed it, so each position is unconstrained and the count is a simple power of `k`. Recognising which part of a problem needs no DP at all is a real skill — it keeps your solutions small.
2. **Part B walks `n`'s digits while maintaining the tight invariant.** Read the loop as: *"I have matched `n` exactly up to here. What are my options now?"*
3. **The `smaller * k^(L-i-1)` term counts an entire family in one step.** Choosing any digit below `n[i]` makes the number smaller than `n` regardless of what follows, so every one of the `k^(L-i-1)` suffixes is valid. This bulk counting is what replaces the exponential enumeration — it is digit DP's core economic trick, visible here without any recursion.
4. **Sorted input lets us `break` early.** Since `digits` is sorted ascending, once we reach a digit that is not below `n[i]`, nothing after it will be either. The `break` is a small optimisation and, more importantly, it makes the three cases (below / equal / above) explicit and easy to verify.
5. **The early `return answer` when `n[i]` is unwritable is essential.** If our digit set cannot produce `n[i]`, there is no way to keep matching, and every remaining tight branch is dead. Omitting this return silently overcounts.
6. **The final `+ 1` counts `n` itself.** We only reach that line by matching every digit, which means `n` is writable with our digit set — and `n <= n`. Leaving this out is the classic off-by-one here.
7. **The recursive template is the one to carry forward.** The iterative version is tighter for *this* problem, but the `dfs(pos, tight, started)` shape transfers to essentially every digit DP you will ever meet: LeetCode 233 (Number of Digit One), 357 (Count Numbers with Unique Digits), 600 (Non-negative Integers without Consecutive Ones), 1012 (Numbers With Repeated Digits), 2376 (Count Special Integers). Add one property field to the state, change the base-case test, and you are done. Memorise the skeleton, not the problems.

---

# Category 8 — Bitmask DP

> **Core Definition:** Utilizing binary bit configurations to compactly represent subsets or visited states in exponential search spaces.

When the problem involves **which subset of items have I used so far**, and `n` is small, you encode that subset as the bits of a single integer. Bit `i` set means "item `i` is used." Now a subset is just an array index, and the whole power set becomes a table of size `2^n`.

**The dead giveaway: `n <= 20` (sometimes `n <= 22`).** That constraint is almost a signed confession. `2^20` is about one million states, which is fast; `2^25` is thirty million, which is borderline; `2^30` is hopeless. Whenever you see a suspiciously tiny `n` alongside a question about orderings or groupings, think bitmask.

### The bit operations you must know cold

| Operation | Code | Meaning |
|---|---|---|
| Is item `i` in the set? | `mask & (1 << i)` | non-zero if yes |
| Add item `i` | `mask \| (1 << i)` | returns a new mask |
| Remove item `i` | `mask & ~(1 << i)` | returns a new mask |
| Toggle item `i` | `mask ^ (1 << i)` | flips membership |
| Size of the set | `__builtin_popcount(mask)` (C++) / `bin(mask).count('1')` (Python) | how many items are used |
| The full set | `(1 << n) - 1` | all `n` bits set |
| Is the set empty? | `mask == 0` | |

### The two template shapes

**Shape A — "the position is implied by the count."** If you are building a permutation, the number of items already placed tells you which position you are filling. So the state is *just* the mask; you do not need a separate position dimension.

```
position being filled = popcount(mask)
```

This is a lovely space saving and it appears constantly. Problem 8.1 uses it.

**Shape B — "the mask plus one extra value."** Sometimes you need a little more, such as "where am I standing right now" (the Travelling Salesman Problem uses `dp[mask][currentCity]`) or "how full is the current bucket" (Problem 8.2).

### The iteration order

Process masks in increasing numeric order. This works because adding an item only ever **sets** bits, so `mask | (1 << i)` is always numerically larger than `mask`. Counting upward therefore visits every state after all of its predecessors — dependency order for free.

**Problems in this category:** LeetCode 526, 473.

---

## Problem 8.1 — LeetCode 526: Beautiful Arrangement

### 1. THE BLUEPRINT (Input/Output Expectation)

Consider the numbers `1` through `n`. An arrangement (a permutation) `perm` is called **beautiful** if, for **every** 1-based position `i`, at least one of these holds:

- `perm[i]` is divisible by `i`, or
- `i` is divisible by `perm[i]`.

Count the beautiful arrangements.

- **Input:** a single integer `n` (with `1 <= n <= 15`).
- **Output:** a single integer — the number of beautiful arrangements.

**Tiny concrete example:** `n = 2`.

The two permutations are:
- `[1, 2]`: position 1 holds 1 (`1 % 1 == 0`, fine); position 2 holds 2 (`2 % 2 == 0`, fine). **Beautiful.**
- `[2, 1]`: position 1 holds 2 (`2 % 1 == 0`, fine); position 2 holds 1 (`2 % 1 == 0`, fine). **Beautiful.**

Answer: **2**.

For `n = 3` the answer is 3; for `n = 15` it is 24,679.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why brute force fails — but only just.** Generating all `n!` permutations and testing each is `15! ≈ 1.3 trillion` operations. Far too slow. The constraint `n <= 15` is the hint: `15!` is impossible but `2^15 = 32,768` is trivial. That gap between `n!` and `2^n` is exactly what bitmask DP buys you.

**Why backtracking alone is not enough.** Plain backtracking with pruning does pass this problem, and it is a perfectly good answer. But it still explores the same *set* of used numbers through many different orderings. Caching on the set collapses all of those into one computation. Understanding why they collapse is the real lesson.

**The collapsing insight.** Suppose we have filled the first three positions. Does it matter *in what order* we placed the numbers `{2, 5, 7}`? For the purpose of filling position 4 onward — **no**. All that matters is:

1. Which numbers are still available (the complement of `{2, 5, 7}`), and
2. Which position comes next (position 4).

And notice: the second fact is *determined* by the first. If three numbers are used, the next position must be 4. So the set alone is a complete description of the state.

> **State definition:** `dp[mask]` = the number of ways to beautifully fill positions `1` through `popcount(mask)` using exactly the numbers whose bits are set in `mask`.

Bit `i` of the mask represents the number `i + 1` (because bits are 0-indexed and the numbers start at 1). Write that mapping down; it is a frequent source of off-by-one errors.

**The transition.** For a given `mask`, let `pos = popcount(mask) + 1` — the next position to fill. For each number `num` from 1 to `n` that is **not** yet in the mask, check the beauty condition:

```
if num % pos == 0 or pos % num == 0:
    dp[mask | (1 << (num - 1))] += dp[mask]
```

This is a **push** formulation: we take the ways already counted at `mask` and push them forward into every legal successor state. (The equivalent **pull** formulation loops over the last-placed number and reads backwards; both are fine. Push often reads more naturally for permutation building.)

**The base case.** `dp[0] = 1`. There is exactly one way to fill zero positions: do nothing. Counting problems always seed with 1.

**The answer** is `dp[(1 << n) - 1]` — every number used, every position filled.

**The pruning that makes it fast.** We only ever push from masks that have a non-zero count. Most masks are reachable, but the divisibility condition rejects many transitions, so the effective work is well below the `2^n · n` worst case.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(2^n · n)** — there are `2^n` masks, and for each we scan all `n` numbers to find the unused ones; for `n = 15` that is about 500,000 cheap operations.
- **Space Complexity: O(2^n)** — one integer per subset, which is 32,768 entries at `n = 15`.

### 4. THE CODE

**C++ — Bottom-Up Bitmask DP (push formulation)**

```cpp
class Solution {
public:
    int countArrangement(int n) {
        int full = (1 << n) - 1;          // all n numbers used

        // dp[mask] = ways to beautifully fill the first popcount(mask) positions
        //            using exactly the numbers in `mask`.
        vector<int> dp(full + 1, 0);
        dp[0] = 1;                        // one way to place nothing

        // Masks only ever grow, so ascending order is a valid dependency order.
        for (int mask = 0; mask <= full; ++mask) {
            if (dp[mask] == 0) continue;  // unreachable state: nothing to push

            // The number of used items tells us which position comes next.
            int pos = __builtin_popcount(mask) + 1;
            if (pos > n) continue;        // mask is already full

            for (int num = 1; num <= n; ++num) {
                int bit = 1 << (num - 1);

                if (mask & bit) continue;                 // num already placed

                // The beauty rule for putting `num` at position `pos`.
                if (num % pos == 0 || pos % num == 0) {
                    dp[mask | bit] += dp[mask];
                }
            }
        }

        return dp[full];
    }
};
```

**C++ — Top-Down Memoization (pull formulation)**

```cpp
class Solution {
public:
    int countArrangement(int n) {
        N = n;
        memo.assign(1 << n, -1);
        return solve((1 << n) - 1);       // start from the full mask, work down
    }

private:
    int N;
    vector<int> memo;

    // Ways to fill positions 1..popcount(mask) using exactly the numbers in mask.
    int solve(int mask) {
        if (mask == 0) return 1;                  // nothing placed: one empty way
        if (memo[mask] != -1) return memo[mask];

        int pos = __builtin_popcount(mask);       // the LAST position filled
        int total = 0;

        // Which number sat at position `pos`? Try each one in the mask.
        for (int num = 1; num <= N; ++num) {
            int bit = 1 << (num - 1);
            if (!(mask & bit)) continue;

            if (num % pos == 0 || pos % num == 0) {
                total += solve(mask ^ bit);       // remove num, recurse
            }
        }

        return memo[mask] = total;
    }
};
```

**Python — Bottom-Up Bitmask DP**

```python
class Solution:
    def countArrangement(self, n: int) -> int:
        full = (1 << n) - 1

        # dp[mask] = ways to fill the first popcount(mask) positions
        #            using exactly the numbers encoded in mask
        dp = [0] * (full + 1)
        dp[0] = 1                        # one way to place nothing

        for mask in range(full + 1):
            if dp[mask] == 0:
                continue                 # unreachable, skip

            pos = bin(mask).count('1') + 1      # next position to fill (1-based)
            if pos > n:
                continue

            for num in range(1, n + 1):
                bit = 1 << (num - 1)

                if mask & bit:
                    continue             # num is already used

                if num % pos == 0 or pos % num == 0:
                    dp[mask | bit] += dp[mask]

        return dp[full]
```

**Python — Top-Down Memoization**

```python
from functools import lru_cache

class Solution:
    def countArrangement(self, n: int) -> int:

        @lru_cache(maxsize=None)
        def solve(mask: int) -> int:
            if mask == 0:
                return 1                      # base case: nothing placed

            pos = bin(mask).count('1')        # the last position that was filled

            total = 0
            for num in range(1, n + 1):
                bit = 1 << (num - 1)
                if not (mask & bit):
                    continue                  # num is not in this set

                if num % pos == 0 or pos % num == 0:
                    total += solve(mask ^ bit)   # peel num off and recurse

            return total

        return solve((1 << n) - 1)
```

### 5. CODE WALKTHROUGH

1. **`full = (1 << n) - 1` is the goal state** — all `n` bits set, meaning every number has been placed. Writing it once as a named constant keeps the loop bounds readable.
2. **`dp[0] = 1` seeds the count.** The empty arrangement is one valid way to have placed nothing. Every other value in the table is ultimately a sum of copies of this 1.
3. **Iterating masks in ascending order is sufficient.** Since `mask | bit` is strictly greater than `mask`, every state is finalised before anything reads it. No `length` loop, no sorting — the integers do the ordering for us.
4. **`if (dp[mask] == 0) continue;` is a real optimisation.** Unreachable states contribute nothing, and skipping them avoids an inner loop over `n` numbers. On this problem many masks are unreachable because the divisibility rule blocks them.
5. **`pos = popcount(mask) + 1` is the trick that removes a whole state dimension.** A naive implementation would store `dp[mask][pos]`, doubling or worse the memory for no reason. Because the mask already encodes how many numbers are placed, the position is derivable. Whenever you are tempted to add a state variable, pause and ask whether it is already implied.
6. **The beauty check is applied at the moment of placement.** We never build an arrangement and then validate it — invalid transitions simply never happen. Validating during construction rather than afterwards is what keeps the state space clean.
7. **Push versus pull.** The bottom-up version *pushes* `dp[mask]` into its successors; the memoized version *pulls* from its predecessors by asking "which number was last?". They compute the same table. Push tends to be shorter for permutation building; pull tends to be shorter when the recursion naturally shrinks the problem. Be comfortable writing either, and say which you are using so an interviewer can follow.
8. **Return `dp[full]`,** the state where every number is placed and every position is filled.

---

## Problem 8.2 — LeetCode 473: Matchsticks to Square

### 1. THE BLUEPRINT (Input/Output Expectation)

You have a collection of matchsticks with given lengths. You must use **every** matchstick exactly once, and you may not break or bend any of them. Can you arrange them to form a **square**?

- **Input:** an integer array `matchsticks` of lengths (with `n <= 15`).
- **Output:** a boolean — `true` if the matchsticks can form a square, `false` otherwise.

**Tiny concrete example:** `matchsticks = [1, 1, 2, 2, 2]`.

The total is 8, so each side must be 2. The sides are `[2]`, `[2]`, `[2]`, and `[1, 1]`. Every stick is used. Answer: **true**.

Counter-example: `matchsticks = [3, 3, 3, 3, 4]`. The total is 16, so each side must be 4. The stick of length 4 fills one side exactly, but the four 3s cannot combine into sides of length 4. Answer: **false**.

### 2. THE THOUGHT PROCESS (Mental Model)

**Restating the problem.** "Form a square" means: partition all the matchsticks into **4 groups with equal sums**. The geometry is a disguise; this is a partitioning problem.

**The immediate rejections.** Two checks before any real work:
1. If `total % 4 != 0`, the side length is not an integer → `false`.
2. If the longest matchstick exceeds `total / 4`, it cannot fit on any side → `false`.

Both are one-liners that remove huge numbers of inputs instantly.

**Why plain backtracking struggles.** The obvious approach assigns each stick to one of 4 sides, giving `4^15 ≈ 10^9` branches. With good pruning (sort descending, skip duplicate side states, abandon when a side overflows) it does pass — and it is a legitimate answer worth mentioning. But the pruning rules are subtle and easy to get wrong under pressure. Bitmask DP is more mechanical and more reliable.

**The reframing that enables the DP.** Do not think "assign each stick to one of four sides." Think instead:

> *Fill the sides **one at a time**, in order. Keep adding sticks to the current side until it is exactly full, then start the next side.*

Now the only thing we need to track is: which sticks are used, and how full the current side is.

**And here is the beautiful part — the second value is free.** If a set of sticks (encoded as `mask`) has total length `S`, and we have been filling sides in order and completing each one exactly, then:

- the number of completed sides is `S / side`, and
- the fill level of the current side is `S % side`.

Both are determined entirely by the subset sum. We do **not** need a second state dimension. One array indexed by mask is enough.

> **State definition:** `dp[mask]` = `true` if the sticks in `mask` can be arranged to fill some whole number of complete sides plus a partial current side of length exactly `sum(mask) % side`; `false` if this subset cannot be arranged legally at all.

Equivalently and more usefully for code: `dp[mask]` is reachable if there exists an ordering of the sticks in `mask` in which no side ever overflows.

**The transition.** From a reachable `mask`, try adding each unused stick `i`:

```
currentFill = sum(mask) % side

if currentFill + matchsticks[i] <= side:
    dp[mask | (1 << i)] = true
```

The condition `currentFill + len <= side` is the only rule. It says: *the stick must fit in the side we are currently building.* If it fits exactly, `currentFill` wraps to 0 and we automatically begin the next side — the modulo arithmetic handles the transition with no extra code. If it would overflow, we are not allowed to place it now; some other stick must come first.

**The base case.** `dp[0] = true` — the empty set is trivially a valid (empty) arrangement.

**The answer** is `dp[(1 << n) - 1]` — every stick placed with no side ever overflowing. Since the total is exactly `4 * side` and no side ever exceeded `side`, all four sides must be exactly full.

**Why this is correct.** Any valid square gives an ordering (side 1's sticks, then side 2's, and so on) that our DP can follow. Conversely, any path our DP accepts fills sides in order without overflow and consumes the full total, which is exactly four complete sides. The two directions match, so the DP is exact.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(2^n · n)** — we visit every subset once and try adding each of the `n` sticks, which is roughly 500,000 operations at `n = 15`.
- **Space Complexity: O(2^n)** — one boolean per subset, plus an optional precomputed subset-sum array of the same size.

### 4. THE CODE

**C++ — Bottom-Up Bitmask DP**

```cpp
class Solution {
public:
    bool makesquare(vector<int>& matchsticks) {
        int n = matchsticks.size();
        if (n < 4) return false;          // need at least one stick per side

        int total = accumulate(matchsticks.begin(), matchsticks.end(), 0);
        if (total % 4 != 0) return false; // side length must be an integer

        int side = total / 4;

        // No single stick may exceed one side.
        for (int len : matchsticks)
            if (len > side) return false;

        int full = (1 << n) - 1;

        // subsetSum[mask] = total length of the sticks in `mask`.
        vector<int> subsetSum(full + 1, 0);
        for (int mask = 1; mask <= full; ++mask) {
            int lowBit = mask & (-mask);              // lowest set bit
            int index  = __builtin_ctz(mask);         // its position
            subsetSum[mask] = subsetSum[mask ^ lowBit] + matchsticks[index];
        }

        // dp[mask] = can the sticks in `mask` be laid down with no side overflow?
        vector<bool> dp(full + 1, false);
        dp[0] = true;                     // the empty arrangement is valid

        for (int mask = 0; mask <= full; ++mask) {
            if (!dp[mask]) continue;      // unreachable: nothing to extend

            // How full is the side we are currently building?
            int currentFill = subsetSum[mask] % side;

            for (int i = 0; i < n; ++i) {
                int bit = 1 << i;
                if (mask & bit) continue;                 // already used

                // The stick must fit in the CURRENT side.
                // If it fits exactly, the modulo wraps to 0 and the next
                // side begins automatically.
                if (currentFill + matchsticks[i] <= side) {
                    dp[mask | bit] = true;
                }
            }
        }

        return dp[full];
    }
};
```

**C++ — Backtracking with pruning (the alternative worth mentioning)**

```cpp
class Solution {
public:
    bool makesquare(vector<int>& matchsticks) {
        int total = accumulate(matchsticks.begin(), matchsticks.end(), 0);
        if (total % 4 != 0) return false;

        int side = total / 4;

        // Descending order: place big sticks first so dead ends appear early.
        sort(matchsticks.rbegin(), matchsticks.rend());
        if (matchsticks[0] > side) return false;

        vector<int> sides(4, 0);
        return place(matchsticks, 0, sides, side);
    }

private:
    bool place(vector<int>& sticks, int i, vector<int>& sides, int target) {
        if (i == (int)sticks.size()) return true;   // all sticks placed

        for (int s = 0; s < 4; ++s) {
            if (sides[s] + sticks[i] > target) continue;

            // Key pruning: if two sides currently have the same length,
            // trying the second one explores an identical search tree.
            bool duplicate = false;
            for (int t = 0; t < s; ++t)
                if (sides[t] == sides[s]) { duplicate = true; break; }
            if (duplicate) continue;

            sides[s] += sticks[i];
            if (place(sticks, i + 1, sides, target)) return true;
            sides[s] -= sticks[i];                  // undo and try the next side
        }

        return false;
    }
};
```

**Python — Bottom-Up Bitmask DP**

```python
class Solution:
    def makesquare(self, matchsticks: List[int]) -> bool:
        n = len(matchsticks)
        if n < 4:
            return False

        total = sum(matchsticks)
        if total % 4 != 0:
            return False

        side = total // 4
        if max(matchsticks) > side:
            return False

        full = (1 << n) - 1

        # subset_sum[mask] = total length of the sticks in mask
        subset_sum = [0] * (full + 1)
        for mask in range(1, full + 1):
            low = mask & -mask                 # isolate the lowest set bit
            idx = low.bit_length() - 1         # its index
            subset_sum[mask] = subset_sum[mask ^ low] + matchsticks[idx]

        # dp[mask] = can these sticks be laid out with no side overflowing?
        dp = [False] * (full + 1)
        dp[0] = True

        for mask in range(full + 1):
            if not dp[mask]:
                continue

            current_fill = subset_sum[mask] % side     # fill of the active side

            for i in range(n):
                bit = 1 << i
                if mask & bit:
                    continue                           # stick already used

                if current_fill + matchsticks[i] <= side:
                    dp[mask | bit] = True

        return dp[full]
```

**Python — Backtracking with pruning**

```python
class Solution:
    def makesquare(self, matchsticks: List[int]) -> bool:
        total = sum(matchsticks)
        if total % 4 != 0:
            return False

        side = total // 4
        matchsticks.sort(reverse=True)      # big sticks first: fail fast
        if matchsticks[0] > side:
            return False

        sides = [0] * 4

        def place(i: int) -> bool:
            if i == len(matchsticks):
                return True                 # every stick found a home

            for s in range(4):
                if sides[s] + matchsticks[i] > side:
                    continue
                # Skip a side that duplicates an earlier one: same subtree.
                if sides[s] in sides[:s]:
                    continue

                sides[s] += matchsticks[i]
                if place(i + 1):
                    return True
                sides[s] -= matchsticks[i]  # backtrack

            return False

        return place(0)
```

### 5. CODE WALKTHROUGH

1. **Do the cheap rejections first.** Fewer than 4 sticks, a total not divisible by 4, or a stick longer than one side — each is a one-line `false`. These guards also protect the later arithmetic from nonsense values.
2. **Precompute `subsetSum` with the lowest-set-bit trick.** The identity `sum(mask) = sum(mask without its lowest bit) + length of that lowest stick` fills the whole array in O(2^n) time. Since masks ascend and `mask ^ lowBit` is always smaller, the dependency is already satisfied. `mask & (-mask)` isolates the lowest set bit, and `__builtin_ctz` (or `bit_length() - 1` in Python) converts it to an index. This is a standard bitmask idiom — learn it once, reuse it forever.
3. **`dp[0] = true` is the start.** From the empty arrangement we grow outward one stick at a time.
4. **`currentFill = subsetSum[mask] % side` is the idea that makes this problem elegant.** Because every completed side is exactly `side` long, the remainder after division tells you precisely how full the in-progress side is. A second state dimension would have been correct but wasteful; the arithmetic already contains the information. Pausing to ask *"is this value implied by what I already store?"* is the habit that turns adequate DP solutions into clean ones.
5. **The fit check is the only rule in the loop.** `currentFill + matchsticks[i] <= side` enforces that we never overflow the side under construction. When the sum hits exactly `side`, the next `% side` evaluates to 0 and we seamlessly start a fresh side — no branch, no counter, no special case.
6. **Skip unreachable masks.** `if (!dp[mask]) continue;` prevents us from propagating from states that cannot occur, which in practice cuts the work substantially.
7. **Return `dp[full]`.** Every stick placed, no side ever over the limit, and the total equals `4 * side` — therefore all four sides are exactly full. State the argument like that in an interview; it shows you know *why* the DP is correct and not just that it passes.
8. **Know the backtracking version too.** Its two prunings are instructive: sorting descending makes large sticks fail fast, and skipping a side whose current length duplicates an earlier side avoids exploring symmetric, identical subtrees. That symmetry-breaking idea generalises to many search problems. Mentioning both approaches, and explaining that the bitmask version trades memory for predictable O(2^n · n) behaviour, is a strong answer.

---

# Category 9 — Game Theory DP

> **Core Definition:** Turn-based simulation assuming optimal moves from alternating players, utilizing minimax strategies.

Two players alternate turns, both play perfectly, and we must determine the outcome. The DP tracks the game state; the alternation is what makes it different from everything so far.

**How to recognise it:** the words *two players*, *take turns*, *both play optimally*, *who wins*.

### The one trick that simplifies everything: track the DIFFERENCE

The naive approach stores two numbers per state — "player 1's score" and "player 2's score" — plus a flag for whose turn it is. That triples your state space and invites bugs.

Instead, store a single number:

> `dp[state]` = **(current player's score) − (other player's score)**, assuming both play optimally from here.

This works because the game is symmetric: whoever is to move faces the same kind of problem. The genius of the difference formulation is in the recursion:

```
dp[state] = max over moves m of ( gain(m) − dp[state after m] )
```

The **minus sign** is the entire minimax. After I move, it becomes my opponent's turn, and `dp[next state]` is *their* advantage measured from *their* point of view. Their advantage is my disadvantage, so I subtract it. The opponent's maximisation automatically becomes my minimisation. No turn flag, no second array, no separate `min` function.

Say it out loud: **"my gain minus their best future advantage."** That one sentence is this entire category.

**Reading the answer:** `dp[initial state] > 0` means player 1 wins; `= 0` means a draw; `< 0` means player 2 wins.

**The usual state shape.** Most "take from either end" games use an interval `[i, j]`, so these are interval DP problems too — the same length-based loop structure from Category 6.

**Problems in this category:** LeetCode 486, 877.

---

## Problem 9.1 — LeetCode 486: Predict the Winner

### 1. THE BLUEPRINT (Input/Output Expectation)

An array of non-negative integers is placed on a table. Two players alternate turns. On each turn, a player takes the number from **either the far left or the far right** of what remains, and adds it to their score. The game ends when nothing is left. Both players play optimally.

Return `true` if **player 1** can win or tie, `false` otherwise.

- **Input:** an integer array `nums`.
- **Output:** a boolean — whether player 1 can guarantee at least a tie.

**Tiny concrete example:** `nums = [1, 5, 2]`.

Player 1's options:
- **Take 1.** Remaining `[5, 2]`. Player 2 takes 5 (optimal), player 1 takes 2. Final: 3 vs 5. Player 1 loses.
- **Take 2.** Remaining `[1, 5]`. Player 2 takes 5, player 1 takes 1. Final: 3 vs 5. Player 1 loses.

Both lines lose, so the answer is **false**.

Another: `nums = [1, 5, 233, 7]` → **true**. Player 1 takes 7, forcing player 2 into a position where 233 becomes reachable for player 1.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why greedy fails.** "Always take the larger end" is wrong, and `[1, 5, 233, 7]` shows why. Greedy takes 7 (larger than 1), then player 2 takes 1 or 233 — a greedy player 2 takes 233 and wins easily. The real optimal line for player 1 is still to take 7, but the *reason* is not that 7 is larger; it is that taking 7 leaves `[1, 5, 233]`, where whatever player 2 does, player 1 can reach 233. Greedy gets the right move for the wrong reason, which means it gets the wrong move whenever the numbers shift. A move must be valued by the position it leaves behind, not by its own size.

**Why brute force fails.** Each turn branches two ways over up to `n` turns → O(2^n). But many different move sequences lead to the same remaining interval, and from an interval the future is identical regardless of how we arrived. Overlapping subproblems confirmed.

**The state.** What remains is always a contiguous block, because we only ever take from the ends.

> **State definition:** `dp[i][j]` = the maximum score **difference** (current player minus opponent) that the player to move can guarantee on the subarray `nums[i..j]`.

**The transition.** The current player has exactly two moves.

*Take the left element `nums[i]`.* I immediately gain `nums[i]`. Then my opponent faces `nums[i+1..j]` and will achieve an advantage of `dp[i+1][j]` **in their favour**. From my perspective that is a loss, so:

```
value of taking left = nums[i] - dp[i+1][j]
```

*Take the right element `nums[j]`.* Symmetrically:

```
value of taking right = nums[j] - dp[i][j-1]
```

The current player picks the better:

```
dp[i][j] = max(nums[i] - dp[i+1][j],
               nums[j] - dp[i][j-1])
```

That is the entire algorithm. Notice there is no `min` anywhere, and no variable tracking whose turn it is. The subtraction does all of that work.

**The base case.** `dp[i][i] = nums[i]`. With one element left, the player to move takes it and the opponent gets nothing, so the difference is exactly that value.

**The answer** is `dp[0][n-1] >= 0`. Player 1 moves first, so `dp[0][n-1]` is player 1's guaranteed margin. Non-negative means a win or a tie, which the problem counts as success.

**The loop order.** `dp[i][j]` depends on `dp[i+1][j]` and `dp[i][j-1]`, both of which cover **shorter** intervals. So iterate by increasing length (as in Category 6), or equivalently loop `i` **downward** and `j` **upward**. Both work; pick one and be consistent.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(n^2)** — one entry for each of the O(n^2) intervals, each computed with a constant-time two-way comparison.
- **Space Complexity: O(n^2)** for the table, reducible to **O(n)** with a single rolling array, since each interval length depends only on the previous length.

### 4. THE CODE

**C++ — Bottom-Up Tabulation (loop by interval length)**

```cpp
class Solution {
public:
    bool predictTheWinner(vector<int>& nums) {
        int n = nums.size();

        // dp[i][j] = best achievable (me - opponent) on nums[i..j]
        //            for whoever moves first on that subarray.
        vector<vector<int>> dp(n, vector<int>(n, 0));

        // Base case: a single element is taken by the mover.
        for (int i = 0; i < n; ++i) dp[i][i] = nums[i];

        // Grow the interval. Shorter intervals are already final.
        for (int length = 2; length <= n; ++length) {
            for (int i = 0; i + length - 1 < n; ++i) {
                int j = i + length - 1;

                // "My gain minus the opponent's best future advantage."
                int takeLeft  = nums[i] - dp[i + 1][j];
                int takeRight = nums[j] - dp[i][j - 1];

                dp[i][j] = max(takeLeft, takeRight);
            }
        }

        // Player 1 moves first; a non-negative margin means win or tie.
        return dp[0][n - 1] >= 0;
    }
};
```

**C++ — Space-Optimized to O(n)**

```cpp
class Solution {
public:
    bool predictTheWinner(vector<int>& nums) {
        int n = nums.size();

        // dp[j] plays the role of dp[i][j] for the current value of i.
        vector<int> dp(nums.begin(), nums.end());   // length-1 intervals

        // i must decrease so that dp[i+1][*] is still in the array
        // from the previous outer iteration.
        for (int i = n - 2; i >= 0; --i) {
            for (int j = i + 1; j < n; ++j) {
                // dp[j]     currently holds dp[i+1][j]  (previous i)
                // dp[j - 1] currently holds dp[i][j-1]  (already updated)
                dp[j] = max(nums[i] - dp[j], nums[j] - dp[j - 1]);
            }
        }

        return dp[n - 1] >= 0;
    }
};
```

**Python — Bottom-Up Tabulation**

```python
class Solution:
    def predictTheWinner(self, nums: List[int]) -> bool:
        n = len(nums)

        # dp[i][j] = best (mover - opponent) margin on nums[i..j]
        dp = [[0] * n for _ in range(n)]

        for i in range(n):
            dp[i][i] = nums[i]           # one element: the mover takes it

        for length in range(2, n + 1):
            for i in range(n - length + 1):
                j = i + length - 1

                take_left  = nums[i] - dp[i + 1][j]    # gain, then opponent moves
                take_right = nums[j] - dp[i][j - 1]

                dp[i][j] = max(take_left, take_right)

        return dp[0][n - 1] >= 0
```

**Python — Space-Optimized to O(n)**

```python
class Solution:
    def predictTheWinner(self, nums: List[int]) -> bool:
        n = len(nums)
        dp = nums[:]                      # dp[j] starts as dp[j][j]

        for i in range(n - 2, -1, -1):    # i decreasing
            for j in range(i + 1, n):
                # dp[j] is still dp[i+1][j]; dp[j-1] is already dp[i][j-1].
                dp[j] = max(nums[i] - dp[j], nums[j] - dp[j - 1])

        return dp[-1] >= 0
```

**Python — Top-Down Memoization (the clearest statement of the idea)**

```python
from functools import lru_cache

class Solution:
    def predictTheWinner(self, nums: List[int]) -> bool:

        @lru_cache(maxsize=None)
        def margin(i: int, j: int) -> int:
            """Best (mover - opponent) difference achievable on nums[i..j]."""
            if i == j:
                return nums[i]            # take the last element

            # Subtracting the opponent's margin flips the perspective,
            # which is what makes their maximisation into my minimisation.
            return max(nums[i] - margin(i + 1, j),
                       nums[j] - margin(i, j - 1))

        return margin(0, len(nums) - 1) >= 0
```

### 5. CODE WALKTHROUGH

1. **The table stores a difference, not a score.** Write that in a comment, because the code is unreadable without it. A reader who thinks `dp[i][j]` is "player 1's points" will find the minus signs incomprehensible.
2. **Seed the diagonal.** `dp[i][i] = nums[i]` is the whole base case: one element, one move, the mover takes it all.
3. **Loop by interval length.** `dp[i][j]` needs `dp[i+1][j]` and `dp[i][j-1]`, which are both one shorter. Processing lengths in increasing order guarantees they are ready.
4. **Two candidate moves, one `max`.** The current player always maximises their own margin; the opponent's optimal play is already baked into the recursive values being subtracted. This is why no `min` appears.
5. **The minus sign is the minimax.** `nums[i] - dp[i+1][j]` reads as: *"I collect `nums[i]`; then my opponent, playing perfectly, builds an advantage of `dp[i+1][j]` over me; so my net margin is the difference."* If you only remember one line from this category, remember this one.
6. **Return `>= 0`, not `> 0`.** The problem statement says player 1 wins ties. Reading that detail carefully is worth a correct submission.
7. **The 1D optimization needs its index discipline.** With `i` descending and `j` ascending, at the moment we write `dp[j]` the array holds `dp[i+1][j]` at position `j` (not yet overwritten this round) and `dp[i][j-1]` at position `j-1` (already overwritten). Both operands are correct, which is exactly the same in-place trick used in Unique Paths and Maximal Square. If the loop directions were reversed, both reads would be wrong.
8. **The memoized version is the one to write first in an interview.** It is five lines, it states the recurrence without any index-order reasoning, and you can then offer the bottom-up and O(n) versions as refinements.

---

## Problem 9.2 — LeetCode 877: Stone Game

### 1. THE BLUEPRINT (Input/Output Expectation)

Alice and Bob play with piles of stones arranged in a row. The rules are identical to Predict the Winner — each turn, take the whole pile from either end — with three extra guarantees:

- The number of piles is **even**.
- The total number of stones is **odd** (so there can be no tie).
- Alice goes first, and both play optimally.

Return `true` if Alice wins.

- **Input:** an integer array `piles` of positive integers, with even length and odd sum.
- **Output:** a boolean — whether Alice wins.

**Tiny concrete example:** `piles = [5, 3, 4, 5]`.

Alice takes the first pile (5). Remaining `[3, 4, 5]`.
- If Bob takes 3, Alice takes 5 → Alice has 10, Bob has 7. Alice wins.
- If Bob takes 5, Alice takes 4 → Alice has 9, Bob has 8. Alice wins.

Either way Alice wins. Answer: **true**.

### 2. THE THOUGHT PROCESS (Mental Model)

**Part 1 — the DP solution.** This is literally Predict the Winner. Same state, same recurrence, same base case:

```
dp[i][j] = max(piles[i] - dp[i+1][j],
               piles[j] - dp[i][j-1])
```

with `dp[i][i] = piles[i]`, and the answer `dp[0][n-1] > 0`. (Strictly greater, because the odd total rules out a tie.) If you solved 9.1, you are done — write the same code.

**Part 2 — the mathematical shortcut, and why it matters.** The answer to this specific problem is **always `true`**. Alice can never lose. Here is the proof, and it is worth knowing because interviewers ask for it.

Label the piles by position: 1st, 2nd, 3rd, ..., `n`th. Split them into two groups:

- **Odd positions:** 1st, 3rd, 5th, ...
- **Even positions:** 2nd, 4th, 6th, ...

Since `n` is even, these groups have the same count, `n/2` each. And since the total is odd, their sums cannot be equal — one group is strictly larger.

Now the key observation. **Alice can force herself to receive an entire group.** Suppose she decides to take all the odd-position piles. On her first move she takes pile 1 (an odd position, available at the left end). That leaves piles 2 through `n`, whose two ends are pile 2 and pile `n` — **both even positions**. Whatever Bob takes, he must take an even-position pile, and after he does, both new ends are odd-position piles again. The parity alternates perfectly, and Alice is always offered an odd pile.

The same argument works if she starts by taking pile `n` (an even position, at the right end) to claim the even group instead.

So Alice can choose, in advance, whichever group has the larger sum, and simply execute that plan. She wins every time.

**Should you give the shortcut or the DP in an interview?** Give **both**. Lead with the DP — it shows you can solve the general problem, and it is what transfers to harder variants. Then mention the parity argument as an observation about this problem's extra constraints. Giving only `return true` looks like you memorised an answer; giving only the DP misses a chance to show mathematical insight.

**One important warning.** The `return true` shortcut depends entirely on the even-count and odd-sum guarantees. Remove either one and it collapses. LeetCode 1563 (Stone Game V) and 1140 (Stone Game II) are *not* always wins for the first player, and the parity argument says nothing about them. Know the scope of your shortcuts.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(n^2)** for the DP — one constant-time computation per interval; **O(1)** for the mathematical shortcut, which needs no computation at all.
- **Space Complexity: O(n^2)** for the full table, reducible to **O(n)** with a rolling array, or **O(1)** for the shortcut.

### 4. THE CODE

**C++ — Bottom-Up Tabulation (the general solution)**

```cpp
class Solution {
public:
    bool stoneGame(vector<int>& piles) {
        int n = piles.size();

        // dp[i][j] = best (mover - opponent) stone margin on piles[i..j].
        vector<vector<int>> dp(n, vector<int>(n, 0));

        for (int i = 0; i < n; ++i) dp[i][i] = piles[i];   // base case

        for (int length = 2; length <= n; ++length) {
            for (int i = 0; i + length - 1 < n; ++i) {
                int j = i + length - 1;

                dp[i][j] = max(piles[i] - dp[i + 1][j],    // take the left pile
                               piles[j] - dp[i][j - 1]);   // take the right pile
            }
        }

        // The total is odd, so a tie is impossible: strict comparison.
        return dp[0][n - 1] > 0;
    }
};
```

**C++ — Space-Optimized to O(n)**

```cpp
class Solution {
public:
    bool stoneGame(vector<int>& piles) {
        int n = piles.size();
        vector<int> dp(piles.begin(), piles.end());

        for (int i = n - 2; i >= 0; --i) {
            for (int j = i + 1; j < n; ++j) {
                // dp[j] is dp[i+1][j]; dp[j-1] is dp[i][j-1].
                dp[j] = max(piles[i] - dp[j], piles[j] - dp[j - 1]);
            }
        }

        return dp[n - 1] > 0;
    }
};
```

**C++ — The O(1) mathematical answer**

```cpp
class Solution {
public:
    bool stoneGame(vector<int>& piles) {
        // With an even pile count and an odd total, Alice can commit in
        // advance to taking every odd-indexed pile or every even-indexed pile.
        // Her first move fixes the parity, and it holds for the whole game.
        // One of those two groups must be larger, so she always wins.
        return true;
    }
};
```

**Python — Bottom-Up Tabulation**

```python
class Solution:
    def stoneGame(self, piles: List[int]) -> bool:
        n = len(piles)

        # dp[i][j] = best (mover - opponent) margin on piles[i..j]
        dp = [[0] * n for _ in range(n)]

        for i in range(n):
            dp[i][i] = piles[i]              # base case: take the lone pile

        for length in range(2, n + 1):
            for i in range(n - length + 1):
                j = i + length - 1
                dp[i][j] = max(piles[i] - dp[i + 1][j],
                               piles[j] - dp[i][j - 1])

        return dp[0][n - 1] > 0
```

**Python — Space-Optimized and the O(1) answer**

```python
class Solution:
    def stoneGame(self, piles: List[int]) -> bool:
        dp = piles[:]                        # length-1 intervals

        for i in range(len(piles) - 2, -1, -1):
            for j in range(i + 1, len(piles)):
                dp[j] = max(piles[i] - dp[j], piles[j] - dp[j - 1])

        return dp[-1] > 0


class SolutionMath:
    def stoneGame(self, piles: List[int]) -> bool:
        # Even count + odd total => Alice can claim all odd-indexed piles
        # or all even-indexed piles, and one of those groups is bigger.
        return True
```

**Python — Top-Down Memoization**

```python
from functools import lru_cache

class Solution:
    def stoneGame(self, piles: List[int]) -> bool:

        @lru_cache(maxsize=None)
        def margin(i: int, j: int) -> int:
            if i == j:
                return piles[i]

            return max(piles[i] - margin(i + 1, j),
                       piles[j] - margin(i, j - 1))

        return margin(0, len(piles) - 1) > 0
```

### 5. CODE WALKTHROUGH

1. **The DP code is character-for-character the same as Predict the Winner,** except for the final comparison. That identity is the point: you are not learning 24 problems, you are learning a much smaller number of templates. When two problems collapse into one, say so.
2. **`dp[i][i] = piles[i]` seeds the diagonal,** representing a single remaining pile taken by whoever moves.
3. **Length-increasing loop order** keeps `dp[i+1][j]` and `dp[i][j-1]` ready before they are read — the standard interval-DP ordering.
4. **The two-term `max` with subtraction is the minimax,** exactly as in 9.1. Taking a pile gains its stones and hands the opponent a position worth `dp[next]` to *them*.
5. **`> 0` rather than `>= 0`.** The odd total guarantees no tie, so a strict comparison is correct here, whereas Predict the Winner needed `>=` because ties count as a win for player 1. Small difference, real consequence — read the problem statement's tie rule every time.
6. **The O(n) version uses the same descending-`i` discipline** described in 9.1: at the moment of the write, `dp[j]` is still the previous `i` row and `dp[j-1]` is the current one.
7. **Present the math as an addition, not a replacement.** In an interview, write the DP, confirm it is correct, then say: *"given the even count and odd sum, there is also a constant-time argument — Alice can pre-commit to the odd-indexed or even-indexed group, and one of them must be larger."* Showing both the general machinery and the specific insight is what a strong answer looks like.

---

# Category 10 — Probability / Expectation DP

> **Core Definition:** Tracking mathematical probabilities or expected values across state paths recursively.

Everything you already know about DP applies. The only change is what lives in the table: instead of counts or maxima, each cell holds a **probability** (a number between 0 and 1) or an **expected value**.

**The two combining rules you need:**

| Situation | Rule |
|---|---|
| A random choice among `m` equally likely options | `dp[state] = (1/m) · Σ dp[next state]` — the **average** of the outcomes |
| Independent sequential events | multiply the probabilities |

Note the contrast with optimisation DP. There you wrote `max` or `min` because *you* chose the move. Here nobody chooses — the process is random — so you take a weighted **average** over all outcomes. Whenever a problem says *uniformly at random*, that is your signal to replace `max` with "divide by the number of options."

### Forward versus backward formulation

Both work; pick whichever makes the base case simpler.

- **Forward:** `dp[t][state]` = the probability of *being in* `state` after `t` steps. Start with `dp[0][start] = 1` and push probability mass outward. The answer is a sum over final states.
- **Backward:** `dp[t][state]` = the probability of *succeeding* given `t` steps remain and we are at `state`. Start with `dp[0][state] = 1` (or 0) and pull. The answer is a single cell, `dp[k][start]`.

Backward is often tidier because the answer is one lookup instead of a sum. We use backward for Problem 10.1 and backward for 10.2 as well.

### Two practical warnings

1. **Floating-point accumulation.** Summing thousands of small doubles loses precision. Where a sliding window lets you avoid repeated re-summation, use it — it is both faster and more accurate.
2. **Probabilities must stay in `[0, 1]`.** If a value drifts outside that range, you have a bug: usually a missing division by the branching factor, or double counting.

**Problems in this category:** LeetCode 688, 837.

---

## Problem 10.1 — LeetCode 688: Knight Probability in Chessboard

### 1. THE BLUEPRINT (Input/Output Expectation)

A knight stands on an `n x n` chessboard at `(row, column)`. It makes **exactly `k` moves**. Each move is chosen **uniformly at random** among the knight's 8 possible L-shaped moves — even if that move would leave the board. Once the knight steps off the board, it is gone and stops moving.

Return the probability that the knight is still on the board after all `k` moves.

- **Input:** integers `n` (board size), `k` (number of moves), `row`, `column` (starting square).
- **Output:** a floating-point number — the probability of remaining on the board.

**Tiny concrete example:** `n = 3`, `k = 2`, `row = 0`, `column = 0`.

From the corner `(0,0)` on a 3x3 board, only 2 of the 8 moves land on the board: `(1,2)` and `(2,1)`. So after one move the survival probability is `2/8 = 1/4`.

From either of those squares, again only 2 of the 8 moves stay on the board. So the second move survives with probability `1/4` as well.

Total: `1/4 × 1/4 = 1/16 = 0.0625`. Answer: **0.0625**.

Special case: `k = 0` means no moves, so the knight is trivially still on the board → probability 1.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why brute force fails.** Simulating every move sequence gives `8^k` paths. With `k = 100` that is astronomically large. But many paths reach the same square after the same number of moves, and from there the future is identical. Overlapping subproblems again — the structure is no different from Climbing Stairs, only the arithmetic has changed.

**Finding the state.** The knight's future depends on exactly two things: where it is, and how many moves it has left. Nothing about how it got there matters.

> **State definition (backward):** `dp[s][r][c]` = the probability that the knight, starting at square `(r, c)` with `s` moves remaining, is still on the board when all `s` moves are done.

**The transition.** From `(r, c)`, the knight picks one of 8 moves, each with probability `1/8`. A move that lands off the board contributes 0 (the knight is lost). A move landing on `(nr, nc)` contributes the survival probability from there with one fewer move remaining.

```
dp[s][r][c] = (1/8) · Σ over the 8 moves:
                  dp[s-1][nr][nc]   if (nr, nc) is on the board
                  0                  otherwise
```

Note how off-board moves are handled: we still divide by 8, not by "the number of legal moves." The knight genuinely picks among all 8; some of those picks kill it. Dividing by the number of legal moves is a tempting and very common mistake that would wrongly assume the knight avoids bad moves. Re-read the problem statement: the choice is uniform over all 8.

**The base case.** `dp[0][r][c] = 1` for every on-board square. With zero moves remaining, the knight is wherever it is, and it is on the board, so survival is certain.

**The answer** is `dp[k][row][column]`.

**The space insight.** Layer `s` reads only from layer `s-1`. Two `n x n` grids are enough; we never need all `k+1` layers. That drops memory from O(k·n²) to O(n²).

**A useful sanity check.** Every value in the table must lie in `[0, 1]`, and the values must be non-increasing as `s` grows (more moves means more chances to fall off). If you ever see a value above 1, you forgot a division.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(k · n² · 8)** — for each of the `k` layers we visit all `n²` squares and examine 8 moves each, which simplifies to O(k · n²).
- **Space Complexity: O(n²)** — only the current and previous layers are kept, since each layer depends solely on the one before it.

### 4. THE CODE

**C++ — Bottom-Up Tabulation with two rolling layers**

```cpp
class Solution {
public:
    double knightProbability(int n, int k, int row, int column) {
        // The knight's eight L-shaped moves.
        const int dr[8] = {-2, -2, -1, -1, 1, 1, 2, 2};
        const int dc[8] = {-1,  1, -2,  2, -2, 2, -1, 1};

        // prev[r][c] = probability of surviving the remaining moves from (r,c).
        // Base case: with 0 moves left, an on-board knight survives for sure.
        vector<vector<double>> prev(n, vector<double>(n, 1.0));
        vector<vector<double>> curr(n, vector<double>(n, 0.0));

        for (int step = 1; step <= k; ++step) {
            for (int r = 0; r < n; ++r) {
                for (int c = 0; c < n; ++c) {
                    double total = 0.0;

                    for (int m = 0; m < 8; ++m) {
                        int nr = r + dr[m];
                        int nc = c + dc[m];

                        // Off-board moves contribute 0 - the knight is lost.
                        if (nr >= 0 && nr < n && nc >= 0 && nc < n) {
                            total += prev[nr][nc];
                        }
                    }

                    // Divide by 8, NOT by the number of legal moves:
                    // the knight picks uniformly among all eight.
                    curr[r][c] = total / 8.0;
                }
            }

            swap(prev, curr);     // the layer we just built becomes the previous
        }

        return prev[row][column];
    }
};
```

**C++ — Top-Down Memoization**

```cpp
class Solution {
public:
    double knightProbability(int n, int k, int row, int column) {
        N = n;
        // memo[movesLeft][r][c]; -1 marks "not yet computed".
        memo.assign(k + 1, vector<vector<double>>(n, vector<double>(n, -1.0)));
        return solve(k, row, column);
    }

private:
    int N;
    vector<vector<vector<double>>> memo;
    const int dr[8] = {-2, -2, -1, -1, 1, 1, 2, 2};
    const int dc[8] = {-1,  1, -2,  2, -2, 2, -1, 1};

    double solve(int movesLeft, int r, int c) {
        // Stepped off the board: the knight is lost.
        if (r < 0 || r >= N || c < 0 || c >= N) return 0.0;

        // No moves left and still on the board: survived.
        if (movesLeft == 0) return 1.0;

        if (memo[movesLeft][r][c] >= 0.0) return memo[movesLeft][r][c];

        double total = 0.0;
        for (int m = 0; m < 8; ++m) {
            total += solve(movesLeft - 1, r + dr[m], c + dc[m]);
        }

        return memo[movesLeft][r][c] = total / 8.0;
    }
};
```

**Python — Bottom-Up Tabulation with two rolling layers**

```python
class Solution:
    def knightProbability(self, n: int, k: int, row: int, column: int) -> float:
        MOVES = [(-2, -1), (-2, 1), (-1, -2), (-1, 2),
                 (1, -2), (1, 2), (2, -1), (2, 1)]

        # prev[r][c] = survival probability from (r,c) with the current
        # number of moves remaining. Base case: 0 moves left => certain survival.
        prev = [[1.0] * n for _ in range(n)]

        for _ in range(k):
            curr = [[0.0] * n for _ in range(n)]

            for r in range(n):
                for c in range(n):
                    total = 0.0
                    for dr, dc in MOVES:
                        nr, nc = r + dr, c + dc
                        if 0 <= nr < n and 0 <= nc < n:
                            total += prev[nr][nc]
                        # off-board contributes 0.0

                    curr[r][c] = total / 8.0      # uniform over ALL eight moves

            prev = curr

        return prev[row][column]
```

**Python — Top-Down Memoization**

```python
from functools import lru_cache

class Solution:
    def knightProbability(self, n: int, k: int, row: int, column: int) -> float:
        MOVES = [(-2, -1), (-2, 1), (-1, -2), (-1, 2),
                 (1, -2), (1, 2), (2, -1), (2, 1)]

        @lru_cache(maxsize=None)
        def survive(moves_left: int, r: int, c: int) -> float:
            if not (0 <= r < n and 0 <= c < n):
                return 0.0                 # fell off the board
            if moves_left == 0:
                return 1.0                 # finished on the board

            # Average over the eight equally likely moves.
            return sum(survive(moves_left - 1, r + dr, c + dc)
                       for dr, dc in MOVES) / 8.0

        return survive(k, row, column)
```

### 5. CODE WALKTHROUGH

1. **Store the eight moves in a table.** Two parallel arrays (C++) or a list of pairs (Python) turns eight near-identical code paths into one loop. Any time you have fixed directional offsets, do this — it removes the copy-paste errors that plague hand-written move lists.
2. **Initialise `prev` to all 1.0.** This encodes the base case *"zero moves remaining, still on the board, therefore survived."* Starting from all-ones and letting probability leak away at the edges is the cleanest way to express this problem.
3. **Rebuild `curr` each layer from `prev`.** The `step` loop runs `k` times, each time advancing the "moves remaining" count by one. Because layer `s` reads only layer `s-1`, two grids suffice and the swap at the end of each iteration is O(1) in C++.
4. **Off-board moves silently contribute nothing.** The `if` that checks bounds simply skips the addition. That skipped term *is* the probability of death — it is 1/8 of the mass vanishing, which is exactly correct.
5. **Divide by 8.0 unconditionally.** This is the line to get right. The knight chooses among all eight moves regardless of whether they are legal, so the denominator is always 8. Dividing by the count of legal moves would model a cautious knight that avoids the edge, which is a different (and easier) problem.
6. **Return `prev[row][column]`** after `k` layers — the survival probability from the starting square with all `k` moves ahead.
7. **Use `double`, not `float`, in C++.** With `k` up to 100, probabilities become extremely small, and single precision loses them. The memo sentinel is `-1.0` rather than 0, because 0 is a perfectly valid computed probability and would be mistaken for "not yet computed."
8. **Compare this to the forward formulation.** You could instead store "probability of being at `(r,c)` after `t` moves," seed `dp[0][row][column] = 1`, push mass outward, and sum the final grid. That is equally valid and some people find it more intuitive. The backward version is shown here because the answer is a single cell rather than a sum, which means one less place to make a mistake.

---

## Problem 10.2 — LeetCode 837: New 21 Game

### 1. THE BLUEPRINT (Input/Output Expectation)

Alice starts with **0 points**. While her score is **strictly less than `k`**, she draws a number chosen **uniformly at random** from the integers `1` through `maxPts`, and adds it to her score. She stops as soon as her score reaches `k` or more.

Return the probability that her final score is **`n` or less**.

- **Input:** integers `n` (the target ceiling), `k` (the stopping threshold), `maxPts` (the largest single draw).
- **Output:** a floating-point probability.

**Tiny concrete example:** `n = 6`, `k = 1`, `maxPts = 10`.

Since `k = 1`, Alice draws exactly once (her score of 0 is below 1), getting a uniformly random value from 1 to 10. She then stops, because any value from 1 to 10 is at least 1. She succeeds if her draw is 6 or less — that is 6 of the 10 outcomes.

Answer: **0.6**.

Another: `n = 21`, `k = 17`, `maxPts = 10` → approximately **0.73278**.

### 2. THE THOUGHT PROCESS (Mental Model)

**Why this is harder than it looks.** The number of draws is not fixed, so you cannot organise the DP around "after `t` draws." You must organise it around the **score**, which is the only thing that determines what happens next.

**Finding the state.**

> **State definition:** `dp[x]` = the probability that Alice's **final** score is at most `n`, given that her score is currently `x`.

The answer we want is `dp[0]`.

**The terminal cases (`x >= k`).** Alice has stopped. Her final score is exactly `x`, so:

```
dp[x] = 1  if x <= n
dp[x] = 0  if x > n
```

What is the largest score she can possibly stop at? She only draws when her score is at most `k - 1`, and the biggest draw is `maxPts`, so the maximum final score is `k - 1 + maxPts`. Scores from `k` up to `k + maxPts - 1` are the full set of terminal states.

**The active case (`x < k`).** Alice must draw. She gets `x+1, x+2, ..., x+maxPts`, each with probability `1/maxPts`:

```
dp[x] = (dp[x+1] + dp[x+2] + ... + dp[x+maxPts]) / maxPts
```

**The direction.** `dp[x]` depends on *larger* scores, so we compute **downward**: from `x = k-1` to `x = 0`.

**The naive cost and the fix.** Written directly, each `dp[x]` sums `maxPts` terms, giving O(k · maxPts) — up to 10^8 operations, too slow. But look at what changes between consecutive states:

```
window(x)   = dp[x+1] + dp[x+2] + ... + dp[x+maxPts]
window(x-1) = dp[x]   + dp[x+1] + ... + dp[x+maxPts-1]
```

Going from `window(x)` to `window(x-1)`, we **add** `dp[x]` at the front and **remove** `dp[x+maxPts]` from the back. That is a classic **sliding window**: one add, one subtract, constant time per state. The whole algorithm becomes O(n + k + maxPts).

**The two edge cases, and why they matter.**

1. **`k == 0`.** Alice never draws. Her final score is 0, and `n >= 0` always, so the answer is exactly **1.0**. Without this guard the loop body never runs and the array indexing goes wrong.

2. **`n >= k + maxPts - 1`.** The ceiling is at or above the highest score Alice could possibly end on, so she cannot fail. Return **1.0**. This guard is not merely an optimisation — it also keeps the array size bounded in the cases where `n` is enormous.

**The answer** is `dp[0]`.

**A sanity check worth doing.** The computed probability must land in `[0, 1]`, and raising `n` must never decrease it. If either fails, suspect the window update.

### 3. THE FRAMEWORK (Complexity)

- **Time Complexity: O(k + maxPts)** — each score from the top down to 0 is handled once, and the sliding window makes each step a single addition and subtraction rather than a `maxPts`-term sum.
- **Space Complexity: O(k + maxPts)** — one array covering every reachable score, from 0 up to the maximum possible final score `k + maxPts - 1`.

### 4. THE CODE

**C++ — Bottom-Up with a Sliding Window**

```cpp
class Solution {
public:
    double new21Game(int n, int k, int maxPts) {
        // Edge case 1: Alice never draws, so her score stays 0 (and 0 <= n).
        if (k == 0) return 1.0;

        // Edge case 2: even the highest possible final score is within n,
        // so she cannot lose.
        if (n >= k + maxPts - 1) return 1.0;

        // dp[x] = probability the FINAL score is <= n, given the current score x.
        // Scores range from 0 up to the maximum stopping score k + maxPts - 1.
        vector<double> dp(k + maxPts, 0.0);

        // Terminal states: Alice has stopped, so her score is final.
        for (int x = k; x < k + maxPts; ++x) {
            dp[x] = (x <= n) ? 1.0 : 0.0;
        }

        // window = dp[x+1] + ... + dp[x+maxPts], maintained incrementally.
        // We start at x = k-1, so the window covers dp[k] .. dp[k+maxPts-1].
        double window = 0.0;
        for (int x = k; x < k + maxPts; ++x) window += dp[x];

        // Walk downward: dp[x] depends only on larger scores.
        for (int x = k - 1; x >= 0; --x) {
            dp[x] = window / maxPts;        // average over the maxPts draws

            // Slide the window one step left:
            //   gain dp[x] at the front, drop dp[x + maxPts] at the back.
            window += dp[x] - dp[x + maxPts];
        }

        return dp[0];
    }
};
```

**C++ — Naive O(k · maxPts) version (correct but too slow; shown for clarity)**

```cpp
class Solution {
public:
    double new21Game(int n, int k, int maxPts) {
        if (k == 0 || n >= k + maxPts - 1) return 1.0;

        vector<double> dp(k + maxPts, 0.0);
        for (int x = k; x < k + maxPts; ++x) dp[x] = (x <= n) ? 1.0 : 0.0;

        for (int x = k - 1; x >= 0; --x) {
            double total = 0.0;
            // Re-summing maxPts terms every time is what makes this slow.
            for (int d = 1; d <= maxPts; ++d) total += dp[x + d];
            dp[x] = total / maxPts;
        }

        return dp[0];
    }
};
```

**Python — Bottom-Up with a Sliding Window**

```python
class Solution:
    def new21Game(self, n: int, k: int, maxPts: int) -> float:
        # Alice never draws at all.
        if k == 0:
            return 1.0

        # Even the worst case lands at or below n.
        if n >= k + maxPts - 1:
            return 1.0

        # dp[x] = probability the final score is <= n, starting from score x.
        dp = [0.0] * (k + maxPts)

        # Terminal scores: she has stopped drawing.
        for x in range(k, k + maxPts):
            dp[x] = 1.0 if x <= n else 0.0

        # The window holds dp[x+1] + ... + dp[x+maxPts] for the current x.
        window = sum(dp[k:k + maxPts])

        for x in range(k - 1, -1, -1):
            dp[x] = window / maxPts          # average of the maxPts outcomes

            # Slide left: add the value we just computed, drop the far end.
            window += dp[x] - dp[x + maxPts]

        return dp[0]
```

**Python — Top-Down Memoization (clear, but risks deep recursion)**

```python
from functools import lru_cache

class Solution:
    def new21Game(self, n: int, k: int, maxPts: int) -> float:
        if k == 0 or n >= k + maxPts - 1:
            return 1.0

        @lru_cache(maxsize=None)
        def prob(score: int) -> float:
            # Alice has stopped: the score is final.
            if score >= k:
                return 1.0 if score <= n else 0.0

            # She must draw: average over all maxPts equally likely results.
            return sum(prob(score + d) for d in range(1, maxPts + 1)) / maxPts

        return prob(0)
```

### 5. CODE WALKTHROUGH

1. **Handle both edge cases before allocating anything.** `k == 0` means no draws ever happen. `n >= k + maxPts - 1` means failure is impossible. Each is a one-line `return 1.0`, and together they also protect the array sizing and the window arithmetic that follow.
2. **Size the array to `k + maxPts`.** The highest reachable score is `(k-1) + maxPts`, so valid indices run from 0 to `k + maxPts - 1`. Getting this bound right is what lets `dp[x + maxPts]` be read safely inside the loop without a check.
3. **Fill the terminal band first.** Every score from `k` to `k + maxPts - 1` is a stopping point, and its value is a simple comparison with `n`. These fixed 1s and 0s are the base cases that everything else averages over.
4. **Initialise the window over the terminal band.** Before the loop starts, `window` must equal `dp[k] + ... + dp[k+maxPts-1]`, which is exactly the set of values that `dp[k-1]` needs. Setting the window to 0 and expecting the loop to fix it is a common and silent error.
5. **Walk `x` downward.** `dp[x]` depends on larger scores, so descending order is the correct dependency order — the mirror image of the ascending loops used everywhere else in this guide. Always derive the direction from the recurrence rather than from habit.
6. **`dp[x] = window / maxPts` is the averaging rule.** No `max`, no `min` — the draw is random, not chosen, so we take the mean of the outcomes. This is the single line that distinguishes probability DP from optimisation DP.
7. **The window update must come after the assignment.** `window += dp[x] - dp[x + maxPts]` adds the value we just computed and removes the one that has fallen out of range, preparing the window for `x - 1`. Reversing these two lines corrupts the sum for every subsequent state.
8. **Return `dp[0]`** — Alice's starting score.
9. **Note why the memoized version is risky here.** With `k` up to 10,000 the recursion depth can exceed Python's default limit, and each call re-sums `maxPts` terms, making it O(k · maxPts). It is excellent for explaining the recurrence on a whiteboard, but the sliding-window bottom-up version is what you should submit. Knowing *when* top-down is the wrong tool is as valuable as knowing how to write it.

---
---

# Part III — Mastery

---

## Pattern Recognition Cheat Sheet

This is the page to re-read the morning of an interview.

### Match the clue to the category

| What you see in the problem | Category | Typical state |
|---|---|---|
| Walking an array, each step depends on the last one or two | **1D DP** | `dp[i]` |
| A grid, moving right and down | **2D DP** | `dp[i][j]` |
| Two strings being compared | **String DP** | `dp[i][j]` = prefixes of each |
| Input is a `TreeNode`, question asks for a max/min/count | **Tree DP** | post-order DFS return value |
| "Choose items under a capacity / reach an exact sum" | **Knapsack** | `dp[item][capacity]` |
| You pick the *order of operations* on a range | **Interval DP** | `dp[i][j]`, loop by length |
| "Count numbers in `[L, R]` with property P", bounds ~10^18 | **Digit DP** | `(pos, tight, property)` |
| `n <= 20` plus subsets, orderings, or assignments | **Bitmask DP** | `dp[mask]` |
| Two players, alternating turns, both optimal | **Game Theory DP** | `dp[state]` = score difference |
| "uniformly at random", "probability", "expected value" | **Probability DP** | `dp[state]` = probability |

### Match the question type to the operator

| Question asks for | Combine with | Seed the base case as |
|---|---|---|
| Maximum / longest / best | `max` | `0` or `-infinity` |
| Minimum / shortest / cheapest | `min` | `0` or `+infinity` (or a safe sentinel) |
| Number of ways | `+` | `1` (the empty way) |
| Is it possible | `or` | `true` |
| Probability | `sum / branching factor` | `1.0` or `0.0` |

### Constraint-to-complexity table

Reading the constraints tells you which algorithm is expected. This is one of the fastest ways to narrow down an approach.

| `n` up to | Allowed complexity | Likely technique |
|---|---|---|
| 10^18 | O(log n) or O(digits) | Digit DP, matrix exponentiation |
| 10^7 | O(n) | 1D DP, single pass |
| 10^6 | O(n log n) | DP with a sort or binary search |
| 5,000 | O(n²) | 2D DP, string DP |
| 500 | O(n³) | Interval DP |
| 20 | O(2^n · n) | Bitmask DP |
| 12 | O(n!) | Brute-force permutations |

### Space optimization quick rules

| If `dp[i]` depends on... | You can reduce to |
|---|---|
| `dp[i-1]`, `dp[i-2]` | 2 variables → O(1) |
| row `i-1` only | 1 or 2 rows → O(n) |
| row `i-1` and the cell up-left | 1 row + a saved `diagonal` → O(n) |
| all previous rows | nothing — you need the full table |
| and you must reconstruct the answer path | nothing — keep the full table |

---

## Common Bugs and How To Avoid Them

Ten mistakes account for most wrong answers. Check this list whenever you are stuck.

**1. The state is not clearly defined.**
*Symptom:* the recurrence feels arbitrary and you cannot tell whether it is right.
*Fix:* stop coding. Write the sentence `dp[i] means ____` on paper. If you cannot finish it, you do not yet understand the problem. This is the single most common failure and the single most valuable habit.

**2. Off-by-one in the table size.**
*Symptom:* index out of range, or the answer is one step short.
*Fix:* size the array to `n + 1` whenever index `n` is meaningful. If `dp[i]` means "the first `i` items," then the current item is `arr[i-1]`. Write that mapping in a comment.

**3. Missing or wrong base cases.**
*Symptom:* the whole table is zeros, or everything is off by a constant.
*Fix:* match the seed to the question type. Counting → 1. Feasibility → `true`. Minimisation → 0 at the origin and a sentinel elsewhere. Edit Distance → `dp[i][0] = i`, not 0.

**4. Wrong loop direction in a 1D knapsack.**
*Symptom:* counts are too large, or items get reused when they should not be.
*Fix:* 0/1 goes **backwards**; unbounded goes **forwards**. Say it aloud as you type the loop.

**5. Wrong loop order in an interval DP.**
*Symptom:* confidently wrong answers with no crash.
*Fix:* loop by interval **length**, shortest first. Never loop `i` and `j` directly.

**6. Reading a stale or already-overwritten value in a space-optimized DP.**
*Symptom:* works on the 2D version, breaks after compression.
*Fix:* before each write, name exactly which cell each operand currently holds. If the recurrence reaches up-and-left, save the diagonal in a variable first.

**7. Forgetting the modulus, or applying it incorrectly.**
*Symptom:* overflow, or negative output.
*Fix:* take the modulus after every addition in a counting DP. After subtraction in C++, normalise with `((x % MOD) + MOD) % MOD`.

**8. A memo that is not cleared, or is keyed on too little.**
*Symptom:* right on the first test, wrong on the second.
*Fix:* reset the cache whenever the underlying bound or input changes. Make sure every variable the answer depends on is part of the key.

**9. Returning the wrong cell.**
*Symptom:* an answer that is close but not right.
*Fix:* the answer is not always `dp[n]`. Sometimes it is `max(dp)` over the whole array, sometimes a separate global variable (as in Diameter of Binary Tree). Re-read what the question asked for.

**10. Using `max`/`min` where an average belongs.**
*Symptom:* probabilities above 1.
*Fix:* when the process is random rather than chosen, average over the outcomes. Divide by the branching factor, and make sure the denominator counts *all* options, not just the legal ones.

---

## A 6-Week Study Plan

Do not try to absorb this guide in one sitting. Spaced practice beats cramming, and DP in particular rewards slow repetition.

### Week 1 — Foundations
**Goal:** make 1D DP automatic.
- Read Part I twice.
- Solve LC 70, 198, 121 from scratch without looking.
- Then solve: 746 (Min Cost Climbing Stairs), 53 (Maximum Subarray), 213 (House Robber II), 91 (Decode Ways).
- **Daily habit:** before coding, say the state definition out loud.

### Week 2 — Grids and Strings
**Goal:** get comfortable with two-dimensional state.
- Solve LC 62, 64, 221.
- Solve LC 1143, 72, 115.
- Then: 63 (Unique Paths II), 1312 (Minimum Insertions to Make Palindrome), 516 (Longest Palindromic Subsequence), 10 (Regular Expression Matching).
- **Focus:** practice converting each 2D solution to O(n) space. Do it every time, even when not required.

### Week 3 — Knapsack
**Goal:** recognise knapsack when it is disguised.
- Solve LC 416, 322, 494.
- Then: 518 (Coin Change II), 1049 (Last Stone Weight II), 279 (Perfect Squares), 139 (Word Break).
- **Focus:** for every problem, state out loud whether it is 0/1 or unbounded, and which loop direction follows.

### Week 4 — Trees and Intervals
**Goal:** DP on non-linear structures.
- Solve LC 337, 543.
- Solve LC 312, 1039.
- Then: 124 (Binary Tree Maximum Path Sum), 968 (Binary Tree Cameras), 1547 (Minimum Cost to Cut a Stick), 1000 (Minimum Cost to Merge Stones).
- **Focus:** for trees, always ask "what must I return upward versus record globally?" For intervals, always ask "which element is handled *last*?"

### Week 5 — Advanced Patterns
**Goal:** the categories that look intimidating.
- Solve LC 2719, 902 (digit).
- Solve LC 526, 473 (bitmask).
- Solve LC 486, 877 (game theory).
- Then: 600 (Non-negative Integers without Consecutive Ones), 2376 (Count Special Integers), 1125 (Smallest Sufficient Team), 1406 (Stone Game III).
- **Focus:** write the digit DP template from memory until it comes out clean.

### Week 6 — Probability and Consolidation
**Goal:** finish the set and prove to yourself that you have it.
- Solve LC 688, 837.
- Then: 808 (Soup Servings), 1230 (Toss Strange Coins).
- **Re-solve ten problems from Weeks 1 to 4 without notes.** Time yourself.
- Any problem you cannot solve in 30 minutes goes back on the list for next week.

### How to practise, not just grind

1. **Always attempt before reading.** Give yourself a real 20 to 30 minutes. Struggle is where the learning happens; reading a solution you did not fight for teaches almost nothing.
2. **Write the five framework answers before any code.** State, choices, transition, base cases, order. Every time. No exceptions.
3. **Explain your solution out loud as if teaching someone.** If the explanation stalls, that is exactly where your understanding is thin.
4. **Re-solve, do not re-read.** A problem you solved two weeks ago and can still solve today is a problem you actually know.
5. **Keep an error log.** One line per bug: what went wrong and why. Re-read it before each session. After a few weeks you will notice the same two or three mistakes repeating — those are your real weaknesses, and they are now fixable.

---

## Final Thoughts

Dynamic Programming feels impossible right up until the moment it feels obvious. There is no gradual slope; most people describe it as a switch flipping. The switch flips after you have derived — not read, *derived* — maybe thirty recurrences yourself.

So do not measure progress by how many solutions you have read. Measure it by how often you can look at a new problem and say: *"the state is this, the choice at each step is that, and the base case is here."* When that sentence comes easily, you are done, and no interviewer will be able to surprise you.

Three things to carry with you:

1. **The state definition is the whole job.** Everything else is mechanical.
2. **Patterns beat problems.** Twenty-four problems collapse into about eight templates. Learn the templates.
3. **Derive, do not memorise.** Memorised solutions fail the moment a problem is reworded. Derived ones never do.

Go write some recurrences. You will be fine.

---
