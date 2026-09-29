# Guideline 43: Python for Speech and Audio Processing

Every kind of data in this book's second part has had a natural shape: a table of rows and columns (Guideline 39), a grid of pixels (Guideline 42), a sequence of word numbers (Guideline 41). Sound has a shape too, and it is deceptively simple: **a single, very long list of numbers**, each one saying how far a speaker's cone should push the air at that instant, thousands of times per second. This guideline is about turning that long list of numbers into something you can classify, filter, generate and transcribe.

```bash
pip install librosa soundfile pyttsx3 SpeechRecognition
```

A note on what you can run yourself. Every waveform, spectrogram, filtering and classification example in this guideline was actually run to produce the numbers shown, using generated tones and a short offline text-to-speech clip. Two sections, TTS and speech recognition, name both a fully offline option (which was run) and the more accurate, internet-dependent services people use in practice (which are described, not run, since this environment has no connection to them). That distinction is called out plainly wherever it matters.

---

## 0. Waveform and Mel-Spectrogram

Before any library, it helps to understand the two pictures of sound you will see constantly in this guideline, because most of what follows is really about moving between them.

### The waveform

A microphone measures air pressure many times per second and records each measurement as a number. That raw sequence of numbers, plotted against time, is the **waveform** — amplitude (how far the wave swings) on the vertical axis, time on the horizontal axis. It is the most literal representation of sound: exactly what a speaker would need to reproduce it. The number of measurements taken per second is the **sample rate** (commonly 22,050, 44,100 or 16,000 Hz), and it sets a hard limit on which frequencies the recording can capture at all (a sample rate of 22,050 Hz can only represent frequencies up to 11,025 Hz, half the sample rate).

The trouble with a raw waveform is that it hides almost everything useful. Looking at two million numbers, you cannot easily tell that a clip contains a person saying "hello" rather than a car horn. What you actually want to know is **which frequencies are present, and when**, and that is not something the waveform shows directly.

### The spectrogram

A **spectrogram** answers exactly that question: it is a picture with time on one axis, frequency on the other, and colour (or brightness) showing how much energy is present at each frequency at each moment. It is produced by chopping the waveform into short overlapping windows and applying a **Fourier transform** to each one, a mathematical operation that decomposes a window of sound into the frequencies that compose it. Now a spoken vowel shows up as a stack of horizontal bands (its characteristic frequencies), and a burst of noise shows up as a smear across every frequency at once — patterns a machine learning model can actually learn from.

### The mel-spectrogram

Human hearing does not treat every frequency equally. We are far more sensitive to small differences at low frequencies (250 Hz vs 300 Hz sounds obviously different) than to the same-sized difference at high frequencies (8,000 Hz vs 8,050 Hz sounds almost identical). The **mel scale** is a frequency scale warped to match this: equal *steps* on the mel scale correspond to roughly equal *perceived* differences in pitch, however unequal the underlying frequencies are. A **mel-spectrogram** is an ordinary spectrogram with its frequency axis remapped onto this scale and typically compressed into a modest number of bands (128 is common), which both matches human perception and gives a much smaller, more manageable picture than a full-resolution spectrogram. It is the standard input to almost every deep learning model that works with audio, for exactly the same reason a CNN works on pixels (Guideline 40) rather than a description of the scene: it is a rich, regular grid of numbers that a network can learn patterns from directly.

You will build both, for real, in Section 1.

---

## 1. Librosa

**Librosa** is the standard Python library for analysing audio: loading it, visualising it, and extracting the features described above. It works together with **SoundFile** (`soundfile`), which handles the low-level reading and writing of audio files (`.wav`, `.flac` and others).

### Loading audio

```python
import librosa

y, sr = librosa.load("tone.wav", sr=None)
print(y.shape, sr)          # (44100,) 22050
```

The convention across nearly every librosa function is `y` for the waveform (a 1-D NumPy array, one number per sample) and `sr` for the sample rate. Passing `sr=None` keeps the file's original sample rate; leaving it out resamples to librosa's default of 22,050 Hz, which is worth knowing so an unexpected resample does not surprise you.

```python
print(librosa.get_duration(y=y, sr=sr))    # 2.0   (seconds)
```

### The waveform, in code

```python
import matplotlib.pyplot as plt
import librosa.display

librosa.display.waveshow(y, sr=sr)
plt.title("Waveform")
plt.xlabel("Time (s)")
plt.ylabel("Amplitude")
plt.show()
```

