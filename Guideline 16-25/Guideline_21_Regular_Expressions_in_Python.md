# Guideline 21: Regular Expressions in Python

A **regular expression** (regex) is a small, specialized language for describing *patterns* in text — not a specific string, but a shape a string might take: "a sequence of digits," "a word that starts with a capital letter," "anything between two quotation marks." Regex isn't unique to Python — it's a concept shared across nearly every programming language and even many text editors — but in Python it lives in the standard library's **`re` module**.

```python
import re
```

Regex earns a reputation for looking cryptic, and honestly, it deserves some of that reputation — a dense pattern can look like line noise until you know how to read it. This guideline is built almost entirely as reference tables for exactly that reason: regex is less about memorizing a workflow and more about recognizing symbols, one at a time, until the whole pattern reads clearly.

---

## 1. Common Regex Functions

| Function | Description | Example |
|---|---|---|
| `re.match(pattern, string)` | Checks for a match only at the **beginning** of the string | `re.match(r"Hi", "Hi there")` → match found |
| `re.search(pattern, string)` | Scans the **entire** string, returns the first match found anywhere | `re.search(r"there", "Hi there")` → match found |
| `re.fullmatch(pattern, string)` | Matches only if the **entire** string fits the pattern, start to end | `re.fullmatch(r"\d+", "12345")` → match found |
| `re.findall(pattern, string)` | Returns a **list** of all non-overlapping matches | `re.findall(r"\d+", "a1 b22 c3")` → `['1', '22', '3']` |
| `re.finditer(pattern, string)` | Same as `findall()`, but returns an **iterator of match objects** instead of plain strings — useful when you need each match's position too | `for m in re.finditer(r"\d+", "a1 b22"): print(m.group())` |
| `re.sub(pattern, replacement, string)` | Replaces all matches with a replacement string | `re.sub(r"\d+", "#", "a1 b22")` → `'a# b#'` |
| `re.split(pattern, string)` | Splits a string wherever the pattern matches | `re.split(r"\s*,\s*", "a, b,c")` → `['a', 'b', 'c']` |
| `re.compile(pattern)` | Pre-compiles a pattern into a reusable pattern object — faster when the same pattern is used repeatedly | `p = re.compile(r"\d+"); p.findall("a1 b2")` |

A `match` object (returned by `match()`, `search()`, and `fullmatch()`) isn't the matched text itself — it's an object you call `.group()` on to retrieve the actual matched string, or `None` if nothing matched at all. This is why checking `if re.search(...)` works as a truthiness test, but `re.search(...).group()` will crash with an `AttributeError` if there was no match to begin with.

---

## 2. Special Characters

These symbols don't match themselves literally — each one carries special meaning inside a pattern.

