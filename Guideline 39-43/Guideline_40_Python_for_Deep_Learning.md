# Guideline 40: Python for Deep Learning

In Guideline 39 you met a small neural network hidden inside scikit-learn: `MLPClassifier`. It worked, but it was a sketch. It ran on the CPU only, its layers were all one kind, and you could not reach inside to change how it learned. Real deep learning, the kind that recognises faces, translates languages and generates images, needs more control and far more speed.

This guideline introduces the tool for that job: **TensorFlow** and its high-level interface **Keras**. You will build a network that reads handwritten digits, watch it learn, and then meet the two architectures that changed the field: networks that *see* (convolutional) and networks that *remember* (recurrent). The guideline ends with the trick that makes most real projects practical: borrowing a network someone else has already trained.

A few notes before we start:

- **This guideline builds on earlier ones.** Arrays and shapes come from NumPy (Guideline 34), plotting from Matplotlib (Guideline 38), and the ideas of overfitting, scaling and the train-test split from Guideline 39. If "overfitting" is not a familiar word, revisit Section 6 of Guideline 39 first.
- **Numbers marked with ≈ will differ on your machine.** Neural networks start from random weights, so accuracy varies slightly from run to run and from computer to computer. Numbers that describe a network's *structure* (shapes, parameter counts) are exact.
- **Some code downloads data.** Loading MNIST, IMDB or pre-trained models fetches files from the internet the first time you run it, so you need a connection.
- **Install:** `pip install tensorflow`. A GPU speeds training up dramatically but is not required for anything in this guideline; every example here runs on an ordinary laptop.

---

## 1. Deep Learning

Artificial intelligence, machine learning and deep learning are nested like Russian dolls:

- **Artificial intelligence** is the broad goal of making machines behave intelligently.
- **Machine learning** is the part of AI where the machine learns its rules from data (Guideline 39).
- **Deep learning** is the part of machine learning that uses neural networks with **many layers**. The word "deep" refers to that depth: the number of layers stacked between input and output.

### What is different about deep learning?

Think back to the workflow of Guideline 39. For a classic model such as a random forest, *you* did the thinking about the features. To recognise a cat in a photo, you would have to decide what a cat looks like in numbers (whisker density? ear angle? fur texture?), compute those features, and hand the table to the model. That step is called **feature engineering**, and it is slow, difficult and requires expertise.

A deep network takes the raw material directly: the pixels of the photo. Its early layers learn to detect simple things such as edges and colour blobs. Middle layers combine those into textures and shapes such as curves, eyes and ears. Later layers combine *those* into whole objects. Nobody programs this hierarchy; it emerges from training. This is called **representation learning**, and it is the reason deep learning conquered images, sound and language, where nobody could write down the right features by hand.

### When should you use it?

Deep learning is not automatically the right tool. A fair comparison:

| | Classic machine learning (Guideline 39) | Deep learning (this guideline) |
|---|---|---|
| Best on | tables of numbers and categories | images, audio, text, video |
| Data needed | hundreds to thousands of rows can be enough | usually many thousands to millions of examples |
| Feature work | you design the features | the network learns its own |
| Training cost | seconds to minutes on a laptop | minutes to weeks, often on a GPU |
| Interpretability | some models are readable (trees, linear) | a black box; hard to explain |

For a small spreadsheet of customer data, a random forest will very likely beat a neural network and will train in a blink. For a photo-classification project with 50,000 pictures, deep learning is the clear choice. Choosing the right tool is part of the skill.

### Why now?

The core ideas of neural networks are decades old. Three things made them practical: enormous datasets (the internet), fast hardware (GPUs, originally built for video games, turn out to be superb at the arithmetic neural networks need), and better training techniques such as the ReLU activation and Dropout, both of which appear below.

---

## 2. TensorFlow and Keras

**TensorFlow** is a library, developed by Google, for numerical computation on **tensors** (Section 4) that can run on CPUs, GPUs and specialised hardware. It can also automatically compute derivatives, which is what makes training possible (Section 7). **Keras** is the friendly, high-level interface for building neural networks. It ships inside TensorFlow, so a single install gives you both.

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers

print(tf.__version__)                             # e.g. 2.x
print(tf.config.list_physical_devices("GPU"))     # [] on a computer with no GPU
```

An empty list from the second line is perfectly fine: it just means TensorFlow will use the CPU.

For repeatable results, fix the random seeds at the top of your script (recall `random_state` in Guideline 39):

```python
keras.utils.set_random_seed(42)    # seeds Python, NumPy and TensorFlow together
```

### The five-step Keras workflow

Nearly every Keras project has the same shape, and the rest of this guideline fills in each step:

1. **Prepare the data** as tensors: scale it, reshape it, split it (Sections 4 and 5).
2. **Build the model** by stacking layers.
3. **Compile** it: choose the loss, the optimizer and the metrics (Section 8).
4. **Fit** it to the training data (Section 9).
5. **Evaluate and predict** on data it has never seen (Section 9).

### Building a model

The simplest way is `keras.Sequential`, a plain stack of layers where the output of each feeds the next:

```python
model = keras.Sequential([
    keras.Input(shape=(28, 28)),
    layers.Flatten(),
    layers.Dense(128, activation="relu"),
    layers.Dropout(0.2),
    layers.Dense(10, activation="softmax"),
])

model.summary()
```

`model.summary()` prints the architecture, and reading it is a skill worth acquiring:

```
┃ Layer (type)          ┃ Output Shape  ┃  Param # ┃
│ flatten (Flatten)     │ (None, 784)   │        0 │
│ dense (Dense)         │ (None, 128)   │  100,480 │
│ dropout (Dropout)     │ (None, 128)   │        0 │
│ dense_1 (Dense)       │ (None, 10)    │    1,290 │
 Total params: 101,770
```

The first entry of every output shape is `None`: that is the **batch size**, left open because it can vary (Section 8). *Param #* counts the numbers the network must learn. We will decode where those counts come from in Section 3.

For networks that are not a simple stack (multiple inputs, branches that merge), Keras also offers the *Functional API*, in which you call layers like functions on tensors. Section 12 uses it. A third style, subclassing `keras.Model`, gives full control and is beyond this guideline.

### Key layers and methods

| Layer | Purpose | Section |
|---|---|---|
| `Input` | declares the shape of the data entering the model | 2 |
| `Flatten` | unrolls a multi-dimensional input into one long vector | 2 |
| `Dense` | fully connected layer: every input connects to every unit | 3 |
| `Dropout` | randomly switches off units during training to fight overfitting | 2 |
| `Conv2D`, `MaxPooling2D` | pattern detectors and downsampling for images | 10 |
| `Embedding`, `SimpleRNN`, `LSTM`, `GRU` | handling text and sequences | 11 |
| `Rescaling`, `RandomFlip`, `RandomRotation` | scaling and data augmentation | 12 |

| Method | Purpose |
|---|---|
| `model.summary()` | print the layers, shapes and parameter counts |
| `model.compile(...)` | set the loss, optimizer and metrics |
| `model.fit(...)` | train |
| `model.evaluate(...)` | measure loss and metrics on a dataset |
| `model.predict(...)` | produce outputs for new data |
| `model.save("file.keras")` | store the architecture and learned weights |
| `keras.models.load_model("file.keras")` | load a saved model |

### Three ideas to preview: Dropout, early stopping and fine-tuning

Three techniques appear throughout deep learning. We introduce each here and use them properly in later sections.

**Dropout** fights overfitting (Guideline 39: doing brilliantly on training data and badly on new data). During each training step, a `Dropout(0.2)` layer randomly sets 20% of its inputs to zero. The network cannot rely on any single unit always being present, so it is forced to spread its knowledge around, like a team where any member might be absent tomorrow. Crucially, **dropout is active only during training**. When the model is used for prediction, nothing is dropped:

```python
d = layers.Dropout(0.5)
x = tf.ones((1, 8))

