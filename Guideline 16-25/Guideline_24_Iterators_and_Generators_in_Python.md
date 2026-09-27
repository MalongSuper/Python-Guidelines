# Guideline 24: Iterators and Generators in Python

Every `for` loop you've written so far has been quietly relying on a mechanism you haven't had to think about directly: how Python actually pulls one item at a time out of a list, a string, or a range. This guideline pulls back that curtain — first with **iterators**, the machinery underneath every loop, and then with **generators**, a remarkably simple way to build your own.

---

## 1. Iterators and Generators (Lazy Evaluation)

So far, most of the sequences you've worked with have been built entirely up front — a list comprehension computes *every* value before you use any of them. This is called **eager evaluation**: all the work happens immediately, and the entire result sits in memory whether you need all of it right now or not.

**Lazy evaluation** is the opposite philosophy: produce a value only at the exact moment it's actually requested, and not a moment sooner. Iterators and generators are Python's two tools for this — instead of handing you a finished list of a million values, they hand you something that can produce the *next* value, on demand, one at a time, which means only one value needs to exist in memory at any given moment. This matters enormously once "how many values" grows large, or is unknown, or is technically infinite.

---

## 2. Iterable Objects

An **iterable** is any object Python knows how to loop over with a `for` statement — a list, a string, a tuple, a dictionary, a set, a `range()`. What makes something iterable, technically, is that it knows how to produce an iterator from itself when asked.

```python
print([1, 2, 3])       # a list -- iterable
print("hello")          # a string -- iterable
print(range(5))         # a range -- iterable
```

It's worth being precise about a distinction that's easy to blur: an **iterable** is something you can *get* an iterator *from* — it is not, itself, the thing producing values one at a time. That role belongs to the iterator itself, covered next.

---

## 3. The `iter()` Method

`iter()` takes an iterable and returns an **iterator** — the actual object responsible for producing one value at a time.

```python
numbers = [10, 20, 30]
it = iter(numbers)
print(it)  # <list_iterator object at 0x...>
```

Notice `it` is not the list — it's a distinct object, a kind of "cursor" positioned just before the first element, ready to produce values one at a time when asked.

---

## 4. The `next()` Method

`next()` asks an iterator for its next value, advancing its internal position by one each time it's called.

```python
numbers = [10, 20, 30]
it = iter(numbers)

print(next(it))  # 10
print(next(it))  # 20
print(next(it))  # 30
print(next(it))  # raises StopIteration -- nothing left
```

That last call isn't a bug — `StopIteration` is exactly how an iterator signals "there are no more values." As you'll see in Section 7, this is the exact exception a `for` loop is silently catching on your behalf every single time one finishes.

---

## 5. Iterators Maintain Their Own State

An iterator remembers exactly where it left off between calls to `next()` — that memory of position is the entire point of the object. Two separate calls to `iter()` on the same iterable, though, produce two completely **independent** iterators, each with its own position:

```python
numbers = [1, 2, 3]

it1 = iter(numbers)
it2 = iter(numbers)

print(next(it1))  # 1
print(next(it1))  # 2
print(next(it2))  # 1  -- it2 has its own separate position, unaffected by it1
```

Advancing `it1` doesn't touch `it2` at all, and neither one changes the original list `numbers` — the state being tracked belongs entirely to the iterator, not the data it's iterating over.

---

## 6. Converting an Iterator Back Into a List

`list()` will drain an iterator and collect whatever values remain into a fresh list:

```python
numbers = [1, 2, 3, 4]
it = iter(numbers)

next(it)  # consumes the first value (1), discarding it here
remaining = list(it)
print(remaining)  # [2, 3, 4]
```

The key word there is **remain** — `list()` doesn't rewind the iterator back to the start, it only collects whatever hasn't been consumed yet. Once an iterator has been converted to a list (or otherwise exhausted), it's genuinely spent — calling `next()` on it again still raises `StopIteration`, and there's no way to "reset" it; you'd need a fresh `iter()` call on the original iterable to start over.

---

## 7. Iterators vs. Normal Loops

A `for` loop has felt like a single, simple construct throughout this book — but underneath, it's doing exactly what Sections 3 and 4 just walked through manually, automatically and invisibly.

```python
for value in [10, 20, 30]:
    print(value)
```

is, under the hood, equivalent to:

```python
numbers = [10, 20, 30]
it = iter(numbers)

while True:
    try:
        value = next(it)
    except StopIteration:
        break
    print(value)
```

