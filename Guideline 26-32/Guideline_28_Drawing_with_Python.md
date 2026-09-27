# Guideline 28: Drawing with Python

Every guideline so far has produced output you read as text. This one produces output you *look* at. Python's built-in `turtle` module simulates a small robot holding a pen — you tell it to move forward, turn, lift the pen, put it down — and a picture appears on screen as it goes. It's a simple idea, but it's enough to draw shapes, logos, and small scenes, and it's a genuinely good way to build intuition for loops, angles, and coordinates.

```python
import turtle
```

## 28.1 The `turtle` Module — Common Methods

A `turtle` program almost always starts the same way: create a screen, create a turtle, draw, then keep the window open until it's closed manually.

```python
import turtle

screen = turtle.Screen()
pen = turtle.Turtle()

# ... drawing goes here ...

turtle.done()
```

The table below covers the methods you'll reach for constantly. Many have a short alias — both spellings work identically.

| Method | Alias | What it does |
|---|---|---|
| `forward(distance)` | `fd()` | Moves the turtle forward, drawing a line if the pen is down |
| `backward(distance)` | `bk()`, `back()` | Moves backward |
| `left(angle)` | `lt()` | Turns left by `angle` degrees |
| `right(angle)` | `rt()` | Turns right by `angle` degrees |
| `penup()` | `pu()` | Lifts the pen — moving now draws nothing |
| `pendown()` | `pd()` | Lowers the pen — moving now draws again |
| `goto(x, y)` | `setpos()` | Jumps straight to a coordinate, drawing a line if the pen is down |
| `speed(n)` | — | Sets drawing speed, 1 (slowest) to 10 (fastest), 0 (instant) |
| `color(c)` | — | Sets the pen (outline) color |
| `pensize(width)` | `width()` | Sets the line thickness |
| `circle(radius)` | — | Draws a circle of the given radius |
| `reset()` | — | Clears the drawing and resets the turtle to the center |
| `clear()` | — | Clears the drawing without moving the turtle |
| `hideturtle()` | `ht()` | Hides the turtle-shaped cursor (keeps the drawing) |
| `showturtle()` | `st()` | Shows it again |
| `done()` / `mainloop()` | — | Keeps the window open after drawing finishes |

A quick example using several of these together — a simple staircase:

```python
pen.speed(3)
for _ in range(5):
    pen.forward(50)
    pen.left(90)
    pen.forward(50)
    pen.right(90)
```

## 28.2 Drawing Shapes

**Square** — four equal sides, turning 90° each time:

```python
for _ in range(4):
    pen.forward(100)
    pen.left(90)
```

**Triangle** — three equal sides. The turn angle for *any* regular polygon is `360 / number_of_sides`, so a triangle turns 120° each time:

```python
for _ in range(3):
    pen.forward(100)
    pen.left(120)
```

**Circle** — `turtle` handles this with a single call, no loop required:

```python
pen.circle(60)   # radius of 60
```

**3D Cube** — a simple illusion of depth: draw two identical squares offset from each other, then connect their matching corners.

```python
def draw_square(t, size):
    for _ in range(4):
        t.forward(size)
        t.left(90)

size = 100
offset = 40   # how far "back" the second square sits

# front face
pen.penup()
pen.goto(-size / 2, -size / 2)
pen.pendown()
draw_square(pen, size)

# back face, offset up and to the right
pen.penup()
pen.goto(-size / 2 + offset, -size / 2 + offset)
pen.pendown()
draw_square(pen, size)

# connecting edges between the two faces
corners = [(-size / 2, -size / 2), (size / 2, -size / 2),
           (size / 2, size / 2), (-size / 2, size / 2)]

for x, y in corners:
    pen.penup()
    pen.goto(x, y)
    pen.pendown()
    pen.goto(x + offset, y + offset)
```