print(d(x, training=True).numpy())    # [[0. 2. 2. 0. 0. 0. 2. 0.]]   (random pattern)
print(d(x, training=False).numpy())   # [[1. 1. 1. 1. 1. 1. 1. 1.]]
```

Notice the survivors were doubled (2.0), not left at 1.0. Keras scales the surviving values by 1 / (1 − rate) so that the total signal stays the same size whether dropout is on or off. You never need to do this yourself; `fit()` switches dropout on and `predict()` switches it off automatically.

**Early stopping** fights overfitting in a different way: it stops training at the right moment. While a network trains, its error on the training data keeps falling. Its error on *held-out* data falls at first and then, once the network begins to memorise, turns around and rises. The best moment to stop is at the turning point. Section 9 shows the `EarlyStopping` callback that finds it automatically.

**Fine-tuning** means taking a network that has already been trained on one task and continuing its training, gently, on a new one. It is the heart of Section 12 and one of the most valuable practical techniques in the field.

---

## 3. Artificial Neural Networks

The term **artificial neural network** (ANN) sounds grand, but the building block is small enough to write in a few lines of Python. Terminology in this field is used loosely, so it helps to see three names as three levels of the same idea:

| Name | What it is |
|---|---|
| **Perceptron** | a single artificial neuron: the simplest network |
| **Multi-Layer Perceptron (MLP)** | layers of neurons, each layer fully connected to the next |
| **Neural network** | the broadest term, covering MLPs *and* convolutional, recurrent, transformer and every other architecture |

Every MLP is a neural network, but not every neural network is an MLP. (Guideline 32 drew networks as layered, weighted graphs; that picture is what we are now building.)

### The perceptron: one neuron

A neuron does three things: multiply each input by a weight, add the results together with a bias, and pass the total through an **activation function** (Section 6) that decides the output.

```
output = activation( w₁·x₁ + w₂·x₂ + … + b )
```

This is exactly the weighted sum of the linear models in Guideline 39, with an activation function added on the end. The weights are what the network *learns*; the bias shifts the decision point.

The original perceptron used the simplest activation of all, a step: output 1 if the total is above zero, otherwise 0. Here is a complete perceptron, in plain NumPy, learning the logical AND gate (output 1 only when both inputs are 1):

```python
import numpy as np

X = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y = np.array([0, 0, 0, 1])            # AND

w = np.zeros(2)                        # start with zero weights
b = 0.0
lr = 0.1                               # learning rate: how big each correction is

for epoch in range(10):
    for xi, yi in zip(X, y):
        prediction = int(xi @ w + b > 0)          # step activation
        error = yi - prediction
        w += lr * error * xi                       # nudge weights toward the right answer
        b += lr * error

print(w.round(2), round(b, 2))                     # [0.2 0.1] -0.2
print([int(xi @ w + b > 0) for xi in X])           # [0, 0, 0, 1]
```

Whenever the perceptron answers wrongly, it nudges each weight in the direction that would have made the answer more correct. After a few passes over the data it has learned a rule that separates the four cases. Mistakes drive learning: this simple loop is the ancestor of everything in this guideline.

### The limit of a single neuron

A perceptron draws a *straight line* between the classes. AND can be separated by a straight line, so it works. The logical XOR (output 1 when the inputs *differ*) cannot: no single line separates {(0,1), (1,0)} from {(0,0), (1,1)}. This famous limitation, pointed out in 1969, stalled the field for years. The remedy is to put neurons in **layers**.

### The MLP: layers of neurons

An MLP arranges neurons in layers:

- The **input layer** is simply the data.
- One or more **hidden layers** each contain many neurons that combine the previous layer's outputs. "Hidden" because you never see their values directly.
- The **output layer** produces the final answer.

In a **fully connected** (or **dense**) layer, every neuron receives every output of the previous layer. With a non-linear activation between the layers, a hidden layer can bend the boundary, so an MLP with even one hidden layer can solve XOR and, in theory, approximate any reasonable function. Stacking *more* hidden layers gives a deep network.

In Keras, a fully connected layer is `Dense(units, activation=...)`, and now the parameter counts from Section 2 make sense. The layer `Dense(128)` receiving 784 inputs has:

- 784 × 128 = 100,352 weights (one per connection)
- plus 128 biases (one per neuron)
- = **100,480** parameters ✓

and the final `Dense(10)` receiving 128 values has 128 × 10 + 10 = **1,290** ✓. The whole model has 101,770 numbers to learn. Modern deep networks have millions to billions, but they are counted in exactly the same way.

---

## 4. Tensors

TensorFlow gets its name from its data structure: the **tensor**. A tensor is a multi-dimensional array of numbers, the same thing as a NumPy array (Guideline 34) with a few extra abilities such as running on a GPU. Everything in deep learning is a tensor: the images going in, the weights inside, the predictions coming out. The number of dimensions is the tensor's **rank**.

| Rank | Name | Example | Shape |
|---|---|---|---|
| 0 | scalar | a single number, such as a loss value | `()` |
| 1 | vector | a list of numbers, such as one row of a table | `(3,)` |
| 2 | matrix | a table, or one grayscale image | `(28, 28)` |
| 3 | 3D tensor | a colour image, or a batch of sequences | `(32, 32, 3)` |
| 4 | 4D tensor | a batch of colour images | `(64, 32, 32, 3)` |

```python
import tensorflow as tf

scalar = tf.constant(5)
vector = tf.constant([1., 2., 3.])
matrix = tf.constant([[1, 2], [3, 4]])
cube   = tf.zeros((2, 3, 4))

for t in (scalar, vector, matrix, cube):
    print(t.ndim, t.shape, t.dtype)

# 0 ()        <dtype: 'int32'>
# 1 (3,)      <dtype: 'float32'>
# 2 (2, 2)    <dtype: 'int32'>
# 3 (2, 3, 4) <dtype: 'float32'>
```

Every tensor has a **shape** (the size along each dimension) and a **dtype** (the type of its numbers). Note that whole integers default to `int32` and decimals to `float32`. Neural networks work in `float32`, so integer data usually needs converting with `tf.cast(t, tf.float32)`.

### Tensors and NumPy

The two convert easily in both directions:

```python
import numpy as np

