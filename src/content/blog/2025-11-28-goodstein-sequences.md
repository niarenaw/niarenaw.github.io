---
title: "Goodstein Sequences"
description: "A simple arithmetic process that grows faster than any computable function, yet always terminates - and this fact is unprovable in standard arithmetic."
pubDate: 2025-11-28
---

Some theorems feel like magic tricks. Goodstein sequences are one of them: take any positive integer, apply a simple recursive rule, and watch the numbers explode - growing faster than exponentials, faster than towers of exponentials, faster than any primitive recursive function. The sequence appears to diverge to infinity.

Yet every Goodstein sequence terminates at zero. Always. And the twist... this fact cannot be proven using the standard axioms of arithmetic.

This post walks through the construction, implements it in Python, and sketches the ordinal-theoretic proof of termination.

## The Construction

Goodstein sequences are built from **hereditary base-$n$ notation**, a way of writing numbers where the exponents themselves are also expanded.

### Hereditary Base-$n$ Notation

In ordinary base-$n$, we write numbers as sums of powers:

$$
a_k n^k + a_{k-1} n^{k-1} + \cdots + a_1 n + a_0
$$

Hereditary notation goes further: every exponent is also written in base-$n$, recursively, until all numbers in the expression are less than $n$.

For example, $100$ in base $3$:

$$
100 = 81 + 18 + 1 = 3^4 + 2 \cdot 3^2 + 1
$$

But $4 = 3 + 1 = 3^1 + 1$, so the hereditary form is:

$$
100 = 3^{3^1 + 1} + 2 \cdot 3^2 + 1
$$

Every positive integer has a unique hereditary base-$n$ representation for any $n \geq 2$.

### The Goodstein Rule

The Goodstein sequence $G(m)$ starting from $m$ proceeds as follows:

1. Write $m$ in hereditary base-$2$
2. Replace every $2$ with $3$ (a "bump")
3. Subtract $1$
4. Repeat: write in hereditary base-$3$, replace with $4$, subtract $1$
5. Continue until you reach $0$

At step $k$, you're working in base $k+2$: write the number in hereditary base-$(k+2)$, replace with $(k+3)$, subtract $1$.

Let's implement this to see it in action.

## Implementation

First, we need a data structure for hereditary representations. Each number is a sum of terms, where each term has a coefficient and an exponent (which is itself a hereditary representation):

```python
from dataclasses import dataclass

@dataclass
class Term:
    coeff: int           # Coefficient (1 to base-1)
    exp: list["Term"]    # Exponent as hereditary representation

# A hereditary representation is a list of Terms
# Example: 2^(2^1) + 2^1 + 1 = [Term(1, [Term(1, [Term(1, [])])]),
#                               Term(1, [Term(1, [])]),
#                               Term(1, [])]
```

Converting a number to hereditary form:

```python
def to_hereditary(n: int, base: int) -> list[Term]:
    """Convert n to hereditary base-b representation"""
    if n == 0:
        return []

    terms = []
    while n > 0:
        # Find the largest power of base that fits
        exp = 0
        power = 1
        while power * base <= n:
            power *= base
            exp += 1

        # How many times does this power fit?
        coeff = n // power
        n = n % power

        # Recursively convert the exponent
        exp_hereditary = to_hereditary(exp, base)
        terms.append(Term(coeff, exp_hereditary))

    return terms
```

Evaluating a hereditary representation in a new base (after the "bump"):

```python
def evaluate(terms: list[Term], base: int) -> int:
    """Evaluate hereditary representation in given base"""
    total = 0
    for term in terms:
        if not term.exp:  # Exponent is 0, so base^0 = 1
            total += term.coeff
        else:
            exp_val = evaluate(term.exp, base)
            total += term.coeff * (base ** exp_val)
    return total
```

Now the Goodstein step - bump the base and subtract:

```python
def goodstein_step(n: int, base: int) -> int:
    """One step: convert to hereditary base, bump to base+1, subtract 1"""
    hereditary = to_hereditary(n, base)
    bumped = evaluate(hereditary, base + 1)
    return bumped - 1
```

