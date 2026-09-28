# Guideline 38: Python for Data Visualization

A table of numbers can hide a pattern that a picture reveals in a second. Which month was the strongest? Do students who study longer score higher? Is one group's spread wider than another's? Guideline 37 taught you to slice, filter, and summarize data, and this guideline teaches you to draw it.

Python has two main libraries for this. **Matplotlib** is the foundation: it can draw almost any chart, one element at a time, with full control. **Seaborn** sits on top of Matplotlib and works directly with pandas tables, so a single line produces a polished statistical chart. Learn Matplotlib first, because everything Seaborn draws is a Matplotlib object underneath, and the two are often used together.

Both are external libraries, so install them once per environment:

```bash
pip install matplotlib seaborn
```

## 38.1 Matplotlib

Matplotlib is almost always imported through its `pyplot` module, with the alias `plt`. This guideline also uses NumPy and pandas from the previous guidelines:

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

**The anatomy of a plot.** Three terms appear throughout the documentation:

| Term | Meaning |
|---|---|
| **Figure** | The whole canvas, or window, that holds everything |
| **Axes** | One individual plot: the data area with its title, labels, and legend. A figure can hold several |
| **Axis** | The x-axis or y-axis of an Axes, with its ticks and tick labels |

**The simplest plot.** Pass x-values and y-values to `plt.plot()`, add labels, and call `plt.show()` to display it:

```python
x = [1, 2, 3, 4, 5]
y = [2, 4, 3, 6, 8]

plt.plot(x, y)
plt.title("My First Plot")
plt.xlabel("x")
plt.ylabel("y")
plt.show()
```

**Two ways to write the same code.** The style above, using `plt.` commands, is the **pyplot interface**. Matplotlib quietly keeps track of a "current" figure, and each command acts on it. It is quick and ideal for single charts. The alternative is the **object-oriented interface**, where you create the figure and axes explicitly and call methods on them:

```python
fig, ax = plt.subplots(figsize=(6, 4))    # one figure containing one Axes

ax.plot(x, y)
ax.set_title("My First Plot")
ax.set_xlabel("x")
ax.set_ylabel("y")

plt.show()
```

The two produce the same chart. The object-oriented form matters once a figure holds several plots (section 38.9), because it lets you say exactly which one you are drawing on.

**Important methods.**

| Method | Purpose |
|---|---|
| `plt.figure(figsize=(8, 5))` | Start a new figure; the size is in inches (width, height) |
| `plt.plot()`, `bar()`, `scatter()`, `pie()`, `hist()`, `boxplot()` | Draw the chart |
| `plt.title("...")` | Title of the plot |
| `plt.xlabel("...")`, `plt.ylabel("...")` | Axis labels |
| `plt.legend()` | Show a legend built from the `label=` given to each item |
| `plt.grid(True, alpha=0.3)` | Add faint grid lines |
| `plt.xlim(a, b)`, `plt.ylim(a, b)` | Set the visible range of an axis |
| `plt.xticks(positions, labels, rotation=45)` | Choose the tick positions, their text, and their angle |
| `plt.axhline(y)`, `plt.axvline(x)` | Draw a horizontal or vertical reference line |
| `plt.text(x, y, "...")`, `plt.annotate()` | Write text, or text with an arrow, at a point |
| `plt.tight_layout()` | Adjust spacing so labels do not overlap |
| `plt.savefig("name.png", dpi=300)` | Save the figure to a file |
| `plt.show()` | Display the figure |
| `plt.close()` | Close the figure and free its memory |

**Styling with keyword arguments.** Almost every drawing function accepts the same styling options:

| Argument | Meaning | Examples |
|---|---|---|
| `color` (or `c`) | Color | `"red"`, `"tab:blue"`, `"#1f77b4"` |
| `linestyle` (or `ls`) | Line pattern | `"-"` solid, `"--"` dashed, `":"` dotted, `"-."` dash-dot |
| `linewidth` (or `lw`) | Line thickness | `2` |
| `marker` | Shape drawn at each data point | `"o"` circle, `"s"` square, `"^"` triangle, `"*"` star, `"x"`, `"+"` |
| `markersize` (or `ms`) | Marker size | `8` |
| `alpha` | Transparency, from 0 (invisible) to 1 (solid) | `0.5` |
| `label` | Name shown in the legend | `"2025"` |

For quick work, a compact **format string** combines color, marker, and line style in one short text: `"ro--"` means red, circle markers, dashed line.

```python
plt.plot(x, y, "ro--")
plt.show()
```

**Styles and saving.** Matplotlib ships with ready-made looks, listed by `plt.style.available`. Using a style inside a `with` block applies it to that chart only. To save a chart, call `savefig()` **before** `show()`, because showing a figure can clear it:

```python
with plt.style.context("ggplot"):
    plt.plot(x, y)
    plt.title("Same Data, Different Style")
    plt.savefig("styled_plot.png", dpi=300, bbox_inches="tight")
    plt.show()
```

The examples below assume a script or notebook where `plt.show()` opens the chart. In a Jupyter notebook, charts appear automatically beneath the cell.

## 38.2 Line Graphs

A **line graph** connects points in order, and it is the natural choice for showing how something changes over time. Two years of monthly sales make a good example:

```python
months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun",
          "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]
sales_2025 = [12, 15, 14, 18, 21, 25, 28, 27, 23, 20, 16, 14]
sales_2026 = [14, 17, 18, 22, 26, 30, 33, 31, 27, 24, 20, 18]

plt.figure(figsize=(8, 4))
plt.plot(months, sales_2025, marker="o", linewidth=2, label="2025")
plt.plot(months, sales_2026, marker="s", linestyle="--", color="tab:orange", label="2026")

plt.title("Monthly Sales")
plt.xlabel("Month")
plt.ylabel("Sales (thousands)")
plt.grid(True, alpha=0.3)
plt.legend()
plt.tight_layout()
plt.show()
```

Each `plt.plot()` call adds one line to the same chart, and the `label` of each becomes its entry in the legend. Both lines rise into the summer and fall away again, with 2026 sitting above 2025 all year.

To point out a specific value, `annotate()` places a note with an arrow. On a chart whose x-values are text, the categories sit at positions 0, 1, 2, ..., so July is position 6:

```python
plt.figure(figsize=(8, 4))
plt.plot(months, sales_2026, marker="o")
plt.annotate("Peak", xy=(6, 33), xytext=(8, 34),
             arrowprops=dict(arrowstyle="->"))
plt.ylim(10, 38)
plt.title("2026 Sales, With the Peak Marked")
plt.show()
```

Other variations include `plt.yscale("log")` for a logarithmic axis, and `plt.fill_between()` to shade the area under a line. With a pandas table, `df.plot()` draws a line per column directly, using Matplotlib behind the scenes.

## 38.3 Bar Graphs

A **bar graph** compares values across categories. The employee table from Guideline 37 works well here:

```python
df = pd.DataFrame({
    "name":       ["Alice", "Bob", "Charlie", "Diana", "Ethan", "Fiona"],
    "department": ["Sales", "IT", "IT", "HR", "Sales", "IT"],
    "age":        [28, 35, 42, 31, 26, 38],
    "salary":     [52000, 68000, 81000, 59000, 48000, 75000],
    "city":       ["Paris", "Tokyo", "Paris", "Lima", "Tokyo", "Paris"],
})

plt.figure(figsize=(7, 4))
bars = plt.bar(df["name"], df["salary"], color="steelblue", edgecolor="black")
plt.bar_label(bars, fmt="%d", padding=3)      # write each value on top of its bar

plt.title("Salary by Employee")
plt.xlabel("Employee")
plt.ylabel("Salary")
plt.ylim(0, 90000)
plt.tight_layout()
plt.show()
```

`plt.bar()` takes the category labels and the bar heights. It returns the bars, which `plt.bar_label()` uses to print their values. Starting the y-axis at zero, as here, keeps the bar heights honest, since a truncated axis makes small differences look large.

**Summaries from `groupby`.** Bar charts pair naturally with the grouping tools from Guideline 37:

```python
avg = df.groupby("department")["salary"].mean()

plt.figure(figsize=(5, 4))
plt.bar(avg.index, avg.values, color=["tab:green", "tab:blue", "tab:orange"])
plt.title("Average Salary by Department")
plt.ylabel("Average salary")
plt.show()
```

**Horizontal bars.** When the labels are long, `barh` turns the chart on its side so they stay readable:

```python
plt.figure(figsize=(6, 4))
plt.barh(df["name"], df["salary"], color="salmon")
plt.title("Salary by Employee (Horizontal)")
plt.xlabel("Salary")
plt.show()
```

**Grouped bars.** To place several series side by side, shift each series left or right of the tick position by half a bar width:

```python
quarters = ["Q1", "Q2", "Q3", "Q4"]
team_a = [20, 35, 30, 35]
team_b = [25, 32, 34, 20]

positions = np.arange(len(quarters))
width = 0.35

plt.figure(figsize=(7, 4))
plt.bar(positions - width / 2, team_a, width, label="Team A")
plt.bar(positions + width / 2, team_b, width, label="Team B")
plt.xticks(positions, quarters)
plt.title("Quarterly Results")
plt.legend()
plt.show()
```

**Stacked bars.** To stack one series on top of another, tell the second one where to start with `bottom`:

```python
plt.figure(figsize=(7, 4))
plt.bar(quarters, team_a, label="Team A")
plt.bar(quarters, team_b, bottom=team_a, label="Team B")
plt.title("Combined Results")
plt.legend()
plt.show()
```

Pandas offers a shortcut too: `avg.plot(kind="bar")` produces a bar chart directly from a Series.

## 38.4 Scatter Plots

A **scatter plot** draws one dot per observation, positioned by two numeric values. It is the standard way to look for a relationship between two variables. The remaining sections use a synthetic table of 120 students, with the hours each studied, the hours each slept, a group label, and an exam score. The random generator is seeded, so everyone gets the same data:

```python
rng = np.random.default_rng(42)
n = 120

students = pd.DataFrame({
    "hours": rng.uniform(0, 10, n).round(1),
    "sleep": rng.normal(7, 1, n).round(1),
    "group": rng.choice(["A", "B", "C"], n),
})

offset = students["group"].map({"A": 0, "B": 5, "C": -4})
students["score"] = (45 + 4 * students["hours"] + 2 * (students["sleep"] - 7)
                     + offset + rng.normal(0, 6, n)).clip(0, 100).round(1)

print(students.head())
```