a = np.array([[1, 2], [3, 4]])
t = tf.convert_to_tensor(a)      # NumPy → tensor
print(t.dtype)                   # int64 (kept from NumPy)
print(t.numpy())                 # tensor → NumPy
```

Operations behave as you would hope. Arithmetic is element by element, and `tf.matmul` is the matrix product from Guideline 34:

```python
print((matrix * 2).numpy())               # [[2 4] [6 8]]
print(tf.matmul(matrix, matrix).numpy())  # [[ 7 10] [15 22]]
print(tf.reshape(tf.range(6), (2, 3)).numpy())   # [[0 1 2] [3 4 5]]
```

### Constants and variables

A `tf.constant` never changes. A `tf.Variable` can be updated in place, and it is what TensorFlow uses for a network's weights, because training is precisely the process of changing them:

```python
v = tf.Variable([1., 2.])
v.assign([3., 4.])
print(v.numpy())    # [3. 4.]
```

### The shape of real data

A recurring point of confusion is that data arrives in *batches*, so tensors have an extra leading dimension for the number of examples. The convention is that **the first axis is always the batch**:

| Data | Shape | Meaning |
|---|---|---|
| a table | `(rows, features)` | 1000 customers with 8 features → `(1000, 8)` |
| grayscale images | `(batch, height, width)` or `(batch, height, width, 1)` | 100 MNIST digits → `(100, 28, 28)` |
| colour images | `(batch, height, width, 3)` | the 3 is the red, green and blue channels |
| sequences (text, audio) | `(batch, timesteps, features)` | 64 sentences of 50 words each → `(64, 50)` before embedding |

Most shape errors in Keras come from mismatching these conventions, so `print(x.shape)` is the most useful debugging line you will ever write. Reshaping is done with NumPy, for example adding the channel axis that convolutional layers expect:

```python
imgs = np.random.randint(0, 256, (100, 28, 28), dtype=np.uint8)

print(imgs.shape)                       # (100, 28, 28)
print(imgs.reshape(100, -1).shape)      # (100, 784)      flatten each image
print(imgs[..., np.newaxis].shape)      # (100, 28, 28, 1) add a channel axis
```

### The validation set

In Guideline 39 you split your data in two: training and test. Deep learning needs a **third** piece, and its importance is hard to overstate.

Here is the problem. A neural network has many choices that are not learned by training: how many layers, how many units, the dropout rate, the learning rate, when to stop. These are **hyperparameters**, and you find good ones by trying several and comparing results. But if you compare them using the *test set*, you are quietly using the test set to make decisions. Each time you pick "whatever scored best on the test set", the test set leaks into the model, and your final test score becomes too optimistic. The test set must remain sealed until the very end.

The solution is a third portion, the **validation set**:

| Set | Used for | How often you look at it |
|---|---|---|
| **Training set** | learning the weights | every step |
| **Validation set** | tuning hyperparameters, early stopping, comparing designs | every epoch |
| **Test set** | one final, honest grade | **once**, at the very end |

A typical division is about 60–80% training, 10–20% validation and 10–20% test. Keras gives you two ways to supply validation data:

```python
# 1. Hold out a fraction of the training data automatically
model.fit(x_train, y_train, epochs=5, validation_split=0.1)

# 2. Provide your own validation set explicitly
model.fit(x_train, y_train, epochs=5, validation_data=(x_val, y_val))
```

Be careful with option 1: `validation_split` takes the **last** 10% of the arrays, *before* shuffling. If your data is sorted (all the zeros, then all the ones, and so on), the validation set could contain only the final classes. For sorted data, shuffle first or split it yourself with `train_test_split` from Guideline 39.

The validation results appear in the training log as `val_loss` and `val_accuracy`, right next to the training numbers. The gap between them is the most important diagnostic in deep learning (Section 9).

---

## 5. The MNIST Dataset

Every field has its "hello world". In deep learning it is **MNIST**: 70,000 small images of handwritten digits collected from American census workers and high-school students. Keras includes it, ready to load:

```python
from tensorflow import keras

(x_train, y_train), (x_test, y_test) = keras.datasets.mnist.load_data()

print(x_train.shape, y_train.shape)   # (60000, 28, 28) (60000,)
print(x_test.shape, y_test.shape)     # (10000, 28, 28) (10000,)
print(x_train.dtype)                  # uint8
print(x_train.min(), x_train.max())   # 0 255
print(y_train[:10])                   # [5 0 4 1 9 2 1 3 1 4]
```

The result is already split for you: **60,000 training images and 10,000 test images**. Each image is a 28 × 28 grid of grayscale pixels, with values from 0 (black) to 255 (white), and each label is the digit, 0 to 9. This is precisely the rank-3 tensor of Section 4.

Look at one with Matplotlib (Guideline 38):

```python
import matplotlib.pyplot as plt

plt.imshow(x_train[0], cmap="gray")
plt.title(f"Label: {y_train[0]}")
plt.axis("off")
plt.show()
```

### Preparing the data

Raw pixels of 0–255 make training unstable, for the same reason unscaled features hurt in Guideline 39. The simplest fix is to divide by 255, so every pixel lies between 0 and 1. Convert to `float32` at the same time:

```python
x_train = x_train.astype("float32") / 255.0
x_test  = x_test.astype("float32") / 255.0
```

Then carve a validation set out of the training data (Section 4). Here, the last 6,000 of the 60,000 images become validation data. (MNIST is shuffled already, so this is safe.)

```python
x_val, y_val = x_train[-6000:], y_train[-6000:]
x_train, y_train = x_train[:-6000], y_train[:-6000]

print(x_train.shape, x_val.shape, x_test.shape)
# (54000, 28, 28) (6000, 28, 28) (10000, 28, 28)
```

### Encoding the labels

Labels can be handled in two ways, and the choice must match the loss function you select in Section 8:

```python
from tensorflow.keras.utils import to_categorical

print(y_train[:3])                       # [5 0 4]    integer labels (what we have)
print(to_categorical([3], 10))           # [[0. 0. 0. 1. 0. 0. 0. 0. 0. 0.]]   one-hot
```

- **Integer labels** (0 to 9) are used with the loss `"sparse_categorical_crossentropy"`. This is the simplest, and what we will use.
- **One-hot labels** (a 1 in the position of the correct class) are used with `"categorical_crossentropy"`. This is the same one-hot idea as the encoders in Guideline 39.

### Other built-in datasets

MNIST is the beginner's dataset, and it is famously easy: even simple models score above 97%. When you want a harder benchmark, Keras includes others in `keras.datasets`, each loaded with the same `load_data()` call:

| Dataset | Content | Size | Task |
|---|---|---|---|
| `mnist` | handwritten digits, 28×28 grayscale | 60,000 train + 10,000 test | 10-class image classification |
| `fashion_mnist` | clothing items, 28×28 grayscale, same format as MNIST | 60,000 + 10,000 | 10-class classification (harder) |
| `cifar10` | small colour photos, 32×32×3 (planes, cars, birds, cats…) | 50,000 + 10,000 | 10-class classification |
| `cifar100` | like CIFAR-10 with 100 finer classes | 50,000 + 10,000 | 100-class classification |
| `imdb` | movie reviews (as sequences of word numbers) | 25,000 + 25,000 | positive/negative sentiment (Section 11) |
| `reuters` | news wires | about 11,000 | topic classification |

Because Fashion-MNIST has exactly the same shape as MNIST, you can swap it in by changing one word and everything else in this guideline still runs. That makes it an excellent exercise: the same network scores noticeably lower, showing that MNIST's "97%" said more about the dataset than about the network.

For a much larger catalogue (hundreds of datasets across images, text, audio and more), the separate `tensorflow-datasets` package (`pip install tensorflow-datasets`, imported as `tfds`) provides a uniform `tfds.load("name")` interface. And your own images can be loaded straight from folders with `keras.utils.image_dataset_from_directory`, which you will use in Section 12.

---

## 6. Activation Functions

We said a neuron passes its weighted sum through an *activation function*. Why is that needed? Because without one, the whole network collapses. A weighted sum of weighted sums is just another weighted sum: a hundred linear layers stacked together can do nothing that a single layer could not, and would be no better than the linear models of Guideline 39. The activation function injects **non-linearity**, and non-linearity is what lets depth pay off.

There are dozens of activation functions, but four cover almost everything you will meet.

We will pass the same five test values, −2 to 2, through the first three:

```python
import tensorflow as tf

