# Guideline 47: Inheritance and Specialization

Suppose you are building a program that models animals. You write a `Dog` class with a name, an `eat()` method, and a `speak()` method. Then you need a `Cat`. It also has a name, also eats, and also speaks, just with a different sound. So you copy the `Dog` class, change a few lines, and move on. Then comes `Bird`, then `Horse`, then `Fish`.

Before long you have five nearly identical classes, and one day you discover a bug in `eat()`. You now have to fix it in five places, and you will almost certainly miss one.

The problem is not the animals. The problem is that we have described the same thing many times instead of once. This guideline is about the tool that fixes it: **inheritance**, the ability of one class to build on another, reusing what it already has and adding only what makes it different. The general idea is written down once, and the specific cases *specialize* it.

## Using Classes to Abstract Behavior

Before writing any inheritance, we need the idea behind it: **abstraction**.

To abstract something is to look at several different things, ignore the differences, and describe what they have in common. A dog, a cat, and a bird are different, but all of them have a name, can eat, and can make a sound. That shared description is a more general idea: an *animal*.

Notice that we have moved *up* a level. "Dog" is a specific kind of thing. "Animal" is a general category that many specific things fit into. In class design, this gives us two roles:

- A **general class** captures what all the related things share.
- **Specific classes** capture only what is unique to each one.

Here is a general class for geometric shapes. Every shape has a name and an area, but *how* the area is calculated depends entirely on the shape, so the general class does not know how:

```python
class Shape:
    def __init__(self, name):
        self.name = name

    def area(self):
        raise NotImplementedError("Each shape must define its own area().")

    def describe(self):
        print(f"{self.name} with area {self.area():.2f}")
```

The `area` method exists to declare a *promise*: every shape can report its area. It deliberately refuses to do the job itself and raises an error if anyone tries. The `describe` method, however, is fully working, and it relies on `area` being provided by whoever specializes the class.

This is abstraction at work. `Shape` is not meant to be used directly, since you would never create a shape that is just "a shape." It exists to define the common behavior and to force the specific classes to fill in the gaps.

Python also provides a formal way to mark a class as unusable on its own, through the built-in `abc` module (short for *abstract base classes*). It uses `@abstractmethod` to make it impossible to create an instance until every abstract method has been filled in. The idea is the same as what we wrote above, only enforced more strictly, and you will meet it once you start building larger designs.

## Understanding Inheritance

**Inheritance** is the mechanism that connects a general class to a specific one. When a class *inherits* from another, it automatically receives all of that class's attributes and methods, without you writing them again.

The vocabulary comes in pairs, and you will see all of these terms used interchangeably:

| General class | Specific class |
|---|---|
| Parent class | Child class |
| Base class | Derived class |
| Superclass | Subclass |

To make a class inherit, put the parent's name in parentheses after the child's name:

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def eat(self):
        print(f"{self.name} is eating.")

    def speak(self):
        print(f"{self.name} makes a sound.")


class Dog(Animal):
    pass


rex = Dog("Rex")
rex.eat()      # Rex is eating.
rex.speak()    # Rex makes a sound.
```

`Dog` contains nothing except `pass`, yet it can already do everything `Animal` can. The constructor, `eat`, and `speak` were all inherited. The bug from the opening story would now need to be fixed exactly once, in `Animal`.

### The "Is-A" Test

Inheritance is not something to use just because two classes share a few lines of code. The test is a simple sentence: **"X is a Y."**

- A dog *is an* animal. Inheritance fits.
- A circle *is a* shape. Inheritance fits.
- A car *is an* engine? No. A car *has an* engine. Inheritance does not fit here.

The second relationship, "has-a," is handled differently, by keeping one object *inside* another as an attribute, exactly as we did with the engine in Guideline 46. Mixing the two up is one of the most common design mistakes. If the "is-a" sentence sounds wrong, do not inherit.

### Adding to What Was Inherited

A child class is not limited to what it inherits. It can add its own attributes and methods. When the child needs its own constructor, it should first let the parent's constructor do its share of the work, using `super()`:

```python
class Cat(Animal):
    def __init__(self, name, indoor=True):
        super().__init__(name)      # let Animal set up the name
        self.indoor = indoor        # then add what is new
```

`super()` means "the parent class." Here it calls `Animal.__init__`, which sets `self.name`, and then `Cat` adds its own `indoor` attribute. If you forget the `super().__init__(...)` line, the parent's setup never runs, and the `name` attribute will not exist.

### Checking Relationships

Python has two functions for asking about inheritance:

```python
tom = Cat("Tom")

