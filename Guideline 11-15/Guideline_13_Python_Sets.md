# Guideline 13: Python Sets

Lists (Guideline 11) preserve order and allow duplicates. Tuples (Guideline 12) preserve order but cannot change. This guideline introduces a collection that gives up order entirely, in exchange for something neither of the other two offers automatically: **every element is guaranteed to be unique.** That collection is the **set**.

---

## 13.1 What is a Set?

A **set** is an unordered collection of **unique** values, written with **curly braces** `{ }`, with elements separated by commas.

```python
fruits = {"apple", "banana", "cherry"}
print(fruits)
# {'banana', 'apple', 'cherry'}   <- order not guaranteed
```

Two properties define a set, and both are visible immediately in that example:

- **Unordered.** The printed order does not necessarily match the order the values were written in, and it is not guaranteed to stay the same between runs. Because there is no reliable order, a set has **no indexing and no slicing** — `fruits[0]` is not a valid operation, since "the first element" is not a meaningful idea for a set.
- **No duplicates.** A set automatically discards repeated values, keeping only one copy of each:

```python
numbers = {1, 2, 2, 3, 3, 3}
print(numbers)
# {1, 2, 3}
```

Writing `2` three times and `3` three times had no effect beyond the first occurrence of each — a set simply cannot contain the same value twice, by definition.

**Creating an empty set** has a gotcha worth flagging immediately: `{}` does **not** create an empty set — it creates an empty **dictionary** (a data type covered in the next guideline). To create an empty set, the `set()` function must be used instead:

```python
empty_dict = {}
print(type(empty_dict))
# <class 'dict'>

empty_set = set()
print(type(empty_set))
# <class 'set'>
```

One more restriction follows from how sets work internally: every element of a set must be **immutable** (technically, *hashable*). Numbers, strings, and tuples are all allowed; lists are not, because a list can change after being added, which a set's internal structure cannot tolerate.

```python
bad_set = {[1, 2], [3, 4]}
# TypeError: unhashable type: 'list'
```

If a collection of collections is genuinely needed inside a set, tuples are the fix — exactly the kind of situation Guideline 12 was preparing you for.

---

## 13.2 From Lists to Sets

The single most common reason to reach for a set is to **remove duplicates** from an existing collection, and the conversion could not be more direct: pass a list into `set()`.

```python
numbers = [1, 2, 2, 3, 3, 3, 4]
unique_numbers = set(numbers)
print(unique_numbers)
# {1, 2, 3, 4}
```

This is a one-line replacement for what would otherwise require a loop and a manual "have I seen this before?" check. If the result needs to go back into an ordered, index-able form afterward, convert it back with `list()`:

```python
unique_list = list(unique_numbers)
print(unique_list)
# [1, 2, 3, 4]
```

Be aware that the order of `unique_list` is **not guaranteed** to match the original list's order — converting to a set and back does deduplicate reliably, but it does not preserve position. If both deduplication *and* original order matter, a set alone is not the right tool (a common workaround, once dictionaries are introduced in the next guideline, uses the fact that dictionary keys preserve insertion order in modern Python — but that is a topic for later).

---

## 13.3 Built-in Methods for Sets: `len()`, `sum()`, `max()`, `min()`

The same aggregate functions from Guideline 11 work on sets exactly as they do on lists, since none of them depend on order — they simply need something to count or compare.

```python
scores = {88, 92, 75, 100, 60}

print(len(scores))   # 5    -> number of unique elements
print(sum(scores))   # 415  -> total of all elements
print(max(scores))   # 100  -> largest value
print(min(scores))   # 60   -> smallest value
```

`len()` on a set has one extra layer of meaning worth noting: because duplicates are automatically removed, `len(set(some_list))` is a quick, reliable way to count **how many distinct values** a list contains, which is a subtly different question than `len(some_list)` alone can answer.

```python
grades = ["A", "B", "A", "C", "A", "B"]
print(len(grades))            # 6  -> total entries
print(len(set(grades)))       # 3  -> distinct grades used
```