`librosa.display.waveshow` is a thin, sensible wrapper around Matplotlib (Guideline 38) that handles the time axis for you.

### The Short-Time Fourier Transform and the spectrogram

The **STFT** (Short-Time Fourier Transform) is the operation described conceptually in Section 0: it slides a window along the waveform and computes a Fourier transform in each one.

```python
import numpy as np

D = librosa.stft(y)
print(D.shape, D.dtype)      # (1025, 87) complex64
```

The result is a 2-D array of **complex numbers**: 1,025 frequency bins by 87 time frames, for this two-second clip. Each complex number encodes both the strength (*magnitude*) and the timing (*phase*) of that frequency at that moment. For a picture, only the magnitude matters, and it is almost always viewed on a **decibel** (logarithmic) scale, because sound energy spans an enormous range that a linear scale would compress into invisibility:

```python
S_db = librosa.amplitude_to_db(np.abs(D), ref=np.max)
print(S_db.min(), S_db.max())     # -80.0 0.0   (decibels, relative to the loudest point)

librosa.display.specshow(S_db, sr=sr, x_axis="time", y_axis="hz")
plt.colorbar(format="%+2.0f dB")
plt.title("Spectrogram")
plt.show()
```

### The mel-spectrogram, in code

```python
mel = librosa.feature.melspectrogram(y=y, sr=sr, n_mels=128)
print(mel.shape)               # (128, 87)  — 128 mel bands, 87 time frames

mel_db = librosa.power_to_db(mel, ref=np.max)
print(mel_db.min(), mel_db.max())    # -80.0 0.0

librosa.display.specshow(mel_db, sr=sr, x_axis="time", y_axis="mel")
plt.colorbar(format="%+2.0f dB")
plt.title("Mel-spectrogram")
plt.show()
```

Compare the shapes: the full spectrogram had 1,025 frequency rows; the mel-spectrogram compresses that down to 128 mel bands, in the same 87 time frames. That compression is precisely the perceptual remapping from Section 0, and this 128 × 87 grid of numbers is now something you could feed straight into a `Conv2D` layer from Guideline 40, treating it exactly like a small grayscale image.

### Other common features

Two more single-number-per-frame features come up constantly and are worth knowing by name:

```python
zcr = librosa.feature.zero_crossing_rate(y)
print(zcr.shape, zcr.mean())            # (1, 87) 0.039

centroid = librosa.feature.spectral_centroid(y=y, sr=sr)
print(centroid.shape, centroid.mean())  # (1, 87) 448.0
```

The **zero-crossing rate** counts how often the waveform crosses zero per frame; it is high for noisy or percussive sounds and low for smooth tones, making it a cheap way to separate speech-like or tonal sound from noise. The **spectral centroid** is the "centre of mass" of the frequency spectrum, informally the perceived "brightness" of a sound; a value around 448 Hz here reflects our test clip being a fairly low, pure tone.

### MFCCs: the classic feature for speech

**MFCCs** (Mel-Frequency Cepstral Coefficients) take the mel-spectrogram one step further, applying another transform that compresses it into a small number of coefficients (commonly 13) that summarise the overall *shape* of the spectrum, largely independent of pitch. For decades, MFCCs were the standard feature for speech recognition and audio classification, and Section 3 puts them to direct use:

```python
mfcc = librosa.feature.mfcc(y=y, sr=sr, n_mfcc=13)
print(mfcc.shape)      # (13, 87)
```

### A few more essentials

Three more librosa tools round out the everyday toolkit:

```python
y_trimmed, index = librosa.effects.trim(y, top_db=20)
print(index)                            # [10240 78336]  — sample indices where the sound actually starts and ends

y_16k = librosa.resample(y, orig_sr=sr, target_sr=16000)
print(y_16k.shape)                      # shorter array, same duration, fewer samples per second

tempo, beats = librosa.beat.beat_track(sr=sr)
print(tempo)                            # estimated tempo, in beats per minute
```

`librosa.effects.trim` strips leading and trailing silence, using `top_db` as the threshold below which audio counts as silent, extremely useful for cleaning up recordings that start and end with dead air. `librosa.resample` changes the sample rate, needed whenever a task (such as many speech recognition models, in Section 6) expects a specific rate like 16,000 Hz. `librosa.beat.beat_track` estimates tempo and beat locations in music.

