# Guideline 15: Python Strings

Strings have already appeared in almost every guideline so far — as a data type in Guideline 4, combined and formatted in Guideline 5, passed through built-in functions in Guideline 6. What none of those guidelines did was treat the string as a **collection** in its own right — something you can index into, slice, search, split apart, and rebuild, the same way Guidelines 11 through 14 treated lists, tuples, sets, and dictionaries. This guideline does exactly that. It is long because a string is one of the most heavily used data types in any real program, and the operations covered here — searching, splitting, joining, validating — are the ones you will reach for constantly from this point forward.

---

## 15.1 Strings in Python

A **string** is a sequence of characters, written between single quotes, double quotes, or triple quotes (Guideline 4 and Guideline 5), and — this is the property that shapes almost everything in this guideline — it is **immutable**. Once created, a string's characters cannot be changed in place; every operation that appears to "modify" a string actually builds and returns a **new** string, leaving the original untouched.

```python
greeting = "Hello, World!"
print(type(greeting))
# <class 'str'>
```

What matters for this guideline specifically is that a string is a **sequence** — an ordered collection of individual characters — in exactly the same structural sense that a list is an ordered collection of elements. This is not a loose comparison: a string supports `len()`, indexing, slicing, iteration with a `for` loop, and the `in` operator, using the *exact same syntax* as a list. The difference is that a list's elements can be anything and can change; a string's elements are always single characters, and can never change once the string exists. Everything from Section 15.4 onward builds directly on this sequence-of-characters view.

---

## 15.2 Strings as Variables: Converting Numbers to Strings, but Not Always the Other Way

A string can hold anything that looks like text — including text that looks like a number:

```python
age_text = "25"
print(type(age_text))
# <class 'str'>
```

`age_text` is a string, full stop — Python does not treat `"25"` as secretly numeric just because it is made of digits. This distinction matters because of an asymmetry worth understanding clearly:

**Converting a number to a string always works.** `str()` can turn *any* number into its text representation, with no exceptions:

```python
print(str(25))       # '25'
print(str(3.14))     # '3.14'
print(str(-7))        # '-7'
```

**Converting a string to a number only works if the string actually represents one.** `int()` and `float()` (first introduced in Guideline 2) will raise an error the moment the text does not look like a valid number:

```python
print(int("25"))       # 25       -> works, "25" is a valid integer
print(float("3.14"))   # 3.14     -> works, "3.14" is a valid float

print(int("twenty-five"))
# ValueError: invalid literal for int() with base 10: 'twenty-five'
```

There is a second, more subtle trap in this same direction: `int()` cannot convert a string that looks like a **decimal** number, even though the value would fit:

```python
print(int("3.14"))
# ValueError: invalid literal for int() with base 10: '3.14'
```

To convert a decimal-looking string into an integer, it must go through `float()` first, and then `int()` on the resulting number:

```python
value = int(float("3.14"))
print(value)
# 3
```

The lesson to carry forward: `str()` is a one-way street that never fails — any number can always become text. Going back the other direction is conditional on the text actually representing a valid number, and this is exactly the kind of situation Guideline 7's `if`/`else` logic (often paired with the validation methods in Section 15.12) is used to guard against before a conversion is even attempted.

---

## 15.3 Addition and Multiplication on Strings: `"1" + "1"` = `"11"`??

Guideline 3 introduced `+` as arithmetic addition, and Guideline 5 used it to combine strings. Placing both uses side by side reveals something that trips up nearly every beginner at least once:

```python
print(1 + 1)
# 2

print("1" + "1")
# 11
```

Both lines use the same `+` symbol, and both produce a two-character result on screen — but they are doing completely different things. `1 + 1` is numeric addition: two integers combine into the single integer `2`. `"1" + "1"` is **string concatenation**: two separate one-character strings are joined end to end into the two-character string `"11"`. Nothing was added in the arithmetic sense at all — the operator's behavior is determined entirely by the type of the values on either side of it, a concept sometimes called *operator overloading*.

This is precisely why mixing a string and a number with `+` fails outright, rather than trying to guess which behavior you meant:

```python
age = 25
print("Age: " + age)
# TypeError: can only concatenate str (not "int") to str
```

Guideline 5's fix — converting the number explicitly with `str()` — is the direct application of Section 15.2's rule: `str(age)` always works, so `"Age: " + str(age)` produces the intended `"Age: 25"` without ambiguity.

**Multiplication with `*`** behaves the same way Guideline 11 showed for lists: a string multiplied by an integer repeats the string that many times, building a new, longer string.

```python
print("ab" * 3)
# ababab

border = "-" * 20
print(border)
# --------------------
```

`border` is a quick, common idiom for drawing a visual divider of a specific width without typing out twenty dashes by hand — the exact same idea behind `[0] * 5` from Guideline 11, applied to characters instead of list elements.

---

## 15.4 Indexing and Slicing on Strings

Because a string is a sequence of characters (Section 15.1), indexing and slicing work **exactly** as they did for lists in Guideline 11 — same zero-based counting, same negative indices, same `[start:stop:step]` slice syntax.

```python
word = "Python"
#        0 1 2 3 4 5
#       -6-5-4-3-2-1

print(word[0])      # P
print(word[-1])      # n
print(word[2:5])     # tho
print(word[::-1])    # nohtyP  -> reversed, same trick as Guideline 11
```

The one rule that does **not** carry over from lists is item assignment, precisely because of the immutability established in Section 15.1:

```python
word = "Python"
word[0] = "J"
# TypeError: 'str' object does not support item assignment
```

To "change" a character, a new string must be built — commonly by slicing around the part you want to replace and concatenating the replacement in:

```python
word = "Python"
new_word = "J" + word[1:]
print(new_word)
# Jython
```

`word` itself is never altered; `new_word` is an entirely separate string. This is the same pattern first seen with numeric strings back in Guideline 4's discussion of immutability, now applied deliberately.

---

## 15.5 Comparing Strings

Strings support the same comparison operators as numbers (`==`, `!=`, `<`, `>`, `<=`, `>=`), but the comparison is not numeric — it is based on the underlying character codes (`ord()`, from Guideline 6), compared position by position, the way words are ordered in a dictionary.

```python
print("apple" == "apple")   # True
print("apple" == "Apple")   # False  -> case matters!
print("apple" != "banana")  # True

print("apple" < "banana")   # True  -> 'a' comes before 'b'
print("Zebra" < "apple")    # True  -> uppercase letters have lower character codes than lowercase
```

That last example is the one worth pausing on: `"Zebra" < "apple"` is `True` not because `Z` is alphabetically "before" `a` in any everyday sense, but because every uppercase letter's character code is numerically lower than every lowercase letter's code (recall `ord("Z")` is smaller than `ord("a")`). This is why comparing user input directly is risky — `"Apple" == "apple"` being `False` will silently reject input that a human would consider identical. The standard fix is to normalize case before comparing, using the methods in Section 15.10:

```python
user_input = "Apple"
if user_input.lower() == "apple":
    print("Match!")
# Match!
```

---

## 15.6 Substrings

A **substring** is simply any contiguous piece of a larger string. The most common question asked about a substring is not "what is it" but "does it exist somewhere inside this larger string" — and that question is answered with the same `in` operator already used for list and set membership in Guidelines 11 and 13.

```python
sentence = "the quick brown fox"
print("quick" in sentence)     # True
print("slow" in sentence)      # False
```

`in` on a string checks for the substring **anywhere** inside it, not just as a whole-word match — `"qui" in sentence` is also `True`, since `"qui"` genuinely appears as a contiguous run of characters inside `"quick"`:

```python
print("qui" in sentence)
# True
```

Extracting a specific substring, once you know where it lives, is exactly the slicing from Section 15.4:

```python
sentence = "the quick brown fox"
print(sentence[4:9])
# quick
```

Section 15.11 (Searching) picks up from here, covering how to find *where* a substring is located when you do not already know its position.

---

## 15.7 Joining a List into a String: `''.join()`

`.join()` runs in the opposite direction from splitting apart (Section 15.8): it takes an iterable of strings — usually a list — and combines them into a **single** string, inserting a chosen separator between each piece. The syntax places the separator first, as the string the method is called on:

```python
words = ["Python", "is", "fun"]
sentence = " ".join(words)
print(sentence)
# Python is fun
```

`" ".join(words)` reads naturally once you notice the separator is the string *before* the dot: join these words together, using a single space between each one. Any string can serve as the separator:

```python
csv_row = ",".join(["Alice", "25", "Engineer"])
print(csv_row)
# Alice,25,Engineer

no_separator = "".join(["a", "b", "c"])
print(no_separator)
# abc
```

`"".join(...)` — an empty string as the separator — is a common and idiomatic way to rebuild a single string out of a list of individual characters, with nothing inserted between them at all.

One firm requirement: every element being joined must already be a string. Joining a list that contains numbers fails directly, and the fix is exactly Section 15.2's rule applied inside a comprehension (Guideline 11):

```python
numbers = [1, 2, 3]
print(", ".join(numbers))
# TypeError: sequence item 0: expected str instance, int found

print(", ".join([str(n) for n in numbers]))
# 1, 2, 3
```

---

## 15.8 Splitting Strings: `.split()` for Multiple Inputs

`.split()` does the reverse of `.join()`: it breaks a single string apart into a **list** of pieces, based on a separator. Called with no argument, it splits on any whitespace (spaces, tabs, newlines) and automatically ignores extra whitespace between words:

```python
sentence = "the quick brown fox"
words = sentence.split()
print(words)
# ['the', 'quick', 'brown', 'fox']
```

A specific separator can be given explicitly:

```python
csv_row = "Alice,25,Engineer"
fields = csv_row.split(",")
print(fields)
# ['Alice', '25', 'Engineer']
```

**One of the most practically useful applications of `.split()`** is reading **multiple values from a single line of input**, revisiting `input()` from Guideline 2. Instead of calling `input()` several times for several numbers, a user can type them all on one line, separated by spaces, and `.split()` breaks them apart in one step:

```python
line = input("Enter two numbers separated by a space: ")
# user types: 12 7
first, second = line.split()
print(first, second)
# 12 7
```

Notice that `first` and `second` are still **strings** at this point — `.split()` only separates text, it does not convert types. Combining this with `map()` (Guideline 6) and Section 15.2's `int()` conversion produces the idiom you will see constantly in practice, converting every split piece to a number in one line:

```python
line = input("Enter two numbers separated by a space: ")
# user types: 12 7
first, second = map(int, line.split())
print(first + second)
# 19
```

`line.split()` produces `['12', '7']`; `map(int, ...)` applies `int()` to each piece; and unpacking the result into `first, second` gives you two ready-to-use integers from a single line of user input.

---

## 15.9 Replacing Strings

`.replace(old, new)` returns a new string with every occurrence of `old` swapped out for `new` — and, consistent with Section 15.1, the original string is never modified.

```python
sentence = "the quick brown fox"
new_sentence = sentence.replace("quick", "slow")
print(new_sentence)
# the slow brown fox
print(sentence)
# the quick brown fox   <- unchanged
```

An optional third argument limits how many occurrences are replaced, starting from the beginning of the string:

```python
text = "cat cat cat cat"
print(text.replace("cat", "dog", 2))
# dog dog cat cat
```

Only the first two occurrences of `"cat"` were replaced — the third and fourth were left alone, since the count argument was `2`.

---

## 15.10 Case in Strings: `.upper()`, `.lower()`, and `.capitalize()`

Three methods adjust the letter case of a string, each returning a new string (again, per Section 15.1) rather than modifying the original.

```python
text = "Hello World"

print(text.upper())          # HELLO WORLD
print(text.lower())           # hello world
print(text.capitalize())      # Hello world
```

`.upper()` and `.lower()` convert every letter in the string, and are the standard tool for **case-insensitive comparison**, exactly as shown in Section 15.5. `.capitalize()` behaves differently from both: it capitalizes only the **first character** of the entire string and forces every other character to lowercase — including letters that were already capitalized elsewhere in the string.

```python
print("HELLO WORLD".capitalize())
# Hello world
```

