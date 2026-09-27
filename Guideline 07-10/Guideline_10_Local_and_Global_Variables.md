# Guideline 10: Python Local and Global Variables

Guideline 9 introduced functions, and along the way, something changed quietly that this guideline now addresses directly: variables stopped living in only one place. Before functions existed, every variable in a script lived in the same single space, visible to every line of code that came after it. Once a function can define its own variables inside its own body, a new question becomes unavoidable: *where* does a variable actually live, and *who* is allowed to see it? This question is called **scope**, and it is one of the most common sources of confusing, hard-to-diagnose bugs for anyone learning functions — which is exactly why it gets a guideline of its own, immediately after functions themselves.

---

## 1. Review About Variables

Guideline 2 introduced the variable as a name that stores a value, created with a single assignment statement — `age = 25` binds the name `age` to the value `25`, and that name can be read, used in calculations, or reassigned to something else later. Every example up through Guideline 8 used variables in exactly this way, all of them living directly in the main body of the script, visible from the point they were created all the way to the end of the program.

Guideline 9 changed that. A function's body is, in a real sense, its own small script-within-a-script — and variables created inside it don't automatically behave the same way as variables created outside it. Understanding exactly how they differ is the entire subject of this guideline.

---

## 2. Local Variables

A variable created *inside* a function's body is called a **local variable**. It exists only for the duration of that one function call — it is created fresh each time the function runs, and it is destroyed the moment the function finishes, whether that's by reaching a `return` statement or simply running out of lines.

```python
def greet():
    message = "Hello!"
    print(message)

greet()
print(message)
```

```
Hello!
Traceback (most recent call last):
  ...
NameError: name 'message' is not defined
```

`greet()` runs perfectly fine on its own — `message` is created, used, and printed entirely within the function. But the moment `greet()` finishes and execution returns to the line `print(message)` outside the function, `message` no longer exists anywhere. It was never available outside `greet()` in the first place; it simply isn't part of that outer scope at all.

This applies to **parameters** too — the values passed into a function are local variables from the moment the function starts running, exactly as if they'd been created with an assignment on the first line of the body.

```python
def add(a, b):
    return a + b

print(a)
```

```
NameError: name 'a' is not defined
```

`a` and `b` exist only while `add()` is actually running — outside of it, on the line `print(a)`, they were never defined at all.

Because local variables are confined to the function that creates them, two different functions can use the exact same variable name without any conflict whatsoever — each one gets its own, completely independent copy.

```python
def function_one():
    x = 10
    print(x)

def function_two():
    x = 20
    print(x)

function_one()   # 10
function_two()   # 20
```

There is only ever one `x` alive at any given moment here — the `x` inside `function_one()` is created, used, and destroyed entirely before `function_two()` ever runs, and `function_two()`'s `x` has no knowledge that the other one ever existed. Local scope means exactly this: a variable is private to the function that made it, invisible everywhere else, including inside other functions.

---

## 3. Global Variables

A variable created *outside* of any function — directly in the main body of the script — is called a **global variable**. Unlike a local variable, it exists for as long as the program keeps running, and it can be *read* from inside any function, without needing to be passed in as a parameter at all.

```python
name = "Iris"

def greet():
    print(f"Hello, {name}!")

greet()
```

```
Hello, Iris!
```

`name` was never passed into `greet()`, and `greet()` never created its own local variable called `name` either — yet the function can still see it and use it. This works because of how Python looks up a name it encounters inside a function: it first checks whether that name is a *local* variable, created inside the current function. If it isn't, Python looks outward, to the *global* scope — the main body of the script — and uses the variable it finds there instead. Only if the name isn't found in either place does Python raise the `NameError` seen in Section 2.

This outward search is one-directional and automatic — you don't have to do anything special to *read* a global variable from inside a function. The complication, covered in the next two sections, appears only once a function tries to *change* one.

---

## 4. Global and Local with the Same Name

