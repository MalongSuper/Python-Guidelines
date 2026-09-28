# Guideline 39: Python for Machine Learning

For thirty-eight guidelines you have been writing rules by hand. If the number is even, print "even". If the score is above 90, print "A". Every decision the program made was a decision you typed out in advance.

Machine learning reverses the arrangement. Instead of writing the rules, you show the program examples and let it work out the rules itself. Show it enough houses with their prices and it learns how price depends on size. Show it enough emails marked "spam" or "not spam" and it learns what spam looks like. The rules still exist, but they are found by an algorithm rather than typed by a person.

Python is the language where most of this happens, and one library does most of the work: **scikit-learn**. This guideline is a tour of it. You already met the supporting cast (NumPy in Guideline 34, Pandas in Guideline 37, Matplotlib in Guideline 38), so here the focus is on the workflow (prepare the data, split it, fit a model, judge the result) and on the families of models that scikit-learn provides.

A word about scope. Each model family below deserves a book of its own, and the theory behind them belongs to a machine learning course. This guideline teaches you to *use* the tools correctly: what to import, what to call, what the results mean, and where beginners usually go wrong.

---

## 1. Machine Learning (Supervised and Unsupervised)

Every machine learning problem starts with a table of data. The columns you use to describe each example are called **features** (conventionally stored in a variable called `X`). If you also have a column you want to predict, that column is the **target** (conventionally `y`).

| House size (m²) | Bedrooms | Age (years) | ← features (`X`) | Price ← target (`y`) |
|---|---|---|---|---|
| 80 | 2 | 10 | | 150,000 |
| 120 | 3 | 5 | | 260,000 |
| 60 | 1 | 30 | | 90,000 |

Whether the target exists divides machine learning into two big families.

### Supervised learning

In **supervised learning**, the training data contains the correct answers, and the model learns the relationship between features and target so that it can predict the answer for new, unseen examples. The "supervision" is the presence of the labelled answers. There are two kinds of supervised problem, distinguished by the type of the target:

- **Classification** predicts a category: spam or not spam, which species of flower, whether a tumour is benign or malignant. The target is discrete.
- **Regression** predicts a number: a house price, tomorrow's temperature, a student's exam score. The target is continuous.

### Unsupervised learning

In **unsupervised learning**, there is no target column. The model is given only `X` and asked to find structure on its own. There are two common tasks:

- **Clustering** groups similar examples together without being told what the groups are: customer segments, clusters of news articles.
- **Dimensionality reduction** compresses many features into fewer ones while keeping as much of the information as possible, useful for visualisation and for simplifying data.

There is no answer key in unsupervised learning, which makes results harder to judge. A clustering is not "correct" or "wrong"; it is more or less useful.

(Two further families exist and are outside this guideline: *semi-supervised* learning, where only some examples are labelled, and *reinforcement* learning, where an agent learns from rewards and penalties.)

### The map of this guideline

Scikit-learn organises its models into modules by the *idea* behind them. The rest of the guideline follows that layout:

| Section | Module | Task | Type |
|---|---|---|---|
| 7 | `sklearn.linear_model` | classification and regression | supervised |
| 8 | `sklearn.tree` | classification and regression | supervised |
| 9 | `sklearn.ensemble` | classification and regression | supervised |
| 10 | `sklearn.naive_bayes` | classification | supervised |
| 11 | `sklearn.discriminant_analysis` | classification (and reduction) | supervised |
| 12 | `sklearn.neighbors` | classification and regression | supervised |
| 13 | `sklearn.svm` | classification and regression | supervised |
| 14 | `sklearn.neural_network` | classification and regression | supervised |
| 15 | `sklearn.cluster` | clustering | unsupervised |
| 16 | `sklearn.decomposition` | dimensionality reduction | unsupervised |

### The standard workflow

Almost every supervised project follows the same steps, and sections 2 to 6 build the tools for them:

1. Load the data into `X` and `y`.
2. Clean it: fill in missing values (imputers), convert text categories to numbers (encoders).
3. Split it into a training set and a test set.
4. Rescale the features if the model needs it (scalers).
5. Fit the model on the training set.
6. Evaluate it on the test set, which the model has never seen.

The order matters more than it looks, as you will see in the section on train-test split.

---

## 2. Scikit-Learn (Some Key Methods, Including the Datasets)

Scikit-learn is an external library (Guideline 25). It is installed under the name `scikit-learn` but imported under the name `sklearn`:

```bash
pip install scikit-learn
```

Unlike most libraries, you rarely import the whole thing. You import the specific class you need from the specific module that contains it:

```python
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
```

### The estimator API

The best thing about scikit-learn is that every model, scaler, imputer and encoder follows the *same small set of methods*. Learn them once and every tool in the library feels familiar. An object that learns from data is called an **estimator**.

| Method | What it does | Used by |
|---|---|---|
| `fit(X, y)` | Learn from the data (`y` is omitted for unsupervised tools) | everything |
| `predict(X)` | Produce predictions for new rows | models, clusterers |
| `predict_proba(X)` | Produce a probability for each class | many classifiers |
| `score(X, y)` | Quick quality measure (accuracy for classifiers, R² for regressors) | supervised models |
| `transform(X)` | Convert data using what `fit` learned | scalers, imputers, encoders, PCA |
| `fit_transform(X)` | `fit` then `transform` in one step | same as above |
| `get_params()` / `set_params()` | Read or change the settings | everything |

The pattern is always the same: **create** the object (choosing its settings), **fit** it, then **predict** or **transform**.

```python
model = SomeModel(setting=value)   # 1. create
model.fit(X_train, y_train)        # 2. learn
predictions = model.predict(X_new) # 3. use
```

By convention, settings you choose before training are called **hyperparameters** (for example, how deep a tree may grow). Values the model discovers during `fit` are stored in attributes ending with an underscore, such as `coef_` or `feature_importances_`. That trailing underscore is a reliable signal: *if it ends in `_`, it was learned from the data.*

### Built-in datasets

Scikit-learn ships with small datasets so that you can practise without hunting for files. They live in `sklearn.datasets`, and their loaders start with `load_`:

| Loader | Rows | Features | Task | Content |
|---|---|---|---|---|
| `load_iris()` | 150 | 4 | classification (3 classes) | flower measurements, three species |
| `load_wine()` | 178 | 13 | classification (3 classes) | chemical analysis of wines |
| `load_breast_cancer()` | 569 | 30 | classification (2 classes) | tumour measurements, benign or malignant |
| `load_digits()` | 1797 | 64 | classification (10 classes) | 8×8 pixel images of handwritten digits |
| `load_diabetes()` | 442 | 10 | regression | disease progression after one year |

```python
from sklearn.datasets import load_iris

iris = load_iris()
print(iris.data.shape)       # (150, 4)
print(iris.feature_names)
# ['sepal length (cm)', 'sepal width (cm)', 'petal length (cm)', 'petal width (cm)']
print(iris.target_names)     # ['setosa' 'versicolor' 'virginica']
print(iris.target[:5])       # [0 0 0 0 0]
```

The loader returns a `Bunch`, which behaves like a dictionary whose keys are also attributes: `data` (the features), `target` (the answers), `feature_names`, `target_names`, and `DESCR` (a full written description of the dataset, worth reading with `print(iris.DESCR)`). Notice that the target is stored as numbers (0, 1, 2), with `target_names` telling you what each number means.

Two shortcuts are worth memorising:

```python
# 1. Get X and y directly
X, y = load_iris(return_X_y=True)

# 2. Get a Pandas DataFrame (features plus a "target" column)
df = load_iris(as_frame=True).frame
print(df.head(2))
#    sepal length (cm)  sepal width (cm)  ...  petal width (cm)  target
# 0                5.1               3.5  ...               0.2       0
# 1                4.9               3.0  ...               0.2       0
```

### Generating your own data

When you want data with known properties (to test an idea, or to demonstrate an algorithm), scikit-learn can fabricate it. These functions start with `make_`:

| Function | Produces |
|---|---|
| `make_classification()` | a classification dataset with a chosen number of features and classes |
| `make_regression()` | a regression dataset with adjustable noise |
| `make_blobs()` | round clusters of points, ideal for clustering demos |
| `make_moons()` | two interleaved half-circles, a classic hard case for clustering |

```python
from sklearn.datasets import make_blobs

X, y = make_blobs(n_samples=300, centers=3, cluster_std=1.0, random_state=42)
print(X.shape)   # (300, 2)
```

The `random_state` argument appears throughout scikit-learn. Many procedures involve randomness (shuffling, random starting points), and setting `random_state` to any fixed integer makes the result repeatable, the same idea as `random.seed()` from Guideline 19. Use it whenever you want your results to be reproducible.

Larger real-world datasets are available through `fetch_` functions (for example `fetch_california_housing()`), which download the data the first time you call them and therefore need an internet connection.

