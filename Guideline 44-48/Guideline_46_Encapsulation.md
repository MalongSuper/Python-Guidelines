# Guideline 46: Encapsulation

In the last two guidelines, we learned to find objects in a problem and to write classes that produce them. There is one habit left that separates a class that merely *works* from a class that is *well designed*: deciding what the outside world is allowed to touch.

That habit is called **encapsulation**. It has two halves, and it helps to keep both in mind:

1. **Bundling.** Data and the methods that work on that data live together, inside one class. You have been doing this since Guideline 44.
2. **Controlling access.** The class decides which parts are open to everyone, which parts are for internal use, and which changes are allowed at all.

Think of a car. You interact with a steering wheel, pedals, and a gear stick. You do not reach into the engine and adjust the fuel injection by hand, and the car is better for it: the engine can be redesigned entirely without you ever noticing, as long as the pedals keep working. Encapsulation gives your classes the same property. The outside sees a simple, stable surface, and the inside stays free to change.

## Understanding the Different Members of a Class

A class is made of **members**: the individual pieces that live inside it. Before we can decide which ones to protect, we need to know what kinds exist. Some you have already met; others are new. This guideline covers eight of them.

| Member | What it is | Belongs to |
|---|---|---|
| Constructor | `__init__`, runs when an instance is created | The instance |
| Destructor | `__del__`, runs when an instance is destroyed | The instance |
| Class field (class attribute) | A variable shared by every instance | The class |
| Instance field (instance attribute) | A variable each instance owns separately | The instance |
| Class method | A method that works with the class itself | The class |
| Nested class | A class defined inside another class | The class |
| Property | A method that behaves like an attribute, with optional getter and setter | The instance |
| Event | A way for an object to notify other code that something happened | The instance |

### Constructor and Destructor

Nothing new here, only a reminder. `__init__` sets up a new instance, and `__del__` runs as it is removed. Both were covered in Guideline 45.

### Class Fields and Instance Fields

The most important distinction in this list is **who owns the data**.

An **instance field** belongs to one instance. It is created through `self` inside the constructor, and every instance gets its own copy. This is what you used in the previous guideline.

A **class field** is created directly in the class body, outside any method. There is only one copy, and every instance can see it.

```python
class Dog:
    species = "Canis familiaris"     # class field: shared

    def __init__(self, name):
        self.name = name             # instance field: one per dog

rex = Dog("Rex")
luna = Dog("Luna")
print(rex.species, luna.species)     # both see the same value
print(rex.name, luna.name)           # Rex Luna
```

Class fields are the right tool for things that are true for the whole class: a constant, a default, or a counter of how many instances exist. Instance fields are the right tool for anything that differs from one object to another.

A common trap: if you write `rex.species = "Wolf"`, Python does not change the shared value. It creates a *new instance field* on `rex` that hides the class field for that one dog. To change the shared value, assign through the class itself: `Dog.species = "Wolf"`.

### Instance Methods and Class Methods

So far, every method we wrote took `self` as its first parameter, meaning it works with one particular instance. These are **instance methods**.

A **class method** works with the class as a whole. It is marked with the `@classmethod` decorator, and its first parameter is `cls` (the class itself) instead of `self`.

```python
class Dog:
    count = 0

    def __init__(self, name):
        self.name = name
        Dog.count += 1

    @classmethod
    def how_many(cls):
        return cls.count

Dog("Rex")
Dog("Luna")
print(Dog.how_many())     # 2
```

Notice that `how_many` is called on the class, with no instance required. Use a class method when the action concerns the class, such as reading a shared counter, and not any single object.

### Nested Classes

A class can be defined inside another class. This is useful when the inner class only makes sense as part of the outer one and nothing else needs to know about it.

```python
class Car:
    class Engine:
        def __init__(self, horsepower):
            self.horsepower = horsepower

    def __init__(self, model, horsepower):
        self.model = model
        self.engine = self.Engine(horsepower)

car = Car("Roadster", 300)
print(car.engine.horsepower)      # 300
```

An engine without a car has no purpose in this program, so `Engine` lives inside `Car`. Nested classes are the least common member on this list, but they communicate a design decision clearly: *this class belongs to that one*.

### Properties and Events

The last two members need more room, so they get their own sections below. In short, a **property** lets you control how an attribute is read and changed, and an **event** lets an object announce that something happened. Both are, in a sense, tools for encapsulation.

## Protecting and Hiding Data

Why hide anything at all? Consider this class:

```python
class BankAccount:
    def __init__(self, balance):
        self.balance = balance

account = BankAccount(100)
account.balance = -5000       # nothing stops this
```

The attribute is wide open. Any part of the program can set it to any value, including nonsense. If a bug puts a negative balance in the account, you now have to search the *entire program* to find which line did it.

Encapsulation solves this by funneling every change through code the class controls, so rules can be enforced in one place.

### Public, Protected, and Private by Convention

