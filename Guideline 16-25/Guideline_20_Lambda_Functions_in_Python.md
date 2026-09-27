# Guideline 20: Lambda Functions in Python

Not every function needs a name, a `def`, and a spot of its own in your code. Sometimes you need a tiny, throwaway piece of logic just long enough to hand off to something else — sort a list a certain way, filter out some values, transform an element in passing. Python's answer to that need is the **lambda function**: a function so small it fits on one line, with no name required.

---

## 1. What is Lambda in Python?

A **lambda** is an anonymous function — a function defined without the `def` keyword or a name, built entirely out of a single expression. The syntax is:

```python
lambda arguments: expression
```

```python
square = lambda x: x ** 2
print(square(5))  # 25

add = lambda x, y: x + y
print(add(3, 4))  # 7
```

Notice what's *missing* compared to a regular function: no `def`, no function name (though we assigned it to a variable here for demonstration), and — crucially — no `return` keyword. A lambda's single expression is automatically what gets returned; there's nowhere else for a value to come from, since a lambda body can only ever be one expression, never a series of statements.

This restriction is the whole trade-off of a lambda: it can't contain multiple lines, `for`/`while` loops as statements, or multiple `return` paths — only one expression, evaluated once, returned automatically. What a lambda *loses* in flexibility, it gains in being usable in exactly the places a full function definition would feel like overkill — which is most of what the rest of this guideline is about.

---

## 2. Using If-Else in Lambda

Since a lambda can only hold a single expression, a normal multi-line `if`/`elif`/`else` block won't fit inside one. What *does* fit is the **conditional expression** — the `value_if_true if condition else value_if_false` form you may recall as a compact one-liner.

```python
classify = lambda n: "even" if n % 2 == 0 else "odd"

print(classify(4))  # even
print(classify(7))  # odd
```

Multiple conditions chain the same way a normal `if`/`elif`/`else` would, just written left to right instead of stacked:

```python
grade = lambda score: "A" if score >= 90 else "B" if score >= 80 else "C" if score >= 70 else "F"

print(grade(95))  # A
print(grade(82))  # B
print(grade(55))  # F
```

It works, but readability degrades fast past two or three branches — a plain `def` function with a real `if`/`elif`/`else` block is almost always the better choice once the logic gets this layered. Lambda earns its keep on *simple* decisions, not long ones.

---

## 3. Using Loops in Lambda — Nested Loop

Here's an important limitation to be upfront about: a lambda **cannot contain a `for` or `while` statement** — statements aren't expressions, and a lambda's body must be a single expression. What a lambda *can* contain is a **list comprehension**, which — as you learned back in Guideline 11 — is itself a single expression that happens to loop internally. That's the loophole that lets "looping" logic live inside a lambda at all.

```python
double_all = lambda lst: [x * 2 for x in lst]
print(double_all([1, 2, 3, 4]))  # [2, 4, 6, 8]
```

Nested loops work the same way — a nested list comprehension, still just one expression overall:

```python
flatten = lambda matrix: [value for row in matrix for value in row]

grid = [[1, 2], [3, 4], [5, 6]]
print(flatten(grid))  # [1, 2, 3, 4, 5, 6]
```

```python
double_grid = lambda matrix: [[x * 2 for x in row] for row in matrix]
print(double_grid(grid))  # [[2, 4], [6, 8], [10, 12]]
```

So when you hear "loop inside a lambda," what's actually happening is a comprehension — never a literal `for` statement. Keep that distinction straight; it's a common point of confusion.

---

## 4. The `filter()` Method

If there's one function that lambda was practically made for, it's `filter()`. Its signature mirrors `map()` from Guideline 17:

```python
filter(function, iterable)
```

`filter()` keeps only the elements for which `function(element)` returns `True`, discarding the rest — and just like `map()`, it returns a lazy filter object, so wrap it in `list()` to see the results.

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

evens = filter(lambda x: x % 2 == 0, numbers)
print(list(evens))  # [2, 4, 6, 8, 10]

long_words = filter(lambda w: len(w) > 4, ["cat", "elephant", "dog", "giraffe"])
print(list(long_words))  # ['elephant', 'giraffe']
```

This pairing — `filter()` supplying the "for every element, keep or discard" machinery, and lambda supplying the one-line test — is one of the most common patterns you'll see in real Python code, and it's worth being completely comfortable with it.

---

## 5. Define a Function, Then Use Lambda

Lambda's one-expression limit means that once your logic needs more than a single line — a loop with an accumulator, multiple checks, an early exit — a lambda genuinely can't hold it anymore. The fix isn't to abandon lambda; it's to write the real logic as a proper function with `def`, and then have the lambda simply *call* it. The lambda becomes a thin, disposable wrapper around a function that already exists.

Recall `is_prime()` from Guideline 9 — real logic, involving a loop, that could never fit inside a lambda by itself:

```python
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True
```

You can't rewrite that *as* a lambda — but you can absolutely *use* it *inside* one, which is exactly what `filter()` needs:

```python
numbers = list(range(2, 30))
primes = filter(lambda n: is_prime(n), numbers)
print(list(primes))  # [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