---

## 3. Scalers

Suppose one feature is a person's age (between 20 and 60) and another is their income (between 20,000 and 200,000). To a human they are different quantities, and the size of the numbers is irrelevant. To many algorithms, the income column simply *looks* thousands of times more important because its numbers are bigger. Any model that measures distances or takes gradient steps will be dominated by the large-valued feature.

**Scaling** puts features on comparable ranges. Scikit-learn's scalers live in `sklearn.preprocessing`.

| Scaler | What it does | Result | Best when |
|---|---|---|---|
| `StandardScaler` | subtract the mean, divide by the standard deviation | mean 0, standard deviation 1 | the default choice |
| `MinMaxScaler` | squeeze into a fixed range (default 0 to 1) | min 0, max 1 | you need bounded values |
| `MaxAbsScaler` | divide by the largest absolute value | range −1 to 1, zeros stay zeros | sparse data |
| `RobustScaler` | subtract the median, divide by the interquartile range | centred on the median | there are outliers |
| `Normalizer` | rescale each *row* to length 1 | every row has unit norm | only the direction of a row matters |

Here is the same tiny table through four of them. Note the outlier in the second column (1000):

```python
import numpy as np
from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler

a = np.array([[1., 200.],
              [2., 300.],
              [3., 400.],
              [4., 1000.]])

print(StandardScaler().fit_transform(a).round(2))
# [[-1.34 -0.88]
#  [-0.45 -0.56]
#  [ 0.45 -0.24]
#  [ 1.34  1.69]]

print(MinMaxScaler().fit_transform(a).round(2))
# [[0.   0.  ]
#  [0.33 0.12]
#  [0.67 0.25]
#  [1.   1.  ]]

print(RobustScaler().fit_transform(a).round(2))
# [[-1.   -0.55]
#  [-0.33 -0.18]
#  [ 0.33  0.18]
#  [ 1.    2.36]]
```

Look at how each treats the outlier. `MinMaxScaler` lets the single value 1000 stretch the whole range, so the three ordinary values are crammed between 0 and 0.25. `RobustScaler` uses the median and the spread of the middle half of the data, so the ordinary values remain well separated and the outlier simply lands far away at 2.36. When outliers are present, that difference matters.

### What `fit` learns

A scaler is an estimator like any other, and `fit` is where it learns its numbers:

```python
scaler = StandardScaler().fit(a)
print(scaler.mean_)               # [  2.5 475. ]
print(scaler.scale_.round(2))     # [  1.12 311.25]   (the standard deviations)
print(scaler.inverse_transform(scaler.transform(a))[0])   # [  1. 200.]
```

`inverse_transform` undoes the scaling, useful when you need to turn scaled predictions or values back into their original units.

### The most important rule: fit on training data only

This is the single most common mistake in beginner machine learning, so it gets its own heading. The test set is supposed to stand in for data the model will meet in the future. If you fit the scaler on the *whole* dataset, the mean and standard deviation it learns include information from the test rows, which means information from the "future" has leaked into your preparation. Your test score becomes slightly too optimistic, and you will not notice.

The correct pattern:

```python
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)   # learn from training data AND apply
X_test_scaled  = scaler.transform(X_test)        # apply only, never fit
```

`fit_transform` on the training set, plain `transform` on the test set. The same rule applies to imputers, encoders and PCA. Section 6 shows how *pipelines* make this rule automatic.

Which models need scaling? Anything based on **distances** (nearest neighbors, clustering, support vector machines), **gradient descent** (neural networks, and linear models fitted by gradient methods) or **penalties on coefficient size** (Ridge and Lasso) needs it. Tree-based models do not: a tree only asks "is this value above or below a threshold?", and scaling does not change the answer.

---

## 4. Imputers

Real data has holes. A survey respondent skips a question, a sensor drops out, a form field is left empty. In Pandas these appear as `NaN`. Most scikit-learn models refuse to run when they meet a `NaN`, so the gaps must be dealt with first.

You could delete every row that contains a gap, but if each row is missing a different column, you may end up deleting most of your data. The alternative is **imputation**: filling each gap with a sensible estimate. The imputers live in `sklearn.impute`.

### SimpleImputer

The simplest strategy replaces missing values using a statistic of that column.

| `strategy=` | Fills with | Works on |
|---|---|---|
| `"mean"` | the column average (default) | numbers |
| `"median"` | the column median | numbers, safer with outliers |
| `"most_frequent"` | the most common value | numbers and text |
| `"constant"` | a value you choose with `fill_value=` | numbers and text |

```python
import numpy as np
from sklearn.impute import SimpleImputer

m = np.array([[1,      2],
              [np.nan, 3],
              [7,      np.nan],
              [4,      6]])

print(SimpleImputer(strategy="mean").fit_transform(m))
# [[1.         2.        ]
#  [4.         3.        ]
#  [7.         3.66666667]
#  [4.         6.        ]]

print(SimpleImputer(strategy="median").fit_transform(m))
# [[1. 2.]
#  [4. 3.]
#  [7. 3.]
#  [4. 6.]]
```

The first column has known values 1, 7 and 4, whose mean is 4, so that is what fills the gap. Text columns need `most_frequent` or `constant`:

```python
import pandas as pd

d = pd.DataFrame({"city": ["A", "B", None, "A"]})

print(SimpleImputer(strategy="most_frequent").fit_transform(d).ravel())
# ['A' 'B' 'A' 'A']

print(SimpleImputer(strategy="constant", fill_value="unknown").fit_transform(d).ravel())
# ['A' 'B' 'unknown' 'A']
```

**Missingness can itself be information.** A patient whose blood pressure was never recorded may differ systematically from those whose readings were taken. Passing `add_indicator=True` appends extra columns of 0s and 1s marking where the gaps were, so the model can learn from the absence as well as the fill value:

```python
imp = SimpleImputer(strategy="mean", add_indicator=True)
print(imp.fit_transform(m))
# [[1.         2.         0.         0.        ]
#  [4.         3.         1.         0.        ]
#  [7.         3.66666667 0.         1.        ]
#  [4.         6.         0.         0.        ]]
```

### KNNImputer

`KNNImputer` fills a gap using the values from the *most similar rows* (the nearest neighbours, a concept developed in section 12). It is more context-sensitive than a column-wide average:

```python
from sklearn.impute import KNNImputer

print(KNNImputer(n_neighbors=2).fit_transform(m).round(2))
# [[1.  2. ]
#  [2.5 3. ]
#  [7.  4. ]
#  [4.  6. ]]
```

For the second row, the two nearest rows (judged by the second column) were rows 1 and 4, whose first-column values are 1 and 4, hence 2.5.

### IterativeImputer

`IterativeImputer` treats each column with gaps as a small regression problem, predicting it from all the other columns, and repeats the process until the estimates settle. It is the most powerful of the three but still officially experimental, so scikit-learn requires you to opt in with an extra import:

```python
from sklearn.experimental import enable_iterative_imputer   # opt in (must come first)
from sklearn.impute import IterativeImputer

imputer = IterativeImputer(random_state=0)
X_filled = imputer.fit_transform(X)
```

On a table as tiny as the one above, iterative imputation has too little to learn from and can give odd values, so it is best used on datasets with a reasonable number of rows.

The same rule as before applies: `fit` the imputer on the training set only, and `transform` the test set with what it learned.

---

## 5. Encoders

Models do arithmetic, and arithmetic needs numbers. Real data is full of categories stored as text: colours, cities, sizes, species. **Encoders** convert them. The right choice depends on one question: *do the categories have a natural order?*

| Encoder | Use for | Turns `"cat", "dog", "bird"` into |
|---|---|---|
| `LabelEncoder` | the **target** `y` | a single column of integers |
| `OrdinalEncoder` | **features** with a real order (small < medium < large) | integers that respect your order |
| `OneHotEncoder` | **features** with no order (colours, cities) | one 0/1 column per category |

All three are in `sklearn.preprocessing`.

### LabelEncoder

`LabelEncoder` numbers the classes alphabetically. It is meant for the target column only:

```python
from sklearn.preprocessing import LabelEncoder

le = LabelEncoder()
codes = le.fit_transform(["cat", "dog", "cat", "bird"])
print(codes)          # [1 2 1 0]
print(le.classes_)    # ['bird' 'cat' 'dog']
print(le.inverse_transform([2, 0]))   # ['dog' 'bird']
```

`classes_` records the mapping (position 0 is `bird`, position 1 is `cat`, and so on), and `inverse_transform` translates predictions back into readable labels. That is how you turn a model's output of `2` back into `"dog"`.

### OrdinalEncoder

When the order carries meaning, tell the encoder that order explicitly. If you leave it to guess, it sorts alphabetically, and `"large"` would come before `"medium"` before `"small"`, which is exactly backwards:

```python
import pandas as pd
from sklearn.preprocessing import OrdinalEncoder

sizes = pd.DataFrame({"size": ["small", "large", "medium", "small"]})

oe = OrdinalEncoder(categories=[["small", "medium", "large"]])
print(oe.fit_transform(sizes).ravel())   # [0. 2. 1. 0.]
```

### OneHotEncoder

Encoding colours as red=0, green=1, blue=2 would tell the model that blue is "twice" green and that red is "less than" blue, none of which is true. **One-hot encoding** avoids inventing a fake order by giving each category its own column, with a 1 marking which category a row belongs to:

```python
from sklearn.preprocessing import OneHotEncoder

col = pd.DataFrame({"color": ["red", "green", "blue", "green"]})

ohe = OneHotEncoder(sparse_output=False)
print(ohe.fit_transform(col))
# [[0. 0. 1.]
#  [0. 1. 0.]
#  [1. 0. 0.]
#  [0. 1. 0.]]

print(ohe.get_feature_names_out())
# ['color_blue' 'color_green' 'color_red']
```

The columns are in alphabetical order (blue, green, red), so the first row, "red", becomes `[0, 0, 1]`. Three details are worth knowing:

- **`sparse_output=False`** returns an ordinary array. The default is a *sparse* matrix (Guideline 35), which is memory-efficient but harder to read while you are learning.
- **`handle_unknown="ignore"`** prevents a crash when the test set contains a category the training set never had. The unknown row simply becomes all zeros:

  ```python
  ohe2 = OneHotEncoder(handle_unknown="ignore", sparse_output=False).fit(col)
  print(ohe2.transform(pd.DataFrame({"color": ["purple"]})))   # [[0. 0. 0.]]
  ```

- **`drop="first"`** removes one column per feature. With three colours, two columns already carry all the information (if it is neither blue nor green, it must be red), and keeping all three can cause trouble for linear models:

  ```python
  print(OneHotEncoder(drop="first", sparse_output=False).fit_transform(col))
  # [[0. 1.]
  #  [1. 0.]
  #  [0. 0.]
  #  [1. 0.]]
  ```

If you only need a quick one-hot encoding while exploring data, `pd.get_dummies()` from Guideline 37 does the same job in one line. The scikit-learn encoder is preferred for real modelling because it *remembers* its columns and can apply exactly the same transformation to new data later.

---

## 6. Train-Test Split

Here is a question worth sitting with. You build a model and it scores 100% on the data you trained it on. Is it a good model?

You cannot tell. A model can score perfectly on its training data simply by *memorising* it, in the way a student who has seen last year's exam questions can recite the answers without understanding the subject. What you actually care about is performance on data the model has never seen. That failure, excellent on training data but poor on new data, is called **overfitting**, and guarding against it is the reason the train-test split exists.

The idea is to hide part of your data before training starts. You train on the larger part and then judge the model on the hidden part, called the **test set**, which behaves like a set of surprise exam questions.

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split

X, y = load_iris(return_X_y=True)

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

print(X_train.shape, X_test.shape)   # (120, 4) (30, 4)
```

The function returns four pieces, always in this order: training features, test features, training targets, test targets.

| Parameter | Meaning |
|---|---|
| `test_size` | fraction (0.2) or number of rows held out for testing. Common choices are 0.2 to 0.3 |
| `train_size` | the mirror image; usually you set only one of the two |
| `random_state` | fixes the shuffle so the split is reproducible |
| `shuffle` | whether to shuffle before splitting (default `True`; keep it unless the rows are time-ordered) |
| `stratify` | pass the target `y` to keep class proportions the same in both parts |

**Why stratify?** Imagine a dataset that is 90% "healthy" and 10% "ill". A random split might, by bad luck, put almost no ill patients in the test set, making your evaluation meaningless. With `stratify=y`, both parts keep the original proportions. For iris, which has 50 flowers of each species:

```python
import numpy as np
print(np.bincount(y_train))   # [40 40 40]
print(np.bincount(y_test))    # [10 10 10]
```

Every class is represented equally in both sets.

**Time-ordered data is different.** If your rows are days in a time series and you want to predict the future, shuffling would let the model train on tomorrow to predict yesterday. In that case split by time instead: earlier rows for training, later rows for testing, with `shuffle=False`.

### The standard workflow, complete

Putting sections 2 to 6 together, this is the shape of nearly every scikit-learn project. It will appear again and again through the rest of the guideline:

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

X, y = load_iris(return_X_y=True)                      # 1. load
X_train, X_test, y_train, y_test = train_test_split(   # 2. split
    X, y, test_size=0.2, random_state=42, stratify=y)

scaler = StandardScaler()                              # 3. scale (fit on train only)
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

model = LogisticRegression(max_iter=200)               # 4. create
model.fit(X_train_s, y_train)                          # 5. fit
print(model.score(X_test_s, y_test))                   # 6. evaluate
```

### Pipelines: making the rules automatic

Notice how many things you must remember: fit the scaler on the training set only, transform the test set with it, keep the order straight. A **pipeline** chains the steps into a single object, so `fit` and `predict` apply everything in the right order and the leakage rule is enforced for you:

```python
from sklearn.pipeline import make_pipeline
from sklearn.svm import SVC

pipe = make_pipeline(StandardScaler(), SVC())
pipe.fit(X_train, y_train)          # scaler fits on training data, then the model fits
print(pipe.score(X_test, y_test))   # test data is only transformed, never fitted
```

Real tables mix numeric and text columns, and each kind needs different preparation. `ColumnTransformer` routes each group of columns through its own steps, and can itself be dropped into a pipeline:

```python
import numpy as np
import pandas as pd
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.linear_model import LogisticRegression

num = Pipeline([("fill", SimpleImputer(strategy="median")),
                ("scale", StandardScaler())])

cat = Pipeline([("fill", SimpleImputer(strategy="most_frequent")),
                ("onehot", OneHotEncoder(handle_unknown="ignore"))])

prep = ColumnTransformer([("num", num, ["age", "income"]),
                          ("cat", cat, ["city"])])

model = Pipeline([("prep", prep), ("clf", LogisticRegression())])

# df has columns: age, income (numbers with gaps) and city (text with gaps)
# model.fit(X_train, y_train); model.predict(X_test)
```

One object now imputes, scales, encodes and predicts, and it can be handed new raw data and will treat it exactly as it treated the training data.

### One split is one opinion

A single split gives a single score, and that score depends on which rows happened to land in the test set. With only 30 test rows, one lucky or unlucky flower moves the accuracy by more than 3 percentage points. **Cross-validation** repeats the split several times, each time holding out a different portion, and gives you a set of scores:

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(make_pipeline(StandardScaler(), SVC()), X, y, cv=5)
print(scores.round(3))          # [0.967 0.967 0.967 0.933 1.   ]
print(scores.mean().round(3))   # 0.967
```

Five scores are a more honest picture than one. Because the pipeline is re-fitted inside every fold, the scaler never sees the held-out rows. Keep in mind while reading the sections below that the scores printed there come from a single 30-row (or 114-row) test set, so small differences between models are not meaningful.

---

## Bonus: Model Evaluation Metrics

You have split the data, trained a model, and asked it to make predictions on the test set. Now comes the important question: *how good are those predictions?* A score of `0.9` might sound impressive, but what does it actually mean? And is it enough to describe the model's performance?

Scikit-learn provides `sklearn.metrics`, a module containing functions for measuring model performance. The right metric depends on the task. Predicting categories is a classification problem, while predicting continuous numbers is a regression problem. Each requires a different way of measuring error.

### Classification metrics

For classification, the model predicts a category, such as whether an email is spam or whether a flower belongs to a particular species. Several metrics help us understand how often the model is correct and what kinds of mistakes it makes.

| Metric    | What it measures                                                 | Better value |
| --------- | ---------------------------------------------------------------- | ------------ |
| Accuracy  | Fraction of all predictions that are correct                     | Higher       |
| Precision | Fraction of predicted positives that are actually positive       | Higher       |
| Recall    | Fraction of actual positives that the model correctly identifies | Higher       |
| F1-score  | Balance between precision and recall                             | Higher       |

**Accuracy** is the simplest metric. If a model correctly classifies 27 out of 30 flowers, its accuracy is 90%. However, accuracy can be misleading when the classes are imbalanced. Imagine a dataset in which 95% of emails are normal and only 5% are spam. A model that always predicts "normal" achieves 95% accuracy while failing to identify a single spam email.

This is where precision and recall become useful. **Precision** asks, "When the model predicts positive, how often is it right?" **Recall** asks, "Of all the actual positive examples, how many did the model find?" A spam filter with high precision avoids incorrectly flagging normal emails, while high recall means it catches more of the spam. The **F1-score** combines these two metrics using their harmonic mean, providing a single measure when both matter.

Let's use the Iris dataset to calculate these metrics:

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import (
    accuracy_score, precision_score,
    recall_score, f1_score,
    classification_report, confusion_matrix
)

X, y = load_iris(return_X_y=True)

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

model = LogisticRegression(max_iter=200)
model.fit(X_train, y_train)

predictions = model.predict(X_test)

print("Accuracy:", accuracy_score(y_test, predictions))
print("Precision:", precision_score(
    y_test, predictions, average="weighted"))
print("Recall:", recall_score(
    y_test, predictions, average="weighted"))
print("F1-score:", f1_score(
    y_test, predictions, average="weighted"))

print(confusion_matrix(y_test, predictions))
print(classification_report(y_test, predictions))
```

