# Guideline 34: Numerical Computation with Python

Python's built-in lists are flexible, but they were never designed for heavy number crunching. Multiplying a million numbers, adding two tables of data together, or solving a set of equations with a plain list means writing loops, and loops in Python are slow. Numerical computation calls for a different tool: one that stores numbers compactly, and applies an operation to every element at once instead of one at a time.

That tool is **NumPy** (Numerical Python), the foundation of almost the entire Python data science ecosystem. `pandas`, `scikit-learn`, `matplotlib`, `TensorFlow`, and `PyTorch` all build on top of it, so the ideas in this guideline (arrays, shapes, indexing, and matrix operations) will keep appearing in every library you meet from here on.

## 34.1 The NumPy Library

NumPy is an external library, so it must be installed once per environment (Guideline 25 covered how):

```bash
pip install numpy
```

By universal convention, it is imported with the alias `np`:

```python
import numpy as np
```

The central object in NumPy is the **`ndarray`** (n-dimensional array), usually just called an *array*. It looks similar to a list, but differs in three important ways:

| Property | Python list | NumPy array |
|---|---|---|
| Element types | Can mix (`[1, "a", 2.5]`) | All the same type (`int`, `float`, etc.) |
| Speed on large data | Slow (Python loops) | Fast (operations run in optimized C code) |
| Math behavior | `*` repeats, `+` joins | `*` and `+` operate on each element |

That last row is the one that surprises beginners:

```python
lst = [1, 2, 3]
arr = np.array([1, 2, 3])

print(lst * 2)   # [1, 2, 3, 1, 2, 3]   (repetition)
print(arr * 2)   # [2 4 6]              (element-wise math)
```

Applying an operation to a whole array without writing a loop is called **vectorization**, and it is the reason NumPy exists.

## 34.2 Create an Array or Matrix

**From normal Python values.** Pass a list to `np.array()` to get a 1D array. Pass a list of lists to get a 2D array, which is a **matrix**:

```python
a = np.array([1, 2, 3, 4])
m = np.array([[1, 2, 3],
              [4, 5, 6]])

print(a)   # [1 2 3 4]
print(m)
# [[1 2 3]
#  [4 5 6]]
```

**Zeros and ones.** These are handy for creating a blank structure to fill in later. The argument is the shape:

```python
print(np.zeros(3))         # [0. 0. 0.]
print(np.zeros((2, 3)))
# [[0. 0. 0.]
#  [0. 0. 0.]]

print(np.ones((2, 2)))
# [[1. 1.]
#  [1. 1.]]
```

Note the decimal points: by default, `zeros` and `ones` create floating-point numbers.

**`arange`.** The NumPy version of `range()`, except it returns an array and accepts decimal steps:

```python
print(np.arange(5))          # [0 1 2 3 4]
print(np.arange(1, 10, 2))   # [1 3 5 7 9]
print(np.arange(0, 1, 0.25)) # [0.   0.25 0.5  0.75]
```

**`linspace`.** Instead of choosing a step size, you choose *how many* evenly spaced points you want between a start and a stop, and both endpoints are included:

```python
print(np.linspace(0, 1, 5))   # [0.   0.25 0.5  0.75 1.  ]
```

Use `arange` when you know the step, and `linspace` when you know the count. `linspace` is the standard choice for generating x-values for a plot.

**Random values.** NumPy has its own random tools (compare with the `random` module from Guideline 19), and they can fill entire arrays at once:

```python
np.random.seed(42)   # makes the "random" results repeatable

print(np.random.rand(2, 3))              # floats between 0 and 1, shape (2, 3)
print(np.random.randint(1, 10, size=(2, 3)))  # integers from 1 to 9
print(np.random.randn(3))                # values from a standard normal distribution
```