print(isinstance(tom, Cat))         # True
print(isinstance(tom, Animal))      # True: a Cat is an Animal
print(isinstance(rex, Cat))         # False
print(issubclass(Cat, Animal))      # True
```

`isinstance(object, Class)` asks whether an object belongs to a class, including through inheritance. `issubclass(Child, Parent)` asks the same about two classes. Also, every class in Python ultimately inherits from a built-in class called `object`, which is why even your simplest class already has some methods you never wrote.

Most of this guideline uses **simple inheritance**: one child, one parent, in a chain that can be as long as you like (`Puppy` inherits from `Dog`, which inherits from `Animal`). A class can also have more than one parent, which we will cover after polymorphism.

## Understanding Method Overloading and Overriding

These two terms sound alike and are easy to confuse, but they describe very different ideas.

### Method Overriding

**Overriding** happens when a child class defines a method with the *same name* as one in its parent, replacing the inherited version for the child. This is how specialization actually happens: the general class provides a default, and the specific class changes it.

```python
class Dog(Animal):
    def speak(self):
        print(f"{self.name} says Woof!")


class Cat(Animal):
    def __init__(self, name, indoor=True):
        super().__init__(name)
        self.indoor = indoor

    def speak(self):
        print(f"{self.name} says Meow!")


Dog("Rex").speak()     # Rex says Woof!
Cat("Tom").speak()     # Tom says Meow!
Animal("Thing").speak()  # Thing makes a sound.
```

Python looks for a method in the child first. If it is found there, that version runs. If not, Python moves up to the parent. That is the whole rule.

Sometimes you do not want to *replace* the parent's behavior but *extend* it. Use `super()` again:

```python
class Puppy(Dog):
    def speak(self):
        super().speak()            # do whatever Dog does first
        print("(wags tail)")


Puppy("Bit").speak()
# Bit says Woof!
# (wags tail)
```

### Method Overloading

**Overloading** means having several methods with the *same name* but different parameter lists, and having the language pick the right one based on how you call it. Some languages, such as Java and C++, do this directly.

**Python does not.** If you define two methods with the same name in one class, the second simply replaces the first, exactly as you saw with functions in Guideline 9:

```python
class Calculator:
    def add(self, a, b):
        return a + b

    def add(self, a, b, c):        # replaces the version above
        return a + b + c


calc = Calculator()
calc.add(1, 2, 3)     # 6
calc.add(1, 2)        # TypeError: missing argument 'c'
```

This does not mean Python cannot handle different numbers of arguments. It just uses different techniques, ones you already know:

```python
class Calculator:
    def add(self, *numbers):       # accepts any number of arguments
        return sum(numbers)


calc = Calculator()
print(calc.add(1, 2))          # 3
print(calc.add(1, 2, 3, 4))    # 10
```

Default values (`def add(self, a, b, c=0)`) and type checks with `isinstance` inside the method cover most of the other cases. So remember the contrast: **overriding** is a child replacing a parent's method, and Python does this naturally. **Overloading** is choosing between same-named methods by their parameters, and Python does not do this, but does not need to.

## Understanding Operator Overloading

Here is something that may have puzzled you: how can `+` add two numbers, join two strings, *and* merge two lists?

```python
print(2 + 3)          # 5
print("py" + "thon")  # python
print([1] + [2])      # [1, 2]
```

The answer is that `+` is not magic. When Python sees `a + b`, it quietly calls a special method on the left-hand object: `a.__add__(b)`. Integers, strings, and lists each define their own `__add__`, so each behaves differently.

That means **your own classes can do the same.** Defining these special methods, known as **operator overloading**, lets instances of your classes work with the ordinary operators.

Consider a two-dimensional vector:

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        return Vector(self.x - other.x, self.y - other.y)

    def __mul__(self, k):                  # vector * number
        return Vector(self.x * k, self.y * k)

    def __eq__(self, other):
        if not isinstance(other, Vector):
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __str__(self):
        return f"Vector({self.x}, {self.y})"


v1 = Vector(1, 2)
v2 = Vector(3, 4)

print(v1 + v2)              # Vector(4, 6)
print(v2 - v1)              # Vector(2, 2)
print(v1 * 3)               # Vector(3, 6)
print(v1 == Vector(1, 2))   # True
```

Without these methods, `v1 + v2` would raise a `TypeError`, and `v1 == Vector(1, 2)` would be `False`, because by default two instances are equal only if they are the very same object.

A few details are worth noting:

- `__str__` controls what `print()` shows for your object. You saw it in the previous guideline's `Transaction` class; now you know what it was doing.
- `__mul__` handles `v1 * 3` (the vector on the *left*). For `3 * v1`, you would also need `__rmul__`, the "reversed" version used when the left operand does not know how to multiply.
- Returning `NotImplemented` inside `__eq__` politely tells Python "I do not know how to compare with this type," so it can try something else instead of crashing.
- If you define `__eq__`, Python stops treating your instances as hashable by default, so they cannot be used as dictionary keys or set members unless you also define `__hash__`. It is rarely a problem in beginner code, but it is a surprise worth knowing about.

