# Guideline 44: Introduction to Object-Oriented Programming with Python

Now we reach the third part of the guidebook.

Everything so far has been about giving instructions: take this input, run this loop, call this function, return that value. Even when we handled large amounts of data, the mindset was the same. We wrote a sequence of steps, and the computer followed them. That approach is called *procedural programming*, and it takes you surprisingly far.

But look at what you have been using all along. When you wrote `"hello".upper()`, the string knew how to convert itself to uppercase. When you wrote `numbers.append(5)`, the list knew how to add an element to itself. Data that carries its own abilities is not a coincidence of Python's design. It is the design. In the very first guideline we said Python is high-level, interpreted, and *object-oriented*. This guideline finally takes that third word seriously.

## The Concept of Object Everywhere

Here is the idea that the rest of this part builds on: **everything is an object**.

Look around the room you are in. A phone, a chair, a cup, a person. Each of these is a distinct *thing*. Each has properties (the cup is white, holds 300 ml, is currently half full) and each can do or have things done to it (the cup can be filled, emptied, or broken). We do not think about the world as a long list of instructions. We think about it as a collection of things, and we describe them by what they are and what they can do.

Object-oriented programming (OOP) asks you to write programs the same way. Instead of scattering variables in one corner and functions in another, you bundle related data and the behavior that goes with it into a single unit: an **object**. A student, a bank account, a bullet in a game, a row in a database. This is why we say objects are everywhere. It is not only that the real world is full of them; it is that almost any problem, once you look at it carefully, turns out to be a set of objects interacting with each other.

Python takes this literally. Integers are objects. Strings are objects. Lists, dictionaries, even functions are objects. You have been working with objects since the first `print()`. The only new step is learning to *design your own*.

The rest of this guideline is about the thinking that comes before any code: how to look at a problem and see the objects inside it.

## Recognizing Objects from Nouns

The easiest way to find the objects in a problem is to read its description and underline the nouns.

Suppose you are asked to build a small library system: *"A library has many books. Members can borrow books, and each borrowing has a due date."* Underline the nouns: **library**, **book**, **member**, **borrowing**, **due date**. These are your candidates for objects.

Not every noun deserves to become an object, and this is where judgment comes in. A useful test: *does this thing have its own data and its own behavior?* A book has a title, an author, and an availability status, and it can be borrowed or returned. That is a strong candidate. A due date, on the other hand, is just a single piece of information attached to a borrowing. It may be better as a plain value inside another object than as an object of its own.

So the process has two steps. First, list every noun. Second, cross out the ones that are really just descriptions of other things. What survives is your set of objects.

There is no single correct answer. Two programmers can look at the same problem and identify slightly different objects, and both can be right. What matters is that the choice makes the problem easier to think about.

## Generating Blueprints for Objects

Once you know which objects you need, notice something: you rarely need just one. A library has hundreds of books. A game has dozens of enemies. A school has thousands of students. Describing each one separately would be absurd.

What these objects share is a *pattern*. Every book has a title and an author. Every student has a name and a grade. The individual books differ, but the structure is the same.

That shared pattern is a **blueprint**. Think of an architect's plan for a house. The blueprint is not a house; you cannot live in it. But from one blueprint, you can build ten identical-in-structure houses, each painted a different color and lived in by a different family.

Programming works the same way, and there are two separate ideas here that beginners often blur together:

- The **blueprint** defines what every object of this kind will contain and what it will be able to do. You write it once.
- Each **object** built from the blueprint is a separate, individual thing with its own values. You can build as many as you like.

A blueprint for "book" says every book has a title. It does not say the title is *Dune*. That detail belongs to one particular book built from the blueprint. Keep this distinction clear, because everything in the following sections depends on it.

## Recognizing Attributes (Fields)

If nouns lead us to objects, the next question is: *what does each object know about itself?*

These pieces of stored information are called **attributes**, or **fields**. They are the variables that live inside an object. A useful trick is to look for the descriptive words attached to your nouns:

- A book has a **title**, an **author**, and a **status** (available or borrowed).
- A member has a **name**, a **member ID**, and a list of **borrowed books**.
- An enemy in a game has **health**, **speed**, and a **position**.

Notice that attributes come in different types. A title is a string. A health value is an integer. A list of borrowed books is a list, and it may even contain other objects. Everything you learned about data types in the earlier guidelines applies here. An object is, in a sense, a well-organized container for the variables you already know how to use.

Attributes also give each object its **state**: the current condition of the object at a given moment. Two books built from the same blueprint can be in different states, one available and one borrowed. When something happens to an object, what usually changes is its state.