---

## 13.4 Checking Membership with `in`

Testing whether a value exists in a set uses the same `in` operator you have already used with lists and tuples.

```python
fruits = {"apple", "banana", "cherry"}
print("apple" in fruits)     # True
print("mango" in fruits)     # False
```

The behavior looks identical to a list's `in` check, but the reason to prefer a set specifically for this task is worth understanding: internally, a set is built on a structure (a hash table) that can check for membership almost instantly, regardless of how large the set is. A list, by contrast, may need to check every single element one by one before it can be sure a value is absent. For a small collection the difference is invisible; for a collection with thousands or millions of elements, checking membership with a set can be dramatically faster than checking with a list. This is the second major reason (alongside automatic deduplication) that sets exist as their own data type rather than being replaced entirely by lists.

---

## 13.5 Adding an Element: `.add()`

Because a set has no order and therefore no positions, there is no `.append()` or `.insert()` — there is nowhere to specify "at the end" or "at index 2" for a collection that has neither. Instead, a set has exactly one way to add a single element: `.add()`, which places the value into the set with no position implied at all.

```python
fruits = {"apple", "banana"}
fruits.add("cherry")
print(fruits)
# {'apple', 'banana', 'cherry'}
```

Adding a value that already exists has no effect and raises no error — it simply confirms what a set already guarantees:

```python
fruits.add("apple")
print(fruits)
# {'apple', 'banana', 'cherry'}   <- unchanged
```

---

## 13.6 Removing Elements: `.clear()`, `.discard()`, `.pop()`, and `.remove()`

Sets offer four ways to remove elements, and each answers a slightly different situation.

### `.clear()` — Remove Everything

Exactly like `list.clear()` from Guideline 11, `.clear()` empties the entire set, permanently and without confirmation.

```python
fruits = {"apple", "banana", "cherry"}
fruits.clear()
print(fruits)
# set()
```

(Notice `print()` shows an empty set as `set()`, not `{}` — a direct consequence of the empty-set gotcha from Section 13.1.) The same caution from Guideline 11 applies here without modification: `.clear()` is irreversible, so only call it on a set you have deliberately decided to empty, and copy it first (`backup = fruits.copy()`) if the contents might still be needed.

### `.discard()` — Remove if Present, No Error if Not

`.discard()` removes a value if it exists, and does **nothing at all** — no error — if it does not.

```python
fruits = {"apple", "banana", "cherry"}
fruits.discard("banana")
print(fruits)
# {'apple', 'cherry'}

fruits.discard("mango")   # "mango" was never in the set
print(fruits)
# {'apple', 'cherry'}   <- no error, no change
```

### `.remove()` — Remove, but Raise an Error if Not Present

`.remove()` behaves like `.discard()` when the value exists, but raises an error when it does not — mirroring the list method with the same name from Guideline 11.

```python
fruits = {"apple", "banana", "cherry"}
fruits.remove("banana")
print(fruits)
# {'apple', 'cherry'}

fruits.remove("mango")
# KeyError: 'mango'
```

**Choosing between the two:** use `.discard()` when it is fine for the value to simply not be there; use `.remove()` when the value *should* be present, and its absence would itself indicate a bug worth knowing about immediately.

### `.pop()` — Remove and Return an Arbitrary Element

`.pop()` removes and returns **one** element from the set — but because a set has no order, there is no index to give it, and no way to control *which* element gets removed.

```python
fruits = {"apple", "banana", "cherry"}
removed = fruits.pop()
print(removed)    # could be any one of the three elements
print(fruits)      # the remaining two
```

This is the clearest practical consequence of "unordered" in this entire guideline: `list.pop()` without an argument reliably removes the *last* element, because a list has a defined last position. `set.pop()` has no such guarantee — it removes whichever element the set's internal structure happens to produce first, which should be treated as effectively random. `.pop()` on a set is appropriate when you need to remove and use "some element" and genuinely do not care which one; it is the wrong tool whenever a specific element needs to be targeted (use `.discard()` or `.remove()` instead).