(In this particular case, `filter(is_prime, numbers)` — passing `is_prime` directly, with no lambda at all — would do exactly the same job, since `is_prime` already takes one argument and returns `True`/`False`. The lambda wrapper earns its place when you need to pass *extra* fixed arguments alongside the loop variable, e.g. `lambda n: is_divisible(n, by=3)`, which a bare function reference can't do on its own.)

The other direction is worth knowing too: some very *simple* `def` functions are direct, one-to-one translations into a lambda —

```python
def add(x, y):
    return x + y

add = lambda x, y: x + y  # exactly equivalent
```

— but this conversion only works because `add` was already a single expression to begin with. The rule of thumb: convert a `def` into a lambda when the whole function is one expression; keep it as a `def` and merely *call it from* a lambda when it's anything more than that.

---

## 6. Sorting in Lambda

`sorted()` accepts a `key` argument — a function applied to each element to decide *what to sort by*, rather than sorting the elements themselves directly. This is lambda's second-biggest use case after `filter()`.

```python
students = [("Amara", 90), ("Ben", 85), ("Cho", 95)]

by_score = sorted(students, key=lambda student: student[1])
print(by_score)
# [('Ben', 85), ('Amara', 90), ('Cho', 95)]
```

Here, `lambda student: student[1]` tells `sorted()` to look at the **second entry of each tuple** — the score — rather than the tuples themselves (which, sorted directly, would compare names alphabetically first). This is exactly the technique to reach for whenever you need to sort a list of tuples, dictionaries, or objects by *one specific field* rather than the whole item.

```python
records = [{"name": "Amara", "age": 20}, {"name": "Ben", "age": 19}]
by_age = sorted(records, key=lambda r: r["age"])
print(by_age)
# [{'name': 'Ben', 'age': 19}, {'name': 'Amara', 'age': 20}]
```

You can also sort by more than one field at once, by returning a tuple from the lambda — Python compares tuples element by element, so this sorts by the first value, then breaks ties using the second:

```python
by_score_then_name = sorted(students, key=lambda s: (s[1], s[0]))
```

---

## 7. Reversing with Lambda

`sorted()`'s `reverse=True` argument flips the entire sort to descending order — and it combines directly with a lambda `key`, letting you sort descending by a *specific field* rather than the default order:

```python
by_score_desc = sorted(students, key=lambda student: student[1], reverse=True)
print(by_score_desc)
# [('Cho', 95), ('Amara', 90), ('Ben', 85)]
```

Where `reverse=True` gets more interesting is when you need **mixed** directions — one field ascending, another descending, in the same sort. `reverse=True` alone applies to the *whole* sort, so it can't do this by itself. The workaround is to negate the field you want descending, directly inside the lambda:

```python
# Sort by score DESCENDING, and for ties, by name ASCENDING
mixed = sorted(students, key=lambda s: (-s[1], s[0]))
```

Negating `-s[1]` flips just that one field's ordering, while `s[0]` stays as-is — a trick that only works cleanly on numbers, but it's the standard answer whenever "mostly ascending, except this one column" comes up.

---

## 8. Return Multiple Results in Lambda

A `return` statement doesn't exist inside a lambda — but that doesn't mean a lambda is limited to producing a single *value*. Since a lambda's body is one expression, and a tuple literal is itself a single expression, a lambda can absolutely package up several results by returning a tuple:

```python
stats = lambda a, b: (a + b, a - b, a * b, a / b)

total, difference, product, quotient = stats(10, 4)
print(total, difference, product, quotient)  # 14 6 40 2.5
```

Nothing exotic is happening here — `(a + b, a - b, a * b, a / b)` is just a tuple, and a tuple is a perfectly ordinary single expression. The lambda "returns multiple values" the same way any regular function does when it writes `return a, b, c` — Python packs them into a tuple either way; the lambda syntax just makes that tuple the entire body instead of hiding it behind a `return`.

---

Lambda is not a replacement for `def` — it's a shorthand for the specific, narrow case where a function is small enough to be a single expression and disposable enough not to need a name. `filter()` and `sorted()`'s `key` argument are where that shorthand earns its keep most often; anywhere the logic grows past one line, reach for a real function instead, and let the lambda simply call it.
