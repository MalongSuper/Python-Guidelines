# Guideline 12: Python Tuples

Guideline 11 built the case for lists as the go-to collection whenever data needs to be added to, removed from, or rearranged over time. Not every collection needs that flexibility, though — and sometimes flexibility is exactly what you *don't* want. A date of birth, once recorded, should not accidentally change. A coordinate on a map should not have its x and y values swapped by a stray line of code three functions away. Python's answer to "a fixed collection of values that should not change" is the **tuple**.

---

## 12.1 What is a Tuple?

A **tuple** is an ordered collection of values, written with **parentheses** `( )` instead of square brackets, with elements separated by commas.

```python
point = (4, 7)
person = ("Alice", 25, "Engineer")
empty_tuple = ()
```

At first glance, this looks like nothing more than a list with different brackets — and structurally, a lot carries over directly: a tuple is ordered (Guideline 11's "preserves order" property applies here too), elements can be any data type, and elements can be mixed types within the same tuple, exactly as with `person` above.

The difference that matters is revealed in the next section, but it is worth naming up front: a tuple is **immutable** — once created, its contents cannot be changed. This single property is what tuples exist for, and it is also why tuples show up constantly in one particular context: **databases**.

A row retrieved from a database table is naturally a fixed, ordered set of values — a customer's ID, name, and email, in that specific order, representing one real record. That record should not be silently rearranged or partially overwritten while your program is working with it; if the email needs to change, that should happen as a deliberate, visible action (typically a new database write), not as a side effect of some unrelated line of code. Representing a database row as a tuple makes that guarantee automatic:

```python
customer_record = (1042, "Alice Chen", "alice@example.com")
```

Anywhere a piece of data is meant to be read, not edited — coordinates, RGB color values, a calendar date, a single row pulled back from a query — a tuple communicates that intent directly, in a way a list does not.

### A Syntax Detail Worth Knowing: the Single-Element Tuple

Because parentheses are also used for grouping expressions in Python — `(2 + 3) * 4` — a tuple with exactly one element needs a trailing comma to actually be recognized as a tuple:

```python
not_a_tuple = (5)
print(type(not_a_tuple))
# <class 'int'>

actual_tuple = (5,)
print(type(actual_tuple))
# <class 'tuple'>
```

Without the comma, Python treats `(5)` as just the number `5` wrapped in parentheses. The comma — not the parentheses — is what actually makes something a tuple. In fact, parentheses are technically optional for tuples with more than one element:

```python
coordinates = 4, 7
print(type(coordinates))
# <class 'tuple'>
```

Parentheses are still recommended in almost all real code, purely for readability — `(4, 7)` is unambiguous at a glance, while `4, 7` on its own can be easy to misread.

---

## 12.2 Tuples vs. Lists

The table below summarizes the practical differences established so far, plus one more that follows directly from immutability: performance.

| Property | List | Tuple |
|---|---|---|
| Syntax | Square brackets `[ ]` | Parentheses `( )` |
| Mutable? | Yes — can be changed after creation | No — fixed once created |
| Preserves order? | Yes | Yes |
| Indexing and slicing | Supported | Supported |
| Typical use case | A collection that will grow, shrink, or be rearranged | A fixed record or a value that should not change |
| Relative performance | Slightly slower to create and iterate | Slightly faster, and uses less memory |

That last row is a direct consequence of immutability: because Python knows a tuple's contents can never change, it can store and manage a tuple more efficiently than a list of the same values. This performance difference is rarely the deciding factor in choosing between the two — clarity of intent almost always is — but it is a genuine, measurable side benefit of using a tuple where one fits.

**The question to ask when choosing between them is simple: will this collection ever need to change after it is created?** If yes, use a list. If no — if it represents something fixed, like a record, a coordinate, or a set of unchanging settings — a tuple documents that guarantee in the code itself, rather than leaving it as an assumption that later code might accidentally violate.

---

## 12.3 Methods in Tuples (Almost Identical to Lists)

Tuples support the same **non-modifying** operations that lists do: `len()`, indexing, slicing, iterating with a `for` loop, and checking membership with `in` all work exactly the same way.

```python
person = ("Alice", 25, "Engineer")

print(len(person))       # 3
print(person[0])          # Alice
print(person[1:])         # (25, 'Engineer')
print("Engineer" in person)   # True

for item in person:
    print(item)
# Alice
# 25
# Engineer
```

Notice that slicing a tuple returns another tuple — `person[1:]` produces `(25, 'Engineer')`, not a list — which makes sense, since a "slice of a fixed record" should still behave like a fixed record.

Where tuples differ sharply from lists is in their **methods**. Every list method from Guideline 11 that *changed* the list — `.append()`, `.extend()`, `.insert()`, `.pop()`, `.remove()`, `.sort()`, `.reverse()`, `.clear()` — simply does not exist for tuples. A tuple cannot be modified, so there is nothing for those methods to do. Attempting to call one raises an error immediately:

```python
person = ("Alice", 25, "Engineer")
person.append("Remote")
# AttributeError: 'tuple' object has no attribute 'append'
```

What remains are exactly **two** methods, both of which only *read* the tuple rather than change it:

### `.count()` — Count Occurrences of a Value

Identical in behavior to the list version from Guideline 11:

```python
numbers = (1, 2, 2, 3, 2, 4)
print(numbers.count(2))
# 3
```

### `.index()` — Find the Position of a Value

`.index()` returns the index of the **first** occurrence of a given value.

```python
numbers = (1, 2, 2, 3, 2, 4)
print(numbers.index(2))
# 1
```

`2` first appears at index `1`, so that is what is returned — even though `2` also appears at indices `2` and `4`. If the value does not exist in the tuple at all, `.index()` raises an error, the same way `.remove()` did for lists in Guideline 11:

```python
numbers.index(99)
# ValueError: tuple.index(x): x not in tuple
```

`.index()` works identically on lists too — it was simply not covered in Guideline 11, since this is the natural place to introduce it alongside its tuple counterpart. Between `.count()` and `.index()`, that is the entire method set a tuple has, and that shortness is itself informative: a tuple's whole design is built around *not* needing more than that.

---

## 12.4 Modifying a Tuple: The Simplest Way is to Turn It Into a List

Immutability is deliberate, not a limitation to be worked around casually — if a collection needs to change regularly, the right move is almost always to use a list from the start rather than fight a tuple's design. That said, there are legitimate situations where you receive a tuple (from a function that returns one, or a database row) and genuinely need to adjust one value before using it further. Direct modification fails exactly as you would now expect:

```python
person = ("Alice", 25, "Engineer")
person[1] = 26
# TypeError: 'tuple' object does not support item assignment
```

The simplest and most common workaround is a three-step conversion: turn the tuple into a list with `list()`, make the change using everything from Guideline 11, and convert it back into a tuple with `tuple()`.

```python
person = ("Alice", 25, "Engineer")

person_list = list(person)      # 1. Convert to a list
person_list[1] = 26              # 2. Modify freely, using list rules
person = tuple(person_list)      # 3. Convert back to a tuple

print(person)
# ('Alice', 26, 'Engineer')
```

This works because `list()` and `tuple()` are conversion functions (first introduced in Guideline 6) that build a new collection of the requested type from the elements of whatever is passed in. Nothing about the *original* tuple was changed — `person` on the last line is a completely new tuple, reassigned over the variable name, which is really no different in spirit from how `word = word + "!"` "changes" an immutable string in Guideline 4 by replacing it entirely rather than editing it in place.

A shorter alternative exists for the specific case of **combining** two tuples, using the `+` operator, which — like string concatenation — produces a new tuple rather than modifying either original:

```python
point = (4, 7)
point = point + (0,)     # note the trailing comma — a one-element tuple
print(point)
# (4, 7, 0)
```

Whichever approach is used, the underlying rule never changes: a tuple itself is never edited — a new tuple is built, and the old one is either discarded or replaced. That rule is not a workaround to memorize around tuples' limitations; it *is* the guarantee that makes a tuple useful in the first place.

*End of Guideline 12.*