| Function | Produces |
|---|---|
| `np.random.rand(r, c)` | Uniform floats in [0, 1) |
| `np.random.randint(low, high, size)` | Random integers (`high` is excluded) |
| `np.random.randn(r, c)` | Normally distributed floats (mean 0, std 1) |
| `np.random.seed(n)` | Fixes the sequence so results are reproducible |

## 34.3 Return the Size of an Array

Three tools describe how big an array is, and they answer three different questions:

```python
m = np.array([[1, 2, 3],
              [4, 5, 6]])

print(len(m))     # 2       -> number of rows (length of the first dimension)
print(m.size)     # 6       -> total number of elements
print(m.shape)    # (2, 3)  -> (rows, columns)
```

| Tool | Answers | For a 2 × 3 matrix |
|---|---|---|
| `len(arr)` | How long is the first dimension? | `2` |
| `arr.size` | How many elements in total? | `6` |
| `arr.shape` | What is the size in every dimension? | `(2, 3)` |

`shape` is the one you will use most. It returns a tuple, so `m.shape[0]` is the number of rows and `m.shape[1]` is the number of columns. Two more attributes worth knowing:

```python
print(m.ndim)    # 2       -> number of dimensions
print(m.dtype)   # int64   -> the data type of the elements
```

## 34.4 Reshape or Flatten an Array

**Reshape** rearranges the same elements into a new shape. The one rule is that the total number of elements must stay the same:

```python
a = np.arange(1, 7)      # [1 2 3 4 5 6]

print(a.reshape(2, 3))
# [[1 2 3]
#  [4 5 6]]

print(a.reshape(3, 2))
# [[1 2]
#  [3 4]
#  [5 6]]
```

Asking for a shape that doesn't fit, such as `a.reshape(4, 2)` on six elements, raises a `ValueError`. Passing `-1` for one dimension tells NumPy to work that dimension out for you:

```python
print(a.reshape(2, -1))   # 2 rows, columns calculated automatically -> shape (2, 3)
print(a.reshape(-1, 1))   # a single column of 6 rows -> shape (6, 1)
```

**Flatten** does the opposite: it collapses any array into 1D.

```python
m = np.array([[1, 2, 3],
              [4, 5, 6]])

print(m.flatten())   # [1 2 3 4 5 6]
print(m.ravel())     # [1 2 3 4 5 6]
print(m.reshape(-1)) # [1 2 3 4 5 6]
```

The difference between the first two is subtle: `flatten()` always returns an independent *copy*, while `ravel()` returns a *view* of the original data when it can, which is faster but means editing the result may change the original.

## 34.5 Indexing and Slicing

Indexing and slicing work just as they did with lists in Guideline 11: counting starts at 0, negative indices count from the end, and slices exclude the stop position.

**1D arrays** use `array[start:stop:step]`:

```python
a = np.array([10, 20, 30, 40, 50, 60])

print(a[0])       # 10
print(a[-1])      # 60
print(a[1:4])     # [20 30 40]
print(a[::2])     # [10 30 50]
print(a[::-1])    # [60 50 40 30 20 10]
```

**Matrices** add a second position, written `array[row, column]` (note the single set of brackets with a comma, instead of the `m[row][col]` chain used for lists of lists):

```python
m = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])

print(m[0, 0])       # 1     -> row 0, column 0
print(m[1, 2])       # 6     -> row 1, column 2
print(m[2, -1])      # 9     -> last row, last column
```

Each of the two positions accepts a full slice, and a bare `:` means "everything":

```python
print(m[1, :])       # [4 5 6]        -> the whole second row
print(m[:, 1])       # [2 5 8]        -> the whole second column
print(m[0:2, 1:3])   # [[2 3]
                     #  [5 6]]       -> rows 0-1, columns 1-2
print(m[::-1])       # rows in reverse order
```

Slices of a NumPy array are **views**, not copies. Changing a slice changes the original array. Use `.copy()` when you need an independent version:

```python
part = m[0, :].copy()
```

Arrays can also be indexed with a condition, which returns only the elements that satisfy it. This is called **boolean indexing**:

```python
a = np.array([10, 20, 30, 40, 50, 60])
print(a[a > 30])        # [40 50 60]
print(a[a % 20 == 0])   # [20 40 60]
```

## 34.6 Common Types of Matrices

Certain matrix shapes appear so often in mathematics and machine learning that they have names, and NumPy has a shortcut for each.

**Square matrix.** A matrix with an equal number of rows and columns (shape `(n, n)`). Most of the special matrices below, along with the determinant, inverse, and eigenvalues later in this guideline, only exist for square matrices.

```python
m = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])

print(m.shape)   # (3, 3)
```

**Diagonal matrix.** A square matrix whose entries outside the main diagonal (top-left to bottom-right) are all zero. `np.diag()` builds one from a list of diagonal values:

```python
print(np.diag([1, 2, 3]))
# [[1 0 0]
#  [0 2 0]
#  [0 0 3]]
```

`np.diag()` also works in reverse: given an existing matrix, it extracts the main diagonal as a 1D array.

```python
print(np.diag(m))   # [1 5 9]
```

**Identity matrix.** A square matrix with **1s on the main diagonal** and 0s everywhere else. It is the matrix version of the number 1: multiplying any matrix by an identity matrix of a matching size leaves it unchanged.

```python
print(np.identity(3))
# [[1. 0. 0.]
#  [0. 1. 0.]
#  [0. 0. 1.]]
```

`np.eye(3)` produces the same result and is the more common spelling in practice.

**Triangular matrix.** A square matrix in which everything on one side of the main diagonal is zero. **`np.triu()`** keeps the *upper* triangle and zeroes the rest (an upper triangular matrix), and **`np.tril()`** keeps the *lower* triangle (a lower triangular matrix):

```python
print(np.triu(m))
# [[1 2 3]
#  [0 5 6]
#  [0 0 9]]

print(np.tril(m))
# [[1 0 0]
#  [4 5 0]
#  [7 8 9]]
```

**Transpose matrix.** Swaps the rows and the columns, so the first row becomes the first column, and so on. This works on any matrix, not only square ones:

```python
print(np.transpose(m))     # or the shorthand: m.T
# [[1 4 7]
#  [2 5 8]
#  [3 6 9]]

r = np.array([[1, 2, 3],
              [4, 5, 6]])
print(r.shape, r.T.shape)   # (2, 3) (3, 2)
```

| Type | Definition | NumPy |
|---|---|---|
| Square | rows == columns | any array with shape `(n, n)` |
| Diagonal | non-zero only on the main diagonal | `np.diag()` |
| Identity | 1s on the main diagonal, 0s elsewhere | `np.identity()`, `np.eye()` |
| Upper triangular | zeros below the main diagonal | `np.triu()` |
| Lower triangular | zeros above the main diagonal | `np.tril()` |
| Transpose | rows and columns swapped | `np.transpose()`, `.T` |

## 34.7 Common Operations of Matrices

For most operations, NumPy accepts either the ordinary operator or an equivalent function. The examples below use two small matrices:

```python
a = np.array([[1, 2],
              [3, 4]])
b = np.array([[5, 6],
              [7, 8]])
```

**Addition and subtraction.** `+` and `-`, or `np.add()` and `np.subtract()`. The operation is applied to matching positions, so both matrices need the same shape:

```python
print(a + b)              # or np.add(a, b)
# [[ 6  8]
#  [10 12]]

print(a - b)              # or np.subtract(a, b)
# [[-4 -4]
#  [-4 -4]]
```

**Division.** `/` or `np.divide()`, also position by position, and the result is always a float:

```python
print(a / b)              # or np.divide(a, b)
# [[0.2        0.33333333]
#  [0.42857143 0.5       ]]
```

Dividing by zero does not crash the program. NumPy prints a warning and returns `inf` (or `nan` for 0/0), so check your data if you see those values.

