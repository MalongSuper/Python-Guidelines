# Guideline 45: Classes and Instances

In the previous guideline, we learned to look at a problem and see objects inside it: nouns became objects, descriptive words became attributes, and verbs became methods. We also saw a first glimpse of a class written in Python. This guideline is where we slow down and write classes properly.

From here on, expect a lot more code. Run every example yourself and change the values. Object-oriented programming is much easier to feel than to memorize.

## Understanding Classes and Instances

Let's restate the two central ideas, because everything in this guideline depends on keeping them apart.

A **class** is a blueprint. It describes what a certain kind of object looks like and what it can do, but it is not itself one of those objects.

An **instance** is an actual object built from that blueprint. It has its own data, and it lives in the computer's memory while your program runs.

A useful comparison is a cookie cutter and the cookies. The cutter defines the shape, and you can make as many cookies as you want from it. Each cookie can have different toppings, and eating one does not affect the others. The cutter is the class. The cookies are the instances.

You have already been creating instances without noticing:

```python
name = str("Ada")       # an instance of the str class
numbers = list([1, 2])  # an instance of the list class
print(type(name))       # <class 'str'>
print(type(numbers))    # <class 'list'>
```

`type()` reports which class an object was built from. Every value in Python is an instance of some class, and now you are going to write your own.

## Understanding Constructors and Destructors

Every object has a life cycle: it is **created**, it is **used**, and eventually it is **destroyed**. Python lets you attach code to the two ends of that life using two special methods.

- A **constructor** runs when an object is created. Its job is to set up the object's starting state, such as giving it its initial attribute values. In Python this is the `__init__` method.
- A **destructor** runs when an object is about to be destroyed. Its job is to clean up, such as closing a file or announcing that the object is gone. In Python this is the `__del__` method.

Both names are wrapped in double underscores. Methods written this way are called *special methods* (or "dunder" methods, from "double underscore"), and Python calls them automatically at the right moment. You almost never call them yourself.

A small technical honesty note: strictly speaking, Python creates the object first, using another special method called `__new__`, and then `__init__` *initializes* it. In everyday programming you only ever write `__init__`, and most people simply call it the constructor. The guidebook will do the same.

## Declaring Classes

Declaring a class takes the `class` keyword, a name, and a colon. The indented block underneath is the body.

```python
class Book:
    pass
```

The `pass` statement means "do nothing here yet." This is already a valid class. Two conventions are worth following from day one:

- Class names use **CapitalizedWords**: `Book`, `BankAccount`, `EnemyShip`. This separates them visually from variables and functions, which use lowercase names with underscores.
- A class name should be a **singular noun**, since each instance represents one thing.

Now let's give the class some behavior. A function written inside a class is a **method**, and its first parameter is always `self`, which stands for the particular instance the method is working on.

```python
class Book:
    def describe(self):
        print("This is a book.")
```

Notice that we never pass anything for `self` when we call the method. Python fills it in automatically with the instance in front of the dot. We will see this in action in the section on creating instances.

## Customizing Constructors

An empty class is not very useful. What makes each instance individual is its data, and the constructor is where that data gets set.

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author
        self.available = True
```

Let's read this line by line:

- `def __init__(self, title, author):` defines the constructor. Besides `self`, it asks for two values, which the caller must provide when creating a book.
- `self.title = title` creates an **attribute** on the instance and stores the value inside it. The part after `self.` is the attribute's name, and the part after `=` is the value, in this case the parameter of the same name. They do not have to match, but matching names make the code easier to read.
- `self.available = True` creates an attribute that does not come from a parameter. Every new book starts as available, so there is no reason to ask the caller.

The distinction matters: parameters go through the constructor, but attributes live on `self`. If you write `title = title` without `self.`, you only create a temporary local variable that disappears when the constructor ends, and the object remembers nothing.

### Default Values

Constructors accept default values just like ordinary functions, which lets some information be optional:

```python
class Book:
    def __init__(self, title, author, pages=100):
        self.title = title
        self.author = author
        self.pages = pages
        self.available = True
```

Now `pages` can be left out, and the book will get 100 unless told otherwise. As with regular functions, parameters with defaults must come after those without.

### Validating in the Constructor

Because the constructor runs every time an object is created, it is also the natural place to reject bad data. Recall what you learned about exception handling:

```python
class Book:
    def __init__(self, title, author, pages=100):
        if pages <= 0:
            raise ValueError("A book must have at least one page.")
        self.title = title
        self.author = author
        self.pages = pages
        self.available = True