| | `for` loop | Manual iterator loop |
|---|---|---|
| Calls `iter()` | Automatically, once, at the start | You call it yourself |
| Calls `next()` | Automatically, once per pass | You call it yourself |
| Handles `StopIteration` | Automatically — simply ends the loop | You catch it yourself with `try`/`except` |
| Code required | Minimal | Considerably more, for identical behavior |

There's no functional difference between the two — the `for` loop is simply the readable, convenient surface Python provides over exactly this mechanism. Knowing what's underneath matters because it's the same mechanism generators plug directly into, which is where the rest of this guideline is headed.

---

## 8. Generators in Python

Building a fully custom iterator by hand — one with its own `__iter__` and `__next__` methods — is possible in Python, but it's verbose for what is often a genuinely simple task. A **generator** is a dramatically simpler way to get the exact same lazy, one-at-a-time behavior, written as an ordinary-looking function with one small but transformative difference: it uses `yield` instead of `return`.

```python
def count_up_to(n):
    i = 1
    while i <= n:
        yield i
        i += 1
```

Calling `count_up_to(5)` doesn't run any of that code yet — it immediately returns a **generator object**, which behaves exactly like the iterators from Sections 3 and 4: it responds to `next()`, it can be looped over with `for`, and it raises `StopIteration` once exhausted.

```python
gen = count_up_to(5)
print(gen)          # <generator object count_up_to at 0x...>
print(next(gen))    # 1
print(next(gen))    # 2

for value in count_up_to(3):
    print(value)
# 1
# 2
# 3
```

---

## 9. The `yield` Method

`yield` is what makes a function a generator at all — its presence anywhere inside a function's body is what changes that function's entire behavior. Where `return` hands back a value and ends the function completely, `yield` hands back a value and **pauses** the function exactly where it is, preserving every local variable, ready to resume from that exact line the next time a value is requested.

```python
def simple_generator():
    print("First value coming up")
    yield 1
    print("Second value coming up")
    yield 2
    print("Done")

gen = simple_generator()

print(next(gen))
# First value coming up
# 1

print(next(gen))
# Second value coming up
# 2

next(gen)
# Done
# raises StopIteration
```

Notice the print statements only appear *between* calls to `next()` — proof that the function's body genuinely pauses at each `yield`, rather than running start to finish the way a normal function call always does.

---

## 10. Normal Function vs. Generator

| | Normal function (`return`) | Generator function (`yield`) |
|---|---|---|
| Calling it | Runs the entire body immediately | Runs **none** of the body yet — returns a generator object |
| Getting a result | Returns one value (or one collection), then the function ends completely | Produces one value at a time, pausing between each |
| Memory use for a large result | Must build the entire result (e.g. a whole list) in memory | Only the current value needs to exist at any moment |
| Can it be resumed? | No — every call starts fresh from the top | Yes — resumes exactly where it last paused |

The memory difference becomes concrete once the size involved grows large:

```python
def squares_list(n):
    result = []
    for i in range(n):
        result.append(i ** 2)
    return result  # the ENTIRE list must exist in memory at once

def squares_generator(n):
    for i in range(n):
        yield i ** 2  # only ONE value exists in memory at any moment
```

For a small `n`, the difference is invisible. For `n = 100_000_000`, `squares_list()` tries to build a hundred-million-element list before returning anything at all, while `squares_generator()` can start handing out values immediately and never holds more than one in memory — this is lazy evaluation, from Section 1, made concrete.

---

## 11. Generators Remember Their State

Just like the iterators from Section 5, a generator remembers exactly where it left off — but a generator goes a step further, preserving its **entire local state**, not just a position. Every local variable inside the function is frozen in place at each `yield`, and picks up exactly as it was the moment `next()` is called again.

```python
def running_total():
    total = 0
    while True:
        value = yield total
        total += value

gen = running_total()
next(gen)               # primes the generator, gets the initial total (0)

print(gen.send(10))      # total becomes 10, yields 10
print(gen.send(5))       # total becomes 15, yields 15
print(gen.send(20))      # total becomes 35, yields 35
```

`total` isn't reset to `0` between calls the way a normal function's local variables would reset on every fresh call — it's the *same* `total` variable, still alive, still remembering its value, because the function itself never actually ended; it only ever paused. This is the deepest difference between a normal function and a generator: a normal function's local state disappears the instant it returns, while a generator's local state survives for as long as the generator object itself exists.

---

Iterators are the quiet mechanism underneath every `for` loop you've ever written; generators are the easiest way to build that exact mechanism yourself, using ordinary function syntax with one keyword — `yield` — doing all the work of pausing, remembering, and resuming. Together, they're how Python handles sequences that are too large, too expensive, or too genuinely infinite to ever build all at once.
