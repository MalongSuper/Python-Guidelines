# Guideline 35: Scientific Computation with Python

NumPy gave Python fast arrays and the basic operations on them. But real scientific work asks for much more than adding and multiplying matrices: describing a dataset statistically, modeling chance with probability distributions, integrating a function, solving a differential equation, finding the shortest route through a network. Writing each of those algorithms from scratch would take months, and getting the numerical details right would take longer.

**SciPy** (Scientific Python) is the library that already did that work. It sits on top of NumPy and bundles decades of tested numerical algorithms, many of them written in Fortran and C, behind a clean Python interface. This guideline tours the parts of SciPy you will reach for most often.

## 35.1 The SciPy Library

SciPy is an external library and needs NumPy to run. Install it once per environment:

```bash
pip install scipy
```

By convention it is imported with the alias `sp`, and its tools are grouped into **submodules** that are reached with a dot:

```python
import numpy as np
import scipy as sp

print(sp.__version__)
print(sp.stats.norm(0, 1).mean())   # 0.0  -> sp.stats is a submodule
```

You will also see the alternative style, which imports a single submodule or function directly:

```python
from scipy import stats
from scipy.integrate import quad
```

Both styles are equivalent, and this guideline uses the `sp.` style for consistency. Each submodule covers one area of scientific computing:

| Submodule | What it covers |
|---|---|
| `sp.stats` | Descriptive statistics, probability distributions, statistical tests |
| `sp.sparse` | Sparse matrices, and `sp.sparse.csgraph` for graph algorithms |
| `sp.integrate` | Numerical integration and differential equations |
| `sp.differentiate` | Numerical derivatives |
| `sp.optimize` | Minimization, curve fitting, root finding |
| `sp.linalg` | Linear algebra (a superset of `np.linalg`) |
| `sp.interpolate` | Filling in values between known data points |
| `sp.fft` | Fourier transforms |
| `sp.signal` | Signal processing and filtering |
| `sp.spatial` | Distances, nearest neighbors, geometry |
| `sp.constants` | Physical and mathematical constants |

SciPy works directly with NumPy arrays: what goes in is an array, and what comes out is an array, so everything from Guideline 34 carries over. A note on versions: SciPy evolves steadily, and a few tools in this guideline (such as `sp.differentiate`) require a recent release. If something is missing, run `pip install --upgrade scipy`.

## 35.2 Sparse vs Dense Matrix

A **dense** matrix, which is what NumPy gives you, stores every single entry, including all the zeros. That is wasteful when most of the entries *are* zeros. Think of a table recording which of a million users watched which of ten thousand videos: nearly every cell is empty. A **sparse** matrix stores only the non-zero values and remembers where they belong, which can shrink memory use by orders of magnitude and speed up calculations by skipping the zeros.

Here is a small matrix used throughout this section:

```python
import numpy as np
import scipy as sp

dense = np.array([[1, 0, 0, 2],
                  [0, 0, 3, 0],
                  [4, 0, 0, 5]])
```

SciPy offers several sparse **formats**, and each stores the same information in a different arrangement. The three most important are COO, CSR, and CSC.

**COO (Coordinate format)** stores three parallel lists: the row of each non-zero value, its column, and the value itself. It is the simplest format and the easiest to build:

```python
coo = sp.sparse.coo_array(dense)

print(coo.row)    # [0 0 1 2 2]
print(coo.col)    # [0 3 2 0 3]
print(coo.data)   # [1 2 3 4 5]
```

A COO matrix can also be built directly from those three lists, without ever creating the dense version, which is how large sparse matrices are usually constructed:

```python
row  = np.array([0, 0, 1, 2, 2])
col  = np.array([0, 3, 2, 0, 3])
vals = np.array([1, 2, 3, 4, 5])

coo2 = sp.sparse.coo_array((vals, (row, col)), shape=(3, 4))
print(np.array_equal(coo2.toarray(), dense))   # True
```

**CSR (Compressed Sparse Row)** stores the values row by row. Instead of a row number for every value, it keeps a compact pointer array marking where each row starts:

