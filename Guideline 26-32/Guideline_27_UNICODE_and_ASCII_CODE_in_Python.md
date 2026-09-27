# Guideline 27: UNICODE and ASCII CODE in Python

Every character you type — a letter, a digit, a punctuation mark, an emoji — is stored on a computer as a number. This guideline covers how Python lets you move back and forth between characters and the numbers underneath them, starting with the older, narrower ASCII standard and ending with Unicode, the standard that actually powers Python's strings today.

## 27.1 ASCII Code

**ASCII** (American Standard Code for Information Interchange) is one of the oldest character-encoding standards still in everyday use. It assigns a number from 0 to 127 to a fixed set of characters — uppercase and lowercase letters, digits, punctuation, and a handful of non-printable "control" characters left over from teletype machines (like newline and tab).

A few landmarks worth memorizing, because they come up constantly:

| Character | ASCII code |
|---|---|
| `'A'` | 65 |
| `'Z'` | 90 |
| `'a'` | 97 |
| `'z'` | 122 |
| `'0'` | 48 |
| `'9'` | 57 |
| `' '` (space) | 32 |
| `'\n'` (newline) | 10 |

Notice the pattern: `'A'` to `'Z'` is one unbroken run (65–90), `'a'` to `'z'` is another (97–122), and `'0'` to `'9'` is another (48–57). That's not a coincidence — it's exactly what makes the arithmetic tricks in the next section possible.

Because ASCII only covers 128 values, it fits in 7 bits (2⁷ = 128) — it was never built to represent every character in every language, which is precisely the gap Unicode was later created to fill (section 27.4).

## 27.2 The `ord()` and `chr()` Functions

Python gives you two built-in functions that move directly between a character and its underlying code — the same functions briefly mentioned back in Guideline 6, covered here in full.

`ord()` takes a single character and returns its numeric code point:

```python
print(ord('A'))   # 65
print(ord('a'))   # 97
print(ord('0'))   # 48
print(ord(' '))   # 32
```

`chr()` is the exact inverse — it takes a number and returns the character it represents:

```python
print(chr(65))    # A
print(chr(97))    # a
print(chr(48))    # 0
```

Because the letter ranges are contiguous (as noted above), you can shift through the alphabet with plain arithmetic. This is the mechanism behind a Caesar-cipher-style letter shift:

```python
def shift_letter(letter, shift):
    code = ord(letter) + shift
    return chr(code)

print(shift_letter('a', 1))   # b
print(shift_letter('a', 25))  # z
```

Both functions only accept — and only return — a single character. Calling `ord()` on a multi-character string raises a `TypeError`; to work through a whole string, you traverse it one character at a time, which is exactly what section 27.5 does.

## 27.3 The `bin()`, `hex()` and `oct()` Methods

Once you have a character's numeric code from `ord()`, you can view that same number in binary, hexadecimal, or octal using three more built-ins:

```python
code = ord('A')   # 65

print(bin(code))  # 0b1000001
print(hex(code))  # 0x41
print(oct(code))  # 0o101
```

Each function returns a **string**, prefixed to show which base it's in: `0b` for binary, `0x` for hexadecimal, `0o` for octal. If you want the digits without the prefix, slice it off:

```python
print(bin(code)[2:])   # 1000001
```

Put together with `ord()`, this is a quick way to see a character from every angle:

```python
def inspect_character(char):
    code = ord(char)
    print(f"{char!r} -> decimal: {code}, binary: {bin(code)}, hex: {hex(code)}, octal: {oct(code)}")

inspect_character('A')
```

```
'A' -> decimal: 65, binary: 0b1000001, hex: 0x41, octal: 0o101
```

## 27.4 UNICODE

ASCII's 128 characters cover English comfortably, but not accented letters, non-Latin scripts (Cyrillic, Arabic, Chinese, Japanese, Korean...), mathematical symbols, or emoji. **Unicode** is the standard that replaces ASCII's tiny range with room for over a million code points — and, importantly, the first 128 Unicode code points are *identical* to ASCII, so every valid ASCII character is automatically also a valid Unicode character.

Python 3 strings are Unicode by default — every string you've written since Guideline 2 has already been a Unicode string, whether or not any character in it happened to be outside the ASCII range. `ord()` and `chr()` work exactly the same way beyond 127; they simply weren't limited to ASCII in the first place:

```python
print(ord('é'))    # 233
print(ord('中'))    # 20013
print(ord('🐍'))    # 128013

print(chr(233))    # é
print(chr(128013)) # 🐍
```

You'll sometimes see Unicode code points written with a `U+` prefix followed by hexadecimal — `U+1F40D` for the snake emoji above. That's just `hex()`'s output with the `0x` swapped for `U+`:

```python
code = ord('🐍')
print(f"U+{code:04X}")   # U+1F40D
```

(`:04X` is a format specifier from Guideline 5 — uppercase hex, padded to at least 4 digits.)

The practical upshot: you almost never need to think about "is this ASCII or Unicode" while writing Python — you're always working in Unicode, and ASCII text is simply a subset that happens to also be Unicode text.

## 27.5 Practical Example: Traversing a String for ASCII Codes

Since `ord()` only takes one character at a time, getting the code for every character in a string means looping over it — something you've already done plenty of since Guideline 8.

```python
def string_to_codes(text):
    codes = []
    for char in text:
        codes.append(ord(char))
    return codes

user_input = input("Enter a string: ")
result = string_to_codes(user_input)

for char, code in zip(user_input, result):
    print(f"{char!r}: {code}")
```

```
Enter a string: Hi!
'H': 72
'i': 105
'!': 33
```

The same idea works in reverse — given a list of codes, rebuild the string with `chr()` and `''.join()` (from Guideline 15):

```python
def codes_to_string(codes):
    return ''.join(chr(code) for code in codes)

print(codes_to_string([72, 105, 33]))   # Hi!
```

---

Underneath every string operation you've used so far — comparing, slicing, searching — is this same idea: characters are numbers wearing a familiar face. `ord()` and `chr()` are how you look underneath that face on demand, and Unicode is simply ASCII's numbering scheme extended far enough to cover every script and symbol in use today.