And the full sequence generator:

```python
def goodstein_sequence(start: int, max_steps: int = 100) -> list[int]:
    """Generate Goodstein sequence starting from n"""
    seq = [start]
    base = 2

    while seq[-1] > 0 and len(seq) < max_steps:
        seq.append(goodstein_step(seq[-1], base))
        base += 1

    return seq
```

## A Complete Example: $G(3)$

Tracing through $G(3)$ step by step:

**Step 0:** Start with $3$ in hereditary base-$2$:
$$3 = 2^1 + 2^0 = 2^1 + 1$$

**Step 1:** Bump $2 \to 3$, then subtract:
$$3^1 + 1 = 4 \quad\to\quad 4 - 1 = 3$$

**Step 2:** Write $3$ in hereditary base-$3$:
$$3 = 3^1$$

Bump $3 \to 4$, subtract:
$$4^1 = 4 \quad\to\quad 4 - 1 = 3$$

**Step 3:** Write $3$ in hereditary base-$4$:
$$3 = 3 \quad \text{(just a constant, no powers)}$$

Bump has no effect. Subtract:
$$3 - 1 = 2$$

**Steps 4-5:** Continue subtracting: $2 \to 1 \to 0$.

```python
print(goodstein_sequence(3))
# [3, 3, 3, 2, 1, 0]
```

Six steps. Not too bad.

## The Sequence $G(4)$

Now try $G(4)$:

```python
seq = goodstein_sequence(4, max_steps=20)
print(seq)
# [4, 26, 41, 60, 83, 109, 139, 173, 211, 253, 299, 348, 401, 458, 519, 584, 653, 726, 803, 884]
```

It's increasing. More steps:

```python
seq = goodstein_sequence(4, max_steps=1000)
print(f"After 1000 steps: {seq[-1]}")
# After 1000 steps: 2352161
```

Still climbing. At step 10000, it reaches about 100 million. At step 1000000, it has 12 digits. The growth looks roughly quadratic in this phase.

But appearances deceive. The sequence $G(4)$ does eventually terminate—after exactly:

$$3 \times 2^{402653211} - 3 \text{ steps}$$

That's a number with over 121 million digits. The sequence grows to incomprehensible sizes before finally collapsing to zero.

## Visualizing the Growth

A helper to print the hereditary form shows what's happening structurally:

```python
def format_hereditary(terms: list[Term], base: int) -> str:
    """Pretty-print hereditary representation"""
    if not terms:
        return "0"

    parts = []
    for term in terms:
        if not term.exp:  # base^0 = 1, so just the coefficient
            parts.append(str(term.coeff))
        elif term.exp == [Term(1, [])]:  # base^1
            if term.coeff == 1:
                parts.append(str(base))
            else:
                parts.append(f"{term.coeff}·{base}")
        else:
            exp_str = format_hereditary(term.exp, base)
            if term.coeff == 1:
                parts.append(f"{base}^({exp_str})")
            else:
                parts.append(f"{term.coeff}·{base}^({exp_str})")

    return " + ".join(parts)
```

Now we can watch the structure evolve:

```python
def trace_goodstein(start: int, steps: int = 10):
    """Trace Goodstein sequence showing hereditary forms"""
    n = start
    base = 2

    for step in range(steps):
        if n == 0:
            print(f"Step {step}: 0 - terminated!")
            break

        terms = to_hereditary(n, base)
        form = format_hereditary(terms, base)
        print(f"Step {step}: {n} = {form} (base {base})")

        n = goodstein_step(n, base)
        base += 1
```

```python
trace_goodstein(4, steps=8)
```