Now plot score against hours studied:

```python
plt.figure(figsize=(6, 4.5))
plt.scatter(students["hours"], students["score"], alpha=0.7, edgecolor="black")
plt.title("Exam Score vs Hours Studied")
plt.xlabel("Hours studied")
plt.ylabel("Score")
plt.grid(True, alpha=0.3)
plt.show()
```

The dots climb from the lower left to the upper right, meaning that more study time goes with higher scores. The `alpha` setting makes the dots slightly transparent, so overlapping points remain visible.

**Adding a trend line.** `np.polyfit()` (a NumPy tool) fits a straight line through the points, and the result can be drawn on top:

```python
slope, intercept = np.polyfit(students["hours"], students["score"], 1)
xs = np.linspace(0, 10, 100)

plt.figure(figsize=(6, 4.5))
plt.scatter(students["hours"], students["score"], alpha=0.6)
plt.plot(xs, slope * xs + intercept, color="red", linewidth=2,
         label=f"trend: {slope:.1f} points per hour")
plt.title("Score vs Hours, With Trend Line")
plt.xlabel("Hours studied")
plt.ylabel("Score")
plt.legend()
plt.show()
```

The slope comes out close to 4, the value the data was generated with.

**Coloring by category.** To separate the groups, draw one scatter call per group, so that each gets its own color and legend entry:

```python
plt.figure(figsize=(6, 4.5))
for group, part in students.groupby("group"):
    plt.scatter(part["hours"], part["score"], label=f"Group {group}", alpha=0.7)

plt.title("Score vs Hours by Group")
plt.xlabel("Hours studied")
plt.ylabel("Score")
plt.legend()
plt.show()
```

**Coloring by a number.** To show a third numeric variable, pass it to `c` and choose a colormap. A **colorbar** acts as the legend:

```python
plt.figure(figsize=(7, 4.5))
points = plt.scatter(students["hours"], students["score"],
                     c=students["sleep"], cmap="viridis", s=50)
plt.colorbar(points, label="Sleep (hours)")
plt.title("Score vs Hours, Colored by Sleep")
plt.xlabel("Hours studied")
plt.ylabel("Score")
plt.show()
```

The size of the dots can also carry a number, through `s`, so a single chart can show up to four variables at once, although it becomes harder to read.

| Argument | Meaning |
|---|---|
| `s` | Marker size (one number, or a list with one size per point) |
| `c` | Color (one color, or a list of numbers mapped through `cmap`) |
| `cmap` | Colormap name, such as `"viridis"`, `"plasma"`, `"coolwarm"` |
| `alpha` | Transparency |
| `marker` | Marker shape |
| `edgecolor` | Outline color of each marker |

## 38.5 Pie Charts

A **pie chart** shows how a whole divides into parts. It works best with a handful of categories that add up to a meaningful total. Here, the employees are split by department:

```python
counts = df["department"].value_counts()      # IT: 3, Sales: 2, HR: 1

plt.figure(figsize=(5, 5))
plt.pie(counts,
        labels=counts.index,
        autopct="%1.1f%%",
        startangle=90,
        colors=["tab:blue", "tab:orange", "tab:green"],
        explode=[0.05, 0, 0])
plt.title("Employees by Department")
plt.show()
```

The important arguments:

| Argument | Effect |
|---|---|
| `labels` | Text placed next to each slice |
| `autopct="%1.1f%%"` | Write each slice's percentage on it (one decimal place) |
| `startangle=90` | Where the first slice begins; 90 starts at the top |
| `explode=[0.05, 0, 0]` | Pull chosen slices out of the pie, one value per slice |
| `colors` | A color for each slice |
| `counterclock=False` | Draw the slices clockwise |

Here, IT takes half the pie (50.0%), Sales a third (33.3%), and HR the remaining sixth (16.7%).

A **donut chart** is a pie with the center removed. It is made by setting the width of the slices:

```python
plt.figure(figsize=(5, 5))
plt.pie(counts, labels=counts.index, autopct="%1.0f%%",
        startangle=90, wedgeprops={"width": 0.45})
plt.title("Employees by Department (Donut)")
plt.show()
```

A word of caution: people are much better at comparing bar lengths than slice angles. If there are more than five or six categories, or the slices are similar in size, a bar chart communicates more clearly.

## 38.6 Histograms

A **histogram** shows the *distribution* of one numeric variable. It sorts the values into equal-width ranges called **bins**, and draws one bar per bin whose height is the number of values that fall inside it. Unlike a bar chart, the bars touch, because the horizontal axis is a continuous scale.

```python
plt.figure(figsize=(6, 4))
counts, edges, patches = plt.hist(students["score"], bins=15,
                                  color="skyblue", edgecolor="black")

plt.axvline(students["score"].mean(), color="red", linestyle="--", label="mean")
plt.title("Distribution of Exam Scores")
plt.xlabel("Score")
plt.ylabel("Number of students")
plt.legend()
plt.show()

print(counts.sum())     # 120.0    every student falls in exactly one bin
print(len(edges))       # 16       15 bins need 16 edges
```