**Filled Star** — a five-pointed star is drawn with a surprisingly small loop: five lines, turning 144° each time (not 72° — that's the trick that makes the lines cross into a star rather than a pentagon). Filling it is covered properly in the next section, but here's the shape and its fill together:

```python
pen.begin_fill()
for _ in range(5):
    pen.forward(150)
    pen.right(144)
pen.end_fill()
```

## 28.3 Filled Color

There are two different things you might want to color, and `turtle` treats them separately.

**The whole window's background** is set once, on the screen object, and has nothing to do with any particular shape:

```python
screen.bgcolor("skyblue")
```

**A single shape's interior** uses `fillcolor()` together with `begin_fill()` and `end_fill()` bracketing the drawing commands. Everything drawn between the two calls gets filled in once `end_fill()` runs:

```python
pen.color("black", "gold")   # pen (outline) color, then fill color
pen.begin_fill()
for _ in range(4):
    pen.forward(100)
    pen.left(90)
pen.end_fill()
```

`pen.color("black", "gold")` is shorthand for calling `pen.pencolor("black")` and `pen.fillcolor("gold")` in one line. If you only call `pen.color("gold")` with a single argument, both the outline and the fill are set to gold.

## 28.4 Displaying Text and Coordinates

`write()` drops text onto the canvas at the turtle's current position:

```python
pen.penup()
pen.goto(0, 100)
pen.write("Hello, Turtle!", align="center", font=("Arial", 16, "normal"))
```

To find out where the turtle actually is — useful while you're still figuring out coordinates for a drawing — three read-only methods report its position:

```python
print(pen.pos())    # (0.00, 100.00)  <- an (x, y) pair
print(pen.xcor())   # 0.0
print(pen.ycor())   # 100.0
```

A handy debugging habit while building a more complex drawing: write the coordinates onto the canvas as you go, so you can see exactly where each piece landed.

```python
pen.penup()
pen.goto(50, 50)
pen.write(f"{pen.pos()}", font=("Arial", 8, "normal"))
```

Turtle's coordinate system puts `(0, 0)` at the exact center of the window — not the top-left corner, which is what you'd get in most graphics libraries. Positive `x` goes right, positive `y` goes *up* (not down), which is the other detail that trips people up coming from other drawing tools.

## 28.5 Practical Examples

**Draw the Olympic Games Logo** — five interlocking rings in blue, black, red, yellow, and green, arranged in two staggered rows:

```python
colors = ["blue", "black", "red"]
positions = [(-220, 0), (-110, 0), (0, 0)]

for color, (x, y) in zip(colors, positions):
    pen.penup()
    pen.goto(x, y)
    pen.pendown()
    pen.color(color)
    pen.circle(50)

colors_row2 = ["yellow", "green"]
positions_row2 = [(-165, -50), (-55, -50)]

for color, (x, y) in zip(colors_row2, positions_row2):
    pen.penup()
    pen.goto(x, y)
    pen.pendown()
    pen.color(color)
    pen.circle(50)
```

**Draw a Car** — a rectangle body with two circular wheels:

```python
pen.penup()
pen.goto(-100, 0)
pen.pendown()
pen.color("black", "red")
pen.begin_fill()
for side in (150, 60):
    pen.forward(side)
    pen.left(90)
    pen.forward(side if side == 150 else 60)
    pen.left(90)
pen.end_fill()

for x_offset in (-70, 70):
    pen.penup()
    pen.goto(x_offset, -20)
    pen.pendown()
    pen.color("black", "black")
    pen.begin_fill()
    pen.circle(20)
    pen.end_fill()
```

**Draw a House** — a square base, a triangular roof, and a rectangular door:

```python
# base
pen.penup()
pen.goto(-75, 0)
pen.pendown()
pen.color("black", "tan")
pen.begin_fill()
for _ in range(4):
    pen.forward(150)
    pen.left(90)
pen.end_fill()

# roof
pen.penup()
pen.goto(-75, 150)
pen.pendown()
pen.color("black", "firebrick")
pen.begin_fill()
pen.setheading(-30)
for _ in range(3):
    pen.forward(170)
    pen.left(120)
pen.end_fill()
pen.setheading(0)

# door
pen.penup()
pen.goto(-15, 0)
pen.pendown()
pen.color("black", "saddlebrown")
pen.begin_fill()
for side in (30, 70):
    pen.forward(side)
    pen.left(90)
    pen.forward(30 if side == 70 else 70)
    pen.left(90)
pen.end_fill()
```

## 28.6 Challenge: Drawing the Python Logo

Everything up to this point has been building toward a shape with a bit more personality than a square or a star: the two interlocking, color-swapped snakes of the Python logo itself. Give it a go using what this guideline covered — `begin_fill()`/`end_fill()` for each snake's body, `goto()` to position the eyes, and whatever combination of `circle()` and straight segments gets the curve of each snake looking right.

*(Your own code goes here.)*