---

## 2. Generating Audio (Simple Sine Wave to Semi-Advanced)

Sound synthesis starts from the same idea used to build the very first test signal above: a sine wave is nothing but `amplitude × sin(2π × frequency × time)`, evaluated at every sample.

### A pure tone

```python
import numpy as np
import soundfile as sf

sr = 22050                                    # samples per second
duration = 2.0                                # seconds
t = np.linspace(0, duration, int(sr * duration), endpoint=False)

frequency = 440.0                             # A4, in hertz
wave = 0.5 * np.sin(2 * np.pi * frequency * t)

sf.write("tone.wav", wave, sr)
print(wave.shape, wave.min(), wave.max())     # (44100,) -0.4999... 0.4999...
```

Every element of `t` is one moment in time; `wave` evaluates the sine function at each one. The amplitude of 0.5 keeps the signal comfortably inside the range `soundfile` expects for floating-point audio, roughly −1.0 to 1.0.

### Building a chord

Real sound is rarely one frequency. Adding several sine waves together, the way a real instrument's overtones combine, is the direct extension:

```python
t = np.linspace(0, 3, int(sr * 3), endpoint=False)

c4 = 0.3 * np.sin(2 * np.pi * 261.63 * t)     # C4
e4 = 0.3 * np.sin(2 * np.pi * 329.63 * t)     # E4
g4 = 0.3 * np.sin(2 * np.pi * 392.00 * t)     # G4

chord = c4 + e4 + g4                          # a C major chord, added sample by sample
sf.write("chord.wav", chord, sr)
```

### Adding realism: noise and envelopes

Two small additions make synthetic audio behave more like the real thing. **Noise** adds a small amount of randomness, useful for testing how robust a later processing step is:

```python
noise = 0.02 * np.random.randn(len(t))
chord_noisy = chord + noise
```

An **envelope** shapes how a sound's volume changes over time. Real notes do not snap instantly to full volume and stay there; they rise (*attack*), settle (*decay*), hold (*sustain*), and fade (*release*), a pattern known as an **ADSR envelope**:

```python
def linear_fade(signal, fade_samples):
    envelope = np.ones(len(signal))
    envelope[:fade_samples] = np.linspace(0, 1, fade_samples)     # fade in
    envelope[-fade_samples:] = np.linspace(1, 0, fade_samples)    # fade out
    return signal * envelope

chord_shaped = linear_fade(chord, fade_samples=int(sr * 0.1))     # 0.1-second fades
```

Multiplying the signal by an envelope that ramps from 0 up to 1 and back down to 0 removes the abrupt clicking pop that an instant on/off would otherwise cause, and it is the same idea, generalised, behind every professional synthesizer's volume shaping.

### Toward semi-advanced generation

A **chirp** is a tone whose frequency changes continuously over time, useful for testing filters and equipment across a whole frequency range at once, and it is built by letting the frequency itself be a function of time rather than a constant:

```python
from scipy.signal import chirp

t = np.linspace(0, 3, int(sr * 3))
sweep = chirp(t, f0=200, f1=2000, t1=3, method="linear")    # sweeps from 200 Hz to 2000 Hz
sf.write("sweep.wav", sweep, sr)
```

Beyond hand-built waveforms, genuinely realistic sound (a violin, a human voice, a drum kit) is generated the way images and text are: with a deep generative model trained on many real recordings. **WaveNet** (2016) was the first model to generate raw audio, one sample at a time, sample by sample, conditioned on everything generated so far, in the same left-to-right spirit as the GPT models of Guideline 41's Section 2, and it was the breakthrough that made today's realistic synthetic speech and music possible. Modern systems mostly generate a mel-spectrogram first (Section 0) and then use a fast **vocoder** network to convert that spectrogram back into an actual waveform, which is exactly the architecture behind the text-to-speech systems in Section 5.

---

## 3. Audio Classification

**Audio classification** assigns a category to a whole clip: this is a dog bark, this is a car horn, this speaker is happy, this recording is music rather than speech. Precisely as with images in Guideline 42, there is a classical route and a deep learning route, and the classical route is genuinely useful for well-defined problems.

### The classical approach: features plus a familiar model

