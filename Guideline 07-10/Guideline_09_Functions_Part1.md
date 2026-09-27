# Guideline 9: Python Functions
## Part 1

Every guideline so far has built individual pieces of logic — a conditional here, a loop there — but each one has lived inside a single, straight-through script. If Guideline 8's prime-number checker needed to run on five different numbers, the only option so far has been to copy and paste that entire loop five times, changing the number by hand each time. Functions are the answer to exactly this problem. They are the point at which a program stops being a single block of instructions and starts being a collection of named, reusable tools that call on each other.

Like Guidelines 7 and 8 before it, this is an important enough topic to split into two parts. Part 1 covers what a function actually is, the crucial and frequently misunderstood difference between a function that *returns* a value and one that merely *prints* one, and the mechanics of defining your own functions. Part 2 will build on this with parameters in more depth, default values, and further practice.

---

## 1. What is a Function?

A **function** is a named, reusable block of code that performs a specific task. You give it a name once, write the steps once, and from then on, you can run those exact steps again — as many times as you like, on as many different inputs as you like — simply by calling that name. This is the single most important idea in this guideline: a function turns "logic you wrote" into "logic you can reuse," without ever having to retype or copy it.

You have already been using functions constantly, from the very first guideline onward. `print()`, `input()`, `len()`, `range()`, `int()`, `enumerate()`, `zip()`, `any()`, `all()` — every one of these is a function. Specifically, they are **built-in functions**: ones that come packaged with Python itself, already written, tested, and ready to use the moment you start the program. You never had to write the code that makes `len()` count the characters in a string; someone else already wrote it, and Python makes it available to every program that needs it.

A **user-defined function** is one that *you* write, using the `def` keyword covered in Section 3 below, to do something specific to the program you're building — something no built-in function already covers, or something you simply want to give a clear, reusable name of your own. Guideline 8's prime-checking loop is a perfect candidate: nothing built into Python checks primality directly, so wrapping that logic into a function you define yourself is exactly the right tool.

The relationship between the two is not competitive — a user-defined function very often *calls* built-in functions inside its own body, exactly the way a script would. The difference is purely about who wrote the function and why: built-in functions solve problems general enough that Python ships them for everyone; user-defined functions solve problems specific to what your particular program needs to do.

---

## 2. Returning a Value vs. Printing

This section addresses a distinction that trips up nearly every beginner at least once, and it's worth reading slowly: **`print()` and `return` are not the same thing, and confusing them is one of the most common — and most confusing to debug — mistakes in early Python code.**

`print()` displays something to the screen, for a human to look at. It does not hand anything back to the rest of the program. `return` does the opposite: it sends a value back out of the function, to whatever code called it, so that value can be stored, reused, or passed along elsewhere — but a `return` on its own displays absolutely nothing on screen.

Consider a function that adds two numbers, written using `print()` instead of `return`:

```python
def add_and_print(a, b):
    print(a + b)

result = add_and_print(3, 4)   # this prints "7" to the screen
print(result)                  # this prints "None"
```

Run this, and you'll see `7` printed — because the `print()` line inside the function ran, exactly as written. But then `print(result)` prints `None`. This is the trap: `add_and_print()` *displayed* the sum, but it never *returned* it. Since the function has no `return` statement at all, Python automatically hands back a special value called `None` (a value you'll see again in later guidelines) to represent "nothing was explicitly returned." `result` is holding that `None`, not the number 7, even though 7 was visibly printed moments earlier. The function's visible output and its actual return value are two entirely separate things.

Now compare that to the same task written with `return`:

```python
def add(a, b):
    return a + b

result = add(3, 4)   # nothing is printed here at all
print(result)         # this prints "7"
```

Nothing appears on screen when `add(3, 4)` is called — no output whatsoever — because there is no `print()` inside `add()` at all. But `result` now genuinely holds the value `7`, because `return a + b` handed that value back out of the function and into the variable that captured it. Only the explicit `print(result)` afterward actually displays anything.

**A second, equally important detail: any line placed *after* a `return` statement inside a function is never executed.** The moment Python reaches `return`, it exits the function immediately and unconditionally — execution does not continue to the next line of that function's body under any circumstances.

```python
def example():
    print("This line runs.")
    return 5
    print("This line never runs — it's unreachable.")

example()
```

Calling `example()` prints `"This line runs."` and then stops — the second `print()`, sitting after `return`, is genuinely dead code; it will never execute no matter how many times or in what way this function is called. This is why the ordering of `print()` and `return` inside a function matters a great deal: `print()` statements that come *before* a `return` will always run when the function is called, but anything written *after* a `return` is unreachable, regardless of what that `return` sends back.

