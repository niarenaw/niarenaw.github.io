---
title: "Buzen's Algorithm"
description: "50 jobs across 10 servers means 12 billion states. Buzen's 1973 algorithm computes exact steady-state probabilities in 500 operations."
pubDate: 2025-12-24
---

Queueing theory studies systems where jobs arrive, wait, get served, and depart. A *queueing network* connects multiple queues - jobs flow from one to the next, like packets through routers or requests through microservices. Analyzing these networks means computing steady-state probabilities: in the long run, what fraction of time does the system spend in each configuration?

Fifty jobs circulating among ten servers can occupy over 12 billion distinct configurations. Yet Buzen's 1973 algorithm computes the exact probability of every state with only a few hundred operations - a reduction from $\sim 10^{10}$ to $500$. This post derives the algorithm from first principles and implements it.

## Queueing Basics

A single queue has jobs arriving at rate $\lambda$ (lambda) and a server processing them at rate $\mu$ (mu). The ratio $\rho = \lambda/\mu$ is the **utilization** - the fraction of time the server is busy. For stability, $\rho < 1$; otherwise jobs accumulate faster than they're served.

The simplest model is the **M/M/1 queue**: Poisson arrivals (memoryless, hence "M"), exponential service times (also "M"), and one server. In steady state, the probability of having exactly $k$ jobs in the system is:

$$P(k) = (1-\rho)\rho^k$$

A geometric distribution - most likely to be empty, exponentially less likely to have more jobs.

### Queueing Networks

Real systems chain queues together. A job might visit a CPU queue, then a disk queue, then return to the CPU. The **routing matrix** $R = [r_{ij}]$ specifies transition probabilities: $r_{ij}$ is the probability a job leaving queue $i$ goes to queue $j$.

In **open networks**, jobs arrive from outside and eventually depart. Jackson (1963) proved that under Poisson arrivals and exponential service, the steady-state distribution has **product form** - the joint probability factors into independent terms for each queue.

In **closed networks**, a fixed population of $N$ jobs circulates forever. No arrivals, no departures - just jobs moving between queues. Gordon and Newell (1967) extended Jackson's result: closed networks also have product-form solutions, but with a normalization constant that's expensive to compute.

### Notation

| Symbol | Meaning |
|--------|---------|
| $N$ | Total number of jobs in the closed network |
| $M$ | Number of queues (service centers) |
| $\mu_i$ | Service rate at queue $i$ |
| $e_i$ | Visit ratio - relative frequency of visits to queue $i$ |
| $X_i = e_i/\mu_i$ | Relative load at queue $i$ (service demand per visit) |
| $k_i$ | Number of jobs at queue $i$ in a given state |
| $G(N)$ | Normalization constant for $N$ jobs |

The visit ratios $e_i$ come from solving the traffic equations $e_j = \sum_i e_i r_{ij}$. They represent how often each queue is visited relative to some reference queue.

## The Normalization Problem

For a closed network with $N$ jobs and $M$ queues, the steady-state probability of configuration $(k_1, \ldots, k_M)$ where $\sum_i k_i = N$ is:

$$\pi(k_1, \ldots, k_M) = \frac{1}{G(N)} \prod_{i=1}^M X_i^{k_i}$$

The product $\prod X_i^{k_i}$ captures the relative likelihood of each configuration - states that concentrate jobs at queues with large $X_i$ contribute more to the un-normalized sum. The normalization constant $G(N)$ ensures probabilities sum to 1:

$$G(N) = \sum_{\mathbf{k} \in S(N,M)} \prod_{i=1}^M X_i^{k_i}$$

where $S(N,M)$ is the **state space** - all ways to distribute $N$ jobs among $M$ queues:

$$S(N,M) = \{(k_1, \ldots, k_M) : k_i \geq 0, \sum_{i=1}^M k_i = N\}$$

For $N=50$ and $M=10$, the state space contains $\binom{59}{9} \approx 1.25 \times 10^{10}$ configurations. Computing $G(N)$ by summing over all of them is infeasible.

## The Naive Approach

Direct enumeration iterates over every state in $S(N,M)$, computes $\prod X_i^{k_i}$, and accumulates the sum. The state space size is the number of ways to place $N$ indistinguishable jobs into $M$ queues - a stars-and-bars combinatorial problem:

$$|S(N,M)| = \binom{N + M - 1}{M - 1}$$

| Network | States | Naive Operations |
|---------|--------|-----------------|
| $N=10$, $M=5$ | 1001 | $\sim 10^3$ |
| $N=50$, $M=5$ | 316251 | $\sim 3 \times 10^5$ |
| $N=20$, $M=10$ | $\sim 10^7$ | $\sim 10^7$ |
| $N=50$, $M=10$ | $\sim 1.26 \times 10^{10}$ | $\sim 10^{10}$ |

A network of 10 queues and 50 jobs requires iterating over tens of billions of states. Even modern hardware chokes on this.

