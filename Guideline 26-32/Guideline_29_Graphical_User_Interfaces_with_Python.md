# Guideline 29: Graphical User Interfaces with Python

Every guideline so far has run in a terminal. This one leaves the terminal behind: **Tkinter** is Python's built-in library for building windows, buttons, text boxes, and everything else you'd expect from an actual application interface — no `pip install` required, since it ships with Python itself.

```python
import tkinter as tk
```

## 29.1 The Tkinter Module — Some Methods

Every Tkinter program needs exactly one **root window** — the main application window everything else lives inside — and it needs to end with `mainloop()`, which keeps that window open and listening for clicks, keystrokes, and other events until the user closes it.

```python
import tkinter as tk

root = tk.Tk()
root.title("My First Window")
root.geometry("400x300")   # width x height, in pixels

root.mainloop()
```

| Method | What it does |
|---|---|
| `Tk()` | Creates the root window |
| `title(text)` | Sets the window's title bar text |
| `geometry("WxH")` | Sets the window's size (and optionally `+x+y` position) |
| `resizable(width, height)` | Whether the window can be resized (`True`/`False` for each axis) |
| `config()` / `configure()` | Changes a widget's options after it's been created |
| `destroy()` | Closes a specific window or widget |
| `quit()` | Exits `mainloop()` without destroying the windows |
| `mainloop()` | Starts the event loop — must be the last line that runs |

Without `mainloop()`, the window would flash open and close immediately, because the script would simply reach the end and exit.

## 29.2 Widget — Displaying Something with Colors and Sizes

Everything you place inside a window — text, buttons, input boxes — is a **widget**. Every widget follows the same basic pattern: create it, configure its appearance, then place it in the window (placement is its own topic, covered in 29.7).

Common appearance options, shared across almost every widget type:

| Option | Controls |
|---|---|
| `bg` (or `background`) | Background color |
| `fg` (or `foreground`) | Text/foreground color |
| `font` | A tuple like `("Arial", 14, "bold")` |
| `width`, `height` | Size, usually in text-character units, not pixels |
| `padx`, `pady` | Internal padding around the widget's content |
| `relief` | Border style: `"flat"`, `"raised"`, `"sunken"`, `"groove"`, `"ridge"` |
| `borderwidth` (or `bd`) | Thickness of that border |

```python
box = tk.Label(root, text="I'm a widget", bg="navy", fg="white",
                font=("Arial", 16, "bold"), width=20, height=2, relief="raised", bd=3)
box.pack()
```

## 29.3 Label and Frame

A **Label** displays text or an image and nothing else — it doesn't react to clicks.

```python
greeting = tk.Label(root, text="Hello, Tkinter!", font=("Arial", 14))
greeting.pack()
```

A **Frame** is a container — an invisible (or visibly bordered) box used purely to group other widgets together, so you can organize and position a cluster of them as a unit instead of individually.

```python
top_frame = tk.Frame(root, bg="lightgray", padx=10, pady=10)
top_frame.pack()

tk.Label(top_frame, text="Inside the frame").pack()
tk.Label(top_frame, text="Also inside the frame").pack()
```

Frames are especially useful once a window has several distinct sections — a header, a form, a footer — each best organized in its own frame.

## 29.4 Entry and Text (Also Read-Only)

An **Entry** is a single-line text input box:

```python
name_entry = tk.Entry(root, width=25)
name_entry.pack()

print(name_entry.get())          # reads the current text
name_entry.insert(0, "Default")  # inserts text at position 0
name_entry.delete(0, tk.END)     # clears it
```

A **Text** widget is Entry's multi-line counterpart — a whole paragraph box instead of one line. Its position arguments work differently: `"1.0"` means "line 1, character 0."

```python
notes = tk.Text(root, width=40, height=8)
notes.pack()

notes.insert("1.0", "Line one\nLine two")
print(notes.get("1.0", tk.END))
notes.delete("1.0", tk.END)
```

Making either **read-only** — so the user can see but not edit the content — uses the `state` option:

```python
name_entry.config(state="readonly")   # Entry
notes.config(state="disabled")        # Text
```