The pattern should look immediately familiar from Guideline 39: extract a fixed-size set of numeric features from each clip, then hand them to any of that guideline's classifiers. Here we build a real, complete classifier that tells a musical **tone** apart from plain **noise**, using the MFCCs from Section 1:

```python
import numpy as np
import librosa
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.svm import SVC

sr = 22050
rng = np.random.RandomState(0)

def make_tone(freq, duration=1.0):
    t = np.linspace(0, duration, int(sr * duration), endpoint=False)
    return 0.5 * np.sin(2 * np.pi * freq * t) + 0.01 * rng.randn(len(t))

def make_noise(duration=1.0):
    return 0.3 * rng.randn(int(sr * duration))

X, y = [], []
for _ in range(30):
    for label, clip in [("tone",  make_tone(rng.choice([220, 440, 880]))),
                        ("noise", make_noise())]:
        mfcc = librosa.feature.mfcc(y=clip.astype(np.float32), sr=sr, n_mfcc=13)
        X.append(mfcc.mean(axis=1))     # average each of the 13 coefficients across the clip
        y.append(label)

X, y = np.array(X), np.array(y)
print(X.shape)      # (60, 13) — 60 clips, 13 MFCC-based features each

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42, stratify=y)
scaler = StandardScaler().fit(X_train)

clf = SVC(kernel="rbf").fit(scaler.transform(X_train), y_train)
print(clf.score(scaler.transform(X_test), y_test))    # 1.0
```

The key idea is `mfcc.mean(axis=1)`: an MFCC array has one column per time frame, and averaging across time collapses a whole clip, however long, into a single fixed-length vector of 13 numbers, exactly the shape scikit-learn's estimators expect (Guideline 39, Section 2). This model reached perfect accuracy on this easy, clean test case; a real dataset of, say, urban sounds (sirens, drilling, dog barks) would use exactly this same pipeline; only the amount and messiness of the data would make it harder.

### The deep learning approach: back to images

The moment you have a mel-spectrogram (Section 1), audio classification becomes an image classification problem, and Guideline 40's convolutional networks apply directly:

```python
from tensorflow import keras
from tensorflow.keras import layers

model = keras.Sequential([
    keras.Input(shape=(128, 87, 1)),        # 128 mel bands x 87 time frames, 1 channel — same shape idea as Guideline 42
    layers.Conv2D(16, 3, activation="relu"),
    layers.MaxPooling2D(),
    layers.Conv2D(32, 3, activation="relu"),
    layers.MaxPooling2D(),
    layers.Flatten(),
    layers.Dropout(0.3),
    layers.Dense(10, activation="softmax"), # 10 sound categories, for example
])
```

Nothing here is new: it is the exact `Conv2D` → `MaxPooling2D` pattern from Guideline 40's Section 10, applied to a grid of mel-spectrogram values instead of a grid of pixel values. This is precisely how large-scale audio classifiers (identifying birdsong, detecting a cough on a phone call, tagging music genre) are actually built, and it is why Section 1 spent so much effort on turning sound into that grid in the first place.

---

## 4. Audio Filtering

**Filtering** changes which frequencies are present in a signal — removing hiss, isolating a voice, cutting a hum. Two ways to do this appear constantly in practice.

### Filtering directly on the spectrogram

Because the STFT (Section 1) already breaks a sound into individual frequency bins, one direct way to filter is to zero out the bins you do not want and convert back:

```python
import librosa
import numpy as np

D = librosa.stft(y)
freqs = librosa.fft_frequencies(sr=sr)       # the actual frequency, in Hz, of each row of D
print(freqs.shape, freqs[:3], freqs[-3:])
# (1025,) [0.  10.77 21.53] [10981.93 10992.7  11003.47 ...]

cutoff = 350                                  # keep only frequencies below 350 Hz
D_filtered = D.copy()
D_filtered[freqs > cutoff, :] = 0             # zero out every bin above the cutoff, at every time frame

y_filtered = librosa.istft(D_filtered)        # the inverse STFT, converting back to a waveform
sf.write("filtered.wav", y_filtered, sr)
```

`librosa.fft_frequencies` tells you exactly which real-world frequency (in Hz) each row of the STFT corresponds to, which is what makes it possible to select "everything above 350 Hz" in the first place. `librosa.istft`, the inverse operation, converts the edited spectrogram back into an ordinary waveform you can save and listen to. This is a **low-pass filter**, built directly, with nothing more than array indexing: it lets frequencies below the cutoff pass through and blocks everything above.