The `average="weighted"` argument calculates each class's metric separately, then combines the results according to the number of actual examples in each class. This is useful for multiclass problems such as Iris. Other averaging options, such as `"macro"` and `"micro"`, are available when different ways of combining class results are needed.

Two additional functions deserve attention. `confusion_matrix()` shows how many examples from each actual class were assigned to each predicted class, making misclassifications easier to inspect. `classification_report()` presents precision, recall, F1-score and support (the number of actual examples) for every class in one report.

### Regression metrics

Classification metrics cannot be used directly to judge ordinary regression predictions. If the model predicts a house price of 250,000 when the actual price is 270,000, we need to measure the numerical difference between the two values.

| Metric | What it measures                                                      | Better value |
| ------ | --------------------------------------------------------------------- | ------------ |
| MAE    | Average absolute prediction error                                     | Lower        |
| MSE    | Average squared prediction error                                      | Lower        |
| RMSE   | Square root of the average squared error                              | Lower        |
| R²     | How much variation the model explains relative to predicting the mean | Higher       |

**Mean Absolute Error (MAE)** averages the absolute differences between actual and predicted values. It is easy to interpret because it uses the same units as the target. If the MAE for house prices is 15,000, the predictions differ from the actual prices by an average absolute amount of 15,000.

**Mean Squared Error (MSE)** squares each error before averaging. This penalises large errors more heavily than small ones. **Root Mean Squared Error (RMSE)** takes the square root of MSE, returning the result to the target's original units.

**R²**, also called the coefficient of determination, measures how well the model explains the variation in the target. A value of `1.0` indicates perfect predictions, `0.0` means performance equivalent to always predicting the training target's mean on the evaluated data, and a negative value means the model performs worse than that baseline. Unlike accuracy, R² is not a percentage of correct predictions.

Here is how to calculate these metrics on a regression problem:

```python
from sklearn.datasets import load_diabetes
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import (
    mean_absolute_error,
    mean_squared_error,
    r2_score
)
import numpy as np

X, y = load_diabetes(return_X_y=True)

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

model = LinearRegression()
model.fit(X_train, y_train)

predictions = model.predict(X_test)

mae = mean_absolute_error(y_test, predictions)
mse = mean_squared_error(y_test, predictions)
rmse = np.sqrt(mse)
r2 = r2_score(y_test, predictions)

print("MAE:", round(mae, 2))
print("MSE:", round(mse, 2))
print("RMSE:", round(rmse, 2))
print("R²:", round(r2, 3))
```

Notice that the regression example uses the same workflow as classification: load the data, split it, fit the model, predict on the test set, and evaluate the results. Only the metrics change to suit the task.

### Choosing the right metric

There is no single metric that describes every aspect of a model. Accuracy is useful when classes are reasonably balanced and mistakes have similar consequences. Precision and recall matter when false positives and false negatives have different costs. For regression, MAE is straightforward to interpret, MSE and RMSE emphasise larger errors, and R² provides a measure of explanatory performance.

One final rule: **evaluate on the test set, not the training set.** Training performance tells you how well the model fits examples it has already seen; test performance gives you evidence of how well it handles unseen examples. Even then, a single test score is not the whole story, and cross-validation can help assess how sensitive the result is to the particular split.

### From metrics to models

You now have the tools to judge a model, but you have not yet explored the algorithms themselves. Different models learn different kinds of relationships, make different assumptions, and respond differently to the same dataset. In the next section, we begin with **linear models**, a useful starting point because they are relatively simple, efficient, and interpretable. We will train them, inspect their learned coefficients, and use the metrics introduced here to understand how well they perform.


## 7. Linear Models

```python
import sklearn.linear_model
```

A linear model predicts by taking a weighted sum of the features. In the house example: *price = w₁·size + w₂·bedrooms + w₃·age + b*. The weights are called **coefficients**, and `b` is the **intercept**. Training means finding the weights that make the predictions as close as possible to the real answers.

Linear models are fast, easy to interpret (each coefficient tells you how much the prediction changes per unit of that feature), and a strong first attempt on almost any problem.

| Class | Task | What is special |
|---|---|---|
| `LinearRegression` | regression | plain least squares, no penalty |
| `Ridge` | regression | penalises large coefficients (L2), which keeps them small and stable |
| `Lasso` | regression | penalises the *sum* of coefficients (L1), which can shrink some to exactly zero |
| `ElasticNet` | regression | a mixture of the Ridge and Lasso penalties |
| `LogisticRegression` | **classification** | predicts class probabilities |
| `SGDClassifier`, `SGDRegressor` | both | fitted by stochastic gradient descent, suited to very large datasets |

### Simple linear regression

Start with the smallest possible example: one feature, four points that lie roughly on a line.

```python
import numpy as np
from sklearn.linear_model import LinearRegression

x = np.array([[1], [2], [3], [4]])        # features must be 2D: rows × columns
y = np.array([2.1, 3.9, 6.2, 7.8])

model = LinearRegression().fit(x, y)
print(model.coef_)             # [1.94]
print(model.intercept_)        # 0.15 (approximately)
print(model.predict([[5]]))    # [9.85]
```

The learned line is *y ≈ 1.94x + 0.15*. The features `x` must be two-dimensional even when there is only one, which is why each value sits inside its own brackets. This is the single most common shape error for beginners.

### Regularisation: Ridge, Lasso, ElasticNet

With many features, plain linear regression can chase noise: the coefficients grow large and erratic and the model overfits. **Regularisation** adds a penalty for large coefficients, trading a little accuracy on the training data for stability on new data. The strength of the penalty is the hyperparameter `alpha` (bigger alpha, stronger penalty).

Here they are on the diabetes dataset, a regression problem with 10 features:

```python
from sklearn.datasets import load_diabetes
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression, Ridge, Lasso, ElasticNet
from sklearn.metrics import r2_score, mean_squared_error

X, y = load_diabetes(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

models = {
    "LinearRegression": LinearRegression(),
    "Ridge":            Ridge(alpha=0.1),
    "Lasso":            Lasso(alpha=0.1),
    "ElasticNet":       ElasticNet(alpha=0.01, l1_ratio=0.5),
}

for name, m in models.items():
    m.fit(X_train, y_train)
    pred = m.predict(X_test)
    print(f"{name:17s} R2 = {r2_score(y_test, pred):.3f}   MSE = {mean_squared_error(y_test, pred):.0f}")

# LinearRegression  R2 = 0.453   MSE = 2900
# Ridge             R2 = 0.461   MSE = 2856
# Lasso             R2 = 0.472   MSE = 2798
# ElasticNet        R2 = 0.374   MSE = 3319
```

(For this dataset the scikit-learn authors have already centred and scaled the features, so no scaler is needed. Your own data will usually need one before using Ridge or Lasso, since the penalty treats all coefficients equally.)

Two metrics appear here, both from `sklearn.metrics`:

- **R²** (`r2_score`) is the fraction of the variation in `y` that the model explains: 1.0 is perfect, 0.0 is no better than always predicting the average, and it can go negative if the model is worse than that. An R² of about 0.45 means the model explains less than half the variation, a modest result on a hard, noisy problem.
- **MSE** (`mean_squared_error`) is the average squared error, in squared units of the target. Lower is better.

Lasso has a distinctive party trick. Because its penalty can push coefficients all the way to zero, it performs automatic *feature selection*:

```python
lasso = Lasso(alpha=0.1).fit(X_train, y_train)
print(lasso.coef_.round(1))
# [   0.  -152.7  552.7  303.4  -81.4   -0.  -229.3    0.   447.9   29.6]
```

Three of the ten features were switched off entirely (their coefficients are zero). If you have hundreds of features and suspect most are irrelevant, this is very useful.

### Logistic regression: a classifier with a misleading name

Despite the word "regression", `LogisticRegression` is a **classification** model. It computes a weighted sum like the others, then squashes the result into a probability between 0 and 1. For more than two classes it produces one probability per class.

```python
from sklearn.datasets import load_iris
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

X, y = load_iris(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y)

clf = LogisticRegression(max_iter=200)
clf.fit(X_train, y_train)

print(accuracy_score(y_test, clf.predict(X_test)))   # 0.9666666666666667
print(clf.predict_proba(X_test[:1]).round(3))        # [[0.985 0.015 0.   ]]
```