Finally, there's a subtler mix-up worth flagging directly: **`print(function_name)`** and **`print(function_name())`** do two completely different things.

```python
def add(a, b):
    return a + b

print(add)         # prints something like: <function add at 0x000001A2B3C4D5E6>
print(add(3, 4))   # prints: 7
```

`print(add)` — no parentheses after the name — prints a description of the function object itself: its name and where it lives in memory. It does not call the function at all. `print(add(3, 4))` — with parentheses and arguments — actually *calls* `add`, runs its body with `a = 3` and `b = 4`, and prints whatever comes back from its `return` statement. The parentheses are what tell Python "run this function now"; leaving them off just refers to the function as an object, without executing anything inside it.

The distinction to keep straight going forward: use `print()` when you want a human to see something on screen right now, with nothing kept for later. Use `return` when you want a value handed back so the rest of your program — an assignment, another calculation, a condition — can actually make use of it. A great many real functions use both together: `print()` for a message along the way, and `return` for the value the rest of the program actually needs.

---

## 3. Define a Function

A function is defined using the `def` keyword, followed by a name, a pair of parentheses (which may contain **parameters** — placeholder names for the values the function expects to receive), and a colon — the exact same colon-and-indented-block shape already familiar from `if` statements and loops.

```python
def function_name(parameter1, parameter2):
    # body: the code that runs every time this function is called
    return some_value
```

The names inside the parentheses at the point of *definition* are called **parameters** — they're placeholders, with no value of their own yet. The actual values supplied when the function is *called* are called **arguments**. In `add(3, 4)`, `3` and `4` are the arguments; inside `def add(a, b):`, `a` and `b` are the parameters that those arguments get assigned to for the duration of that call.

### Example: Arithmetic operations on two numbers

Each of the five arithmetic operators from Guideline 3 becomes its own small, single-purpose function:

```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    return a / b

def floor_divide(a, b):
    return a // b

x = 17
y = 5

print(add(x, y))          # 22
print(subtract(x, y))     # 12
print(multiply(x, y))     # 85
print(divide(x, y))       # 3.4
print(floor_divide(x, y)) # 3
```

Notice that `x` and `y` are defined once, but each function is called on the same pair of values without needing to repeat any of the underlying arithmetic logic — exactly the reusability described in Section 1. Change `x` and `y` to any other pair of numbers, and every single one of these five function calls immediately reflects the new values, with no other code needing to change at all.

### Example: Determining odd or even

Guideline 7 checked odd and even directly in an `if`/`else` block. As a function, the same logic becomes reusable — and it also reveals a small but genuinely useful simplification:

```python
def is_even(n):
    if n % 2 == 0:
        return True
    else:
        return False
```

This works correctly, but it's longer than it needs to be. `n % 2 == 0` is *already* an expression that evaluates directly to `True` or `False` — there's no need to test it with an `if` at all just to hand back the exact same Boolean it already produces. The entire function can be written as a single line:

```python
def is_even(n):
    return n % 2 == 0
```

Both versions behave identically for every possible input. This pattern — returning a comparison or logical expression directly, rather than routing it through an `if`/`else` that just returns `True` or `False` on either side — is worth recognizing; it comes up constantly once you're comfortable writing functions.

```python
print(is_even(4))   # True
print(is_even(7))   # False
```

### Example: Determining prime numbers

This is where functions start to show their real value. Guideline 8's prime-checking loop, wrapped in a function, can now be called on any number at all — as many times as needed — without ever being rewritten:

```python
def is_prime(number):
    if number < 2:
        return False
    for divisor in range(2, number):
        if number % divisor == 0:
            return False
    return True
```

Compare this to Guideline 8's version. There, a Boolean flag (`is_prime = True`, later set to `False`) was needed, because a loop on its own has no way to immediately hand control back to whatever code came after it — `break` only exits the loop, and the flag still had to be checked afterward. Inside a function, `return False` does something stronger: it exits the loop *and* the entire function *and* immediately hands back the answer, all in one step, the moment a divisor is found. No flag variable is needed at all, because `return` itself *is* the mechanism for reporting the result.

```python
for n in range(1, 21):
    if is_prime(n):
        print(n)
```

This prints every prime number from 1 to 20 — and note what's happening in this loop: `is_prime(n)` is called fresh on every single iteration, with a different `n` each time, and the entire multi-line checking process defined once above runs in full for each call. This is the payoff promised at the start of this guideline: the logic for checking primality was written exactly once, and it is now available to be reused for any number, in any context, for the rest of this program, without ever needing to be copied or rewritten again.