The structure of the sum suggests redundancy. Each term $\prod_i X_i^{k_i}$ shares factors with many others. The sum is essentially an $M$-fold convolution - a hint that dynamic programming might help.

## The Insight

Buzen recognized that $G(N)$ can be computed recursively by adding one queue at a time.

Define $g(n, m)$ as the normalization constant for $n$ jobs distributed among the first $m$ queues:

$$g(n, m) = \sum_{\mathbf{k}: \sum_{i=1}^m k_i = n} \prod_{i=1}^m X_i^{k_i}$$

By definition, $g(N, M) = G(N)$.

Now partition the states based on queue $m$'s occupancy. Either queue $m$ has zero jobs, or it has at least one:

- **Zero at queue $m$**: Queue $m$ contributes $X_m^0 = 1$. The remaining $n$ jobs are distributed among queues $1$ through $m-1$. This contributes $g(n, m-1)$.

- **At least one at queue $m$**: Factor out $X_m$ for one job at queue $m$. The remaining $n-1$ jobs (including any additional ones at queue $m$) distribute among all $m$ queues. This contributes $X_m \cdot g(n-1, m)$.

The recurrence:

$$g(n, m) = g(n, m-1) + X_m \cdot g(n-1, m)$$

Base cases:
- $g(0, m) = 1$ for all $m$ (one way to place zero jobs - all queues empty)
- $g(n, 0) = 0$ for $n > 0$ (no queues means no way to place jobs)

This recurrence computes $g(N, M)$ using only $O(NM)$ operations instead of enumerating all $\binom{N+M-1}{M-1}$ states - a count that grows as $O(N^{M-1})$ for fixed $M$.

## Why Dynamic Programming is Optimal

The recurrence has optimal substructure: $g(n,m)$ depends only on $g(n-1,m)$ and $g(n,m-1)$. Without memoization, we'd recompute values exponentially.

An alternative formulation suggests itself. The sum $G(N)$ is the $N$-th coefficient of the generating function:

$$\prod_{i=1}^M \frac{1}{1-X_i z} = \sum_{n=0}^\infty G(n) z^n$$

Using FFT-based polynomial multiplication, we could compute this product in $O(MN \log N)$ time. But Buzen's recurrence achieves $O(MN)$ by exploiting the special structure - each factor is a geometric series, so the convolution simplifies to a single pass.

The $O(MN)$ bound is tight for this recurrence. We must fill an $N \times M$ table, each entry requiring $O(1)$ work.

## Implementation

The algorithm maintains a single array $g[0 \ldots N]$ representing the current "column" $g(\cdot, m)$. Each iteration incorporates one more queue:

```python
def buzen(X: list[float], N: int) -> list[float]:
    """Compute normalization constants G(0), G(1), ..., G(N)"""
    g = [0.0] * (N + 1)
    g[0] = 1.0

    for x in X:
        for n in range(1, N + 1):
            g[n] += x * g[n - 1]

    return g
```

That's it. After processing all $M$ queues, `g[n]` contains $G(n)$ for each $n$ from $0$ to $N$.

### Walkthrough

For $M=2$ queues with $X_1=2$, $X_2=3$, computing $G(3)$:

**Initialize:** $g = [1, 0, 0, 0]$

**Process queue 1** ($X_1 = 2$):
- $g[1] = 0 + 2 \cdot 1 = 2$
- $g[2] = 0 + 2 \cdot 2 = 4$
- $g[3] = 0 + 2 \cdot 4 = 8$

After: $g = [1, 2, 4, 8]$ which matches $[2^0, 2^1, 2^2, 2^3]$.

**Process queue 2** ($X_2 = 3$):
- $g[1] = 2 + 3 \cdot 1 = 5$
- $g[2] = 4 + 3 \cdot 5 = 19$
- $g[3] = 8 + 3 \cdot 19 = 65$

Final: $g = [1, 5, 19, 65]$

Verify directly:
$$G(3) = \sum_{k_1 + k_2 = 3} 2^{k_1} 3^{k_2} = 2^0 3^3 + 2^1 3^2 + 2^2 3^1 + 2^3 3^0 = 27 + 18 + 12 + 8 = 65$$

## Computing Performance Metrics

The real payoff comes from using $G(n)$ values to extract performance metrics without revisiting the state space.

### Marginal Queue Length Distribution

The probability that queue $i$ has at least $k$ jobs:

$$P(n_i \geq k) = X_i^k \cdot \frac{G(N-k)}{G(N)}$$

Intuition: fix $k$ jobs at queue $i$ (contributing $X_i^k$), distribute the remaining $N-k$ arbitrarily (summing to $G(N-k)$), normalize by $G(N)$.

```python
def prob_at_least(g: list[float], X: list[float], i: int, k: int) -> float:
    """P(queue i has >= k jobs)"""
    N = len(g) - 1
    if k > N:
        return 0.0
    return (X[i] ** k) * g[N - k] / g[N]
```