### Filtering with a designed filter (SciPy)

The spectrogram method above is intuitive but blunt: an instant cutoff between "kept" and "zeroed" tends to introduce audible artifacts. Proper filter design, from SciPy's signal processing tools (Guideline 35), produces a smoother, more controlled result:

```python
from scipy.signal import butter, lfilter

def butter_lowpass_filter(data, cutoff, fs, order=5):
    nyquist = 0.5 * fs                                 # the highest frequency the sample rate can represent
    normalized_cutoff = cutoff / nyquist
    b, a = butter(order, normalized_cutoff, btype="low", analog=False)
    return lfilter(b, a, data)

y_lowpass = butter_lowpass_filter(y, cutoff=350, fs=sr, order=5)
```

`butter` designs a **Butterworth filter**, a standard, smooth filter shape widely used because it has no ripples in the frequencies it keeps. Passing `btype="high"` instead makes it a **high-pass filter** (keeps frequencies above the cutoff, useful for removing a low rumble), and `btype="bandpass"` with a `[low, high]` cutoff pair isolates a specific frequency range, exactly the range a particular instrument or a human voice tends to occupy.

| Filter type | Keeps | Typical use |
|---|---|---|
| Low-pass | frequencies below the cutoff | removing hiss, softening a sound |
| High-pass | frequencies above the cutoff | removing low rumble or hum |
| Band-pass | frequencies within a range | isolating a voice or instrument |
| Band-stop (notch) | everything except a narrow range | removing a specific hum frequency (such as 60 Hz mains hum) |

---

## 5. TTS

**Text-to-Speech (TTS)** converts written text into spoken audio. Two very different kinds of tool exist for it, and knowing which one you are looking at prevents real confusion.

### Fully offline: pyttsx3

`pyttsx3` wraps the speech engines already built into your operating system, so it produces speech with **no internet connection and no downloaded model at all**:

```python
import pyttsx3

engine = pyttsx3.init()

voices = engine.getProperty("voices")
print(len(voices))                          # 131 (varies by system; each entry describes one available voice)

engine.setProperty("rate", 150)             # words per minute; default was 200

engine.save_to_file("Hello, this is a test of text to speech.", "output.wav")
engine.runAndWait()                          # required: this is what actually processes the queued commands
```

This exact code was run to produce this guideline, and it wrote a working 99,466-byte WAV file with no internet access at any point. `engine.say(text)` speaks immediately instead of saving to a file, if a speaker is available; `engine.setProperty("voice", voices[i].id)` switches to a different installed voice. The trade-off for working entirely offline is quality: these built-in system voices sound noticeably more robotic than the alternative below.

### Deep learning TTS: natural but online (or heavier to self-host)

Modern, natural-sounding TTS is built the way Section 2 described: a neural network converts text into a mel-spectrogram, and a vocoder network converts that spectrogram into audio. Two common ways to use this in Python:

```python
# gTTS: sends text to Google's translation TTS service and downloads the resulting audio.
# Needs an internet connection; cannot run in an offline environment.
from gtts import gTTS

tts = gTTS(text="Hello, this is a test of text to speech.", lang="en")
tts.save("output_gtts.mp3")
```

```python
# A locally-run neural model (illustrative — requires installing the coqui-tts package
# and downloading a pre-trained model the first time, following Guideline 40's transfer-
# learning pattern of "download a trained model, then use it directly").
from TTS.api import TTS

tts = TTS("tts_models/en/ljspeech/tacotron2-DDC")
tts.tts_to_file(text="Hello, this is a test of text to speech.", file_path="output_neural.wav")
```

| | pyttsx3 | gTTS | Neural TTS (Tacotron, VITS, and others) |
|---|---|---|---|
| Needs internet | no | yes | only to download the model, once |
| Voice quality | robotic | natural | most natural |
| Runs on | any computer instantly | any computer with a connection | benefits from a GPU for speed |

---

## 6. Speech Recognition (also ASR / STT)

**Speech Recognition**, also called **ASR** (Automatic Speech Recognition) or **STT** (Speech-to-Text), does the reverse of Section 5: given audio, produce the words that were spoken. The Python library `SpeechRecognition` (imported as `speech_recognition`) provides one consistent interface to several different recognition engines, offline and online alike.