`plt.hist()` also returns the numbers behind the chart: the count in each bin, and the bin edges. The **bin count** matters. Too few bins hide the shape, and too many make it jagged. Try several values, or pass `bins="auto"` and let Matplotlib choose.

**Overlaying a fitted curve.** If `density=True`, the bar heights are rescaled so that the total area is 1, which allows a probability curve to share the same axes. Here the normal distribution from `scipy.stats` (Guideline 35) is laid over the histogram:

```python
from scipy import stats

scores = students["score"]
mu, sigma = scores.mean(), scores.std()
xs = np.linspace(scores.min(), scores.max(), 200)

plt.figure(figsize=(6, 4))
plt.hist(scores, bins=15, density=True, alpha=0.6, edgecolor="black")
plt.plot(xs, stats.norm.pdf(xs, mu, sigma), color="red", linewidth=2, label="normal curve")
plt.title("Scores With a Fitted Normal Curve")
plt.xlabel("Score")
plt.ylabel("Density")
plt.legend()
plt.show()
```

**Comparing groups.** Draw several histograms in the same axes with transparency, so that each can be seen through the others:

```python
plt.figure(figsize=(6, 4))
for group, part in students.groupby("group"):
    plt.hist(part["score"], bins=12, alpha=0.5, label=f"Group {group}")

plt.title("Scores by Group")
plt.xlabel("Score")
plt.ylabel("Number of students")
plt.legend()
plt.show()
```

| Argument | Meaning |
|---|---|
| `bins` | A number of bins, a list of exact edges, or `"auto"` |
| `range` | Only use values within `(low, high)` |
| `density` | If `True`, scale so the area equals 1 |
| `cumulative` | If `True`, each bar shows the running total |
| `histtype` | `"bar"` (default), `"step"` (outline only), `"stepfilled"` |
| `alpha`, `color`, `edgecolor` | Appearance |

If you only need the counts and not the picture, NumPy's `np.histogram(data, bins=5)` returns them without drawing anything.

## 38.7 Box Plots

A **box plot** (or box-and-whisker plot) summarizes a distribution in five numbers, and it is especially good at comparing several groups side by side. The pieces are the same quartiles that `describe()` reported in Guideline 37:

| Part | What it shows |
|---|---|
| **Line inside the box** | The median (50th percentile) |
| **The box** | From the 25th to the 75th percentile, so it holds the middle half of the data. Its height is the **IQR** (interquartile range) |
| **Whiskers** | Extend to the furthest values within 1.5 × IQR of the box |
| **Dots beyond the whiskers** | **Outliers**: unusually high or low values |

```python
groups = ["A", "B", "C"]
data = [students.loc[students["group"] == g, "score"] for g in groups]

plt.figure(figsize=(6, 4))
plt.boxplot(data, patch_artist=True, showmeans=True)
plt.xticks([1, 2, 3], groups)

plt.title("Scores by Group")
plt.xlabel("Group")
plt.ylabel("Score")
plt.grid(True, axis="y", alpha=0.3)
plt.show()
```

`plt.boxplot()` takes a list with one dataset per box, and the boxes are placed at positions 1, 2, 3. The `plt.xticks()` line then replaces those numbers with the group names. Setting `patch_artist=True` allows the boxes to be filled with color, and `showmeans=True` adds a marker for each mean. Comparing the medians shows Group B scoring above Group A, and Group C below, as the data was built to do.

The outlier rule can also be applied by hand, using pandas, to find out which points would be drawn as dots:

```python
q1, q3 = students["score"].quantile([0.25, 0.75])
iqr = q3 - q1
lower, upper = q1 - 1.5 * iqr, q3 + 1.5 * iqr

outliers = students[(students["score"] < lower) | (students["score"] > upper)]
print(len(outliers))    # how many students the box plot would mark as outliers
```

With pandas, `students.boxplot(column="score", by="group")` draws a grouped box plot in one line. The `showfliers=False` option hides the outlier dots, and `notch=True` adds a notch around the median, a rough visual test of whether two medians differ.

## 38.8 Curves from Equations

To draw a curve from a formula, calculate the formula for many x-values, then plot the pairs. Matplotlib only draws short straight segments between points, so the trick to a smooth curve is to use plenty of points, and `np.linspace()` (Guideline 34) makes that easy:

```python
x = np.linspace(-5, 5, 200)      # 200 evenly spaced values from -5 to 5
y = x**2

plt.figure(figsize=(6, 4))
plt.plot(x, y, color="purple", linewidth=2)
plt.title("y = x²")
plt.xlabel("x")
plt.ylabel("y")
plt.grid(True, alpha=0.3)
plt.show()
```

The NumPy functions from Guideline 34 work on whole arrays, so any formula built from them can be plotted the same way. Here, two trigonometric curves share one chart, with the axis ticks and legend written in mathematical notation. Text between dollar signs is rendered as math:

```python
x = np.linspace(0, 2 * np.pi, 300)

plt.figure(figsize=(7, 4))
plt.plot(x, np.sin(x), label=r"$\sin(x)$")
plt.plot(x, np.cos(x), label=r"$\cos(x)$", linestyle="--")
plt.axhline(0, color="black", linewidth=0.8)

plt.xticks([0, np.pi / 2, np.pi, 3 * np.pi / 2, 2 * np.pi],
           ["0", r"$\pi/2$", r"$\pi$", r"$3\pi/2$", r"$2\pi$"])
plt.title("Sine and Cosine")
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()
```

**Parametric curves.** Some curves are easier to describe with a helper variable `t` that drives both x and y. A circle is the classic case:

```python
t = np.linspace(0, 2 * np.pi, 300)

plt.figure(figsize=(4.5, 4.5))
plt.plot(np.cos(t), np.sin(t))
plt.axis("equal")        # same scale on both axes, so the circle is not squashed
plt.title("A Circle")
plt.show()
```

**Curves with a break.** A function like `1/x` shoots off to infinity near 0, and a plain plot draws a false line across the gap. Replacing the huge values with `nan` (not a number) tells Matplotlib to lift the pen:

```python
x = np.linspace(-5, 5, 1000)
y = 1 / x
y[np.abs(y) > 10] = np.nan

plt.figure(figsize=(6, 4))
plt.plot(x, y)
plt.ylim(-10, 10)
plt.title("y = 1/x")
plt.show()
```

**Shading the area under a curve.** `fill_between()` colors the region between a curve and the axis. This is a picture of a definite integral: the area under `sin(x)` from 0 to π is exactly 2, as the integration in Guidelines 35 and 36 confirmed:

```python
x = np.linspace(0, np.pi, 200)

plt.figure(figsize=(6, 4))
plt.plot(x, np.sin(x), color="black")
plt.fill_between(x, np.sin(x), alpha=0.3)
plt.title("Area Under sin(x) From 0 to π Equals 2")
plt.show()
```

**Plotting a SymPy formula.** Guideline 36 ended with `lambdify`, which turns a symbolic expression into a fast NumPy function. This is the natural bridge to plotting. Here, a function and its exact derivative are drawn together, and the points where the slope is zero are marked:

```python
import sympy as sym

x_sym = sym.symbols("x")
f_expr = x_sym**3 - 3 * x_sym
slope_expr = sym.diff(f_expr, x_sym)                       # 3*x**2 - 3

f = sym.lambdify(x_sym, f_expr, "numpy")
slope = sym.lambdify(x_sym, slope_expr, "numpy")
critical = [float(c) for c in sym.solve(slope_expr, x_sym)]   # [-1.0, 1.0]

xs = np.linspace(-2.5, 2.5, 300)

plt.figure(figsize=(7, 4.5))
plt.plot(xs, f(xs), label="f(x) = x³ − 3x")
plt.plot(xs, slope(xs), linestyle="--", label="f'(x)")
plt.scatter(critical, [f(c) for c in critical], color="red", zorder=3, label="slope = 0")
plt.axhline(0, color="black", linewidth=0.8)
plt.ylim(-5, 5)
plt.title("A Function and Its Derivative")
plt.legend()
plt.show()
```

The red points sit at the peak and the valley of the cubic, exactly where the dashed derivative curve crosses zero. The `zorder=3` setting draws the points on top of the lines. The same approach works for the solution of a differential equation from Guideline 35: plot `sol.t` against `sol.y[0]`.

## 38.9 Subplots

Often several charts belong together in one figure. `plt.subplots(rows, columns)` creates a grid of Axes, and returns the figure and the array of axes. Each Axes is then drawn on with the object-oriented methods from 38.1:

```python
fig, axes = plt.subplots(2, 2, figsize=(10, 8))

axes[0, 0].plot(months, sales_2026, marker="o")
axes[0, 0].set_title("Monthly Sales")

axes[0, 1].bar(df["name"], df["salary"], color="steelblue")
axes[0, 1].set_title("Salary by Employee")
axes[0, 1].tick_params(axis="x", rotation=45)

axes[1, 0].scatter(students["hours"], students["score"], alpha=0.6)
axes[1, 0].set_title("Score vs Hours")
axes[1, 0].set_xlabel("Hours studied")

axes[1, 1].hist(students["score"], bins=15, edgecolor="black")
axes[1, 1].set_title("Score Distribution")

fig.suptitle("Dashboard", fontsize=16)
fig.tight_layout()
plt.show()
```

With a 2 × 2 grid, `axes` is a two-dimensional array indexed as `axes[row, column]`. With a single row or column it is one-dimensional (`axes[0]`, `axes[1]`), and `axes.flat` steps through all of them in a loop.

**Sharing axes.** When plots use the same scale, `sharey=True` (or `sharex=True`) keeps the axes aligned and removes the repeated labels:

```python
fig, (left, right) = plt.subplots(1, 2, figsize=(9, 4), sharey=True)

left.hist(students.loc[students["group"] == "A", "score"], bins=10)
left.set_title("Group A")
right.hist(students.loc[students["group"] == "B", "score"], bins=10, color="tab:orange")
right.set_title("Group B")

left.set_ylabel("Number of students")
fig.tight_layout()
plt.show()
```

**Method names differ slightly on an Axes.** Most `plt.` commands that set a property become `set_` methods:

| pyplot | On an Axes |
|---|---|
| `plt.plot(...)`, `plt.bar(...)`, `plt.scatter(...)` | `ax.plot(...)`, `ax.bar(...)`, `ax.scatter(...)` |
| `plt.title("...")` | `ax.set_title("...")` |
| `plt.xlabel("...")` | `ax.set_xlabel("...")` |
| `plt.xlim(a, b)` | `ax.set_xlim(a, b)` |
| `plt.xticks(...)` | `ax.set_xticks(...)`, `ax.tick_params(...)` |
| `plt.legend()`, `plt.grid()` | `ax.legend()`, `ax.grid()` |
| `plt.suptitle("...")` | `fig.suptitle("...")` |

If a grid has more cells than charts, hide the empty ones with `axes[1, 1].axis("off")`.

**Uneven layouts.** For panels of different sizes, `plt.subplot_mosaic()` lets you sketch the layout as text, where each letter is one panel and repeated letters span several cells:

```python
fig, panels = plt.subplot_mosaic("AAB;CCB", figsize=(9, 5))

panels["A"].plot(months, sales_2025)
panels["A"].set_title("Wide panel")
panels["B"].hist(students["score"], bins=10, orientation="horizontal")
panels["B"].set_title("Tall panel")
panels["C"].bar(quarters, team_a)
panels["C"].set_title("Wide panel below")

fig.tight_layout()
plt.show()
```

## 38.10 3D Plots

Matplotlib can draw in three dimensions, by creating an Axes with the `"3d"` projection. From then on, the usual methods gain a third coordinate:

```python
t = np.linspace(0, 4 * np.pi, 300)

fig = plt.figure(figsize=(6, 5))
ax = fig.add_subplot(projection="3d")

ax.plot(np.cos(t), np.sin(t), t)         # a helix: x, y, and z all given
ax.set_xlabel("x")
ax.set_ylabel("y")
ax.set_zlabel("z")
ax.set_title("A Helix")
plt.show()
```

**3D scatter.** Each point takes three coordinates. Here, all three variables of the student data are shown together:

```python
fig = plt.figure(figsize=(7, 6))
ax = fig.add_subplot(projection="3d")

points = ax.scatter(students["hours"], students["sleep"], students["score"],
                    c=students["score"], cmap="viridis")
ax.set_xlabel("Hours studied")
ax.set_ylabel("Hours slept")
ax.set_zlabel("Score")
fig.colorbar(points, shrink=0.6, label="Score")
plt.show()
```

**Surfaces.** To draw a surface `z = f(x, y)`, first build a grid of coordinates with `np.meshgrid()`, which turns two 1D arrays into two 2D arrays holding every (x, y) combination. Then calculate z across the whole grid at once. The bowl-shaped function from Guideline 36 makes a good example, with the minimum that `solve` found at (2, −3) marked in red:

```python
def g(x, y):
    return x**2 + y**2 - 4 * x + 6 * y

X, Y = np.meshgrid(np.linspace(-2, 6, 60), np.linspace(-8, 2, 60))
Z = g(X, Y)

fig = plt.figure(figsize=(7, 6))
ax = fig.add_subplot(projection="3d")

surface = ax.plot_surface(X, Y, Z, cmap="viridis", alpha=0.85)
ax.scatter(2, -3, g(2, -3), color="red", s=60)      # the minimum, at z = -13
fig.colorbar(surface, shrink=0.6, label="g(x, y)")

ax.set_xlabel("x")
ax.set_ylabel("y")
ax.set_zlabel("g(x, y)")
ax.view_init(elev=25, azim=-60)      # camera angle: elevation and rotation, in degrees
plt.show()
```

The `view_init()` method sets the camera angle, and in a window with an interactive backend, you can also rotate the plot with the mouse.

**A flat alternative: contour plots.** A surface is impressive but often hard to read. A **contour plot** shows the same information as a top-down map, where each line or color band marks one height, just like the contour lines of a hiking map:

```python
plt.figure(figsize=(6, 5))
filled = plt.contourf(X, Y, Z, levels=20, cmap="viridis")
plt.colorbar(filled, label="g(x, y)")
plt.scatter(2, -3, color="red", label="minimum")
plt.title("Contour View of the Same Function")
plt.xlabel("x")
plt.ylabel("y")
plt.legend()
plt.show()
```

| Method | Draws |
|---|---|
| `ax.plot(x, y, z)` | A line through 3D points |
| `ax.scatter(x, y, z)` | Points in 3D space |
| `ax.plot_surface(X, Y, Z)` | A solid, colored surface |
| `ax.plot_wireframe(X, Y, Z)` | The same surface as a see-through mesh |
| `ax.contour(X, Y, Z)` | Contour lines drawn inside a 3D view |
| `ax.view_init(elev, azim)` | Set the camera angle |
| `plt.contourf(X, Y, Z)` | Filled contours in 2D |

## 38.11 Seaborn

**Seaborn** builds statistical charts on top of Matplotlib. The main differences from what you have used so far:

- It works directly with **pandas tables**. You pass the whole table with `data=` and refer to columns by name, instead of extracting each one.
- The `hue` argument colors the data by a category, and builds the legend automatically.
- It calculates statistics for you, such as averages with confidence intervals, density curves, and regression lines.
- Its default look is more polished.

