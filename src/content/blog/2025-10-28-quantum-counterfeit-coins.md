---
title: "Quantum Counterfeit Coins"
description: "A quantum algorithm for the counterfeit coin problem using Bernstein-Vazirani, achieving O(1) query complexity vs classical O(log N)."
pubDate: 2025-10-28
---

Quantum algorithms that provably outperform classical ones are rare. Most require specific problem structures or unproven complexity assumptions. The counterfeit coin problem is different: simple to state, and the quantum speedup is unconditional - $O(1)$ queries versus $O(\log N)$ classically.

This post covers the problem, classical solution, and a detailed walkthrough of the quantum algorithm with full Dirac notation and my Cirq implementation.

## The Counterfeit Coin Problem

The problem dates back to 1945 when E. D. Schell posed it in the *American Mathematical Monthly*:

> You have $N$ coins that look identical. One of them is counterfeit and weighs slightly less than the others. You have a balance scale that tells you whether the left side is heavier, lighter, or equal to the right side. What's the minimum number of weighings needed to identify the counterfeit coin?

For concreteness, imagine you have 16 coins. One is fake. How do you find it?

## The Classical Solution

The classical approach is binary search. At each step, you divide the coins into groups and weigh them against each other:

1. Split the 16 coins into two groups of 8
2. Weigh them - the lighter side contains the counterfeit
3. Split those 8 coins into two groups of 4
4. Continue halving until you find the fake

This requires $\lceil \log_2 N \rceil$ weighings. For 16 coins, that's 4 weighings. For 1 million coins? Just 20 weighings.

Binary search is remarkably efficient, and it's provably optimal for classical algorithms. Any deterministic classical algorithm requires $O(\log N)$ queries to the balance scale.

**But what if the balance scale is quantum?**

## Enter Quantum Computing

In 2010, Iwama, Nishimura, Raymond, and Teruyama showed something remarkable: a quantum algorithm can find the counterfeit coin in $O(1)$ weighings - **independent of the number of coins**.

For 16 coins or 16 billion coins, you need just one quantum query to the balance. Classical binary search requires $O(\log N)$ queries; the quantum algorithm needs $O(1)$.

The key insight is that the counterfeit coin problem can be reformulated as an instance of the **Bernstein-Vazirani problem**, which quantum computers solve in a single query.

## The Bernstein-Vazirani Algorithm

Before diving into counterfeit coins, let's understand Bernstein-Vazirani.

### The Problem

Given a function $f: \{0,1\}^n \rightarrow \{0,1\}$ defined as:

$$
f(x) = x \cdot s = x_1 s_1 \oplus x_2 s_2 \oplus \cdots \oplus x_n s_n \pmod{2}
$$

where $s \in \{0,1\}^n$ is an unknown secret string, find $s$ using as few queries to $f$ as possible.

**Classical complexity:** We need $n$ queries (one for each bit of $s$).

**Quantum complexity:** Just 1 query.

### Why Does This Matter for Coins?

Here's the connection: we can encode the counterfeit coin's position as a secret string. If the counterfeit is at position $k$, let $s = e_k$ (the string with a 1 only at position $k$). Then:

$$
f(x) = x \cdot e_k = x_k
$$

The function tells us whether coin $k$ is in our query or not. The quantum balance acts as this oracle - it returns information about whether the counterfeit is among the coins we're weighing.

## The Quantum Algorithm

Now let's build up the quantum solution step by step.

### Setup