`predict_proba` shows the model's confidence: for the first test flower, 98.5% class 0 (setosa), 1.5% class 1, essentially none for class 2. The `predict` method simply picks the class with the highest probability. `max_iter=200` gives the solver enough iterations to converge; if you see a `ConvergenceWarning`, raise it or scale your features.

**Accuracy** (`accuracy_score`, or `clf.score(X_test, y_test)`) is the fraction of predictions that were correct. Here, 29 of 30.

---

## 8. Tree-Based Models

```python
import sklearn.tree
```

A decision tree predicts by asking a chain of yes/no questions, exactly like the nested `if-elif-else` statements from Guideline 7. The difference is that the tree writes those questions itself: at each step it looks for the feature and threshold that best separates the classes, splits the data there, and repeats on each half.

```python
from sklearn.tree import DecisionTreeClassifier, export_text

dt = DecisionTreeClassifier(max_depth=3, random_state=42)
dt.fit(X_train, y_train)

print(dt.score(X_test, y_test))    # 0.9666666666666667
```

The most attractive property of a tree is that you can *read* what it learned:

```python
from sklearn.datasets import load_iris
names = load_iris().feature_names
print(export_text(dt, feature_names=names))
# |--- petal length (cm) <= 2.45
# |   |--- class: 0
# |--- petal length (cm) >  2.45
# |   |--- petal width (cm) <= 1.65
# |   |   |--- petal length (cm) <= 4.95
# |   |   |   |--- class: 1
# |   |   |--- petal length (cm) >  4.95
# |   |   |   |--- class: 2
# |   |--- petal width (cm) >  1.65
# |   |   |--- petal length (cm) <= 4.85
# |   |   |   |--- class: 2
# |   |   |--- petal length (cm) >  4.85
# |   |   |   |--- class: 2
```

Read it from the top: if the petal length is 2.45 cm or less, it is class 0 (setosa), full stop. Otherwise check the petal width, and so on. No other model in this guideline explains itself this plainly. Two other tools help you see a tree: `sklearn.tree.plot_tree(dt)` draws it with Matplotlib, and the attribute `feature_importances_` ranks how much each feature contributed:

```python
print(dt.feature_importances_.round(3))   # [0.    0.    0.579 0.421]
```

The sepal measurements were never used; the two petal measurements do all the work, a genuinely useful discovery about the data. The importances always sum to 1.

### Trees overfit

If you do not limit a tree, it keeps splitting until every training example sits in its own perfectly pure leaf, which is memorisation in its purest form:

```python
full = DecisionTreeClassifier(random_state=42).fit(X_train, y_train)
print(full.score(X_train, y_train))   # 1.0
print(full.score(X_test, y_test))     # 0.9333333333333333
```

A perfect score on training data and a lower one on unseen data: overfitting, visible in two lines. The remedy is to restrict the tree's growth with hyperparameters:

| Hyperparameter | Effect |
|---|---|
| `max_depth` | the maximum number of questions along any path |
| `min_samples_split` | a node needs at least this many rows to be split further |
| `min_samples_leaf` | every leaf must keep at least this many rows |
| `max_leaf_nodes` | caps the total number of leaves |
| `criterion` | how "best split" is measured (`"gini"` by default, or `"entropy"`) |

### Regression trees

`DecisionTreeRegressor` works the same way, but each leaf predicts the *average* target of the training rows that landed in it:

```python
from sklearn.tree import DecisionTreeRegressor

reg = DecisionTreeRegressor(max_depth=3, random_state=42)
reg.fit(Xd_train, yd_train)          # the diabetes split from section 7
print(reg.score(Xd_test, yd_test))   # 0.329...
```

Because a tree predicts one fixed value per leaf, its predictions form a staircase rather than a smooth line, one reason linear models are sometimes better for smooth numeric relationships. Trees need no scaling, handle mixed feature ranges naturally, and capture non-linear patterns and interactions, at the cost of being unstable: a small change in the data can produce a very different tree. The next section fixes that.

---

## 9. Ensemble Models

```python
import sklearn.ensemble
```

A single decision tree is unstable and prone to overfitting. But there is an old and reliable idea: many mediocre opinions, combined sensibly, often beat one expert opinion. An **ensemble** trains many models and combines their predictions. The ensemble methods differ in *how* the members are built and combined.

| Strategy | Idea | Scikit-learn classes |
|---|---|---|
| **Bagging** | train many models independently on random resamples of the data, then average or vote | `BaggingClassifier`, `RandomForestClassifier`, `ExtraTreesClassifier` |
| **Boosting** | train models one after another, each focusing on the mistakes of the previous ones | `AdaBoostClassifier`, `GradientBoostingClassifier`, `HistGradientBoostingClassifier` |
| **Voting** | combine several *different kinds* of model | `VotingClassifier` |

Each has a regression twin: replace `Classifier` with `Regressor` (for example `RandomForestRegressor`).

We will compare them on the breast cancer dataset, a real medical classification problem with 30 features:

```python
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.ensemble import (RandomForestClassifier, GradientBoostingClassifier,
                              AdaBoostClassifier, BaggingClassifier,
                              ExtraTreesClassifier, HistGradientBoostingClassifier)

bc = load_breast_cancer()
Xb, yb = bc.data, bc.target
Xb_train, Xb_test, yb_train, yb_test = train_test_split(
    Xb, yb, test_size=0.2, random_state=42, stratify=yb)

models = [RandomForestClassifier(random_state=42),
          GradientBoostingClassifier(random_state=42),
          AdaBoostClassifier(random_state=42),
          BaggingClassifier(random_state=42),
          ExtraTreesClassifier(random_state=42),
          HistGradientBoostingClassifier(random_state=42)]

for m in models:
    acc = m.fit(Xb_train, yb_train).score(Xb_test, yb_test)
    print(f"{type(m).__name__:32s} {acc:.3f}")

# RandomForestClassifier           0.956
# GradientBoostingClassifier       0.956
# AdaBoostClassifier               0.956
# BaggingClassifier                0.930
# ExtraTreesClassifier             0.956
# HistGradientBoostingClassifier   0.974
```

Before drawing conclusions, remember the lesson of section 6: the test set here has 114 rows, and the gap between 0.956 and 0.974 is exactly two patients. That is far too small a difference to declare a winner. What the table does tell you is that all of these methods handle the problem well without any tuning or scaling.

### Random forests

A **random forest** is bagging applied to decision trees, with one extra ingredient. Each tree trains on a random resample of the rows, *and* each split considers only a random subset of the features. This forces the trees to be different from one another, so their individual errors tend to cancel when the votes are counted. The key hyperparameters:

| Hyperparameter | Meaning |
|---|---|
| `n_estimators` | number of trees (default 100; more is more stable but slower) |
| `max_depth` | depth limit for each tree |
| `max_features` | how many features each split may consider |
| `n_jobs` | set to `-1` to train trees in parallel on all CPU cores |

A forest keeps the trees' most useful property, `feature_importances_`, now averaged across all trees and therefore more trustworthy:

```python
import numpy as np

rf = RandomForestClassifier(random_state=42).fit(Xb_train, yb_train)
top = np.argsort(rf.feature_importances_)[::-1][:3]
for i in top:
    print(bc.feature_names[i], round(rf.feature_importances_[i], 3))

# worst area 0.14
# worst concave points 0.13
# worst radius 0.098
```

### Boosting

Where bagging builds independent trees in parallel, **boosting** builds them in sequence. Each new (usually shallow) tree is trained to correct the errors left by all the trees before it. `AdaBoost` does this by increasing the weight of misclassified rows; `GradientBoosting` fits each new tree to the remaining errors directly. `HistGradientBoosting` is a much faster implementation for larger datasets, and it even handles missing values natively. Gradient boosting is behind many of the winning solutions on tabular-data competitions. The trade-off is a greater sensitivity to hyperparameters such as `learning_rate` (how big a correction each tree contributes) and `n_estimators`.

### Voting

`VotingClassifier` combines models of different *types*, letting each cast a vote:

```python
from sklearn.ensemble import VotingClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.neighbors import KNeighborsClassifier

vote = VotingClassifier(
    estimators=[("lr",  LogisticRegression(max_iter=5000)),
                ("tree", DecisionTreeClassifier(max_depth=3, random_state=42)),
                ("knn", KNeighborsClassifier())],
    voting="hard")                       # "hard": majority vote; "soft": average the probabilities

vote.fit(X_train, y_train)               # the iris split from section 6
print(vote.score(X_test, y_test))        # 0.9666666666666667
```

The list of `(name, model)` pairs is how scikit-learn labels the members. The idea works best when the members make *different* mistakes.

Ensemble regressors, such as `RandomForestRegressor`, average the members' numeric predictions instead of voting. On the diabetes split from section 7 it reaches an R² of about 0.44, in the same range as the linear models, a reminder that the fanciest model does not automatically win on a noisy dataset.