A good question to ask for each attribute is: *does this object genuinely need to remember this?* If the information can be calculated from other attributes, or is only needed briefly, it probably does not belong as a stored field.

## Recognizing Actions from Verbs: Methods

Nouns gave us objects, and descriptive words gave us attributes. The last piece of the puzzle hides in the **verbs**.

Return to the library description: *"Members can **borrow** books, and each borrowing has a due date."* Books get **borrowed** and **returned**. Members **borrow**, **return**, and **pay fines**. An enemy in a game **moves**, **attacks**, and **takes damage**. These actions are the object's abilities, and in code they are called **methods**.

A method is simply a function that belongs to an object. You have already met the difference between functions and methods: `len(numbers)` is a function you call *on* a value, while `numbers.append(5)` is a method that the list itself carries. Now you understand why the second form exists. The action belongs to the object, so it is called *through* the object.

Methods and attributes work together. A method usually reads or changes the object's attributes. When a book is borrowed, its status attribute changes from available to borrowed. When an enemy takes damage, its health attribute goes down. The method is the *how*; the attribute is the *what changes*.

A practical habit: when you describe a problem in plain language, mark nouns, descriptive words, and verbs in three different ways. Nouns become objects, descriptive words become attributes, verbs become methods. You will be surprised how often a paragraph of ordinary English is already a draft of your program's design.

## Organizing the Blueprints: Classes

We have been calling the shared pattern a "blueprint." In Python, the proper name for it is a **class**.

A **class** is the formal definition that brings the three ideas together: it names the kind of object, declares the attributes each object will have, and defines the methods each object can perform. An **object** built from a class is called an **instance** of that class, and the act of building one is called *instantiation*.

| Everyday idea | Programming term |
|---|---|
| Blueprint | Class |
| A house built from the blueprint | Object (instance) |
| Features every house has | Attributes (fields) |
| Things a house can do | Methods |

Your program then becomes a collection of classes, one for each kind of object you found, and the actual running program is objects created from those classes interacting with one another. A `Member` object borrows a `Book` object. A `Library` object keeps track of both.

Organizing classes well is a skill in itself, and a few habits help from the start:

- **One class, one clear purpose.** If you struggle to describe what a class represents in a single sentence, it is probably doing too much.
- **Keep related data and behavior together.** If a method constantly works with a particular set of attributes, it likely belongs in the same class as them.
- **Name classes as singular nouns.** `Book`, not `Books`. The class describes one thing; you create many of them.

Later guidelines will go deeper into how classes relate to each other, including how one class can build on another. For now, it is enough to see the basic structure.

## Object-Oriented Approaches in Python

Time to see all of this in Python. This guideline deliberately has very little code, so here is a single small example: the blueprint for a book.

```python
class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author
        self.available = True

    def borrow(self):
        self.available = False
```

Read it against what you just learned. `class Book:` declares the blueprint. The three `self.` lines inside `__init__` are the **attributes**: title, author, and availability. The `borrow` function is a **method**, and all it does is change one attribute. That is the entire pattern.

Two details are worth knowing at this stage:

- `__init__` is a special method that runs automatically whenever a new object is created. Its job is to set up the object's starting state.
- `self` refers to *the particular object being worked on*. It is how a single blueprint can produce many separate books, each keeping track of its own title and its own availability.

Creating objects and using them looks like this:

```python
book1 = Book("Dune", "Frank Herbert")
book1.borrow()
print(book1.available)   # False
```

`book1` is an instance of `Book`. Calling `book1.borrow()` changes only that one book, and any other book you create stays available. That separation is exactly what the blueprint idea promised.

### Why Bother with This Approach?

You may wonder why all this is worth the trouble when functions and variables already work. The answer becomes clearer as programs grow:

- **Organization.** Related data and behavior live together, so you know where to look when something breaks.
- **Reusability.** One class can produce as many objects as you need.
- **Modeling.** The structure of your code mirrors the structure of the problem, which makes it far easier to reason about.
- **Scale.** The large libraries you met earlier, such as NumPy, Pandas, and Scikit-Learn, are built from classes. An `ndarray`, a `DataFrame`, and a trained model are all objects with attributes and methods. Understanding OOP means you now understand how those tools are actually put together.

Nothing here replaces what you learned before. Variables, loops, conditionals, and functions are all still the raw material. Object-oriented programming is a way of *organizing* that material so that larger problems stay manageable.

In the next guideline, we will stay with classes and build them properly: writing constructors, working with attributes and methods in depth, and creating objects that actually do useful work.