**Element-wise multiplication.** `*` or `np.multiply()`. Each element is multiplied by the element in the *same position* of the other matrix. This is **not** matrix multiplication:

```python
print(a * b)              # or np.multiply(a, b)
# [[ 5 12]
#  [21 32]]
```

**Scalar multiplication.** Multiplying a matrix by a single number scales every element:

```python
print(a * 3)
# [[ 3  6]
#  [ 9 12]]
```

**Dot product.** `np.dot()`. For two 1D arrays, it multiplies matching elements and adds up the results:

```python
u = np.array([1, 2, 3])
v = np.array([4, 5, 6])

print(np.dot(u, v))   # 1*4 + 2*5 + 3*6 = 32
```

When both inputs are 2D matrices, `np.dot()` performs true **matrix multiplication**: each entry in the result is the dot product of a row from the first matrix and a column from the second. The `@` operator does the same thing and is the more modern spelling:

```python
print(np.dot(a, b))       # or a @ b
# [[19 22]
#  [43 50]]
```

The top-left value, 19, comes from `1*5 + 2*7`. For matrix multiplication to work, the number of columns in the first matrix must equal the number of rows in the second.

**Outer product.** `np.outer()`. Multiplies *every* element of one array by *every* element of the other, producing a matrix:

```python
print(np.outer(u, v))
# [[ 4  5  6]
#  [ 8 10 12]
#  [12 15 18]]
```

| Operation | Operator | Function | Result shape |
|---|---|---|---|
| Addition | `+` | `np.add()` | same as inputs |
| Subtraction | `-` | `np.subtract()` | same as inputs |
| Division | `/` | `np.divide()` | same as inputs |
| Element-wise multiplication | `*` | `np.multiply()` | same as inputs |
| Scalar multiplication | `*` with a number | `np.multiply()` | same as the matrix |
| Dot product / matrix multiplication | `@` | `np.dot()` | scalar for 1D, matrix for 2D |
| Outer product | (none) | `np.outer()` | `(len(u), len(v))` |

## 34.8 Math Operations on Matrices

**Pi and Euler's number.** These are stored as constants, so they are written *without* parentheses:

```python
print(np.pi)   # 3.141592653589793
print(np.e)    # 2.718281828459045
```

**Sum, mean, and median.**

```python
data = np.array([4, 9, 16, 25])

print(np.sum(data))      # 54
print(np.mean(data))     # 13.5
print(np.median(data))   # 12.5
```

**Minimum and maximum.**

```python
print(np.min(data))      # 4
print(np.max(data))      # 25
```

**Variance and standard deviation.**

```python
print(np.var(data))      # 62.25
print(np.std(data))      # 7.889866919029074 (approximately)
```

By default, NumPy calculates the *population* variance (dividing by *n*). For the *sample* variance used in most statistics courses, pass `ddof=1`: `np.var(data, ddof=1)`.

**Square root and absolute value.**

```python
print(np.sqrt(data))                    # [2. 3. 4. 5.]
print(np.abs(np.array([-3, 2, -1])))    # [3 2 1]
```

**Power and exponential.** `np.power()` raises each element to a power, and `np.exp()` calculates *e* raised to each element:

```python
print(np.power(data, 2))                # [ 16  81 256 625]
print(np.exp(np.array([0, 1, 2])))      # [1.         2.71828183 7.3890561 ]
```

**Logarithms.** `np.log()` is the natural log (base *e*), the other two use base 2 and base 10:

```python
print(np.log(np.array([1, np.e, np.e**2])))   # [0. 1. 2.]
print(np.log2(np.array([1, 2, 4, 8])))        # [0. 1. 2. 3.]
print(np.log10(np.array([1, 10, 100, 1000]))) # [0. 1. 2. 3.]
```

**Working along rows and columns.** On a matrix, aggregate functions take an `axis` argument. Without it, they combine every element into a single number:

```python
m = np.array([[1, 2, 3],
              [4, 5, 6],
              [7, 8, 9]])

print(np.sum(m))           # 45      -> everything
print(np.sum(m, axis=0))   # [12 15 18] -> down each column
print(np.sum(m, axis=1))   # [ 6 15 24] -> across each row
print(np.mean(m, axis=0))  # [4. 5. 6.]
```

A simple way to remember: `axis=0` collapses the rows (leaving one value per column), and `axis=1` collapses the columns (leaving one value per row). The same argument works for `mean`, `median`, `min`, `max`, `var`, and `std`.

| Function | Purpose |
|---|---|
| `np.sum()`, `np.mean()`, `np.median()` | Total, average, middle value |
| `np.min()`, `np.max()` | Smallest and largest value |
| `np.var()`, `np.std()` | Spread of the data |
| `np.sqrt()`, `np.abs()` | Square root, absolute value (per element) |
| `np.power()`, `np.exp()` | Powers and exponentials (per element) |
| `np.log()`, `np.log2()`, `np.log10()` | Logarithms (per element) |

## 34.9 Determinant

The **determinant** is a single number calculated from a square matrix. Among other things, it tells you whether the matrix can be inverted, and how much it scales area when used as a transformation. For a 2 × 2 matrix, the formula is simply `ad - bc`:

```
[[a, b],
 [c, d]]   →   a*d - b*c
```

NumPy's linear algebra tools live in the `np.linalg` submodule:

```python
A = np.array([[4, 3],
              [6, 3]])

det = np.linalg.det(A)
print(round(det, 2))   # -6.0     (4*3 - 3*6 = 12 - 18)
```

Floating-point arithmetic means results are sometimes slightly off (for example `-5.999999999999999`), so it is good practice to round the output when displaying it.

A determinant of **zero** is special. It means the matrix is **singular**: its rows are not independent, and it has no inverse.

```python
S = np.array([[1, 2],
              [2, 4]])    # second row is just 2 x the first
print(np.linalg.det(S))   # 0.0
```

## 34.10 Inverse Matrix

The **inverse** of a matrix `A`, written A⁻¹, is the matrix that undoes it: multiplying `A` by its inverse produces the identity matrix. It is the matrix equivalent of dividing, and it exists only for square matrices whose determinant is not zero.

```python
A = np.array([[4, 7],
              [2, 6]])

A_inv = np.linalg.inv(A)
print(A_inv)
# [[ 0.6 -0.7]
#  [-0.2  0.4]]

print(np.round(A @ A_inv, 2))   # check: should be the identity matrix
# [[1. 0.]
#  [0. 1.]]
```

Trying to invert a singular matrix raises an error, which you can handle with the exception techniques from Guideline 23:

```python
try:
    np.linalg.inv(S)
except np.linalg.LinAlgError as e:
    print("Cannot invert:", e)   # Cannot invert: Singular matrix
```

## 34.11 System of Linear Equations

A set of linear equations can be written compactly as **A x = b**, where `A` holds the coefficients, `x` holds the unknowns, and `b` holds the results. Take these two equations:

```
2x +  y = 5
 x -  y = 1
```

```python
A = np.array([[2,  1],
              [1, -1]])
b = np.array([5, 1])

x = np.linalg.solve(A, b)
print(x)   # [2. 1.]   -> x = 2, y = 1
```

The same function scales to any number of unknowns. Here is a three-variable system:

```
 x +  y +  z =  6
      2y + 5z = -4
2x + 5y -  z = 27
```

```python
A = np.array([[1, 1,  1],
              [0, 2,  5],
              [2, 5, -1]])
b = np.array([6, -4, 27])

print(np.round(np.linalg.solve(A, b), 2))   # [ 5.  3. -2.]
```

There is another route, because if `A x = b` then `x = A⁻¹ b`:

```python
x = np.linalg.inv(A) @ b
```

