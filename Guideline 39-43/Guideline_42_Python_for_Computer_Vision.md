# Guideline 42: Python for Computer Vision

Guideline 30 taught you Pillow: opening an image, resizing it, rotating it, adjusting it. That was **image processing** — changing an image. This guideline is about something harder: **image understanding** — asking a computer to tell you *what is in* an image, *where* it is, and *what shape* it has. That is the job of **computer vision**.

The library that made this practical is **OpenCV**, and it is the backbone of this guideline. Along the way you will meet the classical, decades-old techniques that still run instantly on a laptop with no training required, and you will meet the points where classical technique hands off to the deep learning of Guideline 40, because modern computer vision is a partnership between the two, not a replacement of one by the other.

```bash
pip install opencv-python
```

The module is installed under the name `opencv-python` but imported as `cv2` (the "2" is historical, left over from OpenCV's second major C++ interface, but the current version is well past that).

A word on this guideline's examples. Some tasks in this guideline (opening a live webcam) need hardware this book cannot demonstrate on a page, and some (classifying a photo with a deep network) need model files downloaded from the internet. Wherever that is true, the guideline says so plainly, and every example that *can* run on synthetic shapes and saved images (which is most of them) was actually run to produce the output shown.

---

## 1. Computer Vision

**Computer vision** is the field concerned with getting a computer to extract meaning from images and video: what objects are present, where they are, what they are doing, and how the scene is structured. It sits alongside Natural Language Processing (Guideline 41) as one of the two application areas that most visibly benefit from deep learning (Guideline 40), and for a similar reason: raw pixels, like raw words, do not come with their meaning attached.

### Why vision is hard

A photograph is a grid of numbers (Guideline 34, Guideline 40's Section 4 on tensors), and nothing about that grid says "cat" or "stop sign". The same object looks different depending on lighting, angle, distance, occlusion (something partly blocking it) and background clutter. A cat photographed from the side, in shadow, half-hidden behind a chair, is still obviously a cat to a human and a serious challenge to a program. Computer vision is the discipline of closing that gap.

### The spectrum of tasks

Computer vision is not one task. This guideline works through them roughly in order of increasing difficulty:

| Task | Question it answers | Covered in |
|---|---|---|
| Image processing | how do I change this image? | Guideline 30 |
| Feature detection & matching | which parts of this image are distinctive, and do they appear in another image? | Section 2 |
| Image classification | what is the single main subject of this whole image? | Section 3 |
| Image segmentation | which exact pixels belong to which object? | Section 4 |
| Object detection | what objects are present, and where (bounding boxes)? | Section 5 |

### Two toolkits, one field

Modern computer vision code typically reaches for two different kinds of tool, and this guideline covers both:

- **Classical computer vision** (most of OpenCV): hand-designed algorithms — edge detectors, colour filters, geometric transforms — that run instantly on a CPU with no training and no data. They are the right tool for well-defined, controlled problems: finding a barcode, aligning two scans, tracking a bright dot.
- **Deep learning for vision** (Guideline 40's CNNs, extended here): networks trained on large labelled datasets to recognise open-ended, real-world categories — a face, a dog breed, a tumour. They need training data and computing power but handle the variability of the real world far better than hand-designed rules ever could.

Sections 3 and 5 in particular will show both approaches side by side, because knowing when you need the heavier tool and when the lighter one will do is itself a skill.

---

## 2. OpenCV

**OpenCV** ("Open Computer Vision") is a large, mature library, originally built in C++, with excellent Python bindings. It represents images, as you would now expect, as NumPy arrays (Guideline 34), which means everything you already know about array slicing, reshaping and broadcasting works directly on the images in this guideline.

### Reading, showing and writing images

```python
import cv2

img = cv2.imread("photo.jpg")     # reads into a NumPy array
print(img.shape, img.dtype)        # e.g. (200, 300, 3) uint8

cv2.imwrite("copy.png", img)       # write to disk
```

**The single most important trap in OpenCV**, and worth learning before anything else: OpenCV stores colour images as **BGR** (blue, green, red), not the RGB order used by Pillow (Guideline 30), Matplotlib (Guideline 38) and almost everything else in Python. This is a historical accident that OpenCV has never changed, for compatibility. If you load an image with `cv2.imread` and display it with Matplotlib's `plt.imshow` without converting, the reds and blues will be swapped:

```python
import cv2

img_bgr = cv2.imread("photo.jpg")
img_rgb = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2RGB)   # fix the channel order before showing with Matplotlib
```

`cv2.imshow("window title", img)` opens a native window to display an image; it needs a `cv2.waitKey(0)` call afterwards to keep the window open until a key is pressed, and `cv2.destroyAllWindows()` to close it. That native window does not work in every environment (notably inside Jupyter notebooks or headless servers), so in a notebook, converting to RGB and using `plt.imshow` is the more portable habit.

### Drawing

OpenCV can draw directly onto an array of pixels, which is how the illustrations in this guideline were produced. All the drawing functions take BGR colour tuples:

```python
import numpy as np

img = np.zeros((200, 300, 3), dtype=np.uint8)         # a blank black canvas

cv2.rectangle(img, (30, 30), (120, 120), (0, 0, 255), -1)     # filled red rectangle (BGR!)
cv2.circle(img, (220, 80), 50, (0, 255, 0), 3)                 # green circle outline, thickness 3
cv2.line(img, (0, 150), (300, 150), (255, 0, 0), 4)             # blue line
cv2.putText(img, "Hello CV", (30, 190),
           cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2)     # white text
```

| Function | Draws | Key arguments |
|---|---|---|
| `cv2.rectangle(img, pt1, pt2, color, thickness)` | a rectangle | `thickness=-1` fills it |
| `cv2.circle(img, center, radius, color, thickness)` | a circle | same fill convention |
| `cv2.line(img, pt1, pt2, color, thickness)` | a straight line | |
| `cv2.putText(img, text, org, font, scale, color, thickness)` | text | `org` is the bottom-left corner of the text |

Points are always given as `(x, y)` — column first, then row — which is the opposite order from NumPy's own `array[row, column]` indexing. Keeping the two straight is the second classic source of OpenCV bugs.

### Colour spaces

`cv2.cvtColor(img, code)` converts between colour representations. Beyond the BGR/RGB conversion above, the most useful is **HSV** (hue, saturation, value), which separates *which colour* (hue) from *how vivid* (saturation) and *how bright* (value). This separation makes HSV far better than BGR for tasks like "find everything that's red", because red in BGR can be dark red, bright red or pink, three very different-looking triples of numbers, whereas in HSV they all share roughly the same hue.

```python
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
```

### Resizing, rotating, cropping, flipping

These will look familiar from Pillow (Guideline 30); here they operate on arrays directly.

```python
resized = cv2.resize(img, (150, 100))            # exact target size (width, height)
resized2 = cv2.resize(img, None, fx=0.5, fy=0.5)  # or scale by a factor

rot_matrix = cv2.getRotationMatrix2D(center=(150, 100), angle=45, scale=1.0)
rotated = cv2.warpAffine(img, rot_matrix, (300, 200))

cropped = img[30:120, 30:120]      # cropping is just NumPy slicing: [rows, columns]

flipped = cv2.flip(img, 1)         # 1 = horizontal, 0 = vertical, -1 = both
```

Rotating an image is a two-step idea worth understanding rather than memorising: `getRotationMatrix2D` computes the 2×3 matrix that describes the rotation, and `warpAffine` is the general-purpose function that actually applies any such matrix to the image. The same `warpAffine` function also handles translation and scaling, given the right matrix, which is why OpenCV separates "compute the transform" from "apply the transform".

### Filtering, thresholding and edges

**Blurring** smooths out noise and fine detail, and is often a useful first step before further processing:

```python
blurred = cv2.GaussianBlur(img, (7, 7), 0)    # (7,7) is the blur kernel size; must be odd numbers
```

**Thresholding** converts a grayscale image into pure black and white by a cutoff value, which is the basis of separating a subject from its background:

```python
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
ret, thresh = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)

print(ret)          # 127.0 — the threshold value actually used
print(thresh.shape) # same shape as gray, but every pixel is now 0 or 255
```

Every pixel brighter than 127 becomes 255 (white); every pixel at or below it becomes 0 (black). `cv2.THRESH_OTSU`, used as `cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)`, automatically calculates a good threshold from the image's own brightness histogram instead of you guessing one.

**Edge detection** finds the boundaries between regions, the sharp changes in brightness that outline objects. The Canny algorithm is the standard choice:

```python
edges = cv2.Canny(gray, threshold1=100, threshold2=200)
print(edges.shape, edges.dtype, edges.max())   # (200, 300) uint8 255
```

The result is a black image with white lines exactly where the edges were found. The two thresholds control sensitivity: pixels with a gradient strength above `threshold2` are always treated as edges, pixels below `threshold1` never are, and pixels in between are kept only if they connect to a strong edge. Edge detection is frequently the first step before contour detection, next.

### Finding shapes: contours

A **contour** is a curve joining the continuous points along a boundary, in practice the outline of a connected white blob in a black-and-white image. Contours are how you turn "there are some white pixels here" into "there is one shape here, and here is its size and position":

```python
contours, hierarchy = cv2.findContours(thresh, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

print(len(contours))          # 2, for a scene with a rectangle and a circle

for c in contours:
    print(cv2.contourArea(c), cv2.boundingRect(c))
# 7704.0 (170, 30, 101, 101)
# 8100.0 (30, 30, 91, 91)
```

`cv2.RETR_EXTERNAL` keeps only outermost shapes (ignoring holes inside them); `cv2.CHAIN_APPROX_SIMPLE` compresses the contour to just its corner points instead of every single pixel along the boundary. `cv2.contourArea` gives the enclosed area in pixels, and `cv2.boundingRect` returns `(x, y, width, height)`, the smallest upright rectangle that contains the shape, letting you find, count and measure blobs in an image with no training data whatsoever:

```python
output = img.copy()
cv2.drawContours(output, contours, -1, (0, 0, 255), 2)   # -1 means draw all contours found
```

A related, very handy function is `cv2.approxPolyDP`, which simplifies a contour down to its dominant corners, useful for telling shapes apart:

```python
approx = cv2.approxPolyDP(contours[0], epsilon=0.02 * cv2.arcLength(contours[0], True), closed=True)
print(len(approx))    # 4, for a rectangular contour
```

A contour reduced to 4 points is very likely a rectangle; one reduced to 3 is a triangle; one that needs many points to approximate is closer to a circle. This is a genuinely useful, zero-training way to classify simple shapes, and it is the conceptual ancestor of the much more general shape and object understanding that deep learning provides in Sections 3 to 5.

### Feature detection and matching

Contours describe *shape*. **Features** describe *distinctive points* — corners, blobs, textured spots — that a computer can reliably find again in a different photo of the same object, even if it has moved, rotated or is a different size. **ORB** (Oriented FAST and Rotated BRIEF) is a fast, free algorithm for this:

```python
orb = cv2.ORB_create()

keypoints1, descriptors1 = orb.detectAndCompute(img1, None)
keypoints2, descriptors2 = orb.detectAndCompute(img2, None)

print(len(keypoints1), len(keypoints2))    # 64 64

matcher = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)
matches = matcher.match(descriptors1, descriptors2)
matches = sorted(matches, key=lambda m: m.distance)    # best matches first

print(len(matches), matches[0].distance)   # 21 6.0
```

Each **keypoint** is a distinctive location (a corner or blob); each **descriptor** is a short numeric fingerprint describing the patch around it. `BFMatcher` ("brute-force matcher") compares every descriptor in the first image against every descriptor in the second and finds the closest pairs; sorting by `distance` (lower is a better match) puts the most confident matches first. This is the classical technique behind panorama stitching (finding where two overlapping photos line up), simple object tracking, and augmented reality markers, all without a single neural network.

---

## 3. Image Classification

**Image classification** answers one question: what is the single main subject of this whole image? Cat or dog. Digit 0 through 9. Benign or malignant.

### The classical approach: it has real, if limited, uses

Before deep learning, classification meant hand-engineering features (much like the feature engineering problem described in Guideline 40's opening) and feeding them to a classical model from Guideline 39. A simple, genuinely useful example is classifying by **colour signature**, using the HSV conversion from Section 2:

```python
import cv2
import numpy as np

def dominant_hue(img):
    hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
    hist = cv2.calcHist([hsv], [0], None, [180], [0, 180])   # a histogram of hues
    return int(np.argmax(hist))

print(dominant_hue(cv2.imread("green_apple.jpg")))   # a hue value around the green range
print(dominant_hue(cv2.imread("red_apple.jpg")))     # a hue value around the red range
```

`cv2.calcHist` counts how many pixels fall into each of 180 possible hue buckets (OpenCV represents hue on a 0–179 scale), and the bucket with the most pixels is the dominant colour. This is a real, workable approach for a narrow, controlled problem, such as sorting fruit on a conveyor belt by colour, and it needs no training data at all. It falls apart the moment the problem becomes "is this any kind of apple", because colour alone cannot capture shape, texture or context, and it is exactly this ceiling that motivated deep learning.

### The deep learning approach: reusing what you already built

For general-purpose classification (thousands of possible categories, huge variation in lighting and angle), the standard tool is the convolutional neural network from Guideline 40, and you do not need to train one from scratch. Guideline 40's Section 12 already walked through **transfer learning**: taking a network pre-trained on ImageNet (over a million photos across 1,000 categories) and reusing it. That same pre-trained network can classify a photo directly, with no fine-tuning at all, straight out of the box:

```python
from tensorflow import keras
import numpy as np

model = keras.applications.MobileNetV2(weights="imagenet")

img = keras.utils.load_img("photo.jpg", target_size=(224, 224))
arr = keras.utils.img_to_array(img)
arr = np.expand_dims(arr, axis=0)                                    # add the batch axis (Guideline 40, Section 4)
arr = keras.applications.mobilenet_v2.preprocess_input(arr)

predictions = model.predict(arr)
top3 = keras.applications.mobilenet_v2.decode_predictions(predictions, top=3)[0]

for _, label, confidence in top3:
    print(f"{label:20s} {confidence:.3f}")

# golden_retriever     ≈0.87
# Labrador_retriever   ≈0.08
# cocker_spaniel       ≈0.02
```

(This code needs an internet connection the first time it runs, to download the pre-trained weights, and the confidence values above are illustrative rather than reproduced from a run in this book, since they depend on the actual photo supplied.) Every piece of this should look familiar: `expand_dims` for the batch axis is Section 4 of Guideline 40, `preprocess_input` is the model-specific scaling from its transfer-learning section, and `decode_predictions` simply translates the network's 1,000 raw probabilities back into readable category names, sorted highest first.

The lesson of this section is not "always use deep learning." It is: reach for a histogram, a contour count, or a colour check first, when the problem is genuinely narrow, and reach for a pre-trained network when the categories are broad or the visual variation is high. Knowing which situation you are in is the actual skill.

---

## 4. Image Segmentation (Semantic or Instance)

Classification answers "what is in this image?" for the image as a whole. **Segmentation** asks a much more precise question: **which exact pixels** belong to which object? The output is not a label; it is a **mask**, an image the same size as the original where each pixel is marked with which object (or background) it belongs to.

There are two flavours, and the difference matters:

- **Semantic segmentation** labels every pixel by *category* only. Every pixel belonging to any car gets the label "car", with no distinction between car #1 and car #2.
- **Instance segmentation** goes further: every pixel is labelled by *which specific object* it belongs to. Car #1's pixels and car #2's pixels get different labels, even though both are "car".

Picture a photo of a parking lot with three identical parked cars. Semantic segmentation paints all three the same colour ("car"). Instance segmentation paints each of the three a *different* colour, because it has told them apart as individuals.

### Classical segmentation: GrabCut

OpenCV includes a classical algorithm, **GrabCut**, that segments a foreground object from its background given only a rough starting rectangle around it. It works iteratively, refining a guess about which pixels are foreground and which are background based on their colour statistics:

```python
import cv2
import numpy as np

img = cv2.imread("object_on_table.jpg")
mask = np.zeros(img.shape[:2], np.uint8)       # will be filled in by grabCut
bgd_model = np.zeros((1, 65), np.float64)      # working storage GrabCut needs internally
fgd_model = np.zeros((1, 65), np.float64)

rect = (40, 30, 200, 140)     # a rectangle (x, y, width, height) roughly containing the object

cv2.grabCut(img, mask, rect, bgd_model, fgd_model, iterCount=5, mode=cv2.GC_INIT_WITH_RECT)

# GrabCut labels each pixel 0 (background), 1 (foreground), 2 (probable background) or 3 (probable foreground)
final_mask = np.where((mask == 2) | (mask == 0), 0, 1).astype("uint8")

result = img * final_mask[:, :, np.newaxis]    # zero out every background pixel
cv2.imwrite("segmented.png", result)
```

On a synthetic test image (a green rectangle on a grey background, with the rectangle `(40, 30, 200, 140)` given as the starting hint), this correctly isolated the rectangle: it found 12,221 foreground pixels against a rectangle interior of roughly 12,000 pixels, a close match. GrabCut is genuinely useful for a specific, common job: cutting a clearly distinct foreground object out of a background, with no training data, as long as you can supply a reasonable starting rectangle. It struggles when foreground and background share similar colours, or when there are several separate objects to isolate individually, which is exactly the gap deep learning fills.

### Deep learning segmentation

Modern semantic and instance segmentation is done with specialised convolutional architectures, direct descendants of the CNNs in Guideline 40:

- **U-Net** is the standard architecture for semantic segmentation. It has a distinctive "U" shape: an encoder half that shrinks the image while extracting increasingly abstract features (much like the convolution-and-pooling stack from Guideline 40), and a decoder half that grows the image back to its original size while placing those features precisely, producing a full-resolution mask. Connections that skip directly across the "U" preserve the fine spatial detail that pure shrink-then-grow would lose. It is especially prominent in medical imaging, where marking the exact pixels of a tumour matters enormously.
- **Mask R-CNN** extends object detection (Section 5) to instance segmentation: it first finds a bounding box for each individual object (exactly what an object detector does), and then, for each box, predicts a precise pixel mask just for that one object. This two-stage design (find the objects, then mask each one separately) is what allows it to tell apart several objects of the same category.

Both are available pre-trained through libraries such as `segmentation-models` (built on Keras) or `torchvision` (built on PyTorch), following the same download-a-pre-trained-model, and optionally fine-tune, pattern from Guideline 40's transfer learning section and Section 3 above. Building either from scratch is well beyond this guideline, but recognising the "U" shape and the two-stage design tells you what you are looking at when you encounter them.

---

## 5. Object Detection

**Object detection** sits between classification and segmentation in what it demands: for every object in an image, it must say *what* it is (classification) and roughly *where* it is (a bounding box), but not the exact pixel outline that segmentation requires. "There is a dog here, in this rectangle, and a person there, in that rectangle" is a detection result.

### Classical detection: Haar cascades

Long before deep learning, OpenCV popularised **Haar cascades** for detecting specific, well-defined object types, most famously faces. A cascade is trained (by OpenCV's own creators, in advance) on thousands of positive examples (photos containing the object) and negative examples (photos without it), and OpenCV ships several ready-trained cascades:

```python
import cv2

face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + "haarcascade_frontalface_default.xml")

img = cv2.imread("photo.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

faces = face_cascade.detectMultiScale(gray, scaleFactor=1.1, minNeighbors=5)
print(faces)     # an array of (x, y, width, height) rectangles, one per detected face

for (x, y, w, h) in faces:
    cv2.rectangle(img, (x, y), (x + w, y + h), (0, 255, 0), 2)
```

`cv2.data.haarcascades` points to the folder of pre-trained cascade files that ship with OpenCV itself, so no separate download is needed for common ones like faces, eyes and smiles. On an image with no face at all, `detectMultiScale` correctly returns an empty result rather than a false positive, which is what we confirmed when running this exact code against one of this guideline's synthetic test images: it returned nothing, exactly as it should. `scaleFactor` controls how much the search shrinks the image at each scan (smaller values find more sizes of face, more slowly), and `minNeighbors` controls how many overlapping candidate detections must agree before a face is confirmed, trading sensitivity for false positives.

Haar cascades are fast and need no GPU, but they are narrow (a cascade trained on faces detects faces and nothing else) and considerably less accurate than the deep learning alternative on anything but well-lit, front-facing subjects.

### Deep learning detection

General-purpose object detection (find *any* of 80 or more everyday object categories, at any angle, in any lighting) is a deep learning problem, and three families dominate:

| Family | Idea | Trade-off |
|---|---|---|
| **YOLO** ("You Only Look Once") | looks at the whole image once and predicts all boxes and classes in a single pass | very fast, good for real-time video |
| **SSD** (Single Shot Detector) | similar single-pass idea, checking multiple locations and box shapes at once | fast, a common mobile-friendly choice |
| **Faster R-CNN** | first proposes candidate regions likely to contain an object, then classifies each candidate | typically more accurate, but slower |

OpenCV's `cv2.dnn` module can load a pre-trained detector saved in a standard format (ONNX, TensorFlow, Caffe) and run it, without needing TensorFlow or PyTorch installed at all:

```python
import cv2

net = cv2.dnn.readNetFromONNX("yolov5s.onnx")     # a pre-trained detector, downloaded separately

blob = cv2.dnn.blobFromImage(img, scalefactor=1/255.0, size=(640, 640), swapRB=True)
net.setInput(blob)
outputs = net.forward()
# outputs then needs decoding into boxes, class labels and confidence scores,
# following whatever output format the specific model uses
```

`cv2.dnn.blobFromImage` is `cv2.dnn`'s equivalent of the `preprocess_input` step from Section 3: it resizes the image, scales pixel values, and (with `swapRB=True`) fixes the BGR/RGB mismatch from Section 2 automatically. The exact shape of `outputs` and how to turn it into readable boxes differs by model, which is the main reason a higher-level library (such as `ultralytics`, which wraps YOLO directly) is usually more convenient than driving `cv2.dnn` by hand. The underlying pattern, though, is now a familiar one across this entire guideline: load pixels, preprocess them the way the specific model expects, run the network, decode the output.

---

## 6. Camera and Mapping

Every example so far has worked on a saved image. OpenCV's other great strength is working with a **live camera feed**, treating video as nothing more than a rapid sequence of images.

### Opening a camera

```python
import cv2

cap = cv2.VideoCapture(0)     # 0 is usually the default built-in webcam

if not cap.isOpened():
    print("Could not open the camera")
else:
    while True:
        ret, frame = cap.read()      # ret: whether a frame was successfully grabbed; frame: the image itself
        if not ret:
            break

        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)   # process it like any other image
        cv2.imshow("Camera", gray)

        if cv2.waitKey(1) & 0xFF == ord("q"):    # press 'q' to quit
            break

    cap.release()
    cv2.destroyAllWindows()
```

Checking `cap.isOpened()` is not optional caution; it is essential. Running this exact check with no camera attached (as in producing this guideline) correctly returned `False`, and a subsequent `cap.read()` correctly returned `(False, None)` rather than raising an error or hanging — OpenCV fails quietly, so your code must check for it explicitly. `cv2.VideoCapture` can also open a video *file* by passing its filename instead of a camera index, and every technique earlier in this guideline (edges, contours, detection) applies identically to each `frame` pulled from the loop, since a frame is simply an image like any other.

`cv2.waitKey(1)` waits 1 millisecond for a key press and is what keeps the display window responsive between frames; without a `waitKey` call in the loop, the image window will not refresh. `& 0xFF == ord("q")` is a standard idiom that checks whether the key pressed was "q", used to give the loop a clean way to exit.

### Mapping: warping an image onto a new plane

The second half of this section's title, **mapping**, refers to a different but related idea: **geometric mapping**, where you tell OpenCV "these four points in my image should map onto these four points in the output", and it works out how to warp everything else to match, a natural next step once you can capture live frames from an angle rather than straight-on.

This is exactly what document scanner apps do: you photograph a piece of paper at an angle, and the app "flattens" it as if photographed from directly above. `cv2.getPerspectiveTransform` computes the required transformation matrix from four corresponding points, and `cv2.warpPerspective` applies it:

```python
import numpy as np

pts_src = np.float32([[40, 40], [160, 40], [40, 160], [160, 160]])    # four corners in the original
pts_dst = np.float32([[0, 0], [200, 0], [0, 200], [200, 200]])        # where they should end up

matrix = cv2.getPerspectiveTransform(pts_src, pts_dst)
warped = cv2.warpPerspective(img, matrix, (200, 200))

print(matrix.round(2))
# [[  1.67  -0.   -66.67]
#  [  0.     1.67 -66.67]
#  [  0.     0.     1.  ]]
```

The result, `matrix`, is a 3×3 matrix that captures the entire perspective transformation implied by those four point pairs: in this case, mostly a uniform 1.67× scale-up. `warpPerspective` then applies it to every pixel in the image. This same pair of functions is the geometric foundation behind augmented reality overlays (mapping a virtual object onto a real, tilted surface), image stitching (mapping one photo's perspective to align with another's, once ORB from Section 2 has found matching points between them), and camera calibration, which corrects for a specific lens's own distortion using the same underlying idea, comparing where points actually land against where an undistorted image says they should.

---

## 7. Some Other Key Methods

A reference table of further OpenCV tools that come up often enough to be worth knowing by name, several of which were used quietly in earlier sections and deserve a proper introduction here.

| Function | Purpose |
|---|---|
| `cv2.inRange(img, lower, upper)` | builds a mask of every pixel whose value falls between `lower` and `upper` (often used on HSV images to isolate one colour) |
| `cv2.bitwise_and(img1, img2, mask=...)` | combines two images pixel by pixel using logical AND, commonly used to apply a mask and keep only the pixels it selects |
| `cv2.bitwise_or`, `cv2.bitwise_not`, `cv2.bitwise_xor` | the same idea, with OR, NOT and XOR |
| `cv2.addWeighted(img1, w1, img2, w2, gamma)` | blends two images together: `w1 * img1 + w2 * img2 + gamma` |
| `cv2.erode(img, kernel)` | shrinks bright regions in a binary image, useful for removing small noise specks |
| `cv2.dilate(img, kernel)` | grows bright regions in a binary image, useful for closing small gaps in a shape |
| `cv2.morphologyEx(img, op, kernel)` | applies a named combination of erode/dilate, such as `cv2.MORPH_OPEN` (erode then dilate: removes noise) or `cv2.MORPH_CLOSE` (dilate then erode: fills small holes) |
| `cv2.calcHist([img], [channel], mask, [bins], [range])` | computes a histogram of pixel values, used for the colour classification in Section 3 |
| `cv2.equalizeHist(gray_img)` | spreads out a washed-out image's brightness values to improve contrast |
| `cv2.matchTemplate(scene, template, method)` | slides a small template image across a larger one and scores how well it matches at every position |
| `cv2.minMaxLoc(result)` | finds the best (and worst) matching location from `matchTemplate`'s result |

Two of these deserve a quick worked example, because they show up constantly together as the classical way to isolate "everything of one colour":

```python
import cv2
import numpy as np

hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

lower_green = np.array([40, 50, 50])
upper_green = np.array([80, 255, 255])
mask = cv2.inRange(hsv, lower_green, upper_green)      # white where the pixel is green, black elsewhere

masked_result = cv2.bitwise_and(img, img, mask=mask)   # keep only the green pixels, blacken everything else
```

And `matchTemplate`, finding a known small patch inside a larger scene, which is a quick, training-free way to locate a specific known object (a company logo, a game icon, a specific playing card) when it always looks the same:

```python
result = cv2.matchTemplate(scene_gray, template_gray, cv2.TM_CCOEFF_NORMED)
min_val, max_val, min_loc, max_loc = cv2.minMaxLoc(result)

top_left = max_loc                                            # best match location, as (x, y)
h, w = template_gray.shape
bottom_right = (top_left[0] + w, top_left[1] + h)

print(round(max_val, 3), top_left)   # 1.0 (90, 80)
cv2.rectangle(scene_gray, top_left, bottom_right, 255, 2)
```

`max_val` close to 1.0 means an excellent match; `max_loc` gives its top-left corner. One caution worth stating plainly, because it is easy to trip over: `TM_CCOEFF_NORMED` compares *patterns of variation*, so matching against a perfectly flat, single-colour template produces meaningless results (there is no pattern to compare). Always test it against a template that has some internal contrast, such as a patch cropped directly out of a real photo.

---

*End of Guideline 42.*
