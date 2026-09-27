# Guideline 18: Date and Time Functions in Python

Almost every real program eventually needs to know *when* something happened — a timestamp on a log entry, a countdown, a deadline, a day-of-week check. Python splits this responsibility across three standard library modules: **`time`** (raw timestamps and low-level timing), **`datetime`** (calendar-aware dates and times you can do arithmetic on), and **`calendar`** (whole-month and whole-year views). Each needs an `import` before use — none of this is built in by default.

Because so much of this guideline is reference material rather than new logic, most of it is laid out as tables you can come back to.

---

## 1. The Time Module

```python
import time
```

The `time` module works close to the operating system's clock. Its core unit is the **timestamp** — the number of seconds elapsed since January 1, 1970 (the "Unix epoch") — and a structure called `struct_time`, which breaks a moment down into named fields (`tm_year`, `tm_mon`, `tm_mday`, `tm_hour`, `tm_min`, `tm_sec`, `tm_wday`, and a few more).

### Extracting Time

| Function | Description | Example |
|---|---|---|
| `time.time()` | Current timestamp — seconds since the epoch, as a float | `time.time()` → `1731024000.123` |
| `time.localtime()` | Current moment as a `struct_time`, in local timezone | `time.localtime()` → `struct_time(tm_year=2026, tm_mon=9, ...)` |
| `time.gmtime()` | Current moment as a `struct_time`, in UTC | `time.gmtime()` → `struct_time(tm_year=2026, tm_mon=9, ...)` |
| `time.ctime()` | Current local time as a readable string | `time.ctime()` → `'Sun Sep 27 14:03:01 2026'` |
| `time.perf_counter()` | A high-resolution counter for measuring *elapsed intervals* — not tied to the wall clock, only useful for differences | `end - start` after two calls → seconds elapsed |
| `time.sleep(seconds)` | Pauses program execution for the given number of seconds | `time.sleep(2)` → pauses for 2 seconds |

A `struct_time` isn't a string — it's an object with named fields you access with dot notation:

```python
now = time.localtime()
print(now.tm_year)   # 2026
print(now.tm_mon)    # 9
print(now.tm_mday)   # 27
```

`time.perf_counter()` deserves a special note: it's the right tool for timing *how long code takes to run*, since `time.time()` can be affected by the system clock being adjusted mid-run.

```python
start = time.perf_counter()
time.sleep(1.5)
end = time.perf_counter()
print(end - start)  # ~1.5
```

### Time Conversion

| Function | Description | Example |
|---|---|---|
| `time.strftime(format, struct_time)` | Converts a `struct_time` into a formatted string | `time.strftime("%Y-%m-%d", time.localtime())` → `'2026-09-27'` |
| `time.strptime(string, format)` | Parses a string into a `struct_time`, given a matching format | `time.strptime("27/09/2026", "%d/%m/%Y")` → `struct_time(...)` |
| `time.mktime(struct_time)` | Converts a `struct_time` back into a timestamp (seconds since epoch) | `time.mktime(time.localtime())` → `1731024000.0` |

Both `strftime` and `strptime` rely on the same set of format codes — a small reference worth memorizing early, since it reappears in the `datetime` module too:

| Code | Meaning | Example |
|---|---|---|
| `%Y` | 4-digit year | `2026` |
| `%y` | 2-digit year | `26` |
| `%m` | Month (zero-padded) | `09` |
| `%d` | Day of month (zero-padded) | `27` |
| `%H` | Hour, 24-hour clock | `14` |
| `%M` | Minute | `03` |
| `%S` | Second | `01` |
| `%A` | Full weekday name | `Sunday` |
| `%a` | Abbreviated weekday name | `Sun` |
| `%B` | Full month name | `September` |
| `%b` | Abbreviated month name | `Sep` |

---

## 2. The Date Module

A quick note on naming: Python's standard library doesn't ship a module literally called "date" — the relevant module is **`datetime`**, and inside it lives a class actually called `date` (calendar dates, no time-of-day) alongside `time`, `datetime`, and `timedelta` classes. Since `date` — and its sibling `datetime` — are the pieces you'll use constantly, "the Date module" is a fair shorthand for the whole thing.

```python
from datetime import date, datetime, timedelta
```

Where `time` hands you a loose `struct_time`, `datetime` hands you proper objects you can compare, subtract, and format directly — which is why so many of its methods look familiar from Section 1, just attached to an object instead of passed as an argument.

### Getting the Current Moment

| Function | Description | Example |
|---|---|---|
| `date.today()` | Today's date (no time-of-day) | `date.today()` → `2026-09-27` |
| `datetime.now()` | Current date **and** time | `datetime.now()` → `2026-09-27 14:03:01.123456` |

### Reading Parts of a Date

