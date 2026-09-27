# Guideline 25: Python Libraries

Every guideline since 17 has quietly been building toward this one. `math`, `random`, `time`, `datetime`, `calendar`, and `re` weren't just topics in their own right — they were your first hands-on experience with a **library**: a bundle of pre-written code, built by someone else, that you bring into your own program with a single `import` line instead of writing it yourself from scratch. This guideline steps back to look at that whole system directly — how importing actually works, what's already sitting in Python's standard library waiting to be used, and what exists *outside* it, in the much larger world of external libraries this guidebook's second half leans on heavily.

---

## 1. Import Methods

### Two Ways to Import

The first way imports the **entire module**, and everything inside it must be accessed through the module's name, using a dot:

```python
import math

print(math.sqrt(16))   # 4.0
print(math.pi)          # 3.141592653589793
```

The second way imports **specific names** directly out of a module, letting you use them with no prefix at all:

```python
from math import sqrt, pi

print(sqrt(16))   # 4.0
print(pi)          # 3.141592653589793
```

Neither approach is universally "better" — importing the whole module keeps it obvious, everywhere you use it, exactly which library a function came from (`math.sqrt` unmistakably belongs to `math`); importing specific names keeps your code shorter when you're using something constantly. A related shortcut, `from module import *`, imports *everything* a module offers at once — it's generally worth avoiding in real code, since it makes it much harder to tell, later, which module a given name actually came from, and it risks silently overwriting a name you already defined yourself.

### Alias Import

Both import styles can rename what they bring in, using `as`:

```python
import math as m
print(m.sqrt(16))  # 4.0

from math import sqrt as square_root
print(square_root(16))  # 4.0
```

Aliasing exists for convenience — shortening a long module name you'll type constantly — but it also follows some extremely strong, near-universal conventions once you reach external libraries in Section 4. `import numpy as np` and `import pandas as pd` are so consistently used this exact way across the entire Python data science community that deviating from them (writing `import numpy as banana`, however legal) would only make your code harder for others — and future you — to read.

---

## 2. Built-In Libraries

A **built-in library** (more precisely, a *standard library* module) is one that ships with Python itself — the moment Python is installed on a machine, `math`, `random`, `time`, `re`, and every other standard library module are already there, ready to `import` with no additional setup. This is what every library used in this guidebook so far has been.

The distinction that matters is what comes next: an **external library** is *not* included with Python — it has to be installed separately before it can ever be imported, using a tool called `pip` (covered in Section 4). Trying to `import` an external library that hasn't been installed yet raises a `ModuleNotFoundError` — a strong signal, when you see it, that the fix isn't a code error at all, but a missing installation step.

---

## 3. Common Built-In Libraries for Beginners

You've already met several of the standard library's most useful modules in earlier guidelines — `math` (Guideline 17), `random` (Guideline 19), `time`, `datetime`, and `calendar` (Guideline 18), and `re` (Guideline 21). The table below rounds out the set with the other standard library modules a beginner runs into constantly.

| Module | What It's For | Example Method |
|---|---|---|
| `os` | Interacting with the operating system — file paths, directories, environment variables | `os.getcwd()` → current working directory; `os.listdir()` → files in a folder |
| `sys` | Interacting with the Python interpreter itself — command-line arguments, exiting a program | `sys.argv` → list of command-line arguments; `sys.exit()` → stops the program |
| `string` | Ready-made string constants, useful for validation or generating random text | `string.ascii_letters` → `'abc...XYZ'`; `string.digits` → `'0123456789'` |
| `statistics` | Descriptive statistics on numeric data — a complement to `math`'s more general-purpose functions | `statistics.mean([1,2,3])` → `2`; `statistics.median(...)`, `statistics.stdev(...)` |
| `itertools` | Efficient, memory-conscious tools for looping and combining iterables (built directly on the iterator concept from Guideline 24) | `itertools.combinations([1,2,3], 2)` → pairs; `itertools.permutations(...)`, `itertools.cycle(...)` |
| `collections` | Specialized container types beyond the basic list/dict/set | `collections.Counter(["a","b","a"])` → counts each item; `collections.defaultdict(...)`, `collections.deque(...)` |
| `json` | Reading and writing JSON-formatted data — the format most web APIs speak | `json.dumps(data)` → Python object to JSON string; `json.loads(text)` → JSON string to Python object |
| `csv` | Reading and writing CSV (comma-separated values) files | `csv.reader(file)` → read rows; `csv.writer(file)` → write rows |

Every one of these is available the instant Python is installed — no `pip install` required, no internet connection needed to fetch anything. That single fact is what makes the standard library worth reaching for first, whenever it genuinely covers what you need.

---

## 4. External Libraries

Once a task moves beyond what the standard library was designed for — numerical computing at scale, tabular data analysis, plotting, machine learning — you leave Python's built-in toolkit behind and enter the vastly larger ecosystem of **external libraries**, published and maintained by the wider Python community, hosted on a repository called **PyPI** (the Python Package Index).

Getting one of these onto your machine requires **`pip`**, Python's package installer, run from a terminal — not from inside a Python file itself:

```bash
pip install numpy
```

Once installed, importing it works exactly the same as any built-in module — `import`, or `from ... import ...`, alias or no alias:

```python
import numpy as np
print(np.array([1, 2, 3]))
```

The table below is a preview of the specific external libraries this guidebook's second half builds directly on top of.

| Library | What It's For | Example Method |
|---|---|---|
| `numpy` | Fast numerical computing with multi-dimensional arrays (the specialized alternative to nested lists hinted at back in Guideline 16) | `np.array([1,2,3])` → array; `np.mean(arr)`, `np.dot(a, b)` |
| `pandas` | Loading, cleaning, and analyzing tabular data (spreadsheet-like structures called DataFrames) | `pd.read_csv("file.csv")` → loads a file; `df.head()`, `df.describe()` |
| `matplotlib` | Creating charts and plots — line graphs, bar charts, scatter plots | `plt.plot(x, y)` → draws a line chart; `plt.show()` → displays it |
| `seaborn` | Statistical data visualization, built on top of `matplotlib` with more polished defaults | `sns.heatmap(data)` → a color-coded grid; `sns.boxplot(...)` |
| `scikit-learn` | Classical machine learning — classification, regression, clustering models | `from sklearn.linear_model import LinearRegression` → `model.fit(X, y)` |
| `requests` | Making HTTP requests — fetching data from web APIs | `requests.get(url)` → fetches a web page or API response |
| `beautifulsoup4` (`bs4`) | Parsing HTML, commonly used for web scraping | `BeautifulSoup(html, "html.parser")` → a searchable document object |
| `pygame` | Building 2D games — graphics, sound, input handling | `pygame.display.set_mode((800, 600))` → opens a game window |
| `tensorflow` / `pytorch` | Building and training deep learning models (neural networks) | `tf.keras.Sequential([...])`; `torch.nn.Linear(...)` |

You won't need all of these immediately — several are previews of chapters still ahead, in the same spirit as `math` being introduced back in Guideline 17 as "a sneak peek to working with external libraries." What's worth taking away from this table right now is simply the shape of things to come: importing an external library is no harder than importing `math` was, provided the one extra step — `pip install` — happens first.

---

Between two ways to bring a module in, an alias to shorten the ones you'll type constantly, the always-available standard library, and the much larger external ecosystem installed through `pip`, this guideline is less about a specific new skill and more about the doorway itself — the mechanism this book's remaining guidelines will keep walking back through, one library at a time.