---

## 10. Probabilistic Models

```python
import sklearn.naive_bayes
```

**Naive Bayes** classifiers apply Bayes' theorem: given what we observed, how probable is each class? For a class *C* and observed features, the model computes the probability of the class before seeing the data (the *prior*), multiplies by how likely those features are within that class, and picks the class with the highest result.

The "naive" part is a deliberate simplification: the model assumes the features are **independent of each other given the class**. This is almost never literally true (the width and length of a petal are correlated), yet the classifier often works surprisingly well anyway. It is extremely fast, needs little data, and produces probabilities directly.

Scikit-learn provides several variants, each assuming a different distribution for the features:

| Class | Assumes the features are | Typical use |
|---|---|---|
| `GaussianNB` | continuous, bell-shaped within each class | measurements (height, chemistry, sensors) |
| `MultinomialNB` | counts | word counts in text |
| `BernoulliNB` | binary (present / absent) | whether each word appears at all |
| `ComplementNB` | counts, adapted for imbalanced classes | text classification with uneven classes |
| `CategoricalNB` | categories encoded as integers | categorical features |

### GaussianNB on flower measurements

```python
from sklearn.naive_bayes import GaussianNB

nb = GaussianNB()
nb.fit(X_train, y_train)               # the iris split from section 6

print(nb.score(X_test, y_test))        # 0.9666666666666667
print(nb.predict_proba(X_test[:1]).round(3))   # [[1. 0. 0.]]
```

Because `GaussianNB` models each feature as a bell curve within each class, it needs no scaling.

### MultinomialNB for text

Naive Bayes is a classic tool for spam filtering. Text must first be turned into numbers, and the simplest way is to count words. `CountVectorizer` (from `sklearn.feature_extraction.text`) builds a table where each column is a word and each row is a document:

```python
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB

texts = ["win money now", "free money offer", "meeting at noon",
         "project meeting notes", "free offer now", "lunch at noon"]
labels = [1, 1, 0, 0, 1, 0]            # 1 = spam, 0 = normal

cv = CountVectorizer()
X_counts = cv.fit_transform(texts)     # learn the vocabulary, count the words

nb = MultinomialNB().fit(X_counts, labels)

new = cv.transform(["free money", "noon meeting"])
print(nb.predict(new))                 # [1 0]
print(nb.predict_proba(cv.transform(["free money"])).round(3))   # [[0.1 0.9]]
```

The model has never seen the exact message "free money", but it has learned that "free" and "money" appear in spam and "noon" and "meeting" appear in normal mail, so it gives the first message a 90% probability of being spam. Six sentences are a toy; the identical code with thousands of real emails is a working spam filter. (This is also a first taste of the text processing developed in the Natural Language Processing chapter of *Artificial Intelligence in Everything*.)

---

## 11. Discriminant Analysis Models

```python
import sklearn.discriminant_analysis
```

Discriminant analysis is a close relative of Naive Bayes: it also asks which class is most probable, but it assumes each class's features follow a **bell-shaped (Gaussian) distribution in several dimensions at once**, and, unlike Naive Bayes, it accounts for the *correlations* between features. Picture each class as a cloud of points in space, and the model as drawing the boundaries between the clouds.

| Class | Assumption | Decision boundary |
|---|---|---|
| `LinearDiscriminantAnalysis` (LDA) | all classes share the same shape of cloud | straight lines (flat planes) |
| `QuadraticDiscriminantAnalysis` (QDA) | each class has its own shape of cloud | curves |

LDA is the simpler, more stable one; QDA is more flexible but needs more data to estimate a separate shape for every class.

```python
from sklearn.discriminant_analysis import (LinearDiscriminantAnalysis,
                                           QuadraticDiscriminantAnalysis)

lda = LinearDiscriminantAnalysis().fit(X_train, y_train)     # the iris split from section 6
print(lda.score(X_test, y_test))    # 1.0

qda = QuadraticDiscriminantAnalysis().fit(X_train, y_train)
print(qda.score(X_test, y_test))    # 1.0
```

Both score perfectly on these 30 test flowers. Do not read that as "perfect model": with 30 rows it means only that none of the 30 was misclassified, and iris is famously an easy dataset.

### LDA as dimensionality reduction

LDA has a second use that makes it stand out. Because it is looking for the directions along which the classes are *best separated*, it can also be used to **project** the data into fewer dimensions with `transform`. With three classes, at most two such directions exist:

```python
lda2 = LinearDiscriminantAnalysis(n_components=2).fit(X, y)
Z = lda2.transform(X)
print(Z.shape)                                 # (150, 2)
print(lda.explained_variance_ratio_.round(3))  # [0.99 0.01]
```

The four measurements have been compressed into two columns, and the ratio shows that the first direction alone carries 99% of the class-separating information. Contrast this with PCA, coming in section 16: PCA ignores the labels and looks for the directions of greatest *variation*, while LDA uses the labels and looks for the directions of greatest *separation*. That makes LDA supervised and PCA unsupervised, and they may choose quite different directions.

---

## 12. Nearest Neighbors Models

```python
import sklearn.neighbors
```

The k-nearest neighbors (KNN) idea is the most human of all the algorithms: to judge a new example, find the *k* most similar examples you have already seen, and let them decide. If you want to guess whether a new neighbour is friendly, you look at the people living next to them.

- For **classification**, the *k* neighbours vote, and the majority class wins.
- For **regression**, the prediction is the average of the *k* neighbours' target values.

KNN has an unusual property: there is essentially no training. `fit` just stores the data, and all the work happens when you call `predict`, which measures the distance from the new point to every stored point. That makes prediction slow on very large datasets.

| Hyperparameter | Meaning |
|---|---|
| `n_neighbors` | *k*, the number of neighbours consulted (default 5) |
| `weights` | `"uniform"` (all votes equal) or `"distance"` (closer neighbours count more) |
| `metric` | how distance is measured (`"minkowski"` is the default; with `p=2` it is ordinary straight-line distance) |

### Choosing k

A small *k* follows the training data closely and is sensitive to noise; a large *k* smooths things out and can blur real boundaries. There is no universal best value, so you try several. Because KNN is distance-based, the features must be scaled first:

```python
from sklearn.neighbors import KNeighborsClassifier
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler().fit(X_train)                # the iris split from section 6
X_train_s = scaler.transform(X_train)
X_test_s = scaler.transform(X_test)

for k in (1, 3, 5, 7, 9, 15):
    knn = KNeighborsClassifier(n_neighbors=k).fit(X_train_s, y_train)
    print(k, round(knn.score(X_test_s, y_test), 3))

# 1 0.967
# 3 0.933
# 5 0.933
# 7 0.967
# 9 0.967
# 15 0.967
```

An honest caution: choosing the *k* that scores best on the test set quietly turns the test set into part of the training process. The rigorous way is to compare values of *k* with cross-validation (section 6) on the training data, and touch the test set only once at the end.

### Why scaling is not optional

The wine dataset has features on wildly different scales (one column is measured in hundreds, another in fractions). Watch what happens to KNN with and without scaling:

```python
from sklearn.datasets import load_wine

Xw, yw = load_wine(return_X_y=True)
Xw_train, Xw_test, yw_train, yw_test = train_test_split(
    Xw, yw, test_size=0.25, random_state=42, stratify=yw)

raw = KNeighborsClassifier().fit(Xw_train, yw_train)
print(raw.score(Xw_test, yw_test))            # 0.7777...

sw = StandardScaler().fit(Xw_train)
scaled = KNeighborsClassifier().fit(sw.transform(Xw_train), yw_train)
print(scaled.score(sw.transform(Xw_test), yw_test))   # 0.9333...
```

The same algorithm and the same data go from 78% to 93% purely because the large-valued feature no longer drowns out the others in the distance calculation.

### Finding neighbours directly

Sometimes the neighbours themselves are the answer, for example in a "customers similar to this one" or "similar products" feature. `NearestNeighbors` is the unsupervised version: it does no predicting, only searching.

```python
import numpy as np
from sklearn.neighbors import NearestNeighbors

points = np.array([[0, 0], [1, 1], [5, 5], [6, 6]])
nn = NearestNeighbors(n_neighbors=2).fit(points)

distances, indices = nn.kneighbors([[0.9, 0.9]])
print(distances)   # [[0.14142136 1.27279221]]
print(indices)     # [[1 0]]
```

The two points closest to (0.9, 0.9) are point 1, at (1, 1), a distance of 0.14 away, and point 0, at (0, 0), 1.27 away. Regression works identically through `KNeighborsRegressor`.

---

## 13. Support Vector Machines

```python
import sklearn.svm
```