| Attribute / Method | Description | Example |
|---|---|---|
| `.year` / `.month` / `.day` | The individual components of a `date` or `datetime` | `date.today().year` → `2026` |
| `.hour` / `.minute` / `.second` | Time components (`datetime` only, not plain `date`) | `datetime.now().hour` → `14` |
| `.weekday()` | Day of the week as an integer, **Monday = 0** | `date(2026, 9, 27).weekday()` → `6` |
| `.isoweekday()` | Day of the week as an integer, **Monday = 1** | `date(2026, 9, 27).isoweekday()` → `7` |

### Converting To and From Strings

This is where the parallel to `time` is closest — the same format codes from Section 1 apply here too:

| Function | Description | Example |
|---|---|---|
| `date_obj.strftime(format)` | Formats a `date`/`datetime` object into a string | `date.today().strftime("%B %d, %Y")` → `'September 27, 2026'` |
| `datetime.strptime(string, format)` | Parses a string into a `datetime` object | `datetime.strptime("27/09/2026", "%d/%m/%Y")` → `datetime(2026, 9, 27, 0, 0)` |

### Date Arithmetic with `timedelta`

This is the one capability `time`'s raw timestamps don't offer cleanly: adding or subtracting spans of time directly on calendar-aware objects.

| Operation | Description | Example |
|---|---|---|
| `date_obj + timedelta(days=n)` | Adds `n` days to a date | `date(2026, 9, 27) + timedelta(days=7)` → `2026-10-04` |
| `date_obj - timedelta(days=n)` | Subtracts `n` days from a date | `date(2026, 9, 27) - timedelta(days=7)` → `2026-09-20` |
| `date2 - date1` | Subtracting two dates gives a `timedelta` — the span between them | `date(2026, 12, 25) - date.today()` → `timedelta(days=89)` |

```python
today = date.today()
next_week = today + timedelta(weeks=1)
print(next_week)  # 2026-10-04
```

### Quick Comparison: `time` vs. `datetime`

| Task | `time` module | `datetime` module |
|---|---|---|
| Get "now" | `time.time()` → raw timestamp | `datetime.now()` → full object |
| Get today's date only | `time.localtime()` (has extra fields) | `date.today()` (date only, cleaner) |
| Format as string | `time.strftime(fmt, struct_time)` | `date_obj.strftime(fmt)` |
| Parse a string | `time.strptime(str, fmt)` → `struct_time` | `datetime.strptime(str, fmt)` → `datetime` object |
| Do date arithmetic | Not directly supported | `timedelta` supports it natively |

The short version: reach for `time` when you need a raw timestamp or a simple stopwatch; reach for `datetime` when you need to reason about calendar dates — comparing them, adding days to them, or extracting the weekday.

---

## 3. The Calendar Module

```python
import calendar
```

Where `datetime` works with a single date, `calendar` zooms out to entire months and years — useful for anything that needs to *display* a calendar, or answer structural questions like "how many days are in this month" or "is this a leap year."

| Function | Description | Example |
|---|---|---|
| `calendar.month(year, month)` | Returns a formatted multi-line string of a single month | `calendar.month(2026, 9)` → text calendar for September 2026 |
| `calendar.calendar(year)` | Returns a formatted multi-line string of an entire year | `calendar.calendar(2026)` → text calendar for all of 2026 |
| `calendar.isleap(year)` | Checks whether a year is a leap year | `calendar.isleap(2024)` → `True` |
| `calendar.leapdays(y1, y2)` | Counts how many leap years fall between two years | `calendar.leapdays(2000, 2026)` → `7` |
| `calendar.monthrange(year, month)` | Returns `(weekday of the 1st, number of days in the month)` | `calendar.monthrange(2026, 9)` → `(1, 30)` |
| `calendar.weekday(year, month, day)` | Returns the weekday index for a specific date (Monday = 0) | `calendar.weekday(2026, 9, 27)` → `6` |
| `calendar.setfirstweekday(day)` | Changes which day is treated as the start of the week (default Monday = 0) | `calendar.setfirstweekday(6)` → weeks now start on Sunday |
| `calendar.day_name` | A sequence of full weekday names, in order | `calendar.day_name[0]` → `'Monday'` |
| `calendar.month_name` | A sequence of full month names, 1-indexed (`month_name[0]` is empty) | `calendar.month_name[9]` → `'September'` |

```python
print(calendar.month(2026, 9))
#     September 2026
# Mo Tu We Th Fr Sa Su
#     1  2  3  4  5  6
#  7  8  9 10 11 12 13
# 14 15 16 17 18 19 20
# 21 22 23 24 25 26 27
# 28 29 30
```

`monthrange()` in particular is worth remembering: it's the function you'd reach for if you ever needed to *programmatically* know how many days are in a given month — accounting for leap years automatically — without hardcoding "30 days hath September."

---

Between raw timestamps (`time`), calendar-aware objects you can compare and subtract (`datetime`), and whole-month/whole-year views (`calendar`), these three modules cover the overwhelming majority of date-and-time needs you'll run into — from a simple "how long did this take to run" stopwatch, to scheduling logic that needs to know exactly how many days remain until a deadline.