| Character | Meaning | Example |
|---|---|---|
| `.` | Matches any single character except a newline | `c.t` matches `"cat"`, `"cot"`, `"c!t"` |
| `^` | Anchors a match to the **start** of the string (or line, with the `MULTILINE` flag) | `^Hi` matches `"Hi there"` but not `"Say Hi"` |
| `$` | Anchors a match to the **end** of the string (or line, with `MULTILINE`) | `bye$` matches `"goodbye"` but not `"bye now"` |
| `*` | Matches the previous element **zero or more** times | `ab*c` matches `"ac"`, `"abc"`, `"abbbc"` |
| `+` | Matches the previous element **one or more** times | `ab+c` matches `"abc"`, `"abbc"`, but not `"ac"` |
| `?` | Matches the previous element **zero or one** time (optional) | `colou?r` matches `"color"` and `"colour"` |
| `{n,m}` | Matches the previous element between `n` and `m` times | `a{2,4}` matches `"aa"`, `"aaa"`, `"aaaa"` |
| `[]` | A character class — matches any **one** character inside the brackets | `[aeiou]` matches any single vowel |
| `()` | Groups part of a pattern together, and captures the matched text for later use | `(ab)+` matches `"ab"`, `"abab"`, `"ababab"` |
| `\|` | Alternation — matches whichever side is present ("or") | `cat\|dog` matches `"cat"` or `"dog"` |
| `\` | Escapes a special character, treating it literally — or introduces a special sequence (Section 4) | `\.` matches a literal period |

A quick but important note on `[]`: inside brackets, most special characters lose their meaning and become literal. `[.]` matches a literal period, not "any character." The one exception is `^` at the *start* of a bracket, which flips the meaning to "anything **except** these": `[^0-9]` matches any character that is *not* a digit.

---

## 3. RegEx Flags

Flags change how the entire pattern is interpreted, and are passed as an extra argument to most `re` functions.

| Flag | Short Form | Description | Example |
|---|---|---|---|
| `re.IGNORECASE` | `re.I` | Makes matching case-insensitive | `re.findall(r"cat", "CAT cat Cat", re.I)` → `['CAT', 'cat', 'Cat']` |
| `re.MULTILINE` | `re.M` | Makes `^` and `$` match the start/end of **each line**, not just the whole string | `re.findall(r"^\w+", "line one\nline two", re.M)` → `['line', 'line']` |
| `re.DOTALL` | `re.S` | Makes `.` match newlines too (normally it doesn't) | `re.findall(r"a.b", "a\nb", re.S)` → `['a\nb']` |
| `re.VERBOSE` | `re.X` | Allows whitespace and comments inside the pattern itself, for readability | Lets a long pattern be written across multiple indented lines |
| `re.ASCII` | `re.A` | Restricts `\w`, `\d`, `\s` (Section 4) to ASCII characters only, instead of full Unicode | Affects behavior with non-English text |

Multiple flags can be combined with `|`: `re.findall(pattern, text, re.I | re.M)`.

---

## 4. Special Sequences

These are shorthand character classes — each one a compact stand-in for a common set of characters.

| Sequence | Matches | Example |
|---|---|---|
| `\d` | Any digit (0–9) | `\d\d` matches `"42"` |
| `\D` | Any **non**-digit character | `\D+` matches `"hello"` |
| `\w` | Any "word" character — letters, digits, or underscore | `\w+` matches `"hello_123"` |
| `\W` | Any **non**-word character | `\W` matches `"!"`, `" "`, `"@"` |
| `\s` | Any whitespace character (space, tab, newline) | `\s+` matches `"   "` |
| `\S` | Any **non**-whitespace character | `\S+` matches `"hello"` |
| `\b` | A **word boundary** — the edge between a word character and a non-word character (matches a position, not a character) | `\bcat\b` matches `"cat"` in `"a cat sat"` but not in `"category"` |
| `\B` | A **non**-word-boundary | `\Bcat\B` matches `"cat"` only when embedded inside another word |
| `\A` | Matches only at the start of the entire string (like `^`, but unaffected by `MULTILINE`) | `\AHi` |
| `\Z` | Matches only at the end of the entire string (like `$`, but unaffected by `MULTILINE`) | `bye\Z` |
| `\1`, `\2`, ... | A **backreference** — refers back to whatever an earlier group `()` captured | `(\w)\1` matches any repeated character, like `"ll"` or `"oo"` |

`\b` deserves special attention because it trips people up constantly: it's a **zero-width** match — it doesn't consume any characters itself, it just asserts "there's a word boundary right here." That's exactly why `\bcat\b` correctly excludes `category` (there's no boundary between `cat` and `egory`) while still matching a standalone `cat`.

---

## Practice Examples

The rest of this guideline works through concrete problems, pattern by pattern — the kind of tasks regex gets reached for constantly in real code.

### Retrieve Only Those with Text

Given a mixed list, keep only the entries that are made up entirely of letters:

```python
import re

items = ["apple", "123", "banana45", "cherry", "007"]
text_only = [item for item in items if re.fullmatch(r"[A-Za-z]+", item)]
print(text_only)  # ['apple', 'cherry']
```

`fullmatch()` is essential here — `search()` would find the letters *inside* `"banana45"` too, since it doesn't require the whole string to qualify.

### Start and End With

**A single character**, with anything (or nothing) in between:

```python
print(bool(re.fullmatch(r"a.*z", "az")))       # True — nothing between is fine
print(bool(re.fullmatch(r"a.*z", "abcxyz")))   # True
```

**One or more characters in between** — swap `*` (zero or more) for `+` (one or more) to require the middle isn't empty:

```python
print(bool(re.fullmatch(r"a.+z", "az")))    # False — nothing between, fails
print(bool(re.fullmatch(r"a.+z", "abz")))   # True
```

**A specific word** at the start and another at the end:

```python
text = "Hello there, general Kenobi, this is World"
print(bool(re.fullmatch(r"Hello.*World", text)))  # True
```

### Multiple Occurrences

Counting or collecting every time a pattern shows up, not just the first:

```python
text = "the cat sat with the cat and another cat"
matches = re.findall(r"\bcat\b", text)
print(matches)        # ['cat', 'cat', 'cat']
print(len(matches))   # 3
```

### Exact Character Match (Full Match)

Validating that an *entire* string conforms to a shape — common for things like ID codes or simple format checks:

```python
code = "AB-1234"
print(bool(re.fullmatch(r"[A-Z]{2}-\d{4}", code)))  # True