```python
csr = sp.sparse.csr_array(dense)

print(csr.data)      # [1 2 3 4 5]   the non-zero values
print(csr.indices)   # [0 3 2 0 3]   the column of each value
print(csr.indptr)    # [0 2 3 5]     row 0 = data[0:2], row 1 = data[2:3], row 2 = data[3:5]
```

**CSC (Compressed Sparse Column)** is the same idea turned on its side, with values stored column by column:

```python
csc = sp.sparse.csc_array(dense)

print(csc.data)      # [1 4 3 2 5]
print(csc.indices)   # [0 2 1 0 2]   the row of each value
print(csc.indptr)    # [0 2 2 3 5]   column 1 is empty, so its range has zero length
```

| Format | Best for | Weak at |
|---|---|---|
| COO | Building a matrix; converting between formats | Arithmetic and indexing |
| CSR | Row slicing; matrix-vector products; most arithmetic | Changing the structure; column slicing |
| CSC | Column slicing; many linear-algebra solvers | Changing the structure; row slicing |

A common workflow is to build in COO, then convert to CSR or CSC for the actual computation:

```python
csr = coo.tocsr()
csc = coo.tocsc()
back_to_coo = csr.tocoo()
dense_again = csr.toarray()     # a normal NumPy array
```

SciPy also has an older family of classes named `coo_matrix`, `csr_matrix`, and `csc_matrix`. They still work, but the `_array` versions used here are recommended for new code, because they follow NumPy's rules: `*` is element-wise and `@` is matrix multiplication, whereas the old `_matrix` classes used `*` for matrix multiplication.

**Inspecting a sparse matrix.** These tools count and describe what is stored:

```python
print(csr.shape)             # (3, 4)   the size of the full matrix
print(csr.data.shape)        # (5,)     only 5 values are actually stored
print(csr.count_nonzero())   # 5        how many entries are non-zero
print(csr.nnz)               # 5        how many entries are stored
print(csr.getnnz(axis=0))    # [2 0 1 2]   stored entries in each column
print(csr.getnnz(axis=1))    # [2 1 2]     stored entries in each row
```

`getnnz(axis=0)` counts down each column and `getnnz(axis=1)` counts across each row, following the same `axis` convention as NumPy. The pair `nnz` and `count_nonzero()` usually agree, but they differ when a sparse matrix contains *explicit zeros*, meaning zeros that were stored on purpose. `nnz` counts everything stored, while `count_nonzero()` counts only the values that are truly non-zero. The method `.eliminate_zeros()` removes explicit zeros to bring the two back in line.

**How much does it save?** Compare a 1000 × 1000 identity matrix in both forms:

```python
n = 1000
big_dense  = np.eye(n)
big_sparse = sp.sparse.eye_array(n, format="csr")

sparse_bytes = (big_sparse.data.nbytes
                + big_sparse.indices.nbytes
                + big_sparse.indptr.nbytes)

print(big_dense.nbytes)   # 8000000   (about 8 MB)
print(sparse_bytes)       # 16004     (about 16 KB)
```

Only the 1000 ones on the diagonal need to be remembered, so the sparse version is roughly 500 times smaller. As a rule of thumb, sparse formats pay off when the vast majority of entries (often 90% or more) are zero. For a mostly full matrix, a dense array is faster and simpler.

**Other operations.** Sparse matrices support most of the familiar operations, and the result usually stays sparse:

```python
print(csr.T.shape)                   # (4, 3)   transpose
print((csr * 2).toarray())           # scalar multiplication
print(csr.sum(axis=0))               # [5 0 3 7]   column sums
print(csr @ np.array([1, 1, 1, 1]))  # [3 3 9]     matrix-vector product
```

Sparse matrices have their own linear-algebra tools too. To solve a system of equations `A x = b` (as in Guideline 34), use `spsolve`, which prefers CSC or CSR input:

```python
from scipy.sparse.linalg import spsolve

A = sp.sparse.csc_array(np.array([[3, 0, 1],
                                  [0, 2, 0],
                                  [1, 0, 2]]))
b = np.array([5, 4, 5])

print(spsolve(A, b))   # [1. 2. 2.]
```

## 35.3 The `scipy.stats` Module