z = tf.constant([-2., -0.5, 0., 0.5, 2.])
```

### 1. Sigmoid

The sigmoid squashes any number into the range **0 to 1**, with a smooth S-shaped curve:

```python
print(tf.keras.activations.sigmoid(z).numpy().round(3))
# [0.119 0.378 0.5   0.622 0.881]
```

Very negative inputs go towards 0, very positive towards 1, and zero maps to exactly 0.5. Because the output looks like a probability, sigmoid is the standard choice for the **output layer of a yes/no (binary) classifier**. It is rarely used in hidden layers of deep networks any more, because its curve goes almost flat at both ends. There the gradient is close to zero and learning becomes painfully slow (the "vanishing gradient" problem, see Section 7).

### 2. Tanh

Tanh (hyperbolic tangent) is a stretched cousin of the sigmoid, with output between **−1 and 1**, centred on zero:

```python
print(tf.keras.activations.tanh(z).numpy().round(3))
# [-0.964 -0.462  0.     0.462  0.964]
```

The zero-centred output usually makes training easier than sigmoid, and tanh remains in use inside recurrent networks (Section 11). It suffers from the same flat-ends problem, though.

### 3. ReLU

The **Rectified Linear Unit** is the modern default for hidden layers, and its rule is almost embarrassingly simple: *if the input is negative, output 0; otherwise pass it through unchanged*.

```python
print(tf.keras.activations.relu(z).numpy())
# [0.  0.  0.  0.5 2. ]
```

Despite being barely non-linear (just one bend), ReLU works wonderfully. It is extremely cheap to compute, and it does not go flat for positive values, so gradients keep flowing and deep networks train quickly. Its weakness is that a neuron whose input stays negative outputs zero forever and stops learning (a "dead" neuron); variants such as Leaky ReLU exist for that, but plain ReLU is the standard first choice.

### 4. Softmax

Softmax is different from the other three: it acts on a *whole vector at once* instead of one number at a time. It converts a list of raw scores into a list of **probabilities that sum to 1**:

```python
scores = tf.constant([2., 1., 0.1])
print(tf.nn.softmax(scores).numpy().round(3))
# [0.659 0.242 0.099]
```

The largest score gets the largest probability, and the three values add up to 1.0. That makes softmax the standard activation for the **output layer of a multi-class classifier**: the ten outputs of our digit network become "probability of 0, probability of 1, … probability of 9".

### Choosing an activation

| Where | Task | Activation |
|---|---|---|
| Hidden layers | almost everything | `"relu"` |
| Output layer | binary classification (one output) | `"sigmoid"` |
| Output layer | multi-class classification (one output per class) | `"softmax"` |
| Output layer | regression (predicting a number) | none, also called `"linear"` |

The output layer's activation and the loss function (Section 8) go together, so the second column matters: a mismatch here is one of the most common reasons for a network that runs but learns nothing.

---

## 7. Backpropagation

A network starts with random weights, so its first predictions are garbage. Training is the process of improving the weights. That raises two questions: *how do we measure how wrong the network is?* and *which way should each weight move to make it less wrong?*

### The loss

The **loss** (or cost) is a single number measuring how wrong the predictions are: near zero for a good model and large for a bad one. Section 8 lists the standard loss functions. Training means driving this one number down.

### Gradient descent

Imagine standing on a foggy mountainside and wanting to reach the valley. You cannot see the valley, but you can feel the slope beneath your feet. So you take a step downhill, feel the slope again, and repeat. That is **gradient descent**.

The "slope" is the **gradient**: for each weight, it tells you how much the loss would change if you nudged that weight up a little. To reduce the loss, move each weight *against* its gradient:

```
new weight = old weight − learning rate × gradient
```

The **learning rate** sets the size of the step. Too large and you overshoot the valley and bounce around (or blow up); too small and progress is agonisingly slow.

Here is gradient descent on a one-weight problem: minimise the loss (w − 2)², whose lowest point is at w = 2. The gradient of that loss is 2(w − 2). Start at w = 5 and take steps with a learning rate of 0.3:

```python
w = 5.0
for step in range(5):
    gradient = 2 * (w - 2.0)
    w = w - 0.3 * gradient
    print(round(w, 4), end="  ")

# 3.2  2.48  2.192  2.0768  2.0307
```

At w = 5 the gradient is 6, so the first step moves w to 5 − 0.3 × 6 = 3.2. The steps get smaller as the slope flattens near the bottom, and w homes in on 2. Real networks do exactly this, only with millions of weights at once instead of one.

### Backpropagation: finding all the gradients

A network with millions of weights needs a gradient for *every one* of them, and computing each separately would be hopelessly slow. **Backpropagation** is the algorithm that computes all of them efficiently, in a single sweep backwards through the network.

The idea rests on the chain rule from calculus (Guideline 36). One training step has two phases:

1. **Forward pass:** feed a batch of data through the layers, from input to output, and compute the loss.
2. **Backward pass:** starting from the loss, work backwards layer by layer. Each layer uses the chain rule to work out how much *it* contributed to the error, and passes that responsibility back to the layer before it. By the time the sweep reaches the first layer, every weight has its gradient.

Then gradient descent (or one of its improved versions, Section 8) moves each weight a small step. One forward pass, one backward pass and one update is a single **training step**; training is that step repeated thousands of times.

You almost never write backpropagation yourself. When you call `model.fit()`, Keras does all of it. But TensorFlow will show you the gradients on request with `tf.GradientTape`, which records the computation so that it can be differentiated:

```python
w = tf.Variable(3.0)

with tf.GradientTape() as tape:
    loss = (w - 1.0) ** 2