---

## 13.7 Updating a Set

`.update()` adds **multiple** elements to a set at once, from any iterable — a list, a tuple, or another set — merging in the new values while automatically discarding any duplicates.

```python
fruits = {"apple", "banana"}
fruits.update(["cherry", "date", "apple"])
print(fruits)
# {'apple', 'banana', 'cherry', 'date'}
```

Notice that `"apple"` appeared in the list passed to `.update()`, but it did not create a duplicate — it was simply absorbed, since the set already contained it. This is the set equivalent of `list.extend()` from Guideline 11: both add several values at once rather than one, but `.update()` carries the set's uniqueness guarantee along with it automatically.

---

## 13.8 Checking Two Sets: `difference()`, `intersection()`, `union()`, `isdisjoint()`, `issubset()`, `issuperset()`

This is where sets earn their keep beyond deduplication and fast membership checks: sets support the mathematical operations you may already associate with the word "set" from other contexts — comparing two collections to find what they share, what they don't, and how they relate.

Consider two sets of students enrolled in two different clubs:

```python
art_club = {"Amy", "Ben", "Cody", "Dina"}
music_club = {"Cody", "Dina", "Evan", "Fay"}
```

### `.union()` — Everything in Either Set

Returns a new set containing every element that appears in **at least one** of the two sets, with duplicates naturally collapsed.

```python
print(art_club.union(music_club))
# {'Amy', 'Ben', 'Cody', 'Dina', 'Evan', 'Fay'}
```

The `|` operator does the same thing: `art_club | music_club`.

### `.intersection()` — Only What's in Both

Returns a new set containing only the elements that appear in **both** sets.

```python
print(art_club.intersection(music_club))
# {'Cody', 'Dina'}
```

The `&` operator is the shorthand equivalent: `art_club & music_club`.

### `.difference()` — What's in One but Not the Other

Returns a new set containing the elements that are in the **first** set but **not** in the second. Order matters here — `A.difference(B)` and `B.difference(A)` generally give different results.

```python
print(art_club.difference(music_club))
# {'Amy', 'Ben'}   -> in art_club but not in music_club

print(music_club.difference(art_club))
# {'Evan', 'Fay'}   -> in music_club but not in art_club
```

The `-` operator works the same way: `art_club - music_club`.

### `.isdisjoint()` — True if the Sets Share Nothing at All

Returns `True` if the two sets have **no elements in common**, and `False` otherwise.

```python
print(art_club.isdisjoint(music_club))
# False   -> they share Cody and Dina

drama_club = {"Gina", "Hugo"}
print(art_club.isdisjoint(drama_club))
# True   -> no overlap at all
```

### `.issubset()` — Is Every Element of This Set Also in Another?

Returns `True` if **every** element of the calling set also appears in the set passed as the argument.

```python
advanced_art = {"Amy", "Ben"}
print(advanced_art.issubset(art_club))
# True   -> Amy and Ben are both in art_club
```

### `.issuperset()` — Does This Set Contain Every Element of Another?

`.issuperset()` is the mirror image of `.issubset()`: it returns `True` if the calling set contains **all** of the elements of the set passed in.

```python
print(art_club.issuperset(advanced_art))
# True   -> art_club contains everything advanced_art has
```

In fact, for any two sets `A` and `B`, `A.issubset(B)` and `B.issuperset(A)` always agree — they are two ways of asking the exact same question, from opposite directions. Choosing between them in your own code usually comes down to which set reads more naturally as the subject of the sentence: "is `advanced_art` a subset of `art_club`?" versus "does `art_club` cover everything in `advanced_art`?" — the same fact, stated from either side.

---

Between deduplication, fast membership testing, and this final set of comparison operations, a set solves a specific class of problem — "what's unique," "what's shared," "what's different" — more directly and more efficiently than a list or tuple ever could. Recognizing *when* a problem is actually a set problem, rather than reflexively reaching for a list, is the real skill this guideline builds toward.

*End of Guideline 13.*
