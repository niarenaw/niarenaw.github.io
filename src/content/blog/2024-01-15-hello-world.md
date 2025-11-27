---
title: "Hello World"
description: "Welcome to my blog! This is my first post testing markdown, code, and LaTeX."
pubDate: 2024-01-15
---

Welcome to my blog! This is my first post, testing all the features.

## Markdown Features

Here's some **bold text** and *italic text*. You can also use `inline code`.

### Lists

- Item one
- Item two
- Item three

### Links

Check out [Astro](https://astro.build) for more info.

## Code Highlighting

Here's a TypeScript example with syntax highlighting:

```typescript
interface User {
  name: string;
  email: string;
}

function greet(user: User): string {
  return `Hello, ${user.name}!`;
}
```

And some Python:

```python
def fibonacci(n: int) -> int:
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

## LaTeX Support

Inline math works like this: $E = mc^2$

Block equations are rendered beautifully:

$$
\frac{d}{dx} \int_a^x f(t) dt = f(x)
$$

The quadratic formula:

$$
x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}
$$

That's it for the first post!