print(tape.gradient(loss, w).numpy())    # 4.0
```

The loss is (w − 1)² and w = 3, so the gradient is 2 × (3 − 1) = 4, and TensorFlow computed it for us without our writing any calculus. The same tape mechanism is what `fit()` uses internally, applied to every weight in the network.

### Vanishing gradients

The backward sweep multiplies many small numbers together, layer after layer. If those numbers are below 1, the gradient shrinks towards zero as it travels back, and the early layers barely learn. This **vanishing gradient problem** is the reason sigmoid and tanh fell out of fashion in deep hidden layers (their gradients are small when the curve is flat) and ReLU took over. It also motivated the LSTM and GRU designs in Section 11.

---

## 8. Compiling the Model

Building a model defines its structure. **Compiling** it tells Keras how it should learn. `compile()` takes three main choices:

```python
model.compile(
    optimizer="adam",
    loss="sparse_categorical_crossentropy",
    metrics=["accuracy"],
)
```

### The loss function

The loss measures how wrong the predictions are, and it must suit the task and the output layer:

| Task | Output layer | Loss |
|---|---|---|
| Binary classification | 1 unit, sigmoid | `"binary_crossentropy"` |
| Multi-class, integer labels | *k* units, softmax | `"sparse_categorical_crossentropy"` |
| Multi-class, one-hot labels | *k* units, softmax | `"categorical_crossentropy"` |
| Regression | 1 unit, no activation | `"mse"` (mean squared error) |

*Cross-entropy* measures how far the predicted probabilities are from the truth. It punishes a network heavily for being confidently wrong, which is exactly what you want.

### Metrics

Metrics are extra numbers reported during training so *you* can follow progress. Unlike the loss, they are not used to update the weights. `"accuracy"` is the usual choice for classification.

### Epochs and batch size

Two settings, supplied later in `fit()`, control how the training data is fed to the network. Both trip up newcomers, so let us be precise.

An **epoch** is one complete pass through the *entire* training set. Training for 5 epochs means the network sees every training image 5 times.

Updating the weights after every single image would be noisy, while computing the gradient over all 54,000 images before making one update would be slow. The compromise is to process the data in small groups called **batches**. The **batch size** is how many examples are processed before the weights are updated. This gives three regimes, which correspond to the three flavours of gradient descent:

| Batch size | Name | Behaviour |
|---|---|---|
| All the data (54,000) | **Batch gradient descent** | one very accurate but very slow update per epoch |
| 1 | **Stochastic gradient descent (SGD)** | one update per example: fast but very noisy |
| In between (32, 64, 128…) | **Mini-batch gradient descent** | the practical compromise, used almost everywhere |

("Stochastic" means "random": the noise in the small batches makes the path wobble, which can even help the network escape poor spots.) In practice, when people say "SGD" in deep learning, they nearly always mean mini-batch gradient descent, and the default batch size in Keras is **32**.

The two settings determine how many weight updates happen per epoch:

```
steps per epoch = ⌈ number of training examples ÷ batch size ⌉
```

With 60,000 images and a batch size of 32, one epoch is ⌈60,000 ÷ 32⌉ = **1,875 steps**. (With our 54,000 training images it is 1,688.) Larger batches run faster per epoch on a GPU and give smoother gradients but need more memory; small batches are noisier and take more steps. Values from 32 to 128 are a sensible starting range.

### The optimizer

The **optimizer** is the algorithm that turns gradients into weight updates. The basic recipe from Section 7 (weight −= learning rate × gradient) is plain SGD. Better optimizers add refinements.

| Optimizer | Idea |
|---|---|
| `"sgd"` | the basic update; add `momentum=0.9` to build up speed along consistent directions, like a rolling ball |
| `"rmsprop"` | adapts the step size for each weight individually, shrinking steps for weights with consistently large gradients |
| `"adam"` | combines momentum with per-weight adaptive step sizes |

**Adam** is the standard first choice: it works well with almost no tuning, and its default learning rate of 0.001 is right for a great many problems. Use plain SGD with momentum when you have time to tune it, as it sometimes generalises slightly better.

To change the learning rate or momentum, pass an optimizer *object* instead of a string:

```python
model.compile(
    optimizer=keras.optimizers.SGD(learning_rate=0.01, momentum=0.9),
    loss="sparse_categorical_crossentropy",
    metrics=["accuracy"],
)

model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=0.001),
    loss="sparse_categorical_crossentropy",
    metrics=["accuracy"],
)
```

The learning rate is the most influential setting in all of deep learning. If a network's loss explodes or stays flat, it is the first thing to try changing (by a factor of 10 at a time).

---

## 9. Training and Evaluation

Everything is ready. Here is the complete digit-recognition project in one place, from the data of Section 5:

```python
from tensorflow import keras
from tensorflow.keras import layers

keras.utils.set_random_seed(42)

# 1. data (Section 5)
(x_train, y_train), (x_test, y_test) = keras.datasets.mnist.load_data()
x_train = x_train.astype("float32") / 255.0
x_test  = x_test.astype("float32") / 255.0
x_val, y_val = x_train[-6000:], y_train[-6000:]
x_train, y_train = x_train[:-6000], y_train[:-6000]

# 2. model (Sections 2, 3, 6)
model = keras.Sequential([
    keras.Input(shape=(28, 28)),
    layers.Flatten(),
    layers.Dense(128, activation="relu"),
    layers.Dropout(0.2),
    layers.Dense(10, activation="softmax"),
])

# 3. compile (Section 8)
model.compile(optimizer="adam",
              loss="sparse_categorical_crossentropy",
              metrics=["accuracy"])

# 4. train
history = model.fit(x_train, y_train,
                    epochs=10,
                    batch_size=32,
                    validation_data=(x_val, y_val))
```

`fit()` prints one line per epoch. Typical output looks like this (your numbers will differ slightly):

```
Epoch 1/10
1688/1688 ━━━━━━━━━━━━━━━━━━━━ 3s - accuracy: ≈0.92 - loss: ≈0.27 - val_accuracy: ≈0.96 - val_loss: ≈0.13
Epoch 2/10
1688/1688 ━━━━━━━━━━━━━━━━━━━━ 3s - accuracy: ≈0.96 - loss: ≈0.12 - val_accuracy: ≈0.97 - val_loss: ≈0.10
...
Epoch 10/10
1688/1688 ━━━━━━━━━━━━━━━━━━━━ 3s - accuracy: ≈0.99 - loss: ≈0.03 - val_accuracy: ≈0.98 - val_loss: ≈0.08
```

Read the log like a doctor reads a chart. `1688/1688` is the number of steps per epoch (Section 8). `accuracy` and `loss` are measured on the training batches; `val_accuracy` and `val_loss` are measured on the validation set at the end of each epoch. Within ten epochs and a few seconds, a simple network has learned to read digits with roughly 98% accuracy.

### The History object

`fit()` returns a `History` object that stores those numbers for every epoch, so you can plot them:

```python
print(history.history.keys())
# dict_keys(['accuracy', 'loss', 'val_accuracy', 'val_loss'])

import matplotlib.pyplot as plt

plt.plot(history.history["loss"], label="training loss")
plt.plot(history.history["val_loss"], label="validation loss")
plt.xlabel("Epoch")
plt.ylabel("Loss")
plt.legend()
plt.show()
```

This plot is the single most useful diagnostic in deep learning. It has three typical shapes:

| What you see | Diagnosis | What to try |
|---|---|---|
| Both losses fall together and level off close to each other | healthy learning | train longer, or a bigger model, if the loss is still too high |
| Training loss keeps falling but validation loss turns **upward** | **overfitting**: the network is memorising | more Dropout, fewer units, more data, early stopping |
| Both losses stay high | **underfitting**: the model is too weak, or the learning rate is off | a bigger model, more epochs, adjust the learning rate |

In the typical log above, the training loss (0.03) is well below the validation loss (0.08): a small gap that shows mild overfitting, which is normal.

### Early stopping

Rather than guess the right number of epochs, let Keras watch the validation loss and stop when it stops improving. This is done with a **callback**, an object that `fit()` calls at the end of each epoch:

```python
early_stop = keras.callbacks.EarlyStopping(
    monitor="val_loss",          # the number to watch
    patience=3,                  # tolerate 3 epochs without improvement
    restore_best_weights=True,   # roll back to the best epoch when stopping
)

