# Guideline 36: Symbolic Computation with Python

Everything so far in this part of the guidebook has been **numeric**. NumPy and SciPy take numbers in, do arithmetic, and hand numbers back, and those numbers are approximations. Ask NumPy for the square root of 8 and you get `2.8284271247461903`, a decimal that is close but not exact. That is perfect for data, simulations, and machine learning, but it is not how a mathematician works on paper. On paper, the square root of 8 is written `2√2`, the derivative of `x³` is written `3x²`, and the answer to an integral is a *formula*.

**Symbolic computation** does math the way you do it by hand: it manipulates symbols and formulas, and keeps every answer exact. Python's library for this is **SymPy**. It can expand and factor expressions, differentiate and integrate, take limits, expand functions into series, and solve equations, all with exact results and often with the general formula rather than a single number.

| | Numeric (NumPy, SciPy) | Symbolic (SymPy) |
|---|---|---|
| Works with | Numbers and arrays | Symbols and formulas |
| Answers are | Approximate decimals | Exact expressions |
| `√8` becomes | `2.8284271247461903` | `2*sqrt(2)` |
| Derivative of `x³` | A number at a chosen point | The formula `3*x**2` |
| Strength | Speed on large data | Exactness and general formulas |
| Weakness | Rounding error | Slow on large or messy problems |

The two approaches work well together: SymPy derives the formula, then NumPy evaluates it quickly over a million points, a hand-off shown in section 36.8.

## 36.1 The SymPy Library

SymPy is written in pure Python, and is installed with pip:

```bash
pip install sympy
```

The usual import alias for SymPy is `sp`, but the previous guideline used `sp` for SciPy, and mixing the two in one program would be confusing. This guideline therefore imports SymPy as `sym`:

```python
import numpy as np
import sympy as sym
```

**Symbols.** Before SymPy can do algebra with `x`, it must be told that `x` is a symbol rather than a Python variable holding a number. Symbols are created with `symbols()`, which can make several at once:

```python
x, y, z = sym.symbols("x y z")

expr = x**2 + 2*x + 1
print(expr)          # x**2 + 2*x + 1
```

Note that powers are written `**`, as in ordinary Python. The `^` operator is *not* exponentiation.

A symbol can also carry an **assumption** about what values it may take, which lets SymPy simplify more aggressively:

```python
a = sym.symbols("a", positive=True)

print(sym.sqrt(x**2))   # sqrt(x**2)   x could be negative, so it cannot simplify
print(sym.sqrt(a**2))   # a            a is positive, so it can
```

**Exact numbers.** SymPy keeps numbers exact, and offers special objects for fractions and mathematical constants:

```python
print(sym.sqrt(8))                          # 2*sqrt(2)
print(sym.sqrt(2)**2)                       # 2         (NumPy gives 2.0000000000000004)
print(sym.Rational(1, 10) + sym.Rational(2, 10))   # 3/10      (plain Python gives 0.30000000000000004)
```

