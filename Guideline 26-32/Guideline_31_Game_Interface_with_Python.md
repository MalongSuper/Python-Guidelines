# Guideline 31: Game Interface with Python

Tkinter (Guideline 29) is built for interfaces that mostly sit still and wait for a click. Games need something that runs continuously — redrawing the screen dozens of times a second, checking constantly for input, reacting the instant two things touch. **PyGame** is the library built for exactly that.

```python
pip install pygame
```

```python
import pygame
```

## 31.1 What Is PyGame? (Some Key Methods)

Every PyGame program is built around a **game loop** — a `while` loop that runs continuously, handling input, updating positions, and redrawing the screen, over and over, dozens of times per second, until the player quits.

```python
import pygame

pygame.init()

screen = pygame.display.set_mode((800, 600))
pygame.display.set_caption("My First Game")
clock = pygame.time.Clock()

running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    screen.fill((30, 30, 30))   # clear the screen each frame

    pygame.display.flip()       # show what was just drawn
    clock.tick(60)              # cap the loop at 60 frames per second

pygame.quit()
```

| Method | What it does |
|---|---|
| `pygame.init()` | Starts up all of PyGame's internal modules — call this first |
| `pygame.display.set_mode((w, h))` | Creates the game window, returning its drawing surface |
| `pygame.display.set_caption(text)` | Sets the window's title bar |
| `pygame.display.flip()` / `.update()` | Pushes everything drawn this frame onto the actual screen |
| `pygame.time.Clock()` | Creates a clock object for controlling frame rate |
| `clock.tick(fps)` | Pauses just long enough to cap the loop at `fps` frames per second |
| `pygame.quit()` | Shuts down PyGame cleanly when the loop ends |

Skipping `clock.tick()` doesn't crash anything, but the loop then runs as fast as the computer possibly can — which wastes processing power and makes movement speed depend on how fast any given machine happens to run, instead of staying consistent.

## 31.2 Drawing Shapes

Every drawing call in PyGame draws onto a **surface** — usually `screen`, the one returned by `set_mode()` — and none of it actually appears until `display.flip()` runs, which is why every example redraws everything fresh at the top of each frame.

```python
screen.fill((255, 255, 255))   # a fill counts as "drawing a rectangle over everything"

pygame.draw.rect(screen, (200, 30, 30), (100, 100, 120, 80))       # x, y, width, height
pygame.draw.circle(screen, (30, 30, 200), (400, 300), 50)          # center, radius
pygame.draw.line(screen, (0, 0, 0), (0, 0), (800, 600), 3)         # start, end, thickness
pygame.draw.polygon(screen, (30, 200, 30), [(300, 100), (350, 200), (250, 200)])
```

Colors are always `(red, green, blue)` tuples, each from 0 to 255 — the same RGB values from Guideline 30's image work. Every `pygame.draw` function returns a `Rect` describing the area it drew, which is often stored for later use in collision checks (31.6).

## 31.3 Event Handling

PyGame reports everything that happens — key presses, mouse clicks, the window closing — as **events**, collected once per frame with `pygame.event.get()`. The loop in 31.1 already checks for one event type, `pygame.QUIT`; most games check several more.

```python
for event in pygame.event.get():
    if event.type == pygame.QUIT:
        running = False
    elif event.type == pygame.KEYDOWN:
        print(f"Key pressed: {pygame.key.name(event.key)}")
    elif event.type == pygame.MOUSEBUTTONDOWN:
        print(f"Mouse clicked at: {event.pos}")
```

| Event type | Fires when |
|---|---|
| `pygame.QUIT` | The window's close button is clicked |
| `pygame.KEYDOWN` / `pygame.KEYUP` | A key is pressed / released |
| `pygame.MOUSEBUTTONDOWN` / `MOUSEBUTTONUP` | A mouse button is pressed / released |
| `pygame.MOUSEMOTION` | The mouse moves — `event.pos` gives its position |

`KEYDOWN` and `KEYUP` fire once each, exactly when the key changes state — they're events, not a continuous reading, which is the distinction that matters for 31.5.

## 31.4 Working with Texts and Images

Text needs a **font** object before it can be drawn — fonts render text into their own small surface, which then gets drawn (`blit`ted) onto the screen like anything else:

```python
font = pygame.font.Font(None, 36)   # None = default font, 36 = size
text_surface = font.render("Score: 0", True, (255, 255, 255))   # text, anti-alias, color
screen.blit(text_surface, (20, 20))   # draw it at (x, y)
```

`blit()` — "block transfer" — is PyGame's general-purpose "draw this surface onto that surface" operation, and it's exactly how images work too:

```python
player_image = pygame.image.load("player.png").convert_alpha()
screen.blit(player_image, (100, 200))
```

`.convert_alpha()` optimizes the image for PyGame's pixel format and preserves transparency (for PNGs with a transparent background) — worth calling on every loaded image, since it makes `blit()` noticeably faster.

## 31.5 Buttons (Keyboards)

Unlike `KEYDOWN`, which only fires the instant a key changes state, `pygame.key.get_pressed()` reports every key's state *right now* — which is what you want for smooth, continuous movement instead of one twitch per press.

```python
keys = pygame.key.get_pressed()

if keys[pygame.K_LEFT]:
    player_x -= 5
if keys[pygame.K_RIGHT]:
    player_x += 5
if keys[pygame.K_UP]:
    player_y -= 5
if keys[pygame.K_DOWN]:
    player_y += 5
```

This line normally sits inside the game loop, right after the `event.get()` block — checking it every single frame is what makes the character glide instead of stepping once per keypress. `pygame.K_LEFT`, `K_RIGHT`, and similar constants exist for essentially every key, including letters (`K_a`, `K_b`, ...), digits (`K_0`, ...), and the space bar (`K_SPACE`).

## 31.6 Collision Detection (Determine Winning Condition)

The simplest and most common collision check compares two `Rect` objects — the same rectangles `draw.rect()` returns, or ones you build directly with `pygame.Rect(x, y, width, height)`:

```python
player_rect = pygame.Rect(player_x, player_y, 40, 40)
enemy_rect = pygame.Rect(enemy_x, enemy_y, 40, 40)

if player_rect.colliderect(enemy_rect):
    print("Hit!")
```

For circular objects, `colliderect()` isn't accurate near the corners — a distance check against the sum of the two radii is the more honest test:

```python
import math

def circles_collide(x1, y1, r1, x2, y2, r2):
    distance = math.hypot(x2 - x1, y2 - y1)
    return distance < (r1 + r2)
```

Collision detection is also how a game actually decides it's *over* — a win or loss condition is usually just a collision check (or a related state check, like a counter hitting zero) tied to ending the loop:

```python
if player_rect.colliderect(goal_rect):
    print("You win!")
    running = False
```

## 31.7 Challenges

Two classic games, both entirely buildable from what this guideline covers. No code here — these are yours to build.

- **Snake** — a growing chain of rectangles that moves continuously in the direction of the last-pressed arrow key, ends the game on collision with the wall or its own body, and grows by one segment each time it collides with a randomly placed food rectangle.
- **Connect Four** — a grid of circles drawn with `pygame.draw.circle()`, where mouse clicks (31.3) determine which column a piece drops into, alternating between two players' colors, checking after every move for four matching circles in a row — horizontally, vertically, or diagonally — as the winning condition.

---

Every PyGame program, no matter how complex, is the same loop from 31.1 running over and over: handle events, update positions based on input and collisions, redraw the screen, cap the frame rate, repeat. Everything else in this guideline — shapes, text, images, keys, collisions — is just what happens to fill in that loop's middle.