history = model.fit(x_train, y_train,
                    epochs=50,               # an upper limit; early stopping decides
                    batch_size=32,
                    validation_data=(x_val, y_val),
                    callbacks=[early_stop])
```

`patience` matters, because validation loss is noisy and one bad epoch does not mean the end. With `patience=3`, training continues until 3 epochs in a row fail to beat the best value, and `restore_best_weights=True` then reloads the weights from the best epoch instead of keeping the slightly worse final ones. You can now set `epochs` to a generous 50 and let the callback stop at the right moment.

A second useful callback saves the best model to disk while training:

```python
checkpoint = keras.callbacks.ModelCheckpoint("best_model.keras",
                                             monitor="val_loss",
                                             save_best_only=True)
```

Pass it in the same `callbacks=[...]` list.

### Evaluation: the one-time test

Only now, after all decisions are made, do we touch the test set:

```python
test_loss, test_acc = model.evaluate(x_test, y_test)
print(f"Test accuracy: {test_acc:.4f}")       # ≈0.98
```

`evaluate` returns the loss followed by each metric you listed in `compile`. This is the honest number, the one you would quote. If you evaluate on the test set, then change the model because of the result and evaluate again, you have started using the test set as a validation set, and the number is no longer honest.

### Prediction

`predict` returns the network's raw output. For our softmax output layer, that is ten probabilities per image:

```python
probs = model.predict(x_test[:3])
print(probs.shape)                       # (3, 10)
print(probs[0].round(3))                 # ten probabilities that add up to 1

import numpy as np
predicted_digits = np.argmax(probs, axis=1)
print(predicted_digits)                  # e.g. [7 2 1]
print(y_test[:3])                        # [7 2 1]  the true labels
```

`np.argmax(..., axis=1)` picks, for each row, the position of the largest probability, which is the predicted digit. (For a sigmoid binary output there is just one probability per example, and you compare it with 0.5 instead.)

### Saving and loading

Training may take hours, so save the finished model and reload it later without retraining:

```python
model.save("digits.keras")                            # architecture + weights
restored = keras.models.load_model("digits.keras")
print(restored.evaluate(x_test, y_test))
```

---

## 10. Convolutional Neural Networks

The digit network of the last sections has an odd habit: its very first step is `Flatten`, which unrolls the 28 × 28 image into one line of 784 numbers. That throws away the most important fact about images: **neighbouring pixels belong together**. A pixel's meaning depends on the pixels around it, and a "7" is a "7" wherever on the page it is written. A dense layer knows none of this; it treats the pixel in the top-left corner and the pixel in the centre as completely unrelated inputs.

**Convolutional neural networks (CNNs)** are designed around the structure of images. They are the reason computers can now recognise faces, read medical scans and drive cars.

### The convolution

The core operation is the **convolution**. A small grid of numbers, the **filter** (or *kernel*), typically 3 × 3, slides across the image. At each position it multiplies its numbers by the pixels beneath it and adds up the results, producing one output number. The grid of all those outputs is called a **feature map**.

Here is the operation done by hand. This 5 × 5 image has a vertical edge: dark on the left (0), bright on the right (9). The filter is a classic vertical-edge detector:

```python
import numpy as np

image = np.array([[0, 0, 9, 9, 9]] * 5, dtype=float)

kernel = np.array([[1, 0, -1]] * 3, dtype=float)      # left minus right

out = np.array([[np.sum(image[i:i+3, j:j+3] * kernel) for j in range(3)]
                for i in range(3)])
print(out)
# [[-27. -27.   0.]
#  [-27. -27.   0.]
#  [-27. -27.   0.]]
```

Where the 3 × 3 window covers the edge, the filter responds strongly (−27); over the flat bright area at the right, it says nothing (0). The filter is an *edge detector*, and the feature map shows where the edge is.

The magic of a CNN is that **you do not design the filters; the network learns them**. `Conv2D(32, 3)` creates 32 filters of size 3 × 3, starting with random values and training them by backpropagation. In the first layer they typically become edge and colour detectors; in deeper layers they combine into detectors for corners, textures, and eventually parts of objects.

Two properties make this efficient:

- **Weight sharing.** One filter is reused at every position, so it needs only 9 weights (plus a bias) however large the image. A dense layer would need a separate weight for every pixel.
- **Translation invariance.** Because the same filter scans the whole image, an edge is detected wherever it appears.

Useful `Conv2D` settings:

| Setting | Meaning |
|---|---|
| `filters` | how many filters (feature maps) the layer produces |
| `kernel_size` | the filter's size, usually 3 (meaning 3 × 3) |
| `strides` | how many pixels the filter moves each step (default 1) |
| `padding` | `"valid"` (default) does not pad, so the output shrinks; `"same"` pads with zeros so the output keeps the input's size |
| `activation` | applied to each output, normally `"relu"` |

With `"valid"` padding, a 3 × 3 filter cuts the output down by 2 in each direction: a 28 × 28 image becomes 26 × 26 (28 − 3 + 1), as in the by-hand example above (5 → 3).

### Pooling

After a convolution, a **pooling** layer shrinks the feature maps. `MaxPooling2D` divides each map into 2 × 2 blocks and keeps only the largest value in each, halving the height and width. This throws away exact positions, which makes the network less sensitive to small shifts, and it cuts the computation for the layers that follow.

### Building a CNN

The classic recipe is *convolve, pool, convolve, pool, then classify with dense layers*:

```python
from tensorflow import keras
from tensorflow.keras import layers

cnn = keras.Sequential([
    keras.Input(shape=(28, 28, 1)),
    layers.Conv2D(32, 3, activation="relu"),
    layers.MaxPooling2D(),
    layers.Conv2D(64, 3, activation="relu"),
    layers.MaxPooling2D(),
    layers.Flatten(),
    layers.Dropout(0.5),
    layers.Dense(10, activation="softmax"),
])

cnn.summary()
```

Convolutional layers expect an explicit channel axis (the 1 in `(28, 28, 1)` means grayscale), so MNIST needs one added, using the NumPy trick from Section 4:

```python
x_train_cnn = x_train[..., np.newaxis]     # (54000, 28, 28) → (54000, 28, 28, 1)
x_val_cnn   = x_val[..., np.newaxis]
x_test_cnn  = x_test[..., np.newaxis]
```

The summary shows how the image shrinks and grows in depth as it moves through the network:

```
┃ Layer (type)                  ┃ Output Shape       ┃ Param # ┃
│ conv2d (Conv2D)               │ (None, 26, 26, 32) │     320 │
│ max_pooling2d (MaxPooling2D)  │ (None, 13, 13, 32) │       0 │
│ conv2d_1 (Conv2D)             │ (None, 11, 11, 64) │  18,496 │
│ max_pooling2d_1 (MaxPooling2D)│ (None, 5, 5, 64)   │       0 │
│ flatten (Flatten)             │ (None, 1600)       │       0 │
│ dropout (Dropout)             │ (None, 1600)       │       0 │
│ dense (Dense)                 │ (None, 10)         │  16,010 │
 Total params: 34,826