NumPy can compute a mean or a standard deviation, but `sp.stats` goes much further. This section covers the descriptive tools, and the next covers probability distributions. The examples use a small dataset whose mean is 5:

```python
data = np.array([2, 4, 4, 4, 5, 5, 7, 9])
```

**`describe`: several statistics at once.** A single call returns the count, range, mean, variance, skewness, and kurtosis:

```python
result = sp.stats.describe(data)

print(result.nobs)       # 8
print(result.minmax)     # (2, 9)
print(result.mean)       # 5.0
print(result.variance)   # 4.571428571428571
print(result.skewness)   # 0.65625
print(result.kurtosis)   # -0.21875
```

Note that the variance here is the *sample* variance (dividing by *n* − 1), unlike `np.var()`, which divides by *n* by default. On a matrix, `describe` accepts `axis=0` or `axis=1` to compute the statistics per column or per row.

**`mode`: the most frequent value.**

```python
result = sp.stats.mode(data)
print(result.mode)    # 4
print(result.count)   # 3     -> the value 4 appears 3 times
```

With a matrix, the `axis` argument chooses whether the mode is found down each column or across each row. If two values tie, the smaller one is returned:

```python
m = np.array([[1, 2, 3],
              [1, 5, 3],
              [4, 2, 3]])

result = sp.stats.mode(m, axis=0)
print(result.mode)    # [1 2 3]   most common value in each column
print(result.count)   # [2 2 3]
```

**Trimmed statistics: reducing the effect of outliers.** A single extreme value can drag the mean far from where most of the data sits. The trimmed versions ignore values outside a chosen range:

| Function | Computes |
|---|---|
| `sp.stats.tmean(a, limits)` | Mean of the values inside `limits` |
| `sp.stats.tvar(a, limits)` | Sample variance of the values inside `limits` |
| `sp.stats.tstd(a, limits)` | Standard deviation of the values inside `limits` |
| `sp.stats.tmin(a, lowerlimit)` | Smallest value at or above `lowerlimit` |
| `sp.stats.tmax(a, upperlimit)` | Largest value at or below `upperlimit` |

```python
scores = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 100])   # 100 is an outlier

print(np.mean(scores))                         # 14.5
print(sp.stats.tmean(scores, limits=(1, 10)))  # 5.0
print(sp.stats.tvar(scores, limits=(1, 10)))   # 7.5
print(sp.stats.tstd(scores, limits=(1, 10)))   # 2.7386127875258306
print(sp.stats.tmin(scores, lowerlimit=3))     # 3
print(sp.stats.tmax(scores, upperlimit=50))    # 9
```

The `t` functions trim by **value**: you supply the lower and upper limits, and anything outside is dropped. If you would rather trim by **proportion** (for instance, cut the top and bottom 10% of the data), use `trim_mean`:

```python
print(sp.stats.trim_mean(scores, 0.1))   # 5.5   -> drops the lowest 10% and highest 10%
```

Here 10% of ten values is one value, so the 1 and the 100 are removed, and the mean of 2 through 9 is 5.5.

**`skew`: which way does the data lean?** Skewness measures asymmetry:

| Skewness | Meaning |
|---|---|
| `> 0` | The long tail stretches to the right (a few unusually large values) |
| `< 0` | The long tail stretches to the left (a few unusually small values) |
| `= 0` | Symmetric around the center |

```python
symmetric    = np.array([1, 2, 3, 4, 5])
right_skewed = np.array([1, 2, 2, 3, 10])
left_skewed  = -right_skewed

print(sp.stats.skew(symmetric))      # 0.0
print(sp.stats.skew(right_skewed))   # about 1.36
print(sp.stats.skew(left_skewed))    # about -1.36
```

**`kurtosis`: how heavy are the tails?** Kurtosis measures "tailedness", meaning how often extreme values occur compared with a normal distribution. SciPy reports *excess* kurtosis, so a normal distribution scores 0:

| Kurtosis | Meaning |
|---|---|
| `> 0` | More outliers than a normal distribution (heavy tails) |
| `< 0` | Fewer outliers than a normal distribution (light tails) |
| `= 0` | Normal-like |