Two closely related methods are worth knowing even though they were not explicitly listed above: `.title()` capitalizes the first letter of **every** word (`"hello world".title()` → `"Hello World"`), and `.swapcase()` flips every character's case at once (`"Hello".swapcase()` → `"hELLO"`). Both follow the same "returns a new string" rule as everything else in this section.

---

## 15.11 Searching: `.find()`, `.startswith()`, `.endswith()`

Section 15.6 answered "does this substring exist?" with `in`. This section answers the next question: "**where** is it, or does the string begin or end with it?"

### `.find()` — Locate a Substring's Position

`.find()` returns the **index** where a substring first appears, or `-1` if it does not appear at all.

```python
sentence = "the quick brown fox"
print(sentence.find("quick"))
# 4

print(sentence.find("slow"))
# -1
```

This is a deliberately different failure behavior from `.index()` — strings also have an `.index()` method, working identically to `.find()` when the substring **is** found, but raising a `ValueError` instead of returning `-1` when it is not:

```python
print(sentence.index("slow"))
# ValueError: substring not found
```

This is the same `.discard()`-versus-`.remove()` choice from Guideline 13's set methods, applied to strings: use `.find()` when a missing substring is a normal, expected possibility that your code should handle gracefully (typically by checking `if sentence.find(x) != -1:`); use `.index()` when the substring's absence would itself indicate something has gone wrong.

### `.startswith()` and `.endswith()` — Check the Edges of a String

These two methods check whether a string begins or ends with a specific substring, returning a plain `True` or `False` — ideal for use directly inside an `if` statement (Guideline 7).

```python
filename = "report_2026.pdf"

print(filename.startswith("report"))    # True
print(filename.endswith(".pdf"))         # True
print(filename.endswith(".docx"))        # False
```

This pattern — checking a file extension, a URL prefix (`"https://"`), or a command keyword at the start of a line — is one of the most common real-world uses of string methods in this entire guideline.

---

## 15.12 Validating a String: `.count()`, `.isalpha()`, `.isdigit()`, `.isalnum()`

These four methods are especially useful **inside conditions** (Guideline 7), because each answers a specific yes/no or how-many question about a string's contents — exactly the kind of check that belongs directly in an `if` statement before trusting a piece of data.

### `.count()` — How Many Times Does a Substring Appear?

```python
sentence = "the quick brown fox jumps over the lazy dog"
print(sentence.count("the"))
# 2

print(sentence.count("o"))
# 4
```

Unlike `list.count()` from Guideline 11, which only matches whole elements, `str.count()` counts **overlapless occurrences of a substring**, which can be more than one character long, as `"the"` shows above.

### `.isalpha()` — Is Every Character a Letter?

Returns `True` only if the string is non-empty and every character is an alphabetic letter — no digits, no spaces, no punctuation.

```python
print("Python".isalpha())     # True
print("Python3".isalpha())    # False  -> contains a digit
print("Py thon".isalpha())    # False  -> contains a space
```

### `.isdigit()` — Is Every Character a Digit?

Returns `True` only if every character is a numeric digit — this is the check that directly protects Section 15.2's `int()` conversion from raising a `ValueError`:

```python
user_input = "25"
if user_input.isdigit():
    age = int(user_input)
    print(f"Age recorded: {age}")
else:
    print("That doesn't look like a valid age.")
# Age recorded: 25
```

Note that `.isdigit()` returns `False` for a negative number or a decimal, since `-` and `.` are not digits themselves — `"-5".isdigit()` and `"3.14".isdigit()` are both `False`, which is worth testing for directly if your program needs to accept those cases too.

### `.isalnum()` — Is Every Character a Letter or a Digit?

Returns `True` if every character is either alphabetic or a digit — no spaces, no punctuation, no symbols — making it a natural fit for validating something like a username.

```python
print("Player123".isalnum())    # True
print("Player_123".isalnum())    # False  -> underscore is not alphanumeric
print("Player 123".isalnum())    # False  -> space is not alphanumeric
```

Together, these four methods turn "is this input actually usable?" from a guess into a direct, checkable condition — validating a string **before** acting on it, rather than discovering the problem only when a conversion or an operation fails partway through the program.

*End of Guideline 15.*