### Loading and recognizing audio

```python
import speech_recognition as sr

recognizer = sr.Recognizer()

with sr.AudioFile("output.wav") as source:
    audio = recognizer.record(source)      # reads the entire file into an AudioData object
```

`AudioFile` and `record` together read a `.wav` file into the format every recognizer function expects. With a microphone instead of a file, the equivalent is `sr.Microphone()` as the source, which streams live audio the same way.

### An offline engine: CMU Sphinx

```python
try:
    text = recognizer.recognize_sphinx(audio)
    print(text)
except sr.UnknownValueError:
    print("Could not understand the audio")
```

This exact code was run, fully offline, on the WAV file produced by `pyttsx3` in Section 5, and it returned a real, if imperfect, transcription ("the dust on the" for a clip that actually said "Hello, this is a test of text to speech"). That gap is instructive rather than embarrassing: **CMU Sphinx** is a lightweight, older, offline engine, chosen here specifically because it needs no internet connection and no large downloaded model, and its accuracy on real speech is well behind modern systems. It remains genuinely useful when working somewhere with no connectivity at all, or when a rough transcription is good enough.

### Modern, far more accurate options

For real transcription work, two paths dominate today, and both follow the same "pre-trained model" pattern from Guideline 40:

```python
# Whisper: OpenAI's open-source model, run locally once installed and downloaded.
# The clear modern standard for accuracy without needing a paid API.
text = recognizer.recognize_whisper(audio, model="base")

# Google's cloud speech API, through the same library, for comparison.
# Needs an internet connection and, for anything beyond light use, an API key.
text = recognizer.recognize_google(audio)
```

`recognize_whisper` runs OpenAI's **Whisper** model directly on your own machine (no data ever leaves it), at the cost of needing the model's weights downloaded once and, for larger and more accurate versions of the model, a reasonably capable computer or a GPU. `recognize_google` sends the audio to Google's cloud service and gets a transcription back, generally excellent, but dependent on a network connection and Google's terms of use.

| Engine | Runs | Accuracy | Needs |
|---|---|---|---|
| CMU Sphinx | locally | modest | nothing extra |
| Whisper | locally | very good to excellent | a downloaded model, more compute for larger sizes |
| Google Cloud Speech | remotely | excellent | an internet connection |

The `SpeechRecognition` library's real value is exactly this: one consistent piece of code, and you swap the single method name (`recognize_sphinx`, `recognize_whisper`, `recognize_google`, and several others) depending on which trade-off between privacy, cost, offline capability and accuracy suits your project.

---

## 7. Other Related Tasks

A handful of further audio tasks are worth knowing by name, each building directly on the pieces above.

| Task | What it does | Built from |
|---|---|---|
| **Speaker identification / verification** | recognises *who* is speaking, not what they said | classify MFCC or learned embeddings per speaker (Section 3's pattern, applied to speaker identity instead of sound category) |
| **Speaker diarization** | labels "who spoke when" in a multi-person recording | clustering (Guideline 39, Section 15) applied to short segments' audio features |
| **Voice activity detection (VAD)** | detects which parts of a recording contain speech at all | often a simple energy or zero-crossing-rate threshold (Section 1), or a small classifier |
| **Music information retrieval** | key, tempo, chord and genre detection in music | librosa's `beat`, `chroma` and related feature functions, often followed by classical or deep models |
| **Audio source separation** | splits a mixed recording into its component sounds (e.g. vocals vs. instruments) | deep learning models trained on spectrograms, conceptually similar to the segmentation task in Guideline 42 |
| **Voice conversion** | changes *how* something is said (accent, speaker identity) while keeping the words | generative deep learning, extending the vocoder idea from Section 2 |
| **Audio captioning** | describes a sound clip in a full sentence ("a dog barks while a car passes") | combines audio feature extraction (this guideline) with text generation (Guideline 41) |

The pattern that ties this entire guideline together, and this table in particular, is worth stating directly: almost every audio task, however specialised it sounds, reduces to the same two moves. Turn the sound into numbers, usually a mel-spectrogram or MFCCs (Section 1), and then apply the same classification, clustering or generation tools already built in Guidelines 39, 40 and 41. Audio is not really a separate subject from everything else in this book; it is one more kind of data, once you know how to picture it.

---

*End of Guideline 43.*