```python
flat  = np.array([1, 2, 3, 4, 5])
heavy = np.array([-10, 0, 0, 0, 0, 0, 0, 0, 0, 10])

print(sp.stats.kurtosis(flat))    # -1.3
print(sp.stats.kurtosis(heavy))   # 2.0

rng = np.random.default_rng(0)
bell = rng.normal(size=100_000)
print(round(sp.stats.kurtosis(bell), 1))   # about 0.0
```

## 35.4 Probability Distributions

A **probability distribution** describes how likely each possible outcome is. Distributions come in two families:

- **Discrete** distributions deal with countable outcomes (0, 1, 2, ... events). Each outcome has its own probability.
- **Continuous** distributions deal with measurements that can take any value in a range. Any single exact value has a probability of zero, so we talk about *density* and about probabilities over intervals.

SciPy represents each distribution as an object. You create it by passing the distribution's parameters, and then call methods on it:

| Distribution | Type | Models | Create with |
|---|---|---|---|
| Bernoulli | Discrete | A single trial: success or failure | `sp.stats.bernoulli(p)` |
| Binomial | Discrete | Number of successes in `n` independent trials | `sp.stats.binom(n, p)` |
| Poisson | Discrete | Number of events in a fixed time or space | `sp.stats.poisson(mu)` |
| Normal | Continuous | A bell-curve measurement | `sp.stats.norm(loc, scale)` |

The parameters are: `p` is the probability of success on one trial, `n` is the number of trials, `mu` is the average number of events, `loc` is the mean, and `scale` is the standard deviation (not the variance).

The methods you will use most are the same across distributions:

| Method | Returns | Used for |
|---|---|---|
| `pmf(k)` | The probability of *exactly* `k` events | Discrete only |
| `cdf(k)` | The probability of `≤ k` events (for whole numbers, the same as `< k + 1`) | Both |
| `sf(k)` | The probability of `> k` events (for whole numbers, the same as `≥ k + 1`) | Both |
| `pdf(m)` | The probability *density* at `x = m` | Continuous only |
| `mean()`, `var()`, `std()` | Mean, variance, and standard deviation | Both |

The name `sf` stands for *survival function*, and it always equals `1 - cdf`. The two methods split the outcomes into "up to k" and "beyond k", so together they cover every possibility.

**Bernoulli.** One coin flip that lands on success with probability 0.3:

```python
coin = sp.stats.bernoulli(0.3)

print(coin.pmf(1))           # 0.3     probability of success
print(coin.pmf(0))           # 0.7     probability of failure
print(coin.mean())           # 0.3
print(coin.var())            # 0.21
print(round(coin.std(), 4))  # 0.4583
```

**Binomial.** Flipping a fair coin 10 times and counting the heads:

```python
heads = sp.stats.binom(10, 0.5)

print(heads.pmf(5))    # 0.24609375   exactly 5 heads
print(heads.cdf(3))    # 0.171875     3 heads or fewer
print(heads.sf(3))     # 0.828125     4 heads or more
print(heads.mean())    # 5.0
print(heads.var())     # 2.5
print(round(heads.std(), 4))   # 1.5811
```

**Poisson.** A help desk that receives 3 calls per hour on average:

```python
calls = sp.stats.poisson(3)

print(round(calls.pmf(2), 4))   # 0.2240   exactly 2 calls
print(round(calls.cdf(2), 4))   # 0.4232   2 calls or fewer
print(round(calls.sf(2), 4))    # 0.5768   3 calls or more
print(calls.mean(), calls.var())   # 3.0 3.0
```

A distinctive property of the Poisson distribution is that its mean and variance are equal, and both are `mu`.

**Normal.** IQ scores, which are designed to follow a normal distribution with a mean of 100 and a standard deviation of 15:

```python
iq = sp.stats.norm(loc=100, scale=15)

print(round(iq.pdf(100), 4))    # 0.0266   density at the peak
print(round(iq.cdf(115), 4))    # 0.8413   fraction scoring 115 or lower
print(round(iq.sf(130), 4))     # 0.0228   fraction scoring above 130
print(iq.mean(), iq.var(), iq.std())   # 100.0 225.0 15.0
```

A `pdf` value is a *density*, not a probability. It can be compared between points (higher means more likely to be near there), but to get an actual probability you need an interval, which the `cdf` gives you by subtraction:

```python
between = iq.cdf(115) - iq.cdf(85)
print(round(between, 4))   # 0.6827   within one standard deviation of the mean
```

Calling `pmf` on a continuous distribution, or `pdf` on a discrete one, raises an error, because those methods do not exist for that type.

## 35.5 Calculus

SciPy handles the numerical side of calculus: it calculates derivatives, integrals, and solutions to differential equations by approximation, which is what you need whenever a problem has no tidy formula.

### Differentiation

`sp.differentiate.derivative()` estimates the derivative of a function at one or many points. The function must accept a NumPy array:

```python
def f(x):
    return x**3

res = sp.differentiate.derivative(f, 2.0)
print(round(float(res.df), 6))   # 12.0    since d/dx (x^3) = 3x^2 = 12 at x = 2
```

Passing an array evaluates the derivative at every point at once:

```python
x = np.array([0.0, 1.0, 2.0, 3.0])
res = sp.differentiate.derivative(f, x)
print(np.round(res.df, 6))   # [ 0.  3. 12. 27.]
```

The result is a small object: `.df` holds the derivative, `.error` an estimate of the numerical error, and `.success` says whether the calculation converged. For derivatives of measured data points (rather than a function), NumPy's `np.gradient()` is the usual tool. Older tutorials use `scipy.misc.derivative`, which has been removed from current versions of SciPy.

### Integration with `quad`

`quad()` computes a definite integral of a function, and returns two values: the answer and an estimate of its error.

```python
# The area under sin(x) from 0 to pi (exactly 2)
value, error = sp.integrate.quad(np.sin, 0, np.pi)
print(round(value, 6))   # 2.0
print(error)             # a tiny number, around 2e-14
```

Any Python function works, including `lambda` functions, and the limits can be infinite:

```python
value, _ = sp.integrate.quad(lambda x: x**2, 0, 3)
print(round(value, 6))   # 9.0

value, _ = sp.integrate.quad(lambda x: np.exp(-x), 0, np.inf)
print(round(value, 6))   # 1.0
```

### Integration from data points: `trapezoid` and `simpson`

When you have measurements rather than a formula, `quad` cannot be used, and the integral must be estimated from the sample points directly. `trapezoid()` connects neighboring points with straight lines, and `simpson()` fits parabolas through groups of points, which is usually more accurate for smooth data:

```python
x = np.linspace(0, 3, 7)   # 7 points, spaced 0.5 apart
y = x**2                   # the exact integral from 0 to 3 is 9

print(sp.integrate.trapezoid(y, x=x))   # 9.125
print(sp.integrate.simpson(y, x=x))     # 9.0
```

The trapezoid rule overshoots slightly here because the curve bends upward, while Simpson's rule is exact for a parabola like `x²`. The older names `trapz` and `simps` have been removed from current SciPy.

### Differential equations, initial value problems

A differential equation describes how a quantity *changes*, and solving it means recovering the quantity itself. In an **initial value problem (IVP)**, you know the starting state and want to follow it forward in time. `solve_ivp()` takes a function `fun(t, y)` returning the rate of change, a time span, and the starting values:

```python
# Decay: dy/dt = -2y, starting at y = 1. The exact answer is e^(-2t).
def decay(t, y):
    return -2 * y

sol = sp.integrate.solve_ivp(decay, (0, 1), [1.0],
                             t_eval=np.linspace(0, 1, 5),
                             rtol=1e-8, atol=1e-10)

print(sol.success)                 # True
print(sol.t)                       # [0.   0.25 0.5  0.75 1.  ]
print(round(sol.y[0, -1], 6))      # 0.135335   -> e^-2
```

The solution is stored in `sol.y`, with one row per variable and one column per time point. Problems with several variables simply return a list of rates. A frictionless spring, `y'' = -y`, is rewritten as two first-order equations, one for position and one for velocity:

```python
def spring(t, state):
    position, velocity = state
    return [velocity, -position]

sol = sp.integrate.solve_ivp(spring, (0, np.pi), [1.0, 0.0],
                             rtol=1e-8, atol=1e-10)

print(round(sol.y[0, -1], 4))   # -1.0    -> cos(pi)
```