It is imported with the alias `sns`, and one call sets the overall theme for every chart that follows:

```python
import seaborn as sns

sns.set_theme(style="whitegrid", palette="deep", font_scale=1.1)
```

| Setting | Options |
|---|---|
| `style` | `"darkgrid"`, `"whitegrid"`, `"dark"`, `"white"`, `"ticks"` |
| `palette` | `"deep"`, `"muted"`, `"pastel"`, `"bright"`, `"colorblind"`, `"Set2"`, `"viridis"` |
| `font_scale` | A multiplier for all text (1.0 is the default) |

**Two kinds of function.** Seaborn's functions come in two families, and knowing which is which explains most surprises:

| | Axes-level | Figure-level |
|---|---|---|
| Behavior | Draws onto one Axes (the current one, or the one passed as `ax=`) | Creates its own whole figure |
| Examples | `scatterplot`, `lineplot`, `barplot`, `histplot`, `boxplot`, `heatmap` | `relplot`, `displot`, `catplot`, `lmplot`, `pairplot`, `jointplot` |
| Strength | Combines with Matplotlib and subplots | Splits into panels with `col=` and `row=` |
| Returns | The Axes | A grid object (`FacetGrid`) |

The same scatter chart written both ways:

```python
sns.scatterplot(data=students, x="hours", y="score", hue="group")    # axes-level: one chart
plt.show()

sns.relplot(data=students, x="hours", y="score", hue="group", col="group")   # figure-level: one panel per group
plt.show()
```

**Key methods, by the question they answer.**

| Question | Functions |
|---|---|
| How do two numbers relate? | `scatterplot`, `lineplot`, `relplot` |
| How is one number distributed? | `histplot`, `kdeplot`, `ecdfplot`, `displot` |
| How do categories compare? | `barplot`, `countplot`, `boxplot`, `violinplot`, `stripplot`, `swarmplot`, `pointplot`, `catplot` |
| Is there a linear trend? | `regplot`, `lmplot` |
| How correlated is everything? | `heatmap`, `clustermap` |
| What does the whole table look like? | `pairplot`, `jointplot` |

**Common arguments.**

| Argument | Meaning |
|---|---|
| `data` | The pandas table |
| `x`, `y` | Column names for the axes |
| `hue` | Column that decides the color |
| `style`, `size` | Columns that decide the marker shape or size (scatter and line plots) |
| `col`, `row` | Column that splits a figure-level plot into panels |
| `palette` | Colors to use for the `hue` categories |
| `order`, `hue_order` | The order in which categories appear |
| `estimator` | The summary statistic for bars and lines (default: mean) |
| `errorbar` | The uncertainty drawn: `("ci", 95)` (default), `"sd"`, or `None` |
| `ax` | The Matplotlib Axes to draw on (axes-level functions only) |

Seaborn also includes small example datasets, loaded with `sns.load_dataset("tips")`, `"penguins"`, `"iris"`, `"titanic"`, and others. They are downloaded from the internet on first use, so the examples here use the student and employee tables instead.

## 38.12 Drawing Plots with Seaborn

Every example below reuses the `students` and `df` tables from earlier in this guideline.

**Scatter plots and regression.** With `hue` and `style`, one call shows the group by both color and marker shape:

```python
plt.figure(figsize=(6.5, 4.5))
sns.scatterplot(data=students, x="hours", y="score", hue="group", style="group", s=60)
plt.title("Score vs Hours by Group")
plt.show()
```

`regplot` adds a fitted regression line with a shaded confidence band, and `lmplot` does the same separately for each group:

```python
plt.figure(figsize=(6, 4.5))
sns.regplot(data=students, x="hours", y="score", scatter_kws={"alpha": 0.5})
plt.title("Score vs Hours, With Regression Line")
plt.show()

sns.lmplot(data=students, x="hours", y="score", hue="group", height=4.5, aspect=1.3)
plt.show()
```

**Line plots.** When several observations share the same x-value, `lineplot` draws their mean and shades the uncertainty around it:

```python
sales = pd.DataFrame({
    "month":   list(range(1, 7)) * 2,
    "region":  ["North"] * 6 + ["South"] * 6,
    "revenue": [10, 12, 15, 14, 18, 21, 8, 9, 13, 15, 17, 16],
})

plt.figure(figsize=(6.5, 4))
sns.lineplot(data=sales, x="month", y="revenue", hue="region", marker="o")
plt.title("Revenue by Region")
plt.show()
```

**Bar plots and counts.** `barplot` computes the mean of `y` for each category (this replaces the `groupby` step we used in Matplotlib), and `countplot` simply counts rows:

```python
plt.figure(figsize=(5.5, 4))
sns.barplot(data=df, x="department", y="salary", hue="department",
            errorbar=None, palette="pastel", legend=False)
plt.title("Average Salary by Department")
plt.show()

plt.figure(figsize=(5.5, 4))
sns.countplot(data=df, x="city", order=df["city"].value_counts().index)
plt.title("Employees by City")
plt.show()
```