This gives the same answer, but `np.linalg.solve()` is faster and more numerically stable, so it is the preferred choice. As with the inverse, `solve()` raises a `LinAlgError` if the matrix is singular, meaning the system has no single unique solution.

## 34.12 Eigenvalues and Eigenvectors

Most vectors change direction when a matrix acts on them. An **eigenvector** of a matrix is a special vector that does not: the matrix only stretches or shrinks it, and the amount of stretching is the **eigenvalue**. In symbols:

```
A v = λ v
```

where `v` is the eigenvector and `λ` (lambda) is its eigenvalue. Eigenvalues and eigenvectors sit behind principal component analysis (PCA), Google's original PageRank, vibration analysis, and much of machine learning.

```python
A = np.array([[4, 1],
              [2, 3]])

values, vectors = np.linalg.eig(A)

print(values)    # [5. 2.]
print(vectors)
# [[ 0.70710678 -0.4472136 ]
#  [ 0.70710678  0.89442719]]
```

`np.linalg.eig()` returns two things: an array of eigenvalues, and a matrix whose **columns** are the matching eigenvectors. So the eigenvalue `5` pairs with the first column, `[0.707, 0.707]`, and the eigenvalue `2` pairs with the second column. The order of the values, and the signs of the vectors, can vary between machines.

You can verify the definition directly:

```python
v = vectors[:, 0]                 # first eigenvector (a column)
print(A @ v)                      # [3.53553391 3.53553391]
print(values[0] * v)              # [3.53553391 3.53553391]  -> identical
```

For symmetric matrices, `np.linalg.eigh()` is the faster and more reliable version, and it returns the eigenvalues sorted in ascending order.

## 34.13 Other Methods

A quick reference of other NumPy tools you will meet often, with `a`, `b` as arrays and `m` as a matrix:

| Method | Purpose | Example |
|---|---|---|
| `np.full()` | Array filled with one chosen value | `np.full((2, 2), 7)` |
| `np.concatenate()` | Join arrays along an existing axis | `np.concatenate([a, b])` |
| `np.vstack()` / `np.hstack()` | Stack arrays vertically / horizontally | `np.vstack([a, b])` |
| `np.append()` | Add values to the end of an array | `np.append(a, 99)` |
| `np.sort()` | Return a sorted copy | `np.sort(a)` |
| `np.argsort()` | Indices that would sort the array | `np.argsort(a)` |
| `np.unique()` | Sorted unique values | `np.unique(a)` |
| `np.where()` | Positions where a condition is true | `np.where(a > 3)` |
| `np.argmax()` / `np.argmin()` | Index of the largest / smallest value | `np.argmax(a)` |
| `np.cumsum()` | Running total | `np.cumsum(a)` |
| `np.round()` | Round to a number of decimals | `np.round(a, 2)` |
| `np.clip()` | Limit values to a range | `np.clip(a, 0, 10)` |
| `np.any()` / `np.all()` | True if any / all elements meet a condition | `np.all(a > 0)` |
| `np.percentile()` | Value below which a percentage of data falls | `np.percentile(a, 90)` |
| `np.corrcoef()` | Correlation between arrays | `np.corrcoef(a, b)` |
| `np.trace()` | Sum of the main diagonal | `np.trace(m)` |
| `np.linalg.norm()` | Length (magnitude) of a vector | `np.linalg.norm(a)` |
| `np.linalg.matrix_rank()` | Number of independent rows/columns | `np.linalg.matrix_rank(m)` |
| `np.isnan()` | Detect missing (`nan`) values | `np.isnan(a)` |
| `np.copy()` | Independent copy of an array | `np.copy(a)` |

---

NumPy turns Python from a general-purpose language into a serious numerical tool. The habits from this guideline are worth carrying forward: think in whole arrays rather than single numbers, check `shape` whenever something looks wrong, and reach for a vectorized function before writing a loop. Nearly every library in the remaining guidelines expects you to be comfortable with arrays and matrices, and now you are.