### Expected Queue Length

The expected number of jobs at queue $i$:

$$E[n_i] = \sum_{k=1}^{N} P(n_i \geq k) = \sum_{k=1}^{N} X_i^k \cdot \frac{G(N-k)}{G(N)}$$

```python
def expected_queue_length(g: list[float], X: list[float], i: int) -> float:
    """E[jobs at queue i]"""
    N = len(g) - 1
    total = 0.0
    x_power = X[i]
    for k in range(1, N + 1):
        total += x_power * g[N - k]
        x_power *= X[i]
    return total / g[N]
```

### Throughput

The throughput (completion rate) at queue $i$ follows from the arrival theorem:

$$\lambda_i = e_i \cdot \frac{G(N-1)}{G(N)}$$

where $e_i$ is the visit ratio. The ratio $G(N-1)/G(N)$ acts as a system-wide throughput factor.

```python
def throughput(g: list[float], e: list[float]) -> list[float]:
    """Throughput at each queue"""
    N = len(g) - 1
    ratio = g[N - 1] / g[N]
    return [e_i * ratio for e_i in e]
```

All these metrics come essentially free once we have the $G$ array - no state enumeration required.

## Numerical Considerations

For large $N$ or extreme $X_i$ values, $G(N)$ can overflow or underflow floating-point. Two approaches:

**Scaling**: Periodically divide all entries by $g[N]$ during computation. This keeps values bounded but requires care when computing ratios.

**Log-domain**: Work with $\log G(n)$ instead. The recurrence becomes:

$$\log g(n,m) = \log\left(e^{\log g(n,m-1)} + e^{\log X_m + \log g(n-1,m)}\right)$$

Use the log-sum-exp trick for numerical stability. This adds overhead but handles extreme cases.

For most practical networks (moderate $N$, reasonable utilizations), standard double precision suffices.

## Complexity Comparison

| Approach | Time | Space |
|----------|------|-------|
| Naive enumeration | $|S(N,M)| = O(N^{M-1})$ | $O(1)$ |
| Buzen's algorithm | $O(NM)$ | $O(N)$ |
| FFT convolution | $O(NM \log N)$ | $O(N)$ |

For $N=50$, $M=10$: naive requires $\sim 10^{10}$ operations, Buzen requires $500$.

The algorithm also produces all intermediate values $G(1), \ldots, G(N-1)$ as a byproduct - exactly what we need for performance metrics.

## Historical Note

Buzen developed this algorithm during his PhD at Harvard (1971) while studying computer system performance. At the time, Gordon-Newell networks were known to have elegant theoretical properties, but practitioners couldn't compute anything with them - the state spaces were too large.

Buzen's convolution algorithm made closed queueing network analysis practical. It became a foundation of computer performance modeling throughout the 1970s and 1980s, enabling capacity planning for mainframes and early distributed systems.

The same year (1973), Buzen founded BGS Systems, one of the first companies focused on computer performance analysis software - built on these algorithms.

## Conclusion

Buzen's algorithm reduces a sum over billions of states to a few hundred multiply-adds. The trick is recognizing that the normalization constant $G(N)$ decomposes into sequential convolutions - add one queue at a time, and the combinatorial explosion collapses into a simple recurrence.

The algorithm exemplifies a broader pattern: when a massive sum has product structure, dynamic programming often finds a shortcut. Here, the product-form solution of Gordon-Newell networks provides exactly that structure. What looks like an intractable enumeration problem becomes $O(NM)$ arithmetic.

For practitioners in the 1970s, this was transformative. Closed queueing networks had elegant theory but were computationally useless until Buzen's algorithm made them practical. The same ideas - convolution, dynamic programming, exploiting algebraic structure - appear throughout performance modeling, from Mean Value Analysis to modern cloud capacity planning.

## Further Reading

- [Wikipedia: Buzen's algorithm](https://en.wikipedia.org/wiki/Buzen%27s_algorithm) - derivation and pseudocode
- [Wikipedia: Jackson network](https://en.wikipedia.org/wiki/Jackson_network) - open network theory
- [Wikipedia: Gordon-Newell theorem](https://en.wikipedia.org/wiki/Gordon%E2%80%93Newell_theorem) - closed network product form

## References

- Buzen, J. P. (1973). *Computational algorithms for closed queueing networks with exponential servers*. Communications of the ACM, 16(9), 527-531.
- Gordon, W. J., & Newell, G. F. (1967). *Closed queuing systems with exponential servers*. Operations Research, 15(2), 254-265.
- Jackson, J. R. (1963). *Jobshop-like queueing systems*. Management Science, 10(1), 131-142.
- Reiser, M., & Lavenberg, S. (1980). *Mean value analysis of closed multichain queueing networks*. Journal of the ACM, 27(2), 313-322.