By default, `barplot` also draws error bars, and here `errorbar=None` turns them off, because each department has only a few employees. Assigning `hue` to the same column as `x` gives each bar its own color, and `legend=False` hides the redundant legend.

**Distributions.** `histplot` is Seaborn's histogram, with `kde=True` adding a smooth density curve. `kdeplot` draws the smooth curve on its own, which makes it easier to compare groups:

```python
plt.figure(figsize=(6.5, 4))
sns.histplot(data=students, x="score", hue="group", bins=15, kde=True)
plt.title("Score Distribution by Group")
plt.show()

plt.figure(figsize=(6.5, 4))
sns.kdeplot(data=students, x="score", hue="group", fill=True, common_norm=False)
plt.title("Score Density by Group")
plt.show()
```

**Box plots and violin plots.** A **violin plot** is a box plot whose outline is the density curve, so it also shows *where* the values cluster. Drawing the raw points over a box plot combines the summary with the detail:

```python
plt.figure(figsize=(6, 4.5))
sns.boxplot(data=students, x="group", y="score", hue="group", legend=False)
sns.stripplot(data=students, x="group", y="score", color="black", alpha=0.35, size=3)
plt.title("Scores by Group, With Every Student Shown")
plt.show()

plt.figure(figsize=(6, 4.5))
sns.violinplot(data=students, x="group", y="score", hue="group", inner="quartile", legend=False)
plt.title("Scores by Group (Violin)")
plt.show()
```

Calling two axes-level functions in a row draws both onto the same Axes, which is how the box plot and the points end up together.

**Heatmaps.** A **heatmap** shows a table of numbers as colored cells. Its most common use is a **correlation matrix**, which shows how strongly each pair of columns moves together, from −1 (opposite) through 0 (unrelated) to +1 (identical):

```python
corr = students[["hours", "sleep", "score"]].corr()

plt.figure(figsize=(5, 4))
sns.heatmap(corr, annot=True, fmt=".2f", cmap="coolwarm", vmin=-1, vmax=1)
plt.title("Correlation Matrix")
plt.show()
```

Setting `annot=True` prints each value inside its cell. The `vmin` and `vmax` settings keep the color scale fixed between −1 and +1. In this data, hours studied correlates strongly with score, and sleep only weakly.

**Whole-table overviews.** `pairplot` draws every numeric column against every other in a single call, with histograms on the diagonal, and `jointplot` combines a scatter plot with the histograms of both variables:

```python
sns.pairplot(students, hue="group", corner=True)
plt.show()

sns.jointplot(data=students, x="hours", y="score", kind="reg")
plt.show()
```

**Panels with figure-level functions.** The `col` argument splits any figure-level chart into one panel per category:

```python
sns.displot(data=students, x="score", col="group", kde=True, bins=12, height=3.5)
plt.show()

sns.catplot(data=students, x="group", y="score", kind="violin", height=4)
plt.show()
```

**Combining Seaborn with Matplotlib.** Axes-level functions accept `ax=`, so they can fill the panels of a `plt.subplots()` grid. And since every Seaborn chart is a Matplotlib object, the customizing tools from earlier sections still apply:

```python
fig, axes = plt.subplots(1, 2, figsize=(11, 4))

sns.histplot(data=students, x="score", kde=True, ax=axes[0])
axes[0].set_title("All Scores")

sns.boxplot(data=students, x="group", y="score", hue="group", legend=False, ax=axes[1])
axes[1].set_title("By Group")
axes[1].set_xlabel("Group")

fig.tight_layout()
plt.show()
```

A figure-level function returns a grid object instead. Customize it through that object, and save it with its own `savefig`:

```python
grid = sns.displot(data=students, x="score", hue="group", kde=True)
grid.set_axis_labels("Exam score", "Number of students")
grid.figure.suptitle("Scores by Group", y=1.02)
grid.savefig("scores_by_group.png", dpi=300)
plt.show()
```

**Which chart should I draw?** A quick guide to matching the question to the chart:

| You want to... | Use |
|---|---|
| See a trend over time | Line graph (`plt.plot`, `sns.lineplot`) |
| Compare values across categories | Bar graph (`plt.bar`, `sns.barplot`) |
| Show the relationship between two numbers | Scatter plot (`plt.scatter`, `sns.scatterplot`, `sns.regplot`) |
| Show how one number is distributed | Histogram (`plt.hist`, `sns.histplot`) |
| Compare distributions across groups | Box plot or violin plot (`plt.boxplot`, `sns.boxplot`, `sns.violinplot`) |
| Show parts of a whole (few categories) | Pie or donut chart (`plt.pie`), or a stacked bar |
| Plot a formula | Line graph over `np.linspace` values |
| See how all the numeric columns relate | Heatmap of `corr()`, or `sns.pairplot` |
| Show a function of two variables | Surface or contour plot |

---

Matplotlib and Seaborn cover most of what you will need to draw. The habits that matter most are the same ones from the rest of the guidebook: start from a clear question, choose the chart that answers it, and label everything, because a chart with no title, no axis labels, and no units is a puzzle rather than an answer. Use Matplotlib when you need control over each element, use Seaborn when you want a statistical chart from a pandas table in one line, and combine them freely, since they draw on the same canvas.
