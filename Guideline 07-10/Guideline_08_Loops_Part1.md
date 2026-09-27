# Guideline 8: Loops
## Part 1

Guideline 7 gave your programs the ability to make a decision. But a decision, on its own, only ever gets made once per run — the `if` checks its condition a single time and moves on. A great deal of real programming isn't about deciding once; it's about *repeating* something, often a great many times, without having to type out the same lines of code over and over by hand. That repetition is what this guideline is about. Loops are, alongside conditionals, the second fundamental building block of control flow — the two of them together account for almost everything a program actually "does" beyond straight-line execution.

Like Guideline 7, this one is split into two parts. Part 1 covers what a loop actually is, the two loop structures Python provides — `while` and `for` — and what happens when you place one loop inside another. Part 2 will build on this with the tools for controlling a loop's behavior more precisely.

---

## 1. What is a Loop? (Also Known as Iteration)

A **loop** is a block of code that repeats — automatically, and as many times as needed — without you having to write that code out again for every repetition. Each single pass through the loop's body is called an **iteration**. A loop that runs five times has completed five iterations by the time it finishes.

Consider the difference directly. Printing the numbers 1 through 5 without a loop looks like this:

```python
print(1)
print(2)
print(3)
print(4)
print(5)
```

This works, but it doesn't scale. Printing 1 through 500 this way would mean writing five hundred lines of near-identical code, and printing "however many numbers the user asks for" wouldn't be possible at all, since you don't know the number of lines to write in advance. A loop solves both problems at once: it expresses the *pattern* of the repetition rather than each individual repetition.

It's worth being precise about how a loop differs from the conditional statements in Guideline 7, because on the surface both involve checking a condition:

- An **`if` statement** checks a condition once. Based on the result, it runs a block of code either zero times or one time, and then execution moves on permanently — it never returns to check that same condition again.
- A **loop** checks a condition (or works through a sequence) and, as long as the condition holds (or the sequence has items left), keeps running the same block of code again and again — zero times, once, or an unbounded number of times, depending entirely on what's being checked.

Python gives you two distinct loop structures, and the choice between them almost always comes down to one question: **do you know in advance how many times you need to repeat, or are you repeating until some condition changes?** The **`while` loop** is built for the second case. The **`for` loop** is built for the first.

---

## 2. The While Loop

A `while` loop repeats its body for as long as a condition remains `True`. The condition is written using exactly the same comparison and logical operators from Guidelines 3 and 7 — a `while` loop is, in a real sense, an `if` statement that keeps re-checking itself instead of only checking once.

```python
count = 1

while count <= 5:
    print(count)
    count += 1
```

Trace through this carefully, because every `while` loop follows the same rhythm:

1. Python checks the condition, `count <= 5`. It's `True` (1 is less than or equal to 5), so the body runs.
2. The body prints `count`, then increments it by 1 using `count += 1` (shorthand for `count = count + 1`, from the assignment operators covered in Guideline 3).
3. Python goes back to step 1 and checks the condition *again*, with `count` now equal to 2.
4. This repeats — 2, 3, 4, 5 all print — until `count` becomes 6. At that point, `count <= 5` evaluates to `False`, the loop stops, and execution continues with whatever comes after it.

**The line that updates the loop's condition variable is not optional.** If `count += 1` were left out, `count` would stay at 1 forever, `count <= 5` would never become `False`, and the loop would never stop. This is called an **infinite loop**, and it is one of the single most common bugs a beginner writes — not because the logic of the condition is wrong, but because nothing inside the loop body ever moves the program toward making that condition false. Every `while` loop you write should be checked against one question: *is there a line inside this loop that eventually makes the condition false?* If the answer is no, the loop doesn't end.

`while` loops are the right tool specifically when the number of repetitions isn't known ahead of time — it depends on something that can only be discovered while the program is running. A very common shape for this is **input validation**: keep asking the user for something until they provide a value that's actually acceptable.

```python
password = input("Enter a password (at least 8 characters): ")

while len(password) < 8:
    print("Too short. Try again.")
    password = input("Enter a password (at least 8 characters): ")

print("Password accepted.")
```

There's no way to know in advance how many times the user will get this wrong — zero times, if they get it right immediately, or many times, if they keep entering short passwords. The loop simply keeps running until the condition (`len(password) < 8`) becomes false on its own, driven by the user's actual input rather than a fixed counter.

---

## 3. The For Loop

Where a `while` loop repeats based on a condition, a `for` loop repeats based on a **sequence** — it takes something that can be stepped through, one item at a time, and runs its body once for each item, automatically, until the sequence is exhausted. This is called **definite iteration**, because the number of repetitions is fixed by the length of the sequence rather than by a condition that might change unpredictably.