### Differential equations, boundary value problems

In a **boundary value problem (BVP)**, you know the state at *both ends* of an interval instead of just the start, so the solver must find the curve that connects them. `solve_bvp()` needs the equations, a function describing the boundary conditions, and an initial guess on a grid of points.

```python
# y'' = -y, with y(0) = 0 and y(pi/2) = 1. The exact answer is sin(x).
def equations(x, y):
    return np.vstack((y[1], -y[0]))     # y[0] = y, y[1] = y'

def boundary(ya, yb):
    return np.array([ya[0], yb[0] - 1]) # y(0) = 0 and y(pi/2) = 1

x = np.linspace(0, np.pi / 2, 5)
guess = np.zeros((2, x.size))

sol = sp.integrate.solve_bvp(equations, boundary, x, guess)

print(sol.status)                               # 0   -> solved
print(np.round(sol.sol(np.pi / 4), 2))          # [0.71 0.71]   -> sin(pi/4) and cos(pi/4)
```

`sol.sol` is a function you can evaluate at any point inside the interval.

### Minimization

`sp.optimize.minimize()` searches for the input that makes a function as small as possible. You supply the function and a starting guess, and the function receives its inputs as a single array:

```python
def f(v):
    return (v[0] - 3)**2 + 2

res = sp.optimize.minimize(f, x0=[0.0])

print(np.round(res.x, 3))     # [3.]     the input at the minimum
print(round(res.fun, 3))      # 2.0      the smallest value reached
print(res.success)            # True
```

With several variables, `x0` simply gets more entries:

```python
def bowl(p):
    return (p[0] - 1)**2 + (p[1] + 2)**2

res = sp.optimize.minimize(bowl, x0=[0.0, 0.0])
print(np.round(res.x, 3))     # [ 1. -2.]
```

Limits on the inputs are passed with `bounds`:

```python
res = sp.optimize.minimize(f, x0=[0.0], bounds=[(0, 2)])
print(np.round(res.x, 3))     # [2.]    the closest allowed point to 3
print(round(res.fun, 3))      # 3.0
```

To maximize a function instead, minimize its negative. The `method` argument selects the algorithm, but the default, which is chosen automatically, is a good starting point.

## 35.6 Graphs

A **graph** is a set of nodes connected by edges. It models road networks, friendships, web links, and dependencies between tasks. In SciPy, graph algorithms live in `sp.sparse.csgraph`, and a graph is stored as an **adjacency matrix**: entry `(i, j)` holds the weight of the edge from node `i` to node `j`, and a missing edge is simply a zero, which makes sparse matrices a natural fit.

The examples below use a graph with seven nodes. Nodes 0 to 4 are joined by weighted edges, and nodes 5 and 6 form a separate island:

```python
from scipy.sparse import csgraph

row = [0, 0, 2, 1, 2, 3, 5]
col = [1, 2, 1, 3, 3, 4, 6]
w   = [4, 1, 2, 5, 8, 3, 7]     # weight of each edge

G = sp.sparse.csr_array((w, (row, col)), shape=(7, 7))
```

Each edge was entered once, as if it pointed from `row` to `col`. Passing `directed=False` to the algorithms makes them treat every edge as two-way.

**Connected components.** Which nodes can reach each other?

```python
n, labels = csgraph.connected_components(G, directed=False)

print(n)        # 2
print(labels)   # [0 0 0 0 0 1 1]
```

There are two components, and `labels` says which one each node belongs to. For directed graphs, the `connection` argument chooses between `"weak"` (ignore edge directions) and `"strong"` (follow them).

**Shortest paths with `dijkstra`.** Dijkstra's algorithm finds the cheapest route from a starting node to every other node, provided no edge weight is negative:

```python
dist, pred = csgraph.dijkstra(G, directed=False, indices=0,
                              return_predecessors=True)

print(dist)   # [ 0.  3.  1.  8. 11. inf inf]
print(pred)   # [-9999     2     0     1     3 -9999 -9999]
```