What happens if a function creates a local variable that happens to share its name with an existing global variable? The two do not merge, conflict, or interfere with each other in any way — the local variable simply takes priority *inside that function*, for the duration of that function's execution, while the global variable outside remains completely untouched.

```python
x = "global value"

def show_local():
    x = "local value"
    print(x)

show_local()
print(x)
```

```
local value
global value
```

Inside `show_local()`, the line `x = "local value"` does not modify the global `x` at all — it creates a brand-new, entirely separate local variable that simply happens to also be named `x`. For the rest of `show_local()`'s execution, any reference to `x` refers to this new local one, which is why `print(x)` inside the function shows `"local value"`. But the moment the function ends, that local `x` is destroyed, exactly as described in Section 2 — and the global `x`, which was never actually touched, is exactly what it always was when `print(x)` runs outside the function afterward.

This is the single most important rule to take from this guideline, because it explains a mistake that catches nearly everyone the first time they encounter it: **any assignment (`=`) to a name inside a function creates a local variable by default — even if a global variable with that exact same name already exists.** Python does not check whether you "meant" to update the global one; assigning to a name inside a function is always treated as creating (or updating) a local variable, unless you explicitly say otherwise. That "explicitly say otherwise" is the subject of the final section.

---

## 5. Modifying Global Variables Inside a Function

Given the rule from Section 4, consider what happens with a very natural-looking attempt to update a global counter from inside a function:

```python
count = 0

def increment():
    count = count + 1
    print(count)

increment()
```

```
Traceback (most recent call last):
  ...
UnboundLocalError: local variable 'count' referenced before assignment
```

This error is worth understanding precisely, because it looks contradictory at first — `count` clearly already exists, as a global variable, so why does Python complain that it's "referenced before assignment"? The answer comes directly from the rule in Section 4: because `increment()` contains the line `count = count + 1`, Python decides — for the *entire function*, before it even runs — that `count` is going to be a local variable inside `increment()`. But that same line tries to *read* `count`'s current value (the right-hand side, `count + 1`) before that local `count` has ever actually been given a value. The global `count` is sitting right there, untouched, but Python isn't looking at it — as far as `increment()` is concerned, its own local `count` doesn't exist yet, and reading a local variable that hasn't been assigned yet is exactly the error being reported.

To genuinely modify a global variable from inside a function, Python needs to be told explicitly not to treat that name as local. This is done with the **`global`** keyword, stated once at the top of the function, before the variable is used:

```python
count = 0

def increment():
    global count
    count = count + 1
    print(count)

increment()
increment()
print(count)
```

```
1
2
2
```

`global count` changes everything that follows it inside `increment()`: every reference to `count` in this function now refers directly to the real, script-wide global variable — not a new local one. `count = count + 1` reads the current global value, adds 1, and writes the result straight back into that same global variable, which is why calling `increment()` twice leaves the global `count` at `2`, and why `print(count)` outside the function correctly shows that updated value.

This is genuinely useful when a single piece of shared state — a running score, a counter, a setting — needs to be updated by more than one function across a program:

```python
score = 0

def add_points(points):
    global score
    score += points

def reset_score():
    global score
    score = 0

add_points(10)
add_points(5)
print(score)   # 15

reset_score()
print(score)   # 0
```

Both `add_points()` and `reset_score()` declare `global score`, and both can freely read and modify the exact same variable — there's only ever one `score` in this program, and every function that declares it `global` is working with that same one.

That said, `global` is a tool worth using deliberately, not by default. A program where many different functions can silently modify the same global variable becomes genuinely difficult to reason about — tracking down *which* function changed a value, and *when*, gets harder the more places are allowed to touch it. In most cases, the pattern from Guideline 9 — a function that takes what it needs as parameters and hands back a result with `return` — is the clearer and more maintainable choice; reach for `global` specifically when a value is meant to represent one shared piece of state across the whole program, updated by design from more than one place, rather than as a shortcut to avoid passing a value in or returning one out.