```

Follow the shapes: 28 → conv → 26 → pool → 13 → conv → 11 → pool → 5, and the final feature maps of 5 × 5 × 64 flatten into 1,600 numbers. The parameter counts follow the same logic as Section 3: the first convolution has 3 × 3 × 1 × 32 weights plus 32 biases = 320, and the second has 3 × 3 × 32 × 64 + 64 = 18,496.

Compare this with the dense network of Section 2: it had 101,770 parameters, and this CNN has just **34,826**, about a third as many, yet it sees far better. That is the payoff of weight sharing.

Train it exactly as before:

```python
cnn.compile(optimizer="adam",
            loss="sparse_categorical_crossentropy",
            metrics=["accuracy"])

early_stop = keras.callbacks.EarlyStopping(monitor="val_loss", patience=3,
                                           restore_best_weights=True)

cnn.fit(x_train_cnn, y_train,
        epochs=20, batch_size=64,
        validation_data=(x_val_cnn, y_val),
        callbacks=[early_stop])

print(cnn.evaluate(x_test_cnn, y_test))     # test accuracy ≈0.99
```

A test accuracy of about 99%, from a network that trains in about a minute on a laptop, is a noticeable step up from the dense network's 98%. That last percent may look small, but it means roughly *halving* the number of mistakes. On harder image datasets such as CIFAR-10, the gap between CNNs and dense networks is enormous.

---

## 11. Recurrent Neural Networks

Images have space; many other kinds of data have **time**. A sentence is a sequence of words, a song is a sequence of notes, a stock price is a sequence of days. In a sequence, order matters: "dog bites man" and "man bites dog" contain the same words and mean very different things. Dense networks and CNNs take a fixed-size input and have no memory of what came before. **Recurrent neural networks (RNNs)** are built for sequences.

### The recurrent idea

An RNN reads a sequence one step at a time. It keeps a **hidden state**, a small vector that acts as its memory. At each step, it combines the new input with the previous hidden state to produce a new hidden state, then carries that forward to the next step. The same weights are reused at every step (a "loop"), which lets the network handle sequences of any length.

Read the sentence "the film was not good" one word at a time: by the time the network reaches "good", its hidden state still carries a trace of "not", and so it can tell the sentiment is negative. That trace is the memory.

### A sentiment classifier with the IMDB dataset

The IMDB dataset (Section 5) holds 50,000 movie reviews labelled positive or negative. Keras has already converted each review into a list of word numbers (each word replaced by its rank in a frequency table):

```python
from tensorflow import keras
from tensorflow.keras import layers
from tensorflow.keras.preprocessing.sequence import pad_sequences

(x_train, y_train), (x_test, y_test) = keras.datasets.imdb.load_data(num_words=10000)

print(len(x_train), len(x_test))     # 25000 25000
print(x_train[0][:10])               # a list of word numbers
print(y_train[:5])                   # 1 = positive, 0 = negative
```

`num_words=10000` keeps only the 10,000 most frequent words. Reviews vary in length, but a tensor must be rectangular, so we **pad** them all to the same length (shorter ones get zeros at the front, longer ones are cut):

```python
print(pad_sequences([[1, 2, 3], [4, 5]], maxlen=4))
# [[0 1 2 3]
#  [0 0 4 5]]

x_train = pad_sequences(x_train, maxlen=200)
x_test  = pad_sequences(x_test,  maxlen=200)
print(x_train.shape)                 # (25000, 200)
```

A word number such as 4172 is meaningless as a number (word 4172 is not "twice" word 2086). The **Embedding** layer solves this: it converts each word number into a short learned vector, so that words with similar meanings end up with similar vectors. Then the recurrent layer reads the vectors in order:

```python
rnn = keras.Sequential([
    keras.Input(shape=(None,)),
    layers.Embedding(input_dim=10000, output_dim=32),
    layers.SimpleRNN(32),
    layers.Dense(1, activation="sigmoid"),
])

rnn.summary()
```

```
┃ Layer (type)             ┃ Output Shape     ┃  Param # ┃
│ embedding (Embedding)    │ (None, None, 32) │  320,000 │
│ simple_rnn (SimpleRNN)   │ (None, 32)       │    2,080 │
│ dense (Dense)            │ (None, 1)        │       33 │
 Total params: 322,113
```

Note the design: a sigmoid output for yes/no (Section 6), so the loss is `binary_crossentropy` (Section 8):

```python
rnn.compile(optimizer="adam",
            loss="binary_crossentropy",
            metrics=["accuracy"])

rnn.fit(x_train, y_train,
        epochs=5, batch_size=64,
        validation_split=0.2)

print(rnn.evaluate(x_test, y_test))
```

(Here `validation_split` is safe, because the IMDB training reviews were already shuffled.) A simple RNN like this typically reaches a test accuracy somewhere in the 80s, which is respectable for so little code, but it stumbles on longer reviews for a reason explained next.

### The memory problem: LSTM and GRU

The plain RNN has a serious flaw. The gradients in backpropagation must travel back through *every time step* (Section 7), so the vanishing gradient problem is severe: over long sequences the signal fades and the network effectively forgets anything from more than a few steps ago. The beginning of a long review is lost by the time the network reaches the end.

Two improved designs, both available as drop-in replacements for `SimpleRNN`, address this by adding **gates**: small learned switches that control what to remember and what to forget.

**LSTM (Long Short-Term Memory)** was introduced in 1997 and adds a separate **cell state**, a kind of conveyor belt that carries information along the sequence with only small, controlled changes. Three gates manage it: the *forget gate* decides what to erase from the belt, the *input gate* decides what new information to write onto it, and the *output gate* decides what part of it to reveal as the current output. Because information can ride the belt unchanged, an LSTM can remember things across hundreds of steps. The price is that an LSTM layer has four sets of weights instead of one, so it is heavier and slower than a simple RNN.

**GRU (Gated Recurrent Unit)** is a streamlined cousin from 2014. It merges the memory into a single hidden state and uses just two gates: an *update gate*, which balances how much old memory to keep against how much new information to accept, and a *reset gate*, which decides how much of the past to ignore when forming the new information. It has three sets of weights instead of four, so it trains faster and needs fewer parameters, and it often performs about as well as an LSTM.

| | SimpleRNN | LSTM | GRU |
|---|---|---|---|
| Memory | a single hidden state | hidden state plus a separate cell state | a single, gated hidden state |
| Gates | none | three (forget, input, output) | two (update, reset) |
| Long-range memory | poor | strong | strong |
| Speed and size | lightest | heaviest | in between |

A practical rule: start with a GRU or an LSTM instead of a plain SimpleRNN for anything beyond very short sequences, and try both, because neither wins every time.

RNNs are the natural choice for time series, speech and text. Since about 2017, a newer architecture called the **transformer** has taken over most language tasks and also handles sequences without a step-by-step loop. Transformers are the technology behind today's large language models, and are covered in the Natural Language Processing chapter of *Artificial Intelligence in Everything*.

---

## 12. Transfer Learning

Training a good image network from scratch needs enormous amounts of data and computing time: the best ones learn from over a million labelled photos, using days of GPU time. You probably have a few thousand pictures and a laptop. Does that mean serious image recognition is out of reach?

No, and the reason is one of the most useful facts in deep learning. The early layers of an image network learn very general things such as edges, colours, textures and simple shapes, which are useful for *any* kind of picture, not just the ones it was trained on. So instead of starting from random weights, you can start from a network that someone else has already trained on a huge dataset, keep its general-purpose knowledge, and teach it only your specific task. This is **transfer learning**: the knowledge *transfers*.

It is the difference between hiring a fresh graduate and teaching them your company's work versus raising a baby from birth.

### Pre-trained models

`keras.applications` contains well-known architectures with weights already trained on **ImageNet**, a dataset of over a million photos in 1,000 categories:

| Model | Character |
|---|---|
| `MobileNetV2` | small and fast; designed for phones and laptops |
| `ResNet50` | a classic workhorse with 50 layers |
| `VGG16` | older, simple structure, and large |
| `EfficientNetB0` | good accuracy for its size |
| `InceptionV3` | multi-scale filters |

Passing `weights="imagenet"` downloads the trained weights the first time (tens of megabytes). We will use MobileNetV2.

### Preparing your own images

Suppose you have photos of five kinds of flowers, arranged one folder per class:

```
flowers/
    daisy/      img001.jpg  img002.jpg ...
    rose/       ...
    tulip/      ...
    ...