`dist` holds the total cost from node 0 to each node, and unreachable nodes get `inf`. The route to node 4 costs 11, and it is *not* the path with the fewest edges: going 0 → 2 → 1 → 3 → 4 beats the direct 0 → 1 edge, which costs 4 instead of 3. The `pred` array records each node's *predecessor* on its best path (with `-9999` meaning none), so the route itself can be rebuilt by walking backward:

```python
def get_path(pred, start, end):
    path = [end]
    while path[-1] != start:
        path.append(int(pred[path[-1]]))
    return path[::-1]

print(get_path(pred, 0, 4))   # [0, 2, 1, 3, 4]
```

**`shortest_path`: one function, several algorithms.** The general-purpose `shortest_path()` picks or accepts an algorithm through its `method` argument (`"D"` for Dijkstra, `"BF"` for Bellman-Ford, `"FW"` for Floyd-Warshall, `"J"` for Johnson, or `"auto"`):

```python
print(csgraph.shortest_path(G, method="D", directed=False, indices=0))
# [ 0.  3.  1.  8. 11. inf inf]
```

**`floyd_warshall`: every pair at once.** This returns a full table of distances between all pairs of nodes:

```python
D = csgraph.floyd_warshall(G, directed=False)

print(D.shape)   # (7, 7)
print(D[0])      # [ 0.  3.  1.  8. 11. inf inf]
print(D[0, 4])   # 11.0
```

**`bellman_ford`: negative weights allowed.** Dijkstra's algorithm gives wrong answers, and raises an error in SciPy, when an edge has a negative weight. Bellman-Ford handles them, as long as the graph is directed and has no *negative cycle* (a loop whose total weight is negative, which would let you lower the cost forever):

```python
H = np.array([[0,  4, 5],
              [0,  0, 0],
              [0, -2, 0]])      # 0 -> 1 costs 4, 0 -> 2 costs 5, 2 -> 1 costs -2

print(csgraph.bellman_ford(H, directed=True, indices=0))   # [0. 3. 5.]
```

The cheapest route to node 1 is 0 → 2 → 1, costing 5 + (−2) = 3, which beats the direct edge costing 4. A negative cycle raises `NegativeCycleError`.

| Algorithm | Function | Handles negative weights? | Finds |
|---|---|---|---|
| Dijkstra | `dijkstra` | No | Shortest paths from chosen source nodes |
| Bellman-Ford | `bellman_ford` | Yes (detects negative cycles) | Shortest paths from chosen source nodes |
| Floyd-Warshall | `floyd_warshall` | Yes | Shortest paths between all pairs |
| Johnson | `johnson` | Yes | Shortest paths between all pairs (good for sparse graphs) |

**Breadth-first and depth-first search.** These two traversals visit every node reachable from a starting point, and they differ in the order. **Breadth-first search (BFS)** explores level by level, finishing all the neighbors before moving deeper. **Depth-first search (DFS)** follows one branch as far as it goes before backing up. Take a small tree:

```
        0
       / \
      1   2
     / \   \
    3   4   5
```

```python
row = [0, 0, 1, 1, 2]
col = [1, 2, 3, 4, 5]
T = sp.sparse.csr_array((np.ones(5), (row, col)), shape=(6, 6))

bfs_order, bfs_pred = csgraph.breadth_first_order(T, 0, directed=False)
dfs_order, dfs_pred = csgraph.depth_first_order(T, 0, directed=False)

print(bfs_order)   # [0 1 2 3 4 5]   level by level
print(dfs_order)   # [0 1 3 4 2 5]   dives down the left branch first
print(bfs_pred)    # [-9999     0     0     1     1     2]
```

Each function returns the visiting order and the predecessor of each node. BFS is the natural choice for finding the fewest-edge route in an unweighted graph, while DFS is used for exploring structure, detecting cycles, and ordering tasks.

## 35.7 Other Methods

A few more tools that come up constantly.

**`zscore`: standardizing data.** A z-score says how many standard deviations a value sits from the mean, which makes values from different scales comparable:

```python
data = np.array([2, 4, 4, 4, 5, 5, 7, 9])

print(sp.stats.zscore(data))
# [-1.5 -0.5 -0.5 -0.5  0.   0.   1.   2. ]
```

Values with a z-score beyond about 3 in either direction are commonly flagged as outliers.