To update a read-only widget's contents from your own code, you have to temporarily flip `state` back to `"normal"`, make the change, then disable it again.

## 29.5 Button and Selection Widgets

A **Button** runs a function — passed as `command` — whenever it's clicked:

```python
def say_hello():
    print("Hello!")

btn = tk.Button(root, text="Click Me", command=say_hello)
btn.pack()
```

Notice `command=say_hello`, not `command=say_hello()` — with the parentheses, the function would run immediately when the button is created, instead of waiting for a click.

Selection widgets let the user choose from options rather than type freely:

```python
# Checkbutton - independent on/off toggle
agree = tk.IntVar()
tk.Checkbutton(root, text="I agree", variable=agree).pack()

# Radiobutton - pick exactly one from a group
size = tk.StringVar(value="medium")
for option in ("small", "medium", "large"):
    tk.Radiobutton(root, text=option, variable=size, value=option).pack()

# Listbox - a scrollable list of items
fruits = tk.Listbox(root)
for fruit in ("Apple", "Banana", "Cherry"):
    fruits.insert(tk.END, fruit)
fruits.pack()
```

`Checkbutton` and `Radiobutton` both rely on the variable types covered next — that's how the widget and your code stay in sync.

## 29.6 User Input and Configuration

Tkinter has its own variable types — `StringVar`, `IntVar`, `DoubleVar`, `BooleanVar` — that link a widget directly to a value in your code. Set one with `textvariable` (for Entry/Label) or `variable` (for Checkbutton/Radiobutton), and reading or writing the variable automatically updates the widget, and vice versa.

```python
name = tk.StringVar()
tk.Entry(root, textvariable=name).pack()

def show_name():
    print(name.get())   # always reflects whatever's currently typed

tk.Button(root, text="Show", command=show_name).pack()
```

Beyond typed input, widgets can react to raw events — key presses, mouse clicks — through `bind()`:

```python
def on_click(event):
    print(f"Clicked at ({event.x}, {event.y})")

root.bind("<Button-1>", on_click)   # left mouse click anywhere in the window
```

And any widget's options can be changed after creation with `config()`/`configure()` — the same method mentioned in 29.1, used constantly to make an interface feel responsive:

```python
btn.config(text="Clicked!", bg="green")
```

## 29.7 Place, Pack, and Grid

Creating a widget doesn't display it — you also have to tell Tkinter *where* it goes, using one of three **geometry managers**. Mixing more than one inside the same container causes unpredictable layouts, so pick one per container and stick with it.

| Manager | How it positions widgets | Best for |
|---|---|---|
| `pack()` | Stacks widgets edge-to-edge (`side="top"/"left"/...`) | Simple, single-direction layouts |
| `grid()` | Places widgets in a row/column grid (`row=`, `column=`) | Form-like layouts with clear rows and columns |
| `place()` | Places widgets at exact coordinates (`x=`, `y=`) or relative positions | Precise, pixel-specific placement |

```python
# pack()
tk.Label(root, text="Top").pack(side="top")
tk.Label(root, text="Bottom").pack(side="bottom")

# grid()
tk.Label(root, text="Name:").grid(row=0, column=0)
tk.Entry(root).grid(row=0, column=1)

# place()
tk.Label(root, text="Fixed spot").place(x=50, y=100)
```

`grid()` is generally the easiest to reach for once a layout has more than a couple of widgets, since rows and columns line up automatically without manual pixel math.

## 29.8 Canvas and PhotoImage

A **Canvas** is a blank drawing surface inside the window — closer to the `turtle` module from Guideline 28 than to a typical widget, but embedded in a real interface instead of its own standalone window.

```python
canvas = tk.Canvas(root, width=300, height=200, bg="white")
canvas.pack()

canvas.create_line(0, 0, 300, 200, fill="black")
canvas.create_rectangle(50, 50, 150, 120, fill="skyblue")
canvas.create_oval(180, 40, 260, 120, fill="tomato")
canvas.create_text(150, 160, text="Canvas!", font=("Arial", 14))
```