```

Keras can read that structure directly, resizing the images and inferring the labels from the folder names, and can split off a validation set:

```python
from tensorflow import keras

train_ds = keras.utils.image_dataset_from_directory(
    "flowers", validation_split=0.2, subset="training",
    seed=42, image_size=(160, 160), batch_size=32)

val_ds = keras.utils.image_dataset_from_directory(
    "flowers", validation_split=0.2, subset="validation",
    seed=42, image_size=(160, 160), batch_size=32)

class_names = train_ds.class_names
print(class_names)       # ['daisy', 'rose', 'tulip', ...]
```

The result is a `tf.data.Dataset`: a stream of batches that you pass straight to `fit()` (no separate `x` and `y` arrays or `batch_size` argument needed). The `seed` must be the same in both calls, so the two subsets do not overlap.

With a small dataset, overfitting is a serious risk. **Data augmentation** helps by creating varied copies of each image on the fly: randomly flipped and slightly rotated. Like dropout, these layers are active only during training:

```python
from tensorflow.keras import layers

augment = keras.Sequential([
    layers.RandomFlip("horizontal"),
    layers.RandomRotation(0.1),
])
```

### Stage 1: feature extraction

First, load the pre-trained network *without its final classification layers* (`include_top=False`; the original top was built for 1,000 ImageNet classes, not your 5) and **freeze** it, so its weights are not changed by training. Then add a new, small head for your task:

```python
base = keras.applications.MobileNetV2(
    input_shape=(160, 160, 3),
    include_top=False,
    weights="imagenet")

base.trainable = False                       # freeze: keep everything it learned

inputs = keras.Input(shape=(160, 160, 3))
x = augment(inputs)
x = keras.applications.mobilenet_v2.preprocess_input(x)   # scale pixels the way this model expects
x = base(x, training=False)                  # run the frozen network in inference mode
x = layers.GlobalAveragePooling2D()(x)       # collapse each feature map to one number
x = layers.Dropout(0.2)(x)
outputs = layers.Dense(5, activation="softmax")(x)        # our new 5-class head

model = keras.Model(inputs, outputs)
```

This is the Functional API mentioned in Section 2: each layer is called like a function on the previous tensor, and `keras.Model(inputs, outputs)` ties the pieces together. Three details deserve comment:

- **`preprocess_input`** rescales pixels the way this particular model was trained (MobileNetV2 expects values between −1 and 1). Every pre-trained model has its own such function; using the wrong scaling silently ruins the results.
- **`training=False`** keeps the frozen base in inference mode, which matters for its batch-normalisation layers. Leave it out and those layers would keep updating their statistics and undermine the frozen weights.
- **`GlobalAveragePooling2D`** averages each feature map down to a single number, an alternative to `Flatten` that produces far fewer values.

Count what will actually be trained. The base has about 2.26 million weights, all frozen, and the only trainable parts are the new `Dense` layer's weights and biases (1,280 × 5 + 5 = 6,405 values):

```python
model.summary()            # Total params: 2,264,389 — but only 6,405 trainable
```

Compile and train the head only. Because the base does the heavy lifting, this converges within a few epochs even on a laptop:

```python
model.compile(optimizer=keras.optimizers.Adam(learning_rate=0.001),
              loss="sparse_categorical_crossentropy",
              metrics=["accuracy"])

early_stop = keras.callbacks.EarlyStopping(monitor="val_loss", patience=3,
                                           restore_best_weights=True)

history = model.fit(train_ds, validation_data=val_ds,
                    epochs=10, callbacks=[early_stop])
```

Note that `sparse_categorical_crossentropy` is used because `image_dataset_from_directory` produces integer labels by default.

### Stage 2: fine-tuning

After stage 1, the new head is sensible and the model is already good. Sometimes you can go further by **fine-tuning**: unfreezing the *top* few layers of the base network and training them together with the head, so that the network's highest-level features adapt to your specific images. The early layers (edges, colours) remain frozen because they are already general enough.

Two rules make fine-tuning safe:

1. **Train the head first.** The new head starts with random weights, and large, wrong gradients from it would flow back and wreck the carefully trained base layers. Only after the head has stabilised should you unfreeze anything.
2. **Use a very small learning rate.** The pre-trained weights are already good and need only gentle nudges. A rate about 10 to 100 times smaller than in stage 1 is typical.

```python
base.trainable = True                        # unfreeze the base...
for layer in base.layers[:-20]:              # ...but re-freeze all except the last 20 layers
    layer.trainable = False

model.compile(optimizer=keras.optimizers.Adam(learning_rate=1e-5),   # tiny learning rate
              loss="sparse_categorical_crossentropy",
              metrics=["accuracy"])

history_fine = model.fit(train_ds, validation_data=val_ds,
                         epochs=10, callbacks=[early_stop])
```

Two points about this code. Keras has to be told to **compile again** after any change to which layers are trainable; forgetting this is a common bug, because the old setting stays in effect. And `base.layers[:-20]` is Python list slicing from Guideline 11: "everything except the last 20". The other 20 layers are now trainable.

Freezing and fine-tuning summarise the whole strategy:

| Situation | Approach |
|---|---|
| Very little data, images similar to ImageNet | feature extraction only (frozen base) |
| Moderate data | feature extraction, then fine-tune the top layers |
| Lots of data, images very different from ImageNet (such as medical scans) | fine-tune more layers, or train from scratch |

### Beyond images

The idea is not limited to pictures. Text models such as BERT and the large language models behind modern chatbots are pre-trained on enormous amounts of writing and then fine-tuned for specific jobs, and the same two-stage habit (reuse first, adapt gently second) applies. Transfer learning is why a student with a laptop can build systems that would have been research breakthroughs a decade ago.

---

*End of Guideline 40.*