**`lu`: LU decomposition.** This factors a square matrix into a permutation matrix `P`, a lower triangular matrix `L`, and an upper triangular matrix `U`, such that `A = P @ L @ U`. It is the machinery behind solving linear systems and computing determinants (see the triangular matrices in Guideline 34):

```python
A = np.array([[2, 1, 1],
              [4, 3, 3],
              [8, 7, 9]])

P, L, U = sp.linalg.lu(A)

print(np.round(L, 3))
# [[1.    0.    0.   ]
#  [0.25  1.    0.   ]
#  [0.5   0.667 1.   ]]

print(np.round(U, 3))
# [[ 8.     7.     9.   ]
#  [ 0.    -0.75  -1.25 ]
#  [ 0.     0.    -0.667]]

print(np.allclose(P @ L @ U, A))   # True
```

**`linregress`: linear regression.** This fits the straight line `y = slope * x + intercept` that best matches your data points, and reports how well it does:

```python
x = np.array([1, 2, 3, 4, 5])
y = np.array([2.1, 3.9, 6.2, 7.8, 10.1])

fit = sp.stats.linregress(x, y)

print(round(fit.slope, 3))       # 1.99
print(round(fit.intercept, 3))   # 0.05
print(round(fit.rvalue**2, 4))   # 0.9973   the R-squared: closer to 1 means a better fit
print(fit.pvalue)                # a very small number -> the trend is unlikely to be chance
print(fit.stderr)                # uncertainty in the slope
```

Once you have the fitted line, predicting is a matter of plugging in a new `x`:

```python
print(round(fit.slope * 6 + fit.intercept, 2))   # 11.99
```

**`rvs`: random samples from a distribution.** Every distribution from section 35.4 can generate random values that follow it, using `rvs` (random variates). This is how simulations are built:

```python
iq = sp.stats.norm(loc=100, scale=15)
samples = iq.rvs(size=1000, random_state=42)

print(samples.shape)               # (1000,)
print(round(samples.mean()))       # about 100
print(round(samples.std()))        # about 15
```

The `random_state` argument fixes the seed so the results can be repeated, the same purpose as `np.random.seed()`.

**Quick reference.** More tools worth knowing:

| Method | Purpose | Example |
|---|---|---|
| `sp.stats.pearsonr()` | Correlation coefficient between two variables, with a p-value | `sp.stats.pearsonr(x, y)` |
| `sp.stats.ttest_ind()` | Test whether two groups have different means | `sp.stats.ttest_ind(a, b)` |
| `sp.stats.norm.ppf()` | Inverse of the `cdf`: the value below which a given fraction falls | `sp.stats.norm.ppf(0.975)` |
| `sp.stats.sem()` | Standard error of the mean | `sp.stats.sem(data)` |
| `sp.linalg.svd()` | Singular value decomposition, used in PCA and recommendation systems | `U, s, Vt = sp.linalg.svd(A)` |
| `sp.linalg.solve()`, `inv()`, `det()` | The linear algebra of Guideline 34, with extra options | `sp.linalg.solve(A, b)` |
| `sp.optimize.curve_fit()` | Fit any function you define to data | `sp.optimize.curve_fit(f, x, y)` |
| `sp.optimize.root()` | Find where a function equals zero | `sp.optimize.root(f, x0=1.0)` |
| `sp.interpolate.interp1d()` | Estimate values between known data points | `sp.interpolate.interp1d(x, y)` |
| `sp.fft.fft()` | Fast Fourier transform: find the frequencies inside a signal | `sp.fft.fft(signal)` |
| `sp.spatial.distance.euclidean()` | Straight-line distance between two points | `sp.spatial.distance.euclidean(p, q)` |
| `sp.constants.c` | Physical constants such as the speed of light | `sp.constants.c` |

---

SciPy is best understood as a well-stocked toolbox rather than a single tool. You do not need to memorize its contents, only to remember that most numerical problems, from a statistical test to a shortest route to a differential equation, already have a tested solution somewhere in it. When a problem arises, search the submodule list first, read the function's documentation, and check the result against a case whose answer you already know, as the examples in this guideline did. That habit of verifying numerical output is worth as much as any individual function.