```

An instance that could never make sense in real life now cannot exist in your program either. This is one of the quiet advantages of classes: you decide what a valid object looks like, in one place.

## Customizing Destructors

The destructor is the counterpart of the constructor. It is defined with `__del__` and takes only `self`.

```python
class Book:
    def __init__(self, title):
        self.title = title
        print(f"'{self.title}' was created.")

    def __del__(self):
        print(f"'{self.title}' was destroyed.")
```

To see it work, use the `del` statement, which removes a name pointing at an object:

```python
book = Book("Dune")
del book
```

Output:

```
'Dune' was created.
'Dune' was destroyed.
```

How does Python decide an object is ready to be destroyed? It keeps track of how many names refer to each object, a count known as its **reference count**. When the count drops to zero, nothing can reach the object anymore, so Python removes it and calls `__del__` first. This also explains something important:

```python
a = Book("Dune")
b = a          # two names now point at the same object
del a          # one name is gone, but b still points at it
print("still alive")
del b          # now the count reaches zero
```

Here, the destroyed message appears only after `del b`, not after `del a`. `del` removes a *name*, not necessarily the object itself.

### A Word of Caution

Destructors are more delicate than constructors, and beginners tend to expect more from them than they deliver:

- You cannot always predict *exactly when* `__del__` runs, especially in more complex programs, because Python decides when objects are cleaned up.
- When the program ends, Python usually cleans up the remaining objects, but you should not rely on it for anything critical.
- Objects that hold onto outside resources, such as open files, are better handled with the `with` statement you saw in the file handling guideline, which guarantees cleanup at a known point.

So treat `__del__` as a convenience for simple cleanup and for understanding an object's life cycle, and not as the main tool for managing resources. In practice, many Python programmers write constructors constantly and destructors rarely.

## Creating Instances of Classes

Creating an instance is written like calling a function. Use the class name, followed by parentheses containing whatever the constructor asks for (except `self`, which Python supplies):

```python
class Book:
    def __init__(self, title, author, pages=100):
        self.title = title
        self.author = author
        self.pages = pages
        self.available = True

    def describe(self):
        print(f"{self.title} by {self.author}, {self.pages} pages")

book1 = Book("Dune", "Frank Herbert", 412)
book2 = Book("Emma", "Jane Austen")
```

Two instances now exist, built from the same blueprint. Each one has its own separate values.

### Accessing Attributes and Calling Methods

Use a dot to reach into an instance:

```python
print(book1.title)      # Dune
print(book2.pages)      # 100 (the default)
book1.describe()        # Dune by Frank Herbert, 412 pages
book2.describe()        # Emma by Jane Austen, 100 pages
```

When you call `book1.describe()`, Python quietly translates it into "run `describe` with `self` being `book1`." That is why the method printed Dune's information, and why `book2.describe()` printed Emma's from the exact same code.

Attributes can also be changed after creation:

```python
book1.available = False
print(book1.available)  # False
print(book2.available)  # True
```

Changing `book1` left `book2` untouched. The instances are independent, which is the whole point.

### Instances Are Separate Objects

Two instances built with identical values are still two different objects. You can confirm this with the identity operator from Guideline 3:

```python
x = Book("Dune", "Frank Herbert")
y = Book("Dune", "Frank Herbert")
print(x is y)   # False
print(x is x)   # True
```

They look the same, but they are separate objects in memory, like two identical cookies from the same cutter.

### Instances in Collections

Since instances are ordinary values, you can store them in lists, dictionaries, or anywhere else you would store a number or a string. This is where the earlier guidelines pay off:

```python
library = [
    Book("Dune", "Frank Herbert", 412),
    Book("Emma", "Jane Austen", 474),
    Book("Ubik", "Philip K. Dick", 202),
]

for book in library:
    book.describe()
```

A loop over a list of instances, calling the same method on each one, is a pattern you will write again and again.

## Summary of the Workflow

Putting it all together, creating something useful with classes follows a small, repeatable routine:

1. **Declare** the class with the `class` keyword and a singular, capitalized name.
2. **Write the constructor** (`__init__`) to set up each instance's attributes, using parameters and defaults where the caller should be able to choose.
3. **Add methods** for the things the object can do, remembering that `self` always comes first.
4. *(Optional)* **Add a destructor** (`__del__`) if you need something to happen when an instance goes away.
5. **Create instances** by calling the class like a function, then use the dot to reach their attributes and methods.

The next guideline will build on this foundation and look more closely at the attributes and methods that live inside a class, including how data belonging to the whole class differs from data belonging to a single instance.
