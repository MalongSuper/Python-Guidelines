# Guideline 22: Python File Handling

Every program covered so far has lived and died within a single run — close the program, and every list, dictionary, and variable disappears with it. Files are how a program remembers something *after* it ends: written to disk, still there tomorrow, still there next week. This guideline covers the core toolkit for creating, reading, writing, and safely closing files in Python.

---

## 1. Creating and Opening a File (File Modes)

Every interaction with a file starts with `open()`, which takes a filename and a **mode** describing what you intend to do with it:

```python
f = open("notes.txt", "w")
```

The mode is not optional detail — it changes what happens to the file's existing content, and even whether Python expects the file to already exist.

| Mode | Meaning | File Must Already Exist? | Erases Existing Content? |
|---|---|---|---|
| `"r"` | Read (the default if no mode is given) | Yes — error if missing | No |
| `"w"` | Write | No — creates it if missing | **Yes** — wipes it clean first |
| `"a"` | Append | No — creates it if missing | No — new writes go to the end |
| `"x"` | Exclusive creation | Must **not** exist — errors if it does | N/A |
| `"r+"` | Read and write | Yes — error if missing | No |
| `"b"` (suffix) | Binary mode, e.g. `"rb"`, `"wb"` | Depends on the letter it's attached to | Depends on the letter it's attached to |

The single most important row in that table is `"w"` — opening a file in write mode **erases its entire existing content the instant it's opened**, even before you write a single character. This is the source of more accidental data loss than almost any other line of code in this guidebook, and it's exactly why Section 5 exists.

---

## 2. Reading an Entire File

`.read()` pulls the entire file's contents into memory as one single string:

```python
f = open("notes.txt", "r")
contents = f.read()
print(contents)
f.close()
```

This is fine for small files, but worth being cautious with on a large one — `.read()` loads everything at once, so a multi-gigabyte log file would mean a multi-gigabyte string sitting in memory.

---

## 3. Reading One Line at a Time

`.readline()` reads exactly one line — including its trailing newline character `\n` — and remembers its position, so calling it again picks up right where it left off. Once there's nothing left to read, it returns an empty string.

```python
f = open("notes.txt", "r")

line = f.readline()
while line:
    print(line.strip())  # .strip() removes the trailing newline for cleaner printing
    line = f.readline()

f.close()
```

This pattern is the memory-friendly alternative to `.read()` — at any given moment, only one line needs to be in memory, no matter how large the file is.

---

## 4. Reading All Lines into a List

`.readlines()` reads the entire file, just like `.read()`, but instead of one giant string, it hands back a **list of strings** — one entry per line, each still carrying its trailing `\n`.

```python
f = open("notes.txt", "r")
lines = f.readlines()
f.close()

print(lines)
# ['first line\n', 'second line\n', 'third line\n']

print(lines[1])         # second line
print(len(lines))       # 3 -> how many lines the file has
```

Having every line as a separate list element makes `.readlines()` the natural choice whenever you need to loop through a file's contents *and* keep track of things like line numbers, or go back and revisit an earlier line — something `.readline()`'s one-at-a-time approach makes far more awkward.

---

## 5. Writing to a File (Append Mode to Avoid Overwriting)

`.write()` sends a string into the file:

```python
f = open("notes.txt", "w")
f.write("Hello, world!")
f.close()
```

Run that exact code a second time, and `"Hello, world!"` is still all that's in the file — `"w"` mode erased whatever was there first, every single time it's opened. If your goal is to *add* to a file without destroying what's already inside it, `"a"` (append) mode is the fix:

```python
f = open("notes.txt", "a")
f.write("\nAnother line, added without erasing the first.")
f.close()
```

Run *this* version twice, and both lines accumulate — nothing gets wiped, because `"a"` always writes starting from the end of the existing content rather than from a blank slate.

---

## 6. Writing Multiple Lines

`.writelines()` takes a list of strings and writes them all in sequence — but with one detail that catches almost everyone the first time: **it does not add newlines between entries automatically.** Whatever line breaks you want, you have to include yourself.

```python
lines = ["First line\n", "Second line\n", "Third line\n"]

f = open("notes.txt", "w")
f.writelines(lines)
f.close()
```

Leave the `\n` off any entry in that list, and it runs straight into the next one with no separation at all. The equally common alternative — looping and calling `.write()` yourself — makes that newline requirement more visible, since you're the one typing it each time:

```python
f = open("notes.txt", "w")
for line in lines:
    f.write(line + "\n")
f.close()
```

Both approaches produce identical output; `.writelines()` is simply a shortcut for the loop.

---

## 7. Closing a File

Every `open()` should be paired with a `.close()`. Skipping it isn't harmless — while a file stays open, Python may hold recently written data in a buffer rather than committing it to disk immediately, meaning your changes might not actually be saved yet. An unclosed file can also stay locked, preventing other programs (or another part of your own program) from accessing it.

```python
f = open("notes.txt", "w")
f.write("Some important data")
f.close()  # only now is the write guaranteed to be flushed to disk
```

The trouble with relying on `.close()` alone is that it's easy to forget — and if an error is raised *between* `open()` and `.close()`, the closing line never runs at all, leaving the file open indefinitely. That gap is exactly what Section 8 solves.

---

## 8. `with open()`

The `with` statement wraps file handling in a way that guarantees the file gets closed automatically — even if an error occurs partway through — without you ever writing `.close()` yourself.

```python
with open("notes.txt", "r") as f:
    contents = f.read()
    print(contents)

# f is automatically closed here, the moment the indented block ends
```

This is functionally identical to the manual `open()` / `.close()` pattern from Section 7, but safer: the moment execution leaves the `with` block — whether normally or because of an exception — the file is closed, no exceptions to that rule. For this reason, `with open()` is the idiomatic, recommended way to handle files in virtually all modern Python code; treat the manual `open()`/`.close()` pattern in the earlier sections as something to *recognize* when you see it, but reach for `with` in your own code from here on.

---

## 9. File Handling with Conditional Statements or Loops

The real value of file handling shows up once reading, looping, and conditionals combine — creating a file with several lines of data, then traversing it afterward to pull out exactly the information you need. `.readlines()` earns its keep here, since having every line available as a list makes it easy to loop through with a `for` statement and check each one against a condition.

**Step one: create a multi-line file.**

```python
students = [
    "Amara,90\n",
    "Ben,72\n",
    "Cho,95\n",
    "Deshawn,64\n"
]

with open("scores.txt", "w") as f:
    f.writelines(students)
```

**Step two: traverse the file and retrieve exactly the information needed** — here, only the students who scored 80 or above:

```python
with open("scores.txt", "r") as f:
    lines = f.readlines()

for line in lines:
    name, score = line.strip().split(",")
    score = int(score)
    if score >= 80:
        print(f"{name} passed with honors: {score}")

# Amara passed with honors: 90
# Cho passed with honors: 95
```

Each piece here is something you've already learned separately: `.readlines()` from Section 4, `.strip()` and `.split()` from earlier string-handling guidelines, and a simple `if` condition inside a `for` loop. File handling rarely introduces new logic of its own — its real skill is reading a file into a familiar shape (a list, a string) and then applying everything else you already know how to do with that shape.

---

Between reading a file whole, line by line, or into a list; writing while being careful not to erase what's already there; and reliably closing everything with `with open()`, this covers the full lifecycle of working with files in Python — creation, reading, writing, and the safe cleanup that keeps your data intact.