The most common sequence to loop over, especially before lists are introduced in a later guideline, is the one produced by the built-in `range()` function. `range()` generates a sequence of whole numbers, and it comes in three forms:

| Form | Produces | Example | Result |
|---|---|---|---|
| `range(stop)` | 0 up to (not including) `stop` | `range(5)` | `0, 1, 2, 3, 4` |
| `range(start, stop)` | `start` up to (not including) `stop` | `range(2, 6)` | `2, 3, 4, 5` |
| `range(start, stop, step)` | `start` up to `stop`, counting by `step` | `range(0, 10, 2)` | `0, 2, 4, 6, 8` |

Notice that `stop` is never actually included — `range(5)` produces five numbers, `0` through `4`, not `0` through `5`. This trips up almost everyone the first time, so it's worth deliberately remembering: `range()` counts *up to, but not including,* its stopping value.

Here is the same 1-to-5 example from the `while` loop section, written as a `for` loop instead:

```python
for count in range(1, 6):
    print(count)
```

This is shorter, and it also removes an entire category of possible mistakes: there's no separate variable to manually increment, and no way to accidentally write an infinite loop, because `range()` always produces a fixed, finite sequence of values up front. `count` is automatically set to each value in that sequence, one at a time, for exactly as many iterations as the sequence contains — five, in this case — and then the loop ends on its own.

A `for` loop isn't limited to `range()`. Anything Python considers **iterable** — meaning it can be stepped through item by item — can be looped over directly. A string, for instance, is iterable: looping over one steps through its characters one at a time.

```python
word = "Python"

for letter in word:
    print(letter)
```

This prints each character of `"Python"` on its own line — `P`, `y`, `t`, `h`, `o`, `n` — without needing `range()` or `len()` at all. The loop simply asks the string, "what's your next character?" until there are none left.

The decision between `while` and `for` comes down to the question posed at the start of this guideline. If you're stepping through a known sequence, or repeating a specific, calculable number of times, reach for `for`. If you're repeating until some condition — often one that depends on user input or a value that changes unpredictably during the loop — becomes false, reach for `while`.

---

## 4. Nested Loops

Just as an `if` statement can be placed inside another `if` statement (Guideline 7's nested conditionals), a loop can be placed entirely inside the body of another loop. This is called a **nested loop**, and the behavior is precise: for *every single iteration* of the outer loop, the entire inner loop runs from start to finish before the outer loop is allowed to move to its next iteration.

```python
for row in range(1, 4):
    for column in range(1, 4):
        print(f"Row {row}, Column {column}")
```

Trace through what actually happens: the outer loop starts with `row = 1`. Before `row` ever advances to 2, the *entire* inner loop runs to completion — `column` takes the values 1, 2, and 3, printing three lines. Only once the inner loop has fully finished does control return to the outer loop, which then advances to `row = 2`, and the inner loop runs all the way through again from `column = 1`. The total number of times the innermost line runs is the outer count multiplied by the inner count — here, 3 × 3 = 9 lines in total.

A classic, practical use of nested loops is generating a multiplication table:

```python
for i in range(1, 6):
    for j in range(1, 6):
        print(f"{i} x {j} = {i * j}")
    print()
```

The outer loop fixes one number (`i`) at a time; the inner loop runs through every value of `j` for that fixed `i`, printing a full row of the table before the blank `print()` (using no arguments, which simply prints an empty line — a detail from Guideline 2's `print()` coverage) separates it from the next row.

Nested loops also produce simple visual patterns, which is a common and genuinely useful exercise for getting comfortable with how the rows and columns of a nested loop actually correspond to what gets printed:

```python
rows = 5

for i in range(1, rows + 1):
    for j in range(i):
        print("*", end="")
    print()
```

Here, the outer loop's variable `i` doesn't just control repetition — it directly determines how many times the inner loop runs on each pass, since `range(i)` grows as `i` grows. On the first outer iteration (`i = 1`), the inner loop runs once, printing a single `*`. On the second (`i = 2`), the inner loop runs twice, printing `**`. By the fifth outer iteration, the inner loop runs five times, printing `*****`. The `end=""` argument to `print()` (from Guideline 2) is doing essential work here — without it, each `*` would be printed on its own line instead of accumulating on a single row before the `print()` at the end of the outer loop moves to the next line. The result is a left-aligned triangle of asterisks, five rows tall, growing by one character per row — a direct, visible demonstration of exactly how the outer and inner loop counts relate to each other.

Nesting can go deeper than two levels, though it's worth being aware that each additional level multiplies the total number of iterations by whatever that level's range covers — a loop nested three levels deep, each running ten times, runs its innermost line 10 × 10 × 10 = 1,000 times. This isn't a reason to avoid nesting, but it is a reason to be deliberate about it, and it's a consideration this book will return to later when the cost of running code is discussed more formally.