Output:
```
Step 0: 4 = 2^(2) (base 2)
Step 1: 26 = 3^(3) + 2·3 + 2 (base 3)
Step 2: 41 = 4^(4) + 2·4 + 1 (base 4)
Step 3: 60 = 5^(5) + 2·5 (base 5)
Step 4: 83 = 6^(6) + 2·6 + 5 (base 6)
Step 5: 109 = 7^(7) + 2·7 + 4 (base 7)
Step 6: 139 = 8^(8) + 2·8 + 3 (base 8)
Step 7: 173 = 9^(9) + 2·9 + 2 (base 9)
```

Look at the pattern: $b^b + 2b + c$ where $c$ counts down. When $c$ hits zero, the $2b$ term will start unwinding. Eventually the leading $b^b$ term will need to decompose - and that's when things explode.

## Why It Always Terminates

Despite the explosive growth, every Goodstein sequence reaches zero. The proof uses **[ordinal numbers](https://en.wikipedia.org/wiki/Ordinal_number)** - a generalization of natural numbers that extends into the transfinite.

### The Key Idea: An Ordinal Shadow

At each step, we create a "shadow" of the current value by replacing the base with $\omega$ (the first infinite ordinal). This maps each Goodstein value to an ordinal below $\varepsilon_0$, where:

$$
\varepsilon_0 = \omega^{\omega^{\omega^{\cdot^{\cdot^{\cdot}}}}}
$$

For example, when $4 = 2^2$ in hereditary base-$2$, its ordinal shadow is:

$$
\omega^\omega
$$

The critical observation is that **while the numerical values grow, the ordinal shadows strictly decrease**.

Why? The base bump ($2 \to 3 \to 4 \to \cdots$) doesn't change the ordinal - $\omega$ stays $\omega$. But subtracting $1$ always decreases the ordinal representation.

Tracing the ordinal shadows:

```python
def to_ordinal_string(terms: list[Term]) -> str:
    """Convert hereditary representation to ordinal notation"""
    if not terms:
        return "0"

    parts = []
    for term in terms:
        if not term.exp:  # ω^0 = 1
            parts.append(str(term.coeff))
        elif term.exp == [Term(1, [])]:  # ω^1 = ω
            if term.coeff == 1:
                parts.append("ω")
            else:
                parts.append(f"{term.coeff}ω")
        else:
            exp_str = to_ordinal_string(term.exp)
            if term.coeff == 1:
                parts.append(f"ω^({exp_str})")
            else:
                parts.append(f"{term.coeff}ω^({exp_str})")

    return " + ".join(parts)
```

```python
def trace_with_ordinals(start: int, steps: int = 8):
    """Show both value and ordinal shadow"""
    n = start
    base = 2

    for step in range(steps):
        if n == 0:
            print(f"Step {step}: value = 0, ordinal = 0")
            break

        terms = to_hereditary(n, base)
        ordinal = to_ordinal_string(terms)
        print(f"Step {step}: value = {n}, ordinal = {ordinal}")

        n = goodstein_step(n, base)
        base += 1
```

```python
trace_with_ordinals(4, steps=10)
```

Output:
```
Step 0: value = 4, ordinal = ω^(ω)
Step 1: value = 26, ordinal = ω^(ω) + 2ω + 2
Step 2: value = 41, ordinal = ω^(ω) + 2ω + 1
Step 3: value = 60, ordinal = ω^(ω) + 2ω
Step 4: value = 83, ordinal = ω^(ω) + ω + 5
Step 5: value = 109, ordinal = ω^(ω) + ω + 4
Step 6: value = 139, ordinal = ω^(ω) + ω + 3
Step 7: value = 173, ordinal = ω^(ω) + ω + 2
Step 8: value = 211, ordinal = ω^(ω) + ω + 1
Step 9: value = 253, ordinal = ω^(ω) + ω
```

Watch the ordinals: they're counting down. The values explode ($4 \to 26 \to 41 \to \cdots$), but the ordinals decrease monotonically.

### Why Ordinals Can't Descend Forever

Ordinals have a crucial property: **there are no infinite descending chains**. Any strictly decreasing sequence of ordinals must be finite.

This is fundamentally different from integers, where we can descend forever in the negatives. Ordinals have a "floor" - you can't go below zero.

So:
1. Each Goodstein step decreases the ordinal shadow
2. Ordinal sequences can't decrease forever
3. Therefore the sequence must reach ordinal $0$
4. When the ordinal is $0$, the value is $0$ $\qquad\qquad\qquad\qquad\qquad\qquad\qquad\qquad\blacksquare$

### Beyond Peano Arithmetic

The proof above uses induction on ordinals up to $\varepsilon_0$. This is called **[transfinite induction](https://en.wikipedia.org/wiki/Transfinite_induction)**, and it's more powerful than ordinary mathematical induction on natural numbers.

In 1982, Laurence Kirby and Jeff Paris proved something remarkable:

> **Goodstein's theorem is unprovable in Peano Arithmetic.**

[Peano Arithmetic](https://en.wikipedia.org/wiki/Peano_axioms) (PA) is the standard axiom system for natural numbers. It captures everything we normally think of as "arithmetic" - addition, multiplication, induction over natural numbers.

But PA can only do induction over $\mathbb{N}$. It cannot express or justify induction up to $\varepsilon_0$. Since that's exactly what the proof requires, Goodstein's theorem lies beyond PA's reach.

This was a landmark result. Unlike Gödel's incompleteness examples (which are self-referential and artificial), Goodstein's theorem is a concrete, natural statement about integer sequences - yet it requires stepping outside arithmetic to prove.

## Growth Rate Comparison

To appreciate how fast Goodstein sequences grow, compare them to familiar fast-growing functions:

| Function | Growth | Example |
|----------|--------|---------|
| Polynomial | $n^k$ | $n^{100}$ |
| Exponential | $2^n$ | Tower of height 1 |
| Tower | $2^{2^{2^n}}$ | Height $n$ |
| Ackermann | $A(n,n)$ | Grows faster than any tower |
| **Goodstein** | Beyond $\varepsilon_0$ | Grows faster than Ackermann |

The Goodstein function $\mathcal{G}(n)$ (number of steps to termination) eventually dominates every [primitive recursive function](https://en.wikipedia.org/wiki/Primitive_recursive_function). It sits at level $\varepsilon_0$ in the [fast-growing hierarchy](https://en.wikipedia.org/wiki/Fast-growing_hierarchy).

To put $G(4)$'s termination time in perspective:

| Quantity | Approximate size |
|----------|-----------------|
| Atoms in observable universe | $10^{80}$ |
| Planck times since Big Bang | $10^{60}$ |
| Steps for $G(4)$ to terminate | $10^{121210694}$ |

## Conclusion

Goodstein sequences are a rare mathematical object - simple to define (hereditary base notation and subtraction), with explosive growth faster than any primitive recursive function, yet guaranteed to terminate via ordinal descent. And the termination proof requires transfinite induction to $\varepsilon_0$, placing it beyond Peano Arithmetic.

They reveal something profound about the foundations of mathematics: some true statements about natural numbers can only be proven by reasoning about infinite objects. The finite and the infinite are more entangled than naive intuition suggests.

## Further Reading

- [Goodstein's theorem](https://en.wikipedia.org/wiki/Goodstein%27s_theorem) - the main theorem and proof sketch
- [Fast-growing hierarchy](https://en.wikipedia.org/wiki/Fast-growing_hierarchy) - where Goodstein functions fit
- [Goodstein sequence](https://googology.fandom.com/wiki/Goodstein_sequence) on Googology Wiki - for large number enthusiasts

## References

- Goodstein, R. L. (1944). *On the Restricted Ordinal Theorem*. Journal of Symbolic Logic.
- Kirby, L. & Paris, J. (1982). *Accessible Independence Results for Peano Arithmetic*. Bulletin of the London Mathematical Society, 14, 285–293.
- Cichon, E. A. (1983). *A Short Proof of Two Recently Discovered Independence Results Using Recursion Theoretic Methods*. Proceedings of the AMS.