We have $n$ coins (I'll use $n = 16$). We need $n + 1$ qubits:

- Qubits $0$ through $n-1$: represent which coins to place on the scale
- Qubit $n$: ancilla qubit for the balance result

The counterfeit coin is at position $m$ (unknown to us, but the oracle knows).

```python
import cirq
from collections import Counter

n = num_coins = 16
num_qubits = num_coins + 1
m = counterfeit_index = 7  # The oracle's secret

qbs = cirq.LineQubit.range(num_qubits)
```

*Note: In this simulation, we know `m` to construct the oracle. In a real scenario, the oracle would be a black-box physical process (the quantum balance) that we query without knowing which coin is counterfeit.*

### Step 1: Create a Superposition

First, we create an equal superposition over all possible coin selections:

$$
|0\rangle^{\otimes n} \xrightarrow{H^{\otimes n}} \frac{1}{\sqrt{2^n}} \sum_{x \in \{0,1\}^n} |x\rangle
$$

Each basis state $|x\rangle = |x_1 x_2 \cdots x_n\rangle$ represents a weighing configuration: coin $i$ is on the scale if $x_i = 1$.

```python
circuit = cirq.Circuit(cirq.H.on_each(*qbs[:-1]))
```

### Step 2: Filter for Even-Weight Strings

Here's a subtlety: we need our superposition to contain only strings with **even Hamming weight** (even number of 1s).

Why? This constraint comes from the mathematical structure of the algorithm, not from physical balance requirements. The Bernstein-Vazirani reduction requires the query strings to form a subspace where the inner product properties work out correctly. Even-weight strings form exactly such a subspace - they're closed under XOR and have the right interference properties for the final Hadamard transform to reveal the secret.

We compute the parity using a cascade of CNOT gates:

$$
|x_1 x_2 \cdots x_n\rangle |0\rangle \xrightarrow{\text{CNOTs}} |x_1 x_2 \cdots x_n\rangle |x_1 \oplus x_2 \oplus \cdots \oplus x_n\rangle
$$

```python
circuit.append([cirq.CNOT(qbs[i], qbs[n]) for i in range(n)])
```

Then we flip and measure the ancilla:

```python
circuit.append([cirq.X(qbs[n]), cirq.measure(qbs[n], key="is_even")])
```

After the X gate, the ancilla is $|1\rangle$ if the parity was even, $|0\rangle$ if odd.

**If we measure 1:** We have successfully projected onto the even-weight subspace. This happens with probability exactly $\frac{1}{2}$ .

**If we measure 0:** The parity was odd. We must restart.

The state after successful projection is:

$$
|\psi_{\text{even}}\rangle = \frac{1}{\sqrt{2^{n-1}}} \sum_{x: |x| \equiv 0 \pmod 2} |x\rangle
$$

### Step 3: The Quantum Oracle (Balance Query)

Now comes the key step. We query the quantum balance using the Bernstein-Vazirani oracle. This is implemented as a controlled sub-circuit that only runs if our parity check succeeded:

```python
sub_circuit = cirq.Circuit()
sub_circuit.append(cirq.H(qbs[n]))
sub_circuit.append(cirq.CNOT(qbs[m], qbs[n]))
sub_circuit.append(cirq.H.on_each(*qbs[:-1]))
```

Let me explain what's happening mathematically.

#### Preparing the Ancilla

We start by applying Hadamard to the ancilla (which is $|1\rangle$ after our successful parity check):

$$
|1\rangle \xrightarrow{H} \frac{|0\rangle - |1\rangle}{\sqrt{2}} = |-\rangle
$$

Our full state is now:

$$
|\psi\rangle = \frac{1}{\sqrt{2^{n-1}}} \sum_{x: |x| \equiv 0 \pmod 2} |x\rangle \otimes |-\rangle
$$

#### The Oracle Query

The oracle applies a CNOT with the counterfeit qubit $m$ as control and the ancilla as target. This computes:

$$
|x\rangle |-\rangle \xrightarrow{U_f} |x\rangle |{-} \oplus x_m\rangle = (-1)^{x_m} |x\rangle |-\rangle
$$

This is the famous **phase kickback** trick! The oracle flips the ancilla when $x_m = 1$, but because the ancilla is in the $|-\rangle$ state, this manifests as a phase flip on the input register.

After the oracle:

$$
|\psi\rangle = \frac{1}{\sqrt{2^{n-1}}} \sum_{x: |x| \equiv 0 \pmod 2} (-1)^{x_m} |x\rangle \otimes |-\rangle
$$

The phase $(-1)^{x_m}$ encodes the counterfeit position $m$.

#### Final Hadamard Transform

Now we apply Hadamard gates to all coin qubits:

$$
H^{\otimes n} |x\rangle = \frac{1}{\sqrt{2^n}} \sum_{y \in \{0,1\}^n} (-1)^{x \cdot y} |y\rangle
$$

The full state becomes:

$$
|\psi_{\text{final}}\rangle = \frac{1}{\sqrt{2^{n-1}}} \sum_{x: |x| \equiv 0 \pmod 2} (-1)^{x_m} \cdot \frac{1}{\sqrt{2^n}} \sum_y (-1)^{x \cdot y} |y\rangle
$$

Rearranging:

$$
|\psi_{\text{final}}\rangle = \frac{1}{\sqrt{2^{2n-1}}} \sum_y \left( \sum_{x: |x| \equiv 0 \pmod 2} (-1)^{x \cdot (y \oplus e_m)} \right) |y\rangle
$$

where $e_m$ is the unit vector with 1 at position $m$.

The inner sum is non-zero only when $y \oplus e_m$ has even Hamming weight, i.e., when $y$ has the **opposite parity of $e_m$** . Since $e_m$ has Hamming weight 1 (odd), we need $y$ to have odd weight.

For such $y$, the inner sum equals $2^{n-1}$, giving us:

$$
|\psi_{\text{final}}\rangle = \frac{1}{\sqrt{2^{n-1}}} \sum_{y: |y| \equiv 1 \pmod 2} |y\rangle
$$

Wait - this is a superposition over odd-weight strings. How do we find $m$?

### Step 4: Measurement and Identification

Here's what makes this work: the phases from the oracle create destructive interference that eliminates most terms, leaving the final state concentrated on specific strings that reveal $m$.

When we measure, we get one of two outcomes:

- $|e_m\rangle = |00\cdots 010 \cdots 00\rangle$ (all zeros except position $m$ is 1)
- $|\bar{e}_m\rangle = |11\cdots 101 \cdots 11\rangle$ (all ones except position $m$ is 0)

Both are odd-weight strings (Hamming weight 1 or $n-1$), and both reveal $m$ as the unique bit that differs from the rest.

**Why does this happen?** The key is that the oracle phase $(-1)^{x_m}$ correlates perfectly with bit $m$ across all even-weight basis states. When we apply the final Hadamard transform, constructive interference occurs only for strings where bit $m$ is "special" - either the only 1 or the only 0. All other odd-weight strings destructively cancel.

In practice: measure all qubits, find the bit that differs from the majority.

```python
circuit.append(cirq.CircuitOperation(sub_circuit.freeze()).with_classical_controls("is_even"))
circuit.append([cirq.measure(qbs[i], key=f"qubit({i:02})") for i in range(n)])
```

### Running the Algorithm

```python
simulator = cirq.Simulator()
results = simulator.run(circuit)
num_iterations = 1

while True:
    if results.measurements["is_even"][0] == 1:
        print("iteration:", num_iterations)
        print(results)
        break
    results = simulator.run(circuit)
    num_iterations += 1
```

The algorithm repeats until we get a successful parity projection (expected 2 attempts on average).

### Finding the Counterfeit

```python
data = [results.data[f"qubit({i:02})"][0] for i in range(n)]
counts = Counter(data)

# Find the least common measured value
least_common = min(counts, key=counts.get)
least_common_index = data.index(least_common)

print("index of counterfeit coin:", least_common_index)
```

Output:
```
iteration: 1
index of counterfeit coin: 7
```

The counterfeit was at index 7, and we found it with a single oracle query!

## The Complete Circuit

Here's what the full circuit looks like (for 16 coins):

```
0: ───H───@─────────────────────────────────────────────────────────────[...]───M──
          │                                                              ║
1: ───H───┼───@─────────────────────────────────────────────────────────[...]───M──
          │   │                                                          ║
2: ───H───┼───┼───@─────────────────────────────────────────────────────[...]───M──
          │   │   │                                                      ║
...       ...                                                            ║
          │   │   │                                                      ║
7: ───H───┼───┼───┼───@─────────────────────────────────────────────────[BV ]───M──
          │   │   │   │                                                  ║
...       ...                                                            ║
          │   │   │   │                                                  ║
16: ──────X───X───X───X───...───X───M('is_even')────────────────────────╩═══────────
```

The `[BV]` box contains the Bernstein-Vazirani subcircuit that runs conditionally on `is_even = 1`.

## Complexity Analysis

Comparing the quantum and classical approaches:

|                         | Classical       | Quantum                    |
| ----------------------- | --------------- | -------------------------- |
| **Query Complexity**    | $O(\log N)$     | $O(1)$                     |
| **Total Operations**    | $O(\log N)$     | $O(N)$ gates               |
| **Success Probability** | 100%            | 50% per attempt            |

The quantum algorithm uses $O(1)$ oracle queries but requires $O(N)$ quantum gates to prepare the superposition and compute parity. The 50% success rate means we need an expected 2 runs.

The key speedup is in **query complexity** - the number of times we consult the balance. This is the standard measure for oracle problems, and here quantum computing reduces $O(\log N)$ to $O(1)$.

*Note: This analysis assumes ideal, noiseless quantum operations. Real quantum hardware introduces errors that would require error correction or multiple trials, but the fundamental query complexity advantage remains.*

## Multiple Counterfeit Coins

The original paper also considers the case of $k$ counterfeit coins among $N$ total:

- **Classical:** $O(k \log(N/k))$ queries
- **Quantum:** $O(k^{1/4})$ queries

This is a quartic speedup, and notably the quantum complexity is independent of $N$.

## Conclusion

The counterfeit coin problem beautifully illustrates quantum speedup. By recasting a classical search problem as an instance of Bernstein-Vazirani, we reduce the query complexity from logarithmic to constant.

This isn't just a toy example - it demonstrates fundamental principles:

1. **Superposition** lets us query all possible coin configurations simultaneously
2. **Phase kickback** encodes the oracle's answer in quantum phases
3. **Interference** (via Hadamard) extracts the hidden information

The speedup here is provable and unconditional - no complexity assumptions required. For anyone interested in quantum computing, the counterfeit coin problem is a clean example of quantum advantage in action.

## Further Reading

For a comprehensive catalog of quantum algorithms with proven speedups, see the [Quantum Algorithm Zoo](https://quantumalgorithmzoo.org/).

## References

- Iwama, K., Nishimura, H., Raymond, R., & Teruyama, J. (2012). *Quantum Counterfeit Coin Problems*. Theoretical Computer Science.
- Bernstein, E., & Vazirani, U. (1997). *Quantum Complexity Theory*. SIAM Journal on Computing.
- Terhal, B. M., & Smolin, J. A. (1998). *Single quantum querying of a database*. Physical Review A.