Many languages have keywords that enforce hiding. Python takes a different, more trusting approach: it relies on **naming conventions** that tell other programmers what is intended.

| Naming style | Example | Meaning |
|---|---|---|
| No underscore | `balance` | **Public.** Free for anyone to use. |
| One leading underscore | `_balance` | **Protected, by convention.** For internal use; outside code should leave it alone. |
| Two leading underscores | `__balance` | **Private.** Python renames it internally to make accidental access harder. |

The single underscore does nothing technical. It is a polite sign saying "this is an implementation detail." Python programmers respect it out of habit.

The double underscore does something real. Python rewrites the name inside the class, a process called **name mangling**. `__balance` inside `BankAccount` becomes `_BankAccount__balance`:

```python
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance

account = BankAccount(100)
print(account.__balance)               # AttributeError
print(account._BankAccount__balance)   # 100
```

The first attempt fails, which is the protection working. But the second shows that determined code can still get in. Python's philosophy is that programmers are responsible adults: the language makes accidental misuse difficult, but it does not build walls against deliberate access. Treat a private name as a firm signal, not as a security feature.

### Giving the Outside a Proper Door

Hiding a field is only half the job. The outside still needs some way to use the account, so we provide **methods** that enforce the rules:

```python
class BankAccount:
    def __init__(self, balance=0):
        self.__balance = balance

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit must be positive.")
        self.__balance += amount

    def withdraw(self, amount):
        if amount > self.__balance:
            raise ValueError("Insufficient funds.")
        self.__balance -= amount

    def get_balance(self):
        return self.__balance
```

Now the balance can change only through `deposit` and `withdraw`, and both check their input. A negative balance is no longer possible by accident. This pattern of a private field with methods to read and change it is the traditional heart of encapsulation, and Python offers a more elegant version of it.

## Working with Properties

Writing `get_balance()` works, but it feels heavy. Reading a value should look like reading a value: `account.balance`, not `account.get_balance()`. A **property** gives you exactly that. It lets you keep attribute-style syntax while running your own code behind the scenes.

### Getters

A **getter** is created with the `@property` decorator:

```python
class BankAccount:
    def __init__(self, balance=0):
        self.__balance = balance

    @property
    def balance(self):
        return self.__balance
```

Now `account.balance` looks like ordinary attribute access, but it actually calls the method. Note that there are no parentheses. Because we defined only a getter, the property is **read-only**:

```python
account = BankAccount(100)
print(account.balance)     # 100
account.balance = 5000     # AttributeError: property has no setter
```

Read-only properties are a clean way to say "you may look, but you may not touch."

### Setters

When outside code *should* be allowed to change a value, but only under your rules, add a **setter** with `@name.setter`:

```python
class Person:
    def __init__(self, name):
        self.name = name              # goes through the setter

    @property
    def name(self):
        return self.__name

    @name.setter
    def name(self, value):
        if not value.strip():
            raise ValueError("Name cannot be empty.")
        self.__name = value.strip()

p = Person("  Ada  ")
print(p.name)          # Ada
p.name = "Grace"       # runs the setter
p.name = "   "         # ValueError
```

Two details are worth studying here:

- The getter and setter share the **same name**, `name`. The private field underneath is `__name`. Using the same name for both would cause the property to call itself forever, which is why the stored value gets a different one.
- The constructor writes `self.name = name`, not `self.__name = name`. That routes even the *first* assignment through the setter, so the validation applies from the moment the object is created.

Properties can also have a **deleter**, defined with `@name.deleter`, which runs when someone writes `del p.name`. It is rarely needed, so we will not go further with it.

### Why Properties Are Worth It

The strongest argument for properties is flexibility. You can start with a plain public attribute. Later, if you need validation, you convert it to a property, and **no code that uses the class has to change**, since `p.name` looks identical either way. This is why Python programmers usually skip `get_` and `set_` methods and reach for properties only when they need them.

## Understanding the Difference Between Mutability and Immutability

Encapsulation raises a question we have been able to ignore until now: when we protect an object's data, what exactly are we protecting against? The answer depends on whether the data is **mutable** or **immutable**.

- A **mutable** object can be changed after it is created, in place.
- An **immutable** object cannot. Any "change" produces a brand-new object.

You have already met both kinds:

| Immutable | Mutable |
|---|---|
| `int`, `float`, `bool` | `list` |
| `str` | `dict` |
| `tuple` | `set` |
| `frozenset` | Instances of your own classes (by default) |

The `id()` function, which reports an object's identity, makes the difference visible:

```python
x = 5
print(id(x))
x += 1
print(id(x))       # a different id: a new int object was created

nums = [1, 2]
print(id(nums))
nums.append(3)
print(id(nums))    # the same id: the same list was modified
```

With the integer, `x += 1` did not modify the 5. It created a 6 and made `x` point at it. With the list, `append` reached into the existing object and changed it.

### Why This Matters for Encapsulation

Here is the catch. Suppose your class keeps a private list and hands it out through a getter:

```python
class Library:
    def __init__(self):
        self.__books = ["Dune", "Emma"]

    @property
    def books(self):
        return self.__books

lib = Library()
lib.books.append("Nonsense")     # the outside just changed the private list
print(lib.books)                 # ['Dune', 'Emma', 'Nonsense']
```

The field was private, and the property was read-only, yet outside code modified the contents. The getter returned the *actual list*, not a copy, and lists are mutable. Hiding the *name* did not protect the *object*.

The remedy is to return something the caller cannot use to damage the original:

```python
    @property
    def books(self):
        return tuple(self.__books)     # an immutable snapshot
```

A `tuple` cannot be modified, and even if you returned a copy of the list instead (`list(self.__books)`), changes to the copy would leave the original untouched. Both approaches keep the inside safe.

There is one more subtlety worth remembering: immutability is shallow. A tuple cannot change *which* objects it holds, but if one of them is a mutable list, that list can still change from within. Immutability protects the container, not necessarily what is inside it.

So a rule of thumb emerges for designing encapsulated classes: when a getter exposes mutable data, return a copy or an immutable version, unless you deliberately want outside code to modify it.

## Encapsulating Data in Python

Now let's bring every member together in one class. This bank account uses a constructor, a destructor, class and instance fields, a class method, a nested class, properties, private data, and an event.

**Events** deserve a quick explanation, because Python has no built-in event keyword. The idea is that an object keeps a list of functions, called **handlers**, and calls each one when something interesting happens. Other code *subscribes* by adding its own function to that list. The account does not need to know what the handlers do; it just announces the news.

```python
class BankAccount:
    bank_name = "Python Bank"        # class field
    __count = 0                      # class field (private)

    class Transaction:               # nested class
        def __init__(self, kind, amount):
            self.kind = kind
            self.amount = amount

        def __str__(self):
            return f"{self.kind}: {self.amount}"

    def __init__(self, owner, balance=0):        # constructor
        self.owner = owner                       # instance field, via property
        self.__balance = balance                 # instance field (private)
        self.__history = []                      # instance field (private)
        self.__low_balance_handlers = []         # the event's handler list
        BankAccount.__count += 1

    def __del__(self):                           # destructor
        BankAccount.__count -= 1

    @property
    def owner(self):                             # getter
        return self.__owner

    @owner.setter
    def owner(self, value):                      # setter with validation
        if not value.strip():
            raise ValueError("Owner name cannot be empty.")
        self.__owner = value.strip()

    @property
    def balance(self):                           # read-only property
        return self.__balance

    @property
    def history(self):                           # immutable snapshot
        return tuple(self.__history)

    def on_low_balance(self, handler):           # subscribe to the event
        self.__low_balance_handlers.append(handler)

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Deposit must be positive.")
        self.__balance += amount
        self.__history.append(self.Transaction("deposit", amount))

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("Withdrawal must be positive.")
        if amount > self.__balance:
            raise ValueError("Insufficient funds.")
        self.__balance -= amount
        self.__history.append(self.Transaction("withdraw", amount))
        if self.__balance < 100:                 # raise the event
            for handler in self.__low_balance_handlers:
                handler(self)

    @classmethod
    def total_accounts(cls):                     # class method
        return cls.__count
```

Now use it:

```python
def warn(account):
    print(f"Warning: {account.owner}'s balance is low ({account.balance}).")

acc = BankAccount("Ada", 500)
acc.on_low_balance(warn)         # subscribe a handler

acc.deposit(50)
acc.withdraw(480)                # balance drops to 70, the event fires

print(acc.balance)               # 70
print(BankAccount.total_accounts())   # 1

for t in acc.history:
    print(t)
```

Output:

```
Warning: Ada's balance is low (70).
70
1
deposit: 50
withdraw: 480
```

And here is what the class refuses to allow:

```python
acc.balance = 1000               # AttributeError: read-only property
acc.owner = "   "                # ValueError: name cannot be empty
acc.withdraw(10_000)             # ValueError: insufficient funds
acc.history.append("fake")       # AttributeError: tuples have no append
```

Read back through the class and match each piece to its purpose:

- The **balance is private** and exposed only as a read-only property, so it can change only through `deposit` and `withdraw`, which enforce the rules.
- The **owner is a property with a setter**, so bad names are rejected even at construction time.
- The **history getter returns a tuple**, so callers can read the records but cannot tamper with the real list.
- The **event** lets other code react to a low balance without the account knowing anything about that code. The account announces; the handlers decide what to do.
- The **class field and class method** track how many accounts exist across the whole class, something no single instance could know.
- The **nested class** keeps `Transaction` tied to the account it belongs to.

Every outside interaction now goes through a controlled door. If the rules for withdrawals ever change, there is exactly one place to update them, and no code using the account has to change. That is what encapsulation buys you, and it is the reason nearly every large Python library is built this way.

In the next guideline, we will look at how one class can be built on top of another, reusing and extending everything we have just designed.