Here are the most common operators and the special methods behind them:

| Operator or function | Special method |
|---|---|
| `+` | `__add__` |
| `-` | `__sub__` |
| `*` | `__mul__` |
| `/` | `__truediv__` |
| `//` | `__floordiv__` |
| `%` | `__mod__` |
| `**` | `__pow__` |
| `==` | `__eq__` |
| `!=` | `__ne__` |
| `<` | `__lt__` |
| `<=` | `__le__` |
| `>` | `__gt__` |
| `>=` | `__ge__` |
| `len(x)` | `__len__` |
| `str(x)`, `print(x)` | `__str__` |
| `x[i]` | `__getitem__` |

A word of restraint: overload an operator only when the meaning is natural. Adding two vectors with `+` is intuitive. Making `+` on a `Student` object mean "enroll in a course" would confuse every reader.

## Taking Advantage of Polymorphism

We have now arrived at the payoff. The word **polymorphism** comes from Greek and means "many forms." In programming, it is the ability to call the *same method* on *different objects* and have each one respond in its own way.

You have been building toward it all guideline. Look at the shapes:

```python
import math


class Circle(Shape):
    def __init__(self, radius):
        super().__init__("Circle")
        self.radius = radius

    def area(self):
        return math.pi * self.radius ** 2


class Rectangle(Shape):
    def __init__(self, width, height):
        super().__init__("Rectangle")
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height


shapes = [Circle(1), Rectangle(2, 3)]

for shape in shapes:
    shape.describe()
```

Output:

```
Circle with area 3.14
Rectangle with area 6.00
```

Study what happened. The loop does not ask "is this a circle or a rectangle?" It simply calls `describe()`, and `describe()` calls `area()`. Each object supplies the right version by itself. If you add a `Triangle` class next month, this loop works unchanged, and you do not touch a single existing line. That is the power of polymorphism: code that works with the general idea keeps working as new specific cases appear.

### Duck Typing

Python takes polymorphism one step further than many languages. It does not require objects to share a parent class at all. It only asks whether the object has the method you are calling. This is called **duck typing**, after the saying "if it walks like a duck and quacks like a duck, then it is a duck."

```python
class Robot:
    def speak(self):
        print("Beep boop.")


for thing in [Dog("Rex"), Cat("Tom"), Robot()]:
    thing.speak()
```

Output:

```
Rex says Woof!
Tom says Meow!
Beep boop.
```

`Robot` is not an `Animal`, and Python does not care. It has a `speak()` method, so the call works. Inheritance is one way to achieve polymorphism, but in Python it is not the only way.

## Multiple Inheritance

So far, every child class had exactly one parent. Python does not force that limit. A class can inherit from **several parents at once** by listing them in the parentheses, separated by commas. This is called **multiple inheritance**.

The natural use is combining separate abilities. A duck is an animal, but it can also fly and swim, and those abilities are not unique to ducks. Fish swim, and so do penguins. Rather than stuffing everything into `Animal`, we can define each ability as its own small class:

```python
class Flyer:
    def fly(self):
        print(f"{self.name} is flying.")


class Swimmer:
    def swim(self):
        print(f"{self.name} is swimming.")


class Duck(Animal, Flyer, Swimmer):
    def speak(self):
        print(f"{self.name} says Quack!")


donald = Duck("Donald")
donald.eat()      # Donald is eating.      (from Animal)
donald.fly()      # Donald is flying.      (from Flyer)
donald.swim()     # Donald is swimming.    (from Swimmer)
donald.speak()    # Donald says Quack!     (its own)
```

`Duck` receives everything from all three parents and adds its own `speak`. Classes like `Flyer` and `Swimmer`, which exist only to be mixed into other classes and are not meant to be used alone, are commonly called **mixins**. Notice that they quietly rely on `self.name`, which they never define. They assume the class they are mixed into will provide it, a small echo of the duck typing idea above.

### When Two Parents Disagree

What happens if two parents both define a method with the same name? Python needs a rule, and the rule is the **method resolution order**, or **MRO**: the exact order in which Python searches the classes.

```python
class A:
    def hello(self):
        print("Hello from A")


class B:
    def hello(self):
        print("Hello from B")


class C(A, B):
    pass


C().hello()          # Hello from A
print(C.__mro__)
```

The output of the last line is:

```
(<class '__main__.C'>, <class '__main__.A'>, <class '__main__.B'>, <class 'object'>)
```