`PhotoImage` loads an actual image file (GIF or PNG) onto a Canvas or Label. Tkinter requires you to keep a reference to the image around — if the variable holding it gets garbage-collected, the image silently vanishes from the screen, which is a classic Tkinter gotcha.

```python
photo = tk.PhotoImage(file="picture.png")
canvas.create_image(150, 100, image=photo)

# or on a Label:
img_label = tk.Label(root, image=photo)
img_label.pack()
```

## 29.9 Tables and Scrolldown

Plain Tkinter doesn't have a table widget, but its `ttk` extension (a more modern-looking widget set, also built in) includes `Treeview`, which doubles as a straightforward table when you don't use its tree-nesting features:

```python
from tkinter import ttk

table = ttk.Treeview(root, columns=("name", "age"), show="headings")
table.heading("name", text="Name")
table.heading("age", text="Age")

table.insert("", tk.END, values=("Alice", 24))
table.insert("", tk.END, values=("Bob", 31))

table.pack()
```

A **Scrollbar** attaches to any widget whose content can exceed the visible area — a `Listbox`, `Text`, or `Treeview` — by linking the two together in both directions:

```python
scrollbar = tk.Scrollbar(root)
listbox = tk.Listbox(root, yscrollcommand=scrollbar.set)
scrollbar.config(command=listbox.yview)

listbox.pack(side="left", fill="y")
scrollbar.pack(side="right", fill="y")
```

`yscrollcommand` tells the listbox to keep the scrollbar's thumb in sync as its content scrolls, and the scrollbar's `command` tells it to move the listbox's view when dragged — each one drives the other.

## 29.10 Pop-up Window

A **Toplevel** creates a brand-new window on top of the root window — used for dialogs, settings panels, or any secondary window that isn't the whole application.

```python
def open_popup():
    popup = tk.Toplevel(root)
    popup.title("Details")
    popup.geometry("200x100")
    tk.Label(popup, text="This is a pop-up window").pack(pady=20)

tk.Button(root, text="Open Popup", command=open_popup).pack()
```

A `Toplevel` behaves like its own small Tkinter program — it can hold any widget the root window can — but it shares the same `mainloop()`, so you never call `mainloop()` a second time for it.

## 29.11 Message Boxes

The `tkinter.messagebox` module covers the common "quick dialog" cases — informing, warning, or asking the user something — without building a `Toplevel` by hand.

```python
from tkinter import messagebox

messagebox.showinfo("Saved", "Your changes were saved.")
messagebox.showwarning("Careful", "This action can't be undone.")
messagebox.showerror("Error", "Something went wrong.")

if messagebox.askyesno("Confirm", "Delete this item?"):
    print("Deleting...")
```

| Function | Shows | Returns |
|---|---|---|
| `showinfo()` | An informational message | Nothing meaningful |
| `showwarning()` | A warning message | Nothing meaningful |
| `showerror()` | An error message | Nothing meaningful |
| `askquestion()` | A yes/no question | `"yes"` or `"no"` (strings) |
| `askyesno()` | A yes/no question | `True` or `False` |
| `askokcancel()` | An OK/Cancel confirmation | `True` or `False` |

## 29.12 Challenges

Three projects to pull everything in this guideline together, each combining widgets, layout, and the state-tracking tools covered above. No code here — these are yours to build.

- **Student Management System** — an interface for adding, viewing, and removing student records (name, age, grade), using Entry widgets for input, a Treeview or Listbox to display the current roster, and Buttons wired to functions that update it.
- **Drawing Pad** — a Canvas the user can draw on freehand by dragging the mouse, with buttons or a color-selection widget to change the pen color, and a "Clear" button to reset the canvas.
- **Calculator** — a Grid-based layout of number and operator Buttons feeding into a display Entry or Label, evaluating the expression when "=" is pressed.

---

Tkinter's whole design rests on the same handful of ideas repeated everywhere: create a widget, configure how it looks, place it with a geometry manager, and connect it to your code through a variable, a `command`, or `bind()`. Once those four moves feel automatic, building any interface becomes a matter of combining widgets rather than learning something new for each one.