A support vector machine (SVM) approaches classification geometrically. Picture two groups of points on a page. Many straight lines could separate them, but the SVM looks for the *best* one: the line that stays as far as possible from the nearest points on both sides, leaving the widest empty street between the classes. The points that sit on the edge of that street, and therefore define it, are the **support vectors**. Points far from the street are irrelevant to the result.

| Class | Task |
|---|---|
| `SVC` | classification |
| `SVR` | regression |
| `LinearSVC` | classification with a linear boundary only, much faster on large datasets |

### The kernel trick

Real classes are rarely separable by a straight line. SVMs handle this with **kernels**, mathematical shortcuts that let the model behave as if the data had been lifted into a higher-dimensional space where a flat boundary does work, without ever computing that space. You select a kernel with the `kernel` parameter:

| `kernel=` | Boundary |
|---|---|
| `"linear"` | straight line or plane |
| `"rbf"` | smooth, flexible curves (the default and usually the first thing to try) |
| `"poly"` | polynomial curves |
| `"sigmoid"` | S-shaped, similar to a neural-network activation |

```python
from sklearn.svm import SVC

# iris, scaled (X_train_s, X_test_s from section 12)
for kernel in ("linear", "rbf", "poly", "sigmoid"):
    svc = SVC(kernel=kernel).fit(X_train_s, y_train)
    print(f"{kernel:8s} {svc.score(X_test_s, y_test):.3f}")

# linear   1.000
# rbf      0.967
# poly     0.900
# sigmoid  0.900
```

Notice that the simplest kernel wins here. Iris is nearly linearly separable, so extra flexibility has nothing to add. A flexible tool is not automatically a better tool.

### The two dials: C and gamma

- **`C`** controls how strictly the model insists on classifying training points correctly. A large `C` tolerates few mistakes, producing a tight, complicated boundary that risks overfitting; a small `C` accepts some mistakes in exchange for a smoother boundary.
- **`gamma`** (for `rbf`, `poly` and `sigmoid`) controls how far the influence of a single training point reaches. A large `gamma` makes each point's influence very local, giving a wiggly boundary that hugs the data; a small `gamma` gives a smoother one.

Both push in the same direction: bigger values fit the training data more tightly. Tuning them together is the everyday work of using an SVM.

### SVMs demand scaling

Like KNN, SVMs depend on distances, and they can be dramatically worse without scaling. On the breast cancer data (the split from section 9):

```python
from sklearn.preprocessing import StandardScaler

print(SVC().fit(Xb_train, yb_train).score(Xb_test, yb_test))     # 0.9298

sb = StandardScaler().fit(Xb_train)
svc = SVC().fit(sb.transform(Xb_train), yb_train)
print(svc.score(sb.transform(Xb_test), yb_test))                 # 0.9825
```

From 93% to 98% by adding one line. In practice, wrapping the scaler and the SVM in a pipeline (section 6) is the safest habit.

### Support vectors and probabilities

After fitting, the support vectors can be inspected:

```python
svc = SVC(probability=True, random_state=42).fit(X_train_s, y_train)   # iris
print(svc.n_support_)               # [10 18 19]  support vectors per class
print(svc.support_vectors_.shape)   # (47, 4)
```

Only 47 of the 120 training flowers ended up defining the boundary; the rest could be deleted without changing the model. An SVM does not naturally output probabilities. Passing `probability=True` makes it estimate them (through an internal cross-validation), at the cost of slower training, and after that `predict_proba` becomes available.

`SVR` applies the same idea to regression, fitting a tube around the data and ignoring errors smaller than the tube's width (`epsilon`). `LinearSVC` restricts itself to a straight boundary, which makes it far quicker on datasets with many rows; on the scaled breast cancer data it scores about 0.965.

---

## 14. Neural Network Models

```python
import sklearn.neural_network
```

A **neural network** is a stack of layers of simple units. Each unit takes a weighted sum of its inputs, passes the result through a non-linear **activation function**, and hands it to the next layer. Layer by layer, the network builds increasingly abstract combinations of the raw features. Training adjusts all the weights, using gradient descent, to shrink the prediction error.

Scikit-learn offers a basic, CPU-only version of this idea, the **multi-layer perceptron**:

| Class | Task |
|---|---|
| `MLPClassifier` | classification |
| `MLPRegressor` | regression |

| Hyperparameter | Meaning |
|---|---|
| `hidden_layer_sizes` | a tuple giving the number of units in each hidden layer: `(16, 8)` means two hidden layers with 16 and 8 units |
| `activation` | `"relu"` (default), `"tanh"`, `"logistic"` |
| `solver` | the optimiser: `"adam"` (default), `"sgd"`, `"lbfgs"` (good for small datasets) |
| `max_iter` | the maximum number of training passes |
| `alpha` | strength of the L2 penalty on the weights |
| `random_state` | fixes the random starting weights, which otherwise change every run |

```python
from sklearn.neural_network import MLPClassifier

# breast cancer, scaled (sb from section 13)
Xb_train_s = sb.transform(Xb_train)
Xb_test_s = sb.transform(Xb_test)

mlp = MLPClassifier(hidden_layer_sizes=(16, 8), max_iter=1000, random_state=42)
mlp.fit(Xb_train_s, yb_train)

print(mlp.score(Xb_test_s, yb_test))        # 0.956...
print([w.shape for w in mlp.coefs_])        # [(30, 16), (16, 8), (8, 1)]
print(mlp.n_iter_)                          # 345
print(mlp.loss_curve_[0].round(3), mlp.loss_curve_[-1].round(3))   # 1.067 0.014
```

The learned weights are stored in `coefs_`, one matrix per connection between layers: 30 inputs into 16 units, 16 into 8, and 8 into a single output (one output is enough for a two-class problem). `loss_curve_` records the training error at every iteration, and plotting it with Matplotlib (Guideline 38) is the usual way to check that training is going somewhere: it should fall steadily and level off.

Two practical notes:

- **Scale your features.** Neural networks are very sensitive to the size of their inputs. On this same dataset, the same network with the unscaled features scores about 0.930 rather than 0.956.
- **`ConvergenceWarning`** means training ran out of iterations before the error stopped improving. Raise `max_iter`, scale the data, or simplify the network.

A small caution on interpretation: the training loss above ended at 0.014, which is very close to zero. When a network is that confident on data it has already seen, it may have memorised some of it. It is again the *test* score, not the training loss, that tells the truth.

Scikit-learn's networks are a good way to learn the concept and a reasonable tool for small tabular problems, but they lack much of what serious deep learning requires: GPU support, convolutional layers for images, recurrent layers for sequences, and fine control over the architecture. For those tasks you use TensorFlow or PyTorch (introduced in Guideline 25).

---

## 15. Clustering Models

```python
import sklearn.cluster
```

Everything so far had an answer key. Clustering does not: given only `X`, it groups the rows so that examples in the same group are more similar to each other than to those in other groups. We will use synthetic blobs so that we know the true structure:

```python
from sklearn.datasets import make_blobs

Xc, yc = make_blobs(n_samples=300, centers=3, cluster_std=1.0, random_state=42)
```

### K-Means

K-Means is the workhorse. You tell it how many clusters (*k*) you want, and it does the following: place *k* points ("centroids") at random, assign every row to its nearest centroid, move each centroid to the mean of the rows assigned to it, and repeat until nothing moves.

```python
from sklearn.cluster import KMeans

km = KMeans(n_clusters=3, n_init=10, random_state=42)
km.fit(Xc)

print(km.cluster_centers_.round(2))
# [[-2.63  9.04]
#  [-6.88 -6.98]
#  [ 4.75  2.01]]

print(np.bincount(km.labels_))      # [100 100 100]
print(round(km.inertia_, 1))        # 566.9
print(km.predict([[0, 0]]))         # which cluster does a new point belong to?
```

| Attribute / method | Meaning |
|---|---|
| `labels_` | the cluster number assigned to each training row |
| `cluster_centers_` | the coordinates of each centroid |
| `inertia_` | the sum of squared distances from each row to its centroid (lower means tighter clusters) |
| `predict(X)` | assign new rows to the nearest existing centroid |

The random starting points matter, since a bad start can lead to a poor result, so `n_init=10` runs the algorithm ten times from different starts and keeps the best. The cluster *numbers* themselves are arbitrary (the group called 0 in one run may be called 2 in another); only the grouping is meaningful.

### But how many clusters?

K-Means needs *k* in advance, and in real data nobody hands you the answer. Two standard tools help.

The **elbow method** runs K-Means for a range of *k* and looks at how the inertia falls:

```python
inertias = [KMeans(n_clusters=k, n_init=10, random_state=42).fit(Xc).inertia_
            for k in range(1, 8)]
print([round(i) for i in inertias])
# [20402, 5763, 567, 496, 427, 375, 308]
```

Inertia always decreases as clusters are added (with 300 clusters it would be zero), so you look for the "elbow" where the improvement stops being dramatic. Here it plunges from 5763 to 567 when going from two clusters to three, and then only creeps downwards. The bend is at *k* = 3, which matches how the data was built.