code2 = "AB-12"
print(bool(re.fullmatch(r"[A-Z]{2}-\d{4}", code2)))  # False — wrong digit count
```

### Only Uppercase or Lowercase

```python
words = ["HELLO", "hello", "Hello"]

all_upper = [w for w in words if re.fullmatch(r"[A-Z]+", w)]
all_lower = [w for w in words if re.fullmatch(r"[a-z]+", w)]

print(all_upper)  # ['HELLO']
print(all_lower)  # ['hello']
```

### Find Capitalized Words

Words that start with a capital letter, followed by lowercase letters:

```python
text = "Alice went to Paris with bob and Charlie"
capitalized = re.findall(r"\b[A-Z][a-z]*\b", text)
print(capitalized)  # ['Alice', 'Paris', 'Charlie']
```

Notice `bob` is correctly excluded — it starts with a lowercase letter, so the pattern simply doesn't match it.

### Has [a Character] in Between, but Not at the Beginning or End

Requiring the target to appear somewhere in the *middle*, guaranteed by demanding at least one character on both sides:

```python
words = ["banana", "apple", "anagram"]
has_a_in_middle = [w for w in words if re.fullmatch(r".+a.+", w)]
print(has_a_in_middle)  # ['banana', 'anagram']
```

`.+a.+` requires at least one character before the `a` and at least one character after it — which is exactly what rules out `a` sitting right at the very start or very end of the string.

### Two Consecutive Words Both Starting with the Same Letter

This is where backreferences (Section 4) do the real work — capture the first word's starting letter with a group, then demand the second word start with the exact same letter using `\1`:

```python
text = "Peter Piper picked a peck of pickled peppers"
pattern = r"\b([A-Za-z])\w*\s+\1\w*\b"

for match in re.finditer(pattern, text, re.IGNORECASE):
    print(match.group())

# Peter Piper
# picked a   -> not printed, "a" doesn't share picked's letter
# peck of    -> not printed
# pickled peppers
```

### All Words Starting with Either [Letter]

A character class placed right at the start of the word boundary handles "either/or" starting letters in one shot:

```python
text = "Cats, Dogs, and Elephants live at the zoo, but Cows do not"
words = re.findall(r"\b[CD]\w*\b", text)
print(words)  # ['Cats', 'Dogs', 'Cows']
```

### Date or Number Conversion

Reformatting a date from `MM/DD/YYYY` to `YYYY-MM-DD`, using capture groups and backreferences inside `re.sub()`:

```python
date = "09/27/2026"
converted = re.sub(r"(\d{2})/(\d{2})/(\d{4})", r"\3-\1-\2", date)
print(converted)  # 2026-09-27
```

Cleaning a formatted number (removing thousands separators) before converting it to an actual number:

```python
raw = "12,345,678"
clean = re.sub(r",", "", raw)
print(int(clean))  # 12345678
```

### Extract Words or Numbers

```python
text = "There are 3 cats and 12 dogs in 2 houses"

numbers = re.findall(r"\d+", text)
words = re.findall(r"[A-Za-z]+", text)

print(numbers)  # ['3', '12', '2']
print(words)    # ['There', 'are', 'cats', 'and', 'dogs', 'in', 'houses']
```

### Find Occurrences of a Specific Substring

```python
text = "the rain in Spain falls mainly on the plain"
occurrences = re.findall(r"ain", text)
print(len(occurrences))  # 4  -> "rain", "Spain", "mainly", "plain"
```

`re.escape()` is worth knowing here too — if the substring you're searching for might contain regex special characters (like a literal `.` or `$`), wrap it in `re.escape()` first so it's treated as plain text rather than a pattern:

```python
substring = "3.14"
text = "pi is roughly 3.14 and definitely not 3x14"
print(re.findall(re.escape(substring), text))  # ['3.14']
```

### Replacing Words

```python
text = "I have a cat and my friend has a cat too"
replaced = re.sub(r"\bcat\b", "dog", text)
print(replaced)  # I have a dog and my friend has a dog too
```

`re.sub()` also accepts a function instead of a plain string as its replacement, letting each match be transformed individually rather than replaced with the same fixed text every time:

```python
text = "hello world"
capitalized = re.sub(r"\b\w", lambda m: m.group().upper(), text)
print(capitalized)  # Hello World
```

---

Regex is a language you read more than you write from scratch — most working patterns are assembled by recognizing which symbol from these tables solves the piece of the problem in front of you, one at a time, rather than composing something clever in a single pass. The practice examples above cover the shapes you'll run into most often; once those feel familiar, unfamiliar patterns start reading less like noise and more like a sentence.
