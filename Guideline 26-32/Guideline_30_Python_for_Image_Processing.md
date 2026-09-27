# Guideline 30: Python for Image Processing

An image, to Python, is just a large grid of numbers — one set of red/green/blue values per pixel. **Pillow** (imported as `PIL`, its historical name — Python Imaging Library) is the standard tool for working with that grid: opening image files, transforming them, and saving the result, all without touching a single pixel value by hand unless you want to.

```python
pip install pillow
```

```python
from PIL import Image
```

## 30.1 The Pillow Library — Some Key Methods

Nearly everything in this guideline starts from an `Image` object, and ends by either displaying it or saving it back to disk.

| Method / Attribute | What it does |
|---|---|
| `Image.open(path)` | Loads an image file into an `Image` object |
| `image.show()` | Opens the image in your system's default image viewer |
| `image.save(path)` | Writes the image to disk — the file extension decides the format |
| `image.size` | An `(width, height)` tuple, in pixels |
| `image.format` | The original file format (`"JPEG"`, `"PNG"`, ...) |
| `image.mode` | The color mode (`"RGB"`, `"RGBA"`, `"L"` for grayscale, ...) |
| `image.copy()` | Returns an independent duplicate — useful before a destructive edit |

```python
img = Image.open("photo.jpg")
print(img.size, img.format, img.mode)   # (1920, 1080) JPEG RGB
img.save("photo_copy.png")              # Pillow converts based on the new extension
```

## 30.2 Opening and Displaying Images

`Image.open()` doesn't actually read all the pixel data immediately — Pillow waits until the image is actually used (resized, saved, displayed) before doing the real work, which is why opening even a large image is nearly instant.

```python
img = Image.open("landscape.png")
img.show()
```

`show()` is meant for quick, interactive checking while you develop — it saves a temporary file and hands it to whatever program your operating system uses to open images, which means it isn't meant for a program that needs to display images as part of its actual, permanent behavior.

## 30.3 Resizing Images

`resize()` takes an exact target size and stretches or shrinks the image to fit it — which distorts the proportions unless you calculate the new dimensions yourself:

```python
img = Image.open("photo.jpg")
resized = img.resize((400, 300))   # exact width, height — may distort proportions
resized.save("photo_resized.jpg")
```

To resize proportionally, calculate the new height from the ratio of the new width to the old one (or the reverse):

```python
target_width = 400
ratio = target_width / img.width
new_size = (target_width, int(img.height * ratio))
proportional = img.resize(new_size)
```

`thumbnail()` does this automatically, and modifies the image in place — it shrinks the image to fit *within* the given box while preserving its proportions (it never enlarges an image that's already smaller):

```python
img.thumbnail((400, 400))   # fits inside 400x400, keeping proportions, no return value
img.save("photo_thumbnail.jpg")
```

## 30.4 Rotating Images

`rotate()` turns the image counter-clockwise by the given number of degrees:

```python
rotated = img.rotate(90)
rotated.save("photo_rotated.jpg")
```

By default, rotating by an angle that isn't a multiple of 90° keeps the original canvas size, which crops the corners of the rotated image. Pass `expand=True` to grow the canvas so the whole rotated image fits:

```python
rotated_45 = img.rotate(45, expand=True)
```

Flipping (as opposed to rotating) uses `transpose()` with one of Pillow's built-in constants:

```python
flipped_h = img.transpose(Image.FLIP_LEFT_RIGHT)
flipped_v = img.transpose(Image.FLIP_TOP_BOTTOM)
```

## 30.5 Cropping Images

`crop()` cuts a rectangular region out of the image, defined by a 4-value box: `(left, top, right, bottom)`, measured in pixels from the top-left corner.

```python
img = Image.open("photo.jpg")   # size, say, (1920, 1080)

box = (200, 100, 800, 600)      # left, top, right, bottom
cropped = img.crop(box)
cropped.save("photo_cropped.jpg")
```

The resulting image's size is simply `(right - left, bottom - top)` — in the example above, `(600, 500)`. A common pattern is cropping the exact center of an image:

```python
def crop_center(image, target_width, target_height):
    left = (image.width - target_width) // 2
    top = (image.height - target_height) // 2
    right = left + target_width
    bottom = top + target_height
    return image.crop((left, top, right, bottom))

centered = crop_center(img, 500, 500)
```

## 30.6 Filtering Images (Grayscale, HSV, Adjusting Brightness)

**Grayscale** is a mode conversion — Pillow's `"L"` mode stores one brightness value per pixel instead of separate red, green, and blue values:

```python
gray = img.convert("L")
gray.save("photo_grayscale.jpg")
```

**HSV** (Hue, Saturation, Value) is an alternate way of describing the same colors — instead of red/green/blue amounts, each pixel is described by its hue (which color), saturation (how vivid), and value (how bright). Converting to it is useful when you want to adjust one of those properties without disturbing the others:

```python
hsv_img = img.convert("HSV")
```

**Adjusting brightness** goes through the `ImageEnhance` module rather than `convert()`, since it's a gradual adjustment, not a mode change:

```python
from PIL import ImageEnhance

enhancer = ImageEnhance.Brightness(img)
brighter = enhancer.enhance(1.5)   # 1.0 = unchanged, >1 brighter, <1 darker
darker = enhancer.enhance(0.6)

brighter.save("photo_bright.jpg")
```

`ImageEnhance` has matching classes for `Contrast`, `Color` (saturation), and `Sharpness`, all used the same way — construct the enhancer around the image, then call `enhance(factor)`.

## 30.7 Retrieving Metadata (Pixels and Others)

Beyond `size`, `format`, and `mode` from 30.1, Pillow can inspect — and read or write — individual pixels directly.

`getpixel()` reads the color value at a specific `(x, y)` coordinate:

```python
img = Image.open("photo.jpg")
print(img.getpixel((0, 0)))   # e.g. (255, 240, 210) for an RGB image
```

`putpixel()` writes one back — rarely fast enough for editing a whole image, but useful for small, targeted changes:

```python
img.putpixel((0, 0), (255, 0, 0))   # set the top-left pixel to pure red
```

`getdata()` returns every pixel's value as one long sequence, in row-major order (left to right, then top to bottom) — useful for statistics across the whole image, like the average brightness:

```python
pixels = list(img.convert("L").getdata())
average_brightness = sum(pixels) / len(pixels)
print(average_brightness)
```

Finally, `.info` is a dictionary holding whatever extra metadata the file format included — this varies a lot by file and format, and can include things like DPI or, for JPEGs, raw EXIF data:

```python
print(img.info)   # e.g. {'jfif': 257, 'jfif_version': (1, 1), 'dpi': (72, 72), ...}
```

---

Every operation in this guideline follows the same shape: open an image, transform it with one method call, then save or display the result. The transformations themselves are what change — resize, rotate, crop, convert, enhance — but the surrounding pattern of `Image.open()` in, something out, `.save()` stays exactly the same, which is why Pillow is comfortable to pick up a piece at a time.