| Object | Meaning |
|---|---|
| `sym.pi` | π |
| `sym.E` | Euler's number *e* |
| `sym.I` | The imaginary unit √−1 |
| `sym.oo` | Infinity (written with two lowercase letter o's) |
| `sym.Rational(a, b)` | The exact fraction a/b |

**Getting a decimal.** When a numeric answer is wanted, `evalf()` (or `sym.N()`) converts an exact expression to a decimal, to as many digits as you like:

```python
print(sym.pi.evalf())            # 3.14159265358979
print(sym.pi.evalf(20))          # 3.1415926535897932385
print(sym.N(sym.sqrt(2), 10))    # 1.414213562
```

**Substitution.** `subs()` replaces symbols with values or with other expressions:

```python
print(expr.subs(x, 3))              # 16
print(expr.subs(x, y + 1))          # (y + 1)**2 + 2*y + 3
print((x*y).subs({x: 2, y: 5}))     # 10
```

**Reshaping expressions.** Three functions do most of the day-to-day rearranging:

```python
print(sym.expand((x + 1)**2))                  # x**2 + 2*x + 1
print(sym.factor(x**2 + 2*x + 1))              # (x + 1)**2
print(sym.simplify(sym.sin(x)**2 + sym.cos(x)**2))   # 1
```

**A common trap: `==` is not "mathematically equal".** In SymPy, `==` checks whether two expressions have exactly the same *structure*, not whether they are algebraically equivalent:

```python
print((x + 1)**2 == x**2 + 2*x + 1)     # False   different structure
print(sym.simplify((x + 1)**2 - (x**2 + 2*x + 1)) == 0)   # True    equivalent
```

To *state* an equation (for solving, as in 36.7), use `sym.Eq(left, right)`.

**Reading input from text.** `sympify()` turns a string into an expression, which is handy for user input:

```python
print(sym.sympify("x**2 + 1"))   # x**2 + 1
```

**Printing nicely.** `print(expr)` gives plain text. In a Jupyter notebook, an expression on the last line of a cell is drawn as typeset math, and `sym.latex(expr)` returns LaTeX code for a document:

```python
print(sym.latex(sym.sqrt(x) / 2))   # \frac{\sqrt{x}}{2}
```

## 36.2 Derivative

`sym.diff(expression, variable)` differentiates. The result is a new **expression**: the derivative *function* as a formula, not just a number at one point (compare with `sp.differentiate.derivative` in Guideline 35, which approximates the value at a point).

```python
print(sym.diff(x**3 + 2*x, x))           # 3*x**2 + 2
print(sym.diff(sym.sin(x**2), x))        # 2*x*cos(x**2)         chain rule, applied automatically
print(sym.diff(x**2 * sym.sin(x), x))    # x**2*cos(x) + 2*x*sin(x)     product rule
print(sym.diff(sym.log(x), x))           # 1/x
```

SymPy applies the power, product, quotient, and chain rules for you. For a **higher-order** derivative, add the number of times after the variable:

```python
print(sym.diff(x**4, x, 2))   # 12*x**2    second derivative
print(sym.diff(x**4, x, 3))   # 24*x       third derivative
```

Because the result is a formula, it can be evaluated at any point with `subs()`:

```python
d = sym.diff(x**3, x)          # 3*x**2
print(d.subs(x, 2))            # 12
```

Derivatives can be written in an *unevaluated* form with `sym.Derivative`, and calculated later with `.doit()`, which is useful for displaying a derivative before showing its result:

```python
unevaluated = sym.Derivative(x**3, x)
print(unevaluated.doit())      # 3*x**2
```

**A worked example: finding a maximum and a minimum.** Symbolic derivatives make the classic calculus recipe almost mechanical. For `f(x) = x³ − 3x`, find where the slope is zero, then use the second derivative to classify each point:

```python
f = x**3 - 3*x

slope = sym.diff(f, x)                   # 3*x**2 - 3
critical = sym.solve(slope, x)           # [-1, 1]

curvature = sym.diff(f, x, 2)            # 6*x
for point in critical:
    print(point, curvature.subs(x, point))
# -1 -6     negative curvature -> a local maximum
#  1  6     positive curvature -> a local minimum
```

## 36.3 Integral

`sym.integrate()` is the inverse operation. With just a variable, it finds the **indefinite integral** (the antiderivative):

```python
print(sym.integrate(x**2, x))           # x**3/3
print(sym.integrate(sym.cos(x), x))     # sin(x)
print(sym.integrate(1/x, x))            # log(x)
print(sym.integrate(x * sym.exp(x), x)) # (x - 1)*exp(x)     integration by parts
```

SymPy leaves out the constant of integration `+ C`, so remember to add it yourself when writing a result by hand.

With a tuple `(variable, lower, upper)`, it finds a **definite integral**, and the answer is an exact value:

```python
print(sym.integrate(x**2, (x, 0, 3)))                  # 9
print(sym.integrate(sym.sin(x), (x, 0, sym.pi)))       # 2
print(sym.integrate(sym.sin(x)**2, (x, 0, sym.pi)))    # pi/2
```

Infinite limits use `sym.oo`, so improper integrals work too:

```python
print(sym.integrate(sym.exp(-x), (x, 0, sym.oo)))                # 1
print(sym.integrate(sym.exp(-x**2), (x, -sym.oo, sym.oo)))       # sqrt(pi)
```

The second result is the famous Gaussian integral, and `quad()` from Guideline 35 could only ever have given the decimal `1.7724538509055159`. SymPy gives the exact `√π`, and `.evalf()` converts it when a decimal is needed.

Some integrals have no answer in ordinary elementary functions. SymPy either expresses them with special functions, or returns the integral unevaluated:

```python
print(sym.integrate(sym.exp(-x**2), x))    # sqrt(pi)*erf(x)/2      the error function
print(sym.integrate(sym.sin(x) / x, x))    # Si(x)                  the sine integral
print(sym.integrate(x**x, x))              # Integral(x**x, x)      no closed form found
```

When SymPy hands back an `Integral(...)`, it is not an error. It means no formula was found, and you can still get a number from it with `.evalf()`, or by using the numeric tools from Guideline 35.

**Multiple integrals** just list one range after another:

```python
print(sym.integrate(x * y, (x, 0, 1), (y, 0, 2)))   # 1
```

**Checking an answer.** Differentiating an antiderivative should return the original function, which gives a built-in test:

```python
F = sym.integrate(x * sym.exp(x), x)
print(sym.simplify(sym.diff(F, x) - x * sym.exp(x)))   # 0
```

## 36.4 Partial Derivatives

A function of several variables changes in more than one direction. A **partial derivative** measures the change along one variable while all the others are held fixed. In SymPy, nothing new is needed: `diff()` simply takes the variable you want, and treats the others as constants.

```python
f = x**2 * y**3

print(sym.diff(f, x))      # 2*x*y**3      the rate of change in the x direction
print(sym.diff(f, y))      # 3*x**2*y**2   the rate of change in the y direction
```

**Higher and mixed derivatives.** List the variables in the order of differentiation, and repeat a variable, or give a count, to differentiate it more than once:

```python
print(sym.diff(f, x, 2))     # 2*y**3       second derivative with respect to x
print(sym.diff(f, x, y))     # 6*x*y**2     differentiate by x, then by y
print(sym.diff(f, y, x))     # 6*x*y**2     the order does not matter here
```

That the two mixed derivatives agree is not a coincidence: for smooth functions, the order of differentiation does not change the result.

**The gradient** collects all the first partial derivatives into a vector. It points in the direction in which the function rises fastest:

```python
gradient = sym.Matrix([sym.diff(f, v) for v in (x, y)])
print(gradient)                              # Matrix([[2*x*y**3], [3*x**2*y**2]])
print(gradient.subs({x: 1, y: 2}))           # Matrix([[16], [12]])
```

**The Hessian** collects all the second partial derivatives into a matrix, which describes the curvature of the function:

```python
print(sym.hessian(f, (x, y)))
# Matrix([[2*y**3, 6*x*y**2], [6*x*y**2, 6*x**2*y]])
```

**The Jacobian** does the same for a *vector* of functions, giving the matrix of every partial derivative of every output with respect to every input:

```python
F = sym.Matrix([x * y, x + y])
print(F.jacobian([x, y]))     # Matrix([[y, x], [1, 1]])
```

**Finding a minimum in two variables.** Just as with one variable, the minimum of a smooth function sits where the gradient is zero. Setting both partial derivatives to zero gives a system of equations, and `solve()` handles it (see 36.7):

```python
g = x**2 + y**2 - 4*x + 6*y

grad_g = [sym.diff(g, x), sym.diff(g, y)]     # [2*x - 4, 2*y + 6]
print(sym.solve(grad_g, [x, y]))              # {x: 2, y: -3}
```

The bowl-shaped function `g` has its lowest point at `(2, -3)`. This is the same problem that `sp.optimize.minimize` solved numerically in Guideline 35, now solved exactly.

## 36.5 Limits

A **limit** describes the value a function approaches as its input approaches some point, even when the function is undefined at that exact point. In SymPy, the form is `sym.limit(expression, variable, point)`:

```python
print(sym.limit(sym.sin(x) / x, x, 0))             # 1
print(sym.limit((x**2 - 1) / (x - 1), x, 1))       # 2
```

Both expressions above are undefined at the point itself (they would give 0/0), yet their limits exist. Using `sym.oo` gives limits at infinity:

```python
print(sym.limit((3*x**2 + 1) / (x**2 + 5), x, sym.oo))   # 3
print(sym.limit(sym.exp(-x), x, sym.oo))                 # 0
print(sym.limit((1 + 1/x)**x, x, sym.oo))                # E
```

The last one defines the number *e*.

**One-sided limits.** Approaching a point from the right or from the left can give different answers. A fourth argument selects the direction: `'+'` for from the right (from larger values), `'-'` for from the left:

```python
print(sym.limit(1/x, x, 0, "+"))     # oo
print(sym.limit(1/x, x, 0, "-"))     # -oo

print(sym.limit(sym.Abs(x) / x, x, 0, "+"))    # 1
print(sym.limit(sym.Abs(x) / x, x, 0, "-"))    # -1
```

Be careful here: when no direction is given, SymPy assumes `'+'`, so `sym.limit(1/x, x, 0)` quietly returns `oo`. When the left and right limits disagree, as they do for `|x|/x` above, the ordinary two-sided limit does not exist. Passing `dir="+-"` makes SymPy check both sides and raise an error if they differ.

**The definition of the derivative.** The derivative is itself a limit, and SymPy can confirm it by working from the definition, without using `diff()` at all:

```python
h = sym.symbols("h")

definition = ((x + h)**2 - x**2) / h
print(sym.limit(definition, h, 0))    # 2*x
```

The result, `2x`, matches `sym.diff(x**2, x)`. As with integrals, a limit can be written unevaluated as `sym.Limit(...)` and calculated later with `.doit()`.

## 36.6 Series (Taylor and Maclaurin)

Many functions, such as `sin`, `exp`, and `log`, are hard to compute directly, but they can be approximated by a **polynomial**, and polynomials only need addition and multiplication. A **Taylor series** builds such a polynomial around a chosen point using the function's derivatives at that point. A **Maclaurin series** is the special case where that point is 0.

In SymPy, the method is `expression.series(variable, point, n)`, where `n` says how many terms to keep: the series is computed up to, but not including, order `n`:

```python
print(sym.exp(x).series(x, 0, 5))
# 1 + x + x**2/2 + x**3/6 + x**4/24 + O(x**5)

print(sym.sin(x).series(x, 0, 6))
# x - x**3/6 + x**5/120 + O(x**6)

print(sym.cos(x).series(x, 0, 6))
# 1 - x**2/2 + x**4/24 + O(x**6)
```

The `O(x**5)` is called the **big-O term**. It is SymPy's way of saying "and everything after this is of order `x⁵` or smaller", and it reminds you that the polynomial is an approximation, not an equality.

**A Taylor series around a different point.** Change the second argument. The logarithm is undefined at 0, so its series is built around `x = 1`:

```python
print(sym.log(x).series(x, 1, 4))
# x - 1 - (x - 1)**2/2 + (x - 1)**3/3 + O((x - 1)**4, (x, 1))
```

**Using the polynomial.** To calculate with the approximation, remove the big-O term with `.removeO()` and substitute a number. Here, the three-term series for `sin(x)` is compared with the true value at `x = 0.5`:

```python
poly = sym.sin(x).series(x, 0, 6).removeO()   # x - x**3/6 + x**5/120

print(poly.subs(x, 0.5))       # 0.479427083333333
print(sym.sin(0.5))            # 0.479425538604203
```

The two agree to five decimal places, from a polynomial that only needed multiplication and addition. Keeping more terms (a larger `n`) improves the accuracy, and the approximation is best close to the center point and gets worse further away.

**Summing a series.** A related tool, `sym.summation()`, adds up terms of a sequence, including infinite ones, and gives an exact result:

```python
n = sym.symbols("n", integer=True, positive=True)

print(sym.summation(n, (n, 1, 100)))                  # 5050
print(sym.summation(1 / n**2, (n, 1, sym.oo)))        # pi**2/6
```

The second result is the famous solution to the Basel problem.

## 36.7 Equation Solving

In Guideline 34, `np.linalg.solve()` solved a system of *linear* equations and returned decimals. SymPy's `solve()` does the same job, and much more: it handles non-linear equations, gives exact answers, and can solve for one symbol in terms of others.

**One equation.** `solve()` treats an expression as "equals zero", and takes the variable to solve for:

```python
print(sym.solve(x**2 - 4, x))                        # [-2, 2]
print(sym.solve(x**3 - 6*x**2 + 11*x - 6, x))        # [1, 2, 3]
print(sym.solve(x**2 - 2, x))                        # [-sqrt(2), sqrt(2)]
print(sym.solve(x**2 + 1, x))                        # [-I, I]        complex roots, found automatically
```

To write the equation in its natural form, use `sym.Eq`:

```python
print(sym.solve(sym.Eq(2*x + 3, 11), x))    # [4]
```

**Solving with unknown coefficients.** Because the coefficients can be symbols too, `solve()` can produce a general formula, here the quadratic formula:

```python
a, b, c = sym.symbols("a b c")

print(sym.solve(a*x**2 + b*x + c, x))
# [(-b - sqrt(-4*a*c + b**2))/(2*a), (-b + sqrt(-4*a*c + b**2))/(2*a)]
```

**Systems of equations.** Pass a list of equations and a list of unknowns. The system from Guideline 34, `2x + y = 5` and `x − y = 1`, becomes:

```python
print(sym.solve([2*x + y - 5, x - y - 1], [x, y]))     # {x: 2, y: 1}
```

The answer is a dictionary, and the exactness matters. Take a system whose solution is not a whole number:

```python
print(sym.solve([3*x + 2*y - 1, x - 4*y - 2], [x, y]))     # {x: 4/7, y: -5/14}
```

NumPy would return `[0.57142857, -0.35714286]`, whereas SymPy returns the exact fractions.

**Non-linear systems** work the same way. Where does the circle `x² + y² = 25` cross the line `y = x − 1`?

```python
print(sym.solve([x**2 + y**2 - 25, x - y - 1], [x, y]))     # [(-3, -4), (4, 3)]
```

**Matrix form.** For linear systems written as `A x = b`, the matrix methods mirror what NumPy does, but stay exact:

```python
A = sym.Matrix([[1, 1,  1],
                [0, 2,  5],
                [2, 5, -1]])
b = sym.Matrix([6, -4, 27])

print(A.solve(b))                       # Matrix([[5], [3], [-2]])
print(sym.linsolve((A, b), x, y, z))    # {(5, 3, -2)}
```

**`solveset`.** A newer alternative, `sym.solveset(expression, variable, domain)`, returns the answer as a mathematical *set* and lets you say which numbers are allowed, which matters when there are no solutions among the real numbers:

```python
print(sym.solveset(x**2 - 4, x))                       # {-2, 2}
print(sym.solveset(x**2 + 1, x, domain=sym.S.Reals))   # EmptySet
```

**When there is no formula: `nsolve`.** Some equations have no closed-form solution. For those, `nsolve()` finds a decimal answer numerically, starting from a guess:

```python
print(sym.nsolve(sym.cos(x) - x, x, 1))     # 0.739085133215161
```

**Differential equations.** `sym.dsolve()` solves an equation involving a function and its derivatives. The unknown function is declared with `sym.Function`, and the equation `f'(x) = 2 f(x)` becomes:

```python
f = sym.Function("f")
ode = sym.Eq(f(x).diff(x), 2 * f(x))

print(sym.dsolve(ode, f(x)))                       # Eq(f(x), C1*exp(2*x))
print(sym.dsolve(ode, f(x), ics={f(0): 1}))        # Eq(f(x), exp(2*x))
```

The first answer is the general solution, with an arbitrary constant `C1`, and giving a starting value through `ics` (initial conditions) pins down that constant. This is the exact counterpart of `solve_ivp` in Guideline 35.

**Always check.** Substituting a solution back in is a cheap safeguard:

```python
solution = sym.solve(x**2 - 2, x)
print([sym.simplify((x**2 - 2).subs(x, s)) for s in solution])   # [0, 0]
```

## 36.8 Other Methods

**Bridging to NumPy with `lambdify`.** A SymPy expression is a formula, and evaluating it one `subs()` at a time is slow. `lambdify()` converts it into an ordinary Python function that works on NumPy arrays, so you get the exactness of SymPy for deriving the formula and the speed of NumPy for using it:

```python
f = x**2 * sym.sin(x)

fast = sym.lambdify(x, f, "numpy")
print(fast(np.array([0, 1, 2])))    # [0.         0.84147098 3.63718971]
```

**Matrices.** SymPy has its own exact matrices, with the same operations as in Guideline 34:

```python
M = sym.Matrix([[1, 2],
                [3, 4]])

print(M.det())          # -2                  NumPy usually gives -2.0000000000000004
print(M.inv())          # Matrix([[-2, 1], [3/2, -1/2]])
print(M.T)              # Matrix([[1, 3], [2, 4]])

E = sym.Matrix([[4, 1],
                [2, 3]])
print(E.eigenvals())    # {2: 1, 5: 1}        each eigenvalue and how many times it occurs
```

**Rearranging expressions.** Beyond `expand`, `factor`, and `simplify` from 36.1:

```python
print(sym.cancel((x**2 - 1) / (x - 1)))       # x + 1
print(sym.apart(1 / (x * (x + 1))))           # -1/(x + 1) + 1/x       partial fractions
print(sym.trigsimp(2 * sym.sin(x) * sym.cos(x)))   # sin(2*x)
print(sym.expand_trig(sym.sin(2*x)))          # 2*sin(x)*cos(x)
```

**Number theory.** SymPy also has tools for whole numbers:

```python
print(sym.factorial(5))       # 120
print(sym.factorint(360))     # {2: 3, 3: 2, 5: 1}     360 = 2^3 x 3^2 x 5
print(sym.isprime(97))        # True
```

**Quick reference.**

| Function | Purpose | Example |
|---|---|---|
| `sym.symbols()` | Create symbols | `x, y = sym.symbols("x y")` |
| `sym.Rational()` | Exact fraction | `sym.Rational(1, 3)` |
| `expr.subs()` | Substitute values or expressions | `expr.subs(x, 2)` |
| `expr.evalf()`, `sym.N()` | Convert to a decimal | `sym.pi.evalf(10)` |
| `sym.simplify()` | Find a simpler form | `sym.simplify(expr)` |
| `sym.expand()`, `sym.factor()` | Multiply out / factor | `sym.factor(x**2 - 1)` |
| `sym.collect()` | Group terms by powers of a symbol | `sym.collect(expr, x)` |
| `sym.cancel()`, `sym.apart()` | Reduce a fraction / split into partial fractions | `sym.apart(expr)` |
| `sym.diff()` | Derivative | `sym.diff(f, x)` |
| `sym.integrate()` | Integral | `sym.integrate(f, (x, 0, 1))` |
| `sym.limit()` | Limit | `sym.limit(f, x, 0)` |
| `expr.series()` | Taylor / Maclaurin series | `f.series(x, 0, 6)` |
| `sym.summation()`, `sym.product()` | Sums and products of sequences | `sym.summation(n, (n, 1, 10))` |
| `sym.solve()`, `sym.solveset()` | Solve equations | `sym.solve(f, x)` |
| `sym.linsolve()` | Solve linear systems | `sym.linsolve((A, b), x, y)` |
| `sym.nsolve()` | Numerical solution of an equation | `sym.nsolve(f, x, 1)` |
| `sym.dsolve()` | Differential equations | `sym.dsolve(ode, f(x))` |
| `sym.Matrix()` | Exact matrices (`.det()`, `.inv()`, `.eigenvals()`) | `sym.Matrix([[1, 2], [3, 4]])` |
| `sym.lambdify()` | Turn an expression into a fast NumPy function | `sym.lambdify(x, f, "numpy")` |
| `sym.plot()` | Quick plot of an expression (needs Matplotlib) | `sym.plot(sym.sin(x), (x, -6, 6))` |
| `sym.latex()` | LaTeX code for an expression | `sym.latex(expr)` |
| `sym.sympify()` | Turn a string into an expression | `sym.sympify("x**2 + 1")` |

---

Symbolic and numeric computation are two halves of the same toolkit, and knowing which half a problem needs is part of the skill. Reach for SymPy when you want an exact answer, a general formula, or a derivation you can check by hand, and reach for NumPy and SciPy when the data is large, the problem is messy, or a decimal is all you need. Very often the best workflow is both: let SymPy find the formula, confirm it on a case you can verify by hand, then `lambdify` it and let NumPy do the heavy lifting.