The **silhouette score** (`sklearn.metrics.silhouette_score`) measures, for every point, how much closer it is to its own cluster than to the next-nearest one. It ranges from −1 to 1, and higher is better:

```python
from sklearn.metrics import silhouette_score

for k in range(2, 7):
    labels = KMeans(n_clusters=k, n_init=10, random_state=42).fit_predict(Xc)
    print(k, round(silhouette_score(Xc, labels), 3))

# 2 0.705
# 3 0.848
# 4 0.664
# 5 0.49
# 6 0.517
```

The score peaks at *k* = 3, agreeing with the elbow. (`fit_predict` fits and returns the labels in one call.) With messier real data, the two methods will often disagree, and the final decision is a judgment about what is useful.

### DBSCAN: clusters by density

K-Means assumes clusters are round blobs and forces every point into one of them. **DBSCAN** defines a cluster as a dense region of points, separated from other dense regions by sparse space. It does not need *k*; instead you set the neighbourhood radius `eps` and the number of neighbours `min_samples` that makes a region "dense". Points that belong to no dense region are labelled **noise** with the special label −1:

```python
from sklearn.cluster import DBSCAN

db = DBSCAN(eps=1.0, min_samples=5).fit(Xc)
print(set(db.labels_))            # {0, 1, 2, -1}
print((db.labels_ == -1).sum())   # 5  (points treated as noise)
```

DBSCAN found the same three groups and set aside five stray points as outliers, something K-Means cannot do. Its real strength shows on shapes that are not blobs. The two interleaved half-moons from `make_moons` defeat K-Means completely but not DBSCAN:

```python
from sklearn.datasets import make_moons
from sklearn.metrics import adjusted_rand_score

Xm, ym = make_moons(300, noise=0.05, random_state=42)

km_labels = KMeans(2, n_init=10, random_state=42).fit_predict(Xm)
db_labels = DBSCAN(eps=0.2).fit_predict(Xm)

print(adjusted_rand_score(ym, km_labels))   # 0.247...
print(adjusted_rand_score(ym, db_labels))   # 1.0
```

The **adjusted Rand index** compares a clustering against known true groups: 1.0 is a perfect match and values near 0 are no better than random. It is only usable when you happen to have the true labels, as we do with synthetic data. K-Means cuts each moon in half by a straight line (0.25); DBSCAN traces each moon along its curve (1.0).

### Agglomerative (hierarchical) clustering

**Agglomerative clustering** starts with every point as its own cluster and repeatedly merges the two closest clusters until the desired number remains. The `linkage` setting defines "closest": `"ward"` (the default, merging so as to increase variance least), `"complete"`, `"average"` or `"single"`.

```python
from sklearn.cluster import AgglomerativeClustering

ag = AgglomerativeClustering(n_clusters=3, linkage="ward").fit(Xc)
print(np.bincount(ag.labels_))          # [100 100 100]
print(adjusted_rand_score(yc, ag.labels_))   # 1.0
```

Recording the order of the merges produces a tree of nested groupings (a *dendrogram*), which is valuable when you care about how groups relate to one another, not merely what they are.

### Do clusters find the "real" categories?

Try K-Means on the iris measurements, pretending we do not know the species, then compare with the truth:

```python
from sklearn.datasets import load_iris

X, y = load_iris(return_X_y=True)
Xs = StandardScaler().fit_transform(X)

labels = KMeans(n_clusters=3, n_init=10, random_state=42).fit_predict(Xs)
print(adjusted_rand_score(y, labels))    # 0.620...
```

A score of 0.62 is a partial match: the setosa flowers separate cleanly, but versicolor and virginica overlap and get partly mixed. Clusters reflect the structure of the *features you gave it*, which is not always the categories humans care about. As with any unsupervised result, treat clusters as a hypothesis to examine, not a fact to accept. And as with KNN, scaling first is important, since clustering depends on distances.

---

## 16. Dimensionality Reduction

```python
import sklearn.decomposition
```

Data with dozens or thousands of features has two problems: it is impossible to *see*, and many of the features are redundant. Height in centimetres and height in inches carry the same information; pixels next to each other in a photo are nearly identical. **Dimensionality reduction** compresses the data into fewer new features while keeping as much of the information as possible.

### PCA

**Principal Component Analysis (PCA)** is the standard technique. It finds new axes, called **principal components**, which are combinations of the original features. The first component points in the direction along which the data varies most; the second points along the greatest remaining variation at right angles to the first; and so on. Keep only the first few, and you have a compact summary of the data.

```python
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import load_iris

X, y = load_iris(return_X_y=True)
Xs = StandardScaler().fit_transform(X)      # PCA needs scaled features

pca = PCA(n_components=2)
Z = pca.fit_transform(Xs)

print(Z.shape)                              # (150, 2)
print(pca.explained_variance_ratio_.round(3))         # [0.73  0.229]
print(pca.explained_variance_ratio_.sum().round(3))   # 0.958
```

Four features have become two, and `explained_variance_ratio_` reports how much of the original variation each new component keeps: the first holds 73%, the second 22.9%, together 95.8%. Nearly all of the information in four columns survives in two. Scaling first matters, because PCA chases variance, and an unscaled feature with big numbers would hijack the first component.

### How many components?

Fit PCA without a limit and look at the cumulative total:

```python
full = PCA().fit(Xs)
print(full.explained_variance_ratio_.round(3))              # [0.73  0.229 0.037 0.005]
print(np.cumsum(full.explained_variance_ratio_).round(3))   # [0.73  0.958 0.995 1.   ]
```

Two components reach 95.8%, three 99.5%. Rather than counting yourself, you can pass a fraction and let PCA decide how many components are needed to keep that share of the variance:

```python
print(PCA(n_components=0.95).fit(Xs).n_components_)   # 2
```

The savings become dramatic with larger data. The digits dataset has 64 pixel features per image; keeping 90% of the variance takes only 21 components:

```python
from sklearn.datasets import load_digits

digits = load_digits()
print(PCA(n_components=0.9).fit(digits.data).n_components_)   # 21
```

### Looking at the components

Each component is a recipe, a weighted mix of the original features, stored in `components_`:

```python
print(pca.components_.round(2))
# [[ 0.52 -0.27  0.58  0.56]
#  [ 0.38  0.92  0.02  0.07]]
```

The first component gives large positive weights to sepal length (0.52), petal length (0.58) and petal width (0.56), so it acts as an overall "size" axis. The second is dominated by sepal width (0.92). Interpreting components this way is not always possible, but when it is, it explains what the compressed data actually represents.

### Visualising

The reason PCA is so often the first step of data exploration is that two components can be plotted (Guideline 38):

```python
import matplotlib.pyplot as plt

plt.scatter(Z[:, 0], Z[:, 1], c=y)
plt.xlabel("Principal component 1")
plt.ylabel("Principal component 2")
plt.title("Iris flowers in two dimensions")
plt.show()
```

Coloured by species, the plot shows setosa cleanly isolated, with versicolor and virginica touching, echoing the clustering result from section 15. A 4-dimensional dataset has become a picture a human can read.

Compression can also be undone approximately. `inverse_transform` maps the reduced data back into the original feature space, and the difference between the original and the reconstruction is the information that was discarded:

```python
X_back = pca.inverse_transform(Z)
print(X_back.shape)     # (150, 4)
```

### Other tools in the module

| Class | Purpose |
|---|---|
| `PCA` | the standard choice for dense numeric data |
| `TruncatedSVD` | PCA-like reduction that works on sparse matrices (such as the word counts from section 10) without centring them |
| `NMF` | non-negative matrix factorisation: components and weights are never negative, so results read as "parts" (topics in text, features in images) |
| `FastICA` | separates mixed signals into independent sources, as in separating overlapping voices |
| `KernelPCA` | PCA with kernels (as in section 13), able to capture curved structure |

```python
from sklearn.decomposition import TruncatedSVD, NMF

svd = TruncatedSVD(n_components=2, random_state=42).fit(X)
print(svd.explained_variance_ratio_.round(3))     # [0.529 0.448]

nmf = NMF(n_components=2, init="nndsvda", random_state=42, max_iter=1000)
W = nmf.fit_transform(X)      # NMF requires every input value to be non-negative
print(W.shape)                # (150, 2)
```

A word about what these tools are *for*. Dimensionality reduction serves three different goals, and PCA is used for all of them: **visualisation** (reduce to two or three dimensions and plot), **compression** (store or process fewer columns), and **preprocessing** (feed the reduced features into another model, most often in a pipeline: `make_pipeline(StandardScaler(), PCA(n_components=0.95), SVC())`). Keep in mind that the new features are less interpretable than the original ones: "principal component 2" does not mean anything to a customer or a doctor.

---

*End of Guideline 39.*