Python searches the class itself first, then the parents **from left to right** in the order you listed them, and finally `object`. The first match wins. Since `A` was listed before `B`, its `hello` runs. Swap them to `class C(B, A)` and the answer changes.

### super() Follows the Order, Not Just "the Parent"

This also refines what `super()` really means. It does not simply mean "my one parent." It means "the *next class in the MRO*." That distinction matters in the classic diamond shape, where two classes inherit from the same base and a fourth inherits from both:

```python
class Base:
    def hello(self):
        print("Base")


class Left(Base):
    def hello(self):
        print("Left")
        super().hello()


class Right(Base):
    def hello(self):
        print("Right")
        super().hello()


class Bottom(Left, Right):
    def hello(self):
        print("Bottom")
        super().hello()


Bottom().hello()
```

Output:

```
Bottom
Left
Right
Base
```

Follow the chain: `Bottom` calls the next class in line, `Left`; and inside `Left`, `super()` goes to `Right`, not directly to `Base`, because `Right` comes next in the order (`Bottom`, `Left`, `Right`, `Base`, `object`). `Base` is reached last and runs only once, instead of twice. Python builds this order automatically, so the diamond does not cause duplicate calls.

### A Word of Caution

Multiple inheritance is powerful and easy to overuse. Once a class has several parents, it becomes harder to answer a basic question: *where does this method actually come from?* Constructors get trickier too, since each parent may need its `__init__` to run. A few habits keep it manageable:

- Keep the extra parents small and focused, like the mixins above, ideally with no constructor of their own.
- Apply the "is-a" test to the *main* parent, and treat the others as "can-do" abilities.
- If you find yourself untangling a complicated MRO, step back and consider keeping one object *inside* another instead, the "has-a" approach from earlier.

## Working with Simple Inheritance in Python

Let's put every idea together in a realistic example: a small company payroll. The general class describes any employee, and each specific kind of employee specializes it.

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self._salary = salary             # protected by convention

    def monthly_pay(self):
        return self._salary / 12

    def __str__(self):
        return f"{self.name} ({type(self).__name__})"


class Developer(Employee):
    def __init__(self, name, salary, language):
        super().__init__(name, salary)
        self.language = language


class Manager(Employee):
    def __init__(self, name, salary, team_size):
        super().__init__(name, salary)
        self.team_size = team_size

    def monthly_pay(self):                # override
        bonus = 100 * self.team_size
        return super().monthly_pay() + bonus


staff = [
    Developer("Ada", 60000, "Python"),
    Manager("Grace", 72000, 5),
]

for person in staff:
    print(f"{person}: {person.monthly_pay():.2f}")
```

Output:

```
Ada (Developer): 5000.00
Grace (Manager): 6500.00
```

Trace what each piece does:

- `Employee` holds everything that is true of *every* employee: a name, a salary, a way to compute monthly pay, and a readable string form.
- `Developer` inherits all of that and adds one attribute, `language`. It never redefines `monthly_pay`, so Ada is paid by the parent's rule.
- `Manager` **overrides** `monthly_pay`. It still needs the base calculation, so it calls `super().monthly_pay()` and adds a bonus on top, instead of copying the formula.
- Both child constructors call `super().__init__(...)` so that `name` and `_salary` are set up.
- `__str__` is written once in `Employee` and inherited by both. `type(self).__name__` reports the actual class of the instance, so Ada shows as a `Developer` and Grace as a `Manager`.
- The loop at the bottom is polymorphism: it calls `monthly_pay()` on each person and gets the right answer for each.

Checking the relationships gives what you would expect:

```python
print(isinstance(staff[1], Employee))    # True
print(isinstance(staff[0], Manager))     # False
print(issubclass(Manager, Employee))     # True
```

### Inheritance and Encapsulation Together

One last detail connects this guideline to the previous one. Notice that `Employee` stored the salary as `_salary`, with a single underscore. That was deliberate.

If it had been `__salary`, with two underscores, Python's name mangling would have renamed it to `_Employee__salary`. Inside `Manager`, the name `self.__salary` would have been mangled into `_Manager__salary`, which does not exist, and the code would fail. Double underscores hide data *even from child classes*.

So a practical guideline emerges:

- Use **double underscores** for details that belong to one class alone and that no subclass should touch.
- Use a **single underscore** for details that are internal but that subclasses are expected to use.
- Use **public names** for anything the outside world may rely on.

Choosing between these is part of designing a class that others will extend.

You now have the core tools of object-oriented design: classes that model things, encapsulation that protects them, and inheritance that lets them grow without being rewritten. The guidelines that follow will put these tools to work on larger designs.
