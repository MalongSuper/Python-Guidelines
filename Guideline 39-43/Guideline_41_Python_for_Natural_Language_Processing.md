# Guideline 41: Python for Natural Language Processing

Every model in Guidelines 39 and 40 wanted numbers. A spreadsheet of measurements, a grid of pixels, even the movie reviews of Guideline 40 had to be turned into lists of word numbers before a network would touch them. But most of what humans write and say is not a spreadsheet. It is a sentence, and a sentence is a strange kind of data: it has grammar, ambiguity, tone, and meaning that depends on the words around it.

**Natural Language Processing (NLP)** is the field concerned with getting a computer to work with human language: to read it, to understand something about it, and sometimes to produce it. This guideline is shorter than the last two, but it still has real ground to cover: how language becomes numbers, the architecture that changed the field overnight, the tasks NLP is actually used for, and three Python libraries you will meet in almost every NLP project.

A note on what you can run yourself. **NLTK** and **spaCy** work entirely offline once their data is downloaded, and every example in Sections 4 and 5 was run to produce the output shown. **HuggingFace** models (Section 6) are downloaded from the internet the first time you use them, so if you are working somewhere offline, treat those examples as a guide to what the code produces rather than something to run immediately. Their labels and structure are exact; scores are marked with ≈ because they depend on the exact model checkpoint you download and, occasionally, its version.

```bash
pip install nltk spacy transformers
```

---

## 1. Natural Language Processing

### Why language is hard

A table of numbers has one meaning. A sentence often has several. Consider:

> "I saw the man on the hill with a telescope."

Who has the telescope, the speaker or the man? English does not say. Or consider a single word:

> "bank"

A place to keep money, or the edge of a river? A human resolves both instantly using context that never appears explicitly in the sentence. This is **ambiguity**, and it is the reason NLP is a genuinely difficult problem rather than a slightly harder version of the tables and images from Guidelines 39 and 40. Language also has **structure** (grammar dictates which orderings of words are valid and which are not) and **compositionality** (the meaning of a sentence is built from the meaning of its parts, but not always simply). A working NLP system needs to deal with all three.

### The general pipeline

Whatever the final task, most NLP work moves through the same broad stages:

1. **Tokenize** — break text into pieces (usually words or subwords).
2. **Clean and normalise** — lowercase it, remove punctuation, reduce words to a base form.
3. **Represent** — turn the tokens into numbers.
4. **Model** — feed the numbers to an algorithm that performs the task.

You already met the oldest way to do step 3 in Guideline 39: `CountVectorizer` turned sentences into columns of word counts (a **bag of words**), and `MultinomialNB` classified them as spam or not. A refinement of that idea, **TF-IDF** (term frequency–inverse document frequency, available in scikit-learn as `TfidfVectorizer`), weighs a word by how often it appears in a document *and* how rare it is across all documents, so that common words like "the" count for less than distinctive ones like "refund". Both approaches share the same blind spot: they treat "great" and "excellent" as two completely unrelated columns, with no notion that they mean similar things. Section 2 is about how deep learning fixed that.

---

## 2. Deep Learning for NLP

### Embeddings: giving words a meaningful address

You met **embeddings** briefly in Guideline 40's recurrent-network section: a `layers.Embedding` turned each word number into a short vector, so the network could learn its own numeric representation of every word during training. That idea, generalised, is the foundation of every modern NLP system.

An embedding places every word at a point in space (typically 50 to a few hundred dimensions) such that words with similar meaning end up near each other. Trained on enough text, the geometry becomes strikingly sensible: "king" sits close to "queen" and "man" close to "woman", and the *direction* from "man" to "woman" is roughly the same as the direction from "king" to "queen", which is often demonstrated with the arithmetic:

```
vector("king") − vector("man") + vector("woman") ≈ vector("queen")
```

The earliest widely used embedding methods, **Word2Vec** and **GloVe**, learned one fixed vector per word from huge amounts of text, before any specific task existed. Those vectors could then be dropped into any model, the same way you would reuse a pre-trained image network in Guideline 40's transfer learning. Their limitation is that each word gets exactly one vector: "bank" gets a single point in space, blending the riverbank and the money sense together, because the vector does not know which sentence it is in.

### Seq2Seq: mapping a sequence to a sequence

Many NLP tasks are not classification at all: translation takes a sentence and produces a *different* sentence, and summarisation takes a long document and produces a short one. The **sequence-to-sequence (Seq2Seq)** architecture, introduced in 2014, handles exactly this. It pairs two recurrent networks from Guideline 40:

- An **encoder** reads the entire input sequence, one token at a time, and compresses everything it has read into a single vector, its final hidden state.
- A **decoder** takes that vector and generates the output sequence, one token at a time, feeding each token it produces back in as the input for the next step.

The weakness is the encoder's single vector: for a long sentence, cramming everything into one fixed-size summary loses detail, the same long-range memory problem that motivated LSTM and GRU. The fix, called **attention**, let the decoder look back at *every* step of the encoder's output when producing each word, rather than relying on one compressed summary. Attention was the single most consequential idea that led to the next architecture.

### Transformers: attention is all you need

In 2017, a paper literally titled "Attention Is All You Need" removed the recurrent networks entirely and kept only the attention mechanism. The result was the **Transformer**, the architecture behind essentially every major language model since, including the one answering you right now.

Two properties explain why transformers won:

- **Self-attention.** For every word in a sentence, the model directly computes how much attention to pay to *every other word* when building that word's representation. In "the animal didn't cross the street because **it** was too tired", self-attention lets the model connect "it" directly back to "animal", regardless of the distance between them, because nothing has to be carried step by step through a chain the way an RNN's hidden state does.
- **Parallelism.** An RNN must process word 1, then word 2, then word 3, in strict order, because each step depends on the last. A transformer looks at every word in the sentence at the same time. Since a self-attention layer has no built-in sense of order, the model separately adds a **positional encoding** to each word's embedding, a pattern that tells the model where in the sentence each word sits. This parallelism is what let transformers scale up to the enormous sizes that make today's language models possible, trained on hardware that can process many words at once.

### BERT and the pre-train, fine-tune pattern

A full transformer, as in the original paper, has an encoder half (for understanding input) and a decoder half (for generating output), which is the natural shape for translation. Two influential models split that pair apart and specialised.

**BERT** (Bidirectional Encoder Representations from Transformers, 2018) keeps only the encoder half. It is trained by a clever self-supervised trick: take ordinary text, hide about 15% of the words, and ask the model to guess the hidden words from the words on *both* sides, left and right at once — the "bidirectional" in its name. (An RNN reading left to right can never do this naturally.) Having learned to do that on a vast amount of raw text with no manual labels at all, BERT has built up a deep, general understanding of language. It is then **fine-tuned**, exactly as in Guideline 40's transfer learning: keep BERT's learned weights, add a small new head, and retrain briefly on your labelled data for a specific task such as classification or question answering.

**GPT** (Generative Pre-trained Transformer) keeps only the decoder half, and is trained the more old-fashioned way: given the words so far, predict the *next* word, over and over, across enormous amounts of text. This left-to-right, next-word design is what makes GPT-style models naturally suited to *generating* text, one word at a time, which is exactly what the large language models behind today's chatbots do at their core.

A few other names worth recognising, briefly:

| Model | Idea |
|---|---|
| **T5** | frames every NLP task (translation, summarisation, classification) as "input text in, output text in" |
| **RoBERTa** | BERT, retrained more carefully and on more data |
| **DistilBERT** | a smaller, faster, distilled copy of BERT that keeps most of its accuracy |
| **GPT-2 / GPT-3 / GPT-4 and successors** | progressively larger GPT-style models; the lineage behind modern chatbots |

The practical upshot for you as a Python programmer: you will almost never train a transformer from scratch. You will download a pre-trained one and fine-tune or directly use it, which is exactly what Section 6 shows you how to do.

---

## 3. Common Tasks in NLP

NLP is not one task but a family of them, and it helps to have a map before you start using libraries that name these tasks directly.

| Task | What it does | Example |
|---|---|---|
| **Text classification** | assign a category to a whole piece of text | is this email spam? Is this review positive or negative? |
| **Named Entity Recognition (NER)** | find and label real-world entities in text | "**Tim Cook** flew to **London**" → Tim Cook = person, London = place |
| **Question Answering (QA)** | given a passage and a question, extract or generate the answer | passage about the solar system + "how many moons does Mars have?" → "two" |
| **Text generation** | produce new, plausible text continuing a prompt | autocomplete, chatbots, story writing |
| **Summarisation** | compress a long document into a short one that keeps the key points | a news article reduced to two sentences |
| **Translation** | rewrite text in another language | English to French |
| **Part-of-speech (POS) tagging** | label each word with its grammatical role | "dog" → noun, "runs" → verb |
| **Sentiment analysis** | a specific, very common case of text classification | is this tweet positive, negative or neutral? |

Some of these are old and solved well by classical methods (POS tagging has worked reliably for decades); others (open-ended generation, high-quality translation) only became genuinely good once transformers arrived. The libraries in the rest of this guideline sit at different points along that line: NLTK leans classical, spaCy is a fast, practical middle ground, and HuggingFace gives direct access to state-of-the-art transformer models.

---

## 4. NLTK

**NLTK** (Natural Language Toolkit) is the oldest and most academic of the three libraries, built originally for teaching and researching linguistics. It exposes the building blocks of NLP individually, which makes it an excellent place to see each step of Section 1's pipeline in isolation.

```python
import nltk

nltk.download("punkt_tab")                          # tokenizer data
nltk.download("stopwords")                           # common-word lists
nltk.download("wordnet")                              # for lemmatization
nltk.download("averaged_perceptron_tagger_eng")       # for POS tagging
nltk.download("maxent_ne_chunker_tab")                # for named entity chunking
nltk.download("words")
```

NLTK ships its algorithms but not its data; `nltk.download(...)` fetches the specific resource each tool needs, once, and caches it locally. Forgetting the right download is the most common error when starting with NLTK, and the error message will name the missing resource.

### Tokenization

**Tokenizing** splits raw text into pieces: sentences, or words.

```python
from nltk.tokenize import sent_tokenize, word_tokenize

text = "Dr. Smith went to Washington. He works at NASA, and he loves NLP!"

print(sent_tokenize(text))
# ['Dr. Smith went to Washington.', 'He works at NASA, and he loves NLP!']

print(word_tokenize(text))
# ['Dr.', 'Smith', 'went', 'to', 'Washington', '.', 'He', 'works', 'at', 'NASA', ',', 'and', 'he', 'loves', 'NLP', '!']
```

Notice two things a naive `text.split()` would get wrong: `sent_tokenize` correctly did not treat "Dr." as the end of a sentence, and `word_tokenize` separated the punctuation ("," and "!") from the words next to it, so that "NASA," is not treated as a different token from "NASA".

### Removing stopwords

**Stopwords** are extremely common words ("the", "is", "a", "and") that carry little meaning on their own and are often removed before analysis, particularly for the bag-of-words methods of Section 1:

```python
from nltk.corpus import stopwords

stop_words = set(stopwords.words("english"))
words = word_tokenize("This is a simple example showing how stopwords are removed")

filtered = [w for w in words if w.lower() not in stop_words]
print(filtered)
# ['simple', 'example', 'showing', 'stopwords', 'removed']
```

Six words vanished ("This", "is", "a", "how", "are"), leaving the words that actually carry the sentence's content. Whether to remove stopwords depends on the task: for a bag-of-words classifier it usually helps, but for a transformer-based model (Section 6) it usually hurts, since transformers use those small words to understand grammar and relationships.

### Stemming and lemmatization

Both **stemming** and **lemmatization** reduce a word to a base form, so that "run", "running" and "ran" can be treated as related, but they do it very differently.

**Stemming** chops off endings using fast, mechanical rules, without knowing any actual grammar:

```python
from nltk.stem import PorterStemmer

ps = PorterStemmer()
words = ["running", "flies", "easily", "better", "studies"]
print([ps.stem(w) for w in words])
# ['run', 'fli', 'easili', 'better', 'studi']
```

Look closely: "flies" became "fli" and "easily" became "easili", neither of which is an actual English word. Stemming is fast but crude.

**Lemmatization** looks a word up against real vocabulary (WordNet, in NLTK's case) and returns its proper dictionary form, the **lemma**:

```python
from nltk.stem import WordNetLemmatizer

wl = WordNetLemmatizer()
print([wl.lemmatize(w) for w in words])
# ['running', 'fly', 'easily', 'better', 'study']
```

"Flies" correctly becomes "fly" and "studies" becomes "study", but "running" stayed as "running". That is because, without being told otherwise, the lemmatizer assumes every word is a noun, and "running" is already a valid noun (as in "running is good exercise"). Tell it the word is a verb, and it lemmatizes correctly:

```python
print([wl.lemmatize(w, pos="v") for w in ["running", "flies", "studies"]])
# ['run', 'fly', 'study']
```

This is a genuine practical annoyance: lemmatization is more accurate than stemming, but only if you supply the correct part of speech, which brings us to the next tool.

### Part-of-speech tagging

`pos_tag` labels each token with its grammatical role, using NLTK's own abbreviations (from the Penn Treebank tag set):

```python
tokens = word_tokenize("The quick brown fox jumps over the lazy dog")
print(nltk.pos_tag(tokens))
# [('The', 'DT'), ('quick', 'JJ'), ('brown', 'NN'), ('fox', 'NN'), ('jumps', 'VBZ'),
#  ('over', 'IN'), ('the', 'DT'), ('lazy', 'JJ'), ('dog', 'NN')]
```

`DT` is a determiner, `JJ` an adjective, `NN` a singular noun, `VBZ` a third-person-singular present verb, and `IN` a preposition. Notice that "brown" and "fox" were both tagged `NN` (noun) here, an example of the tagger occasionally getting a modifier wrong when context is thin; POS tagging is very reliable but not perfect.

### Named Entity Recognition

`ne_chunk` groups tagged tokens into named entities such as people, organisations and places:

```python
sentence = "Barack Obama was born in Hawaii and worked with Google"
tagged = nltk.pos_tag(word_tokenize(sentence))
tree = nltk.ne_chunk(tagged)
print(tree)
# (S
#   (PERSON Barack/NNP)
#   (PERSON Obama/NNP)
#   was/VBD
#   born/VBN
#   in/IN
#   (GPE Hawaii/NNP)
#   and/CC
#   worked/VBD
#   with/IN
#   (PERSON Google/NNP))
```

"Barack" and "Obama" were correctly tagged as parts of a person, and "Hawaii" as a **GPE** (geopolitical entity). But look at the last line: "Google" was labelled **PERSON**, which is wrong; it is a company. This is a genuine and instructive limitation. NLTK's built-in NER is a classical, rule-and-statistics-based tagger, trained on relatively old data, and it is noticeably less accurate on modern proper nouns than the models in the next two sections. It is fine for teaching how NER works, and NLTK's real strength lies in the transparent, step-by-step tools above it, but production NER work generally moves to spaCy or a transformer.

---

## 5. spaCy

**spaCy** was built with a different philosophy from NLTK: instead of exposing separate functions for each step, you load one trained pipeline, run your text through it once, and get a rich object holding *everything* — tokens, tags, entities, sentences, grammar — computed together. It is faster, generally more accurate out of the box, and the standard choice for production NLP code.

```bash
python -m spacy download en_core_web_sm
```

`en_core_web_sm` is a small, pre-trained pipeline for English (spaCy also offers larger `md` and `lg` versions, and pipelines for many other languages).

```python
import spacy

nlp = spacy.load("en_core_web_sm")
doc = nlp("Apple is looking at buying a U.K. startup for $1 billion. Tim Cook flew to London.")
```

That single line, `nlp(text)`, ran the whole pipeline at once: tokenizing, tagging, parsing and finding entities all happened together, producing a `Doc` object. Check what ran with `nlp.pipe_names`:

```python
print(nlp.pipe_names)
# ['tok2vec', 'tagger', 'parser', 'attribute_ruler', 'lemmatizer', 'ner']
```

### Tokens and their attributes

A `Doc` is a sequence of `Token` objects, each carrying several pieces of information computed in that one pass:

```python
for token in doc[:5]:
    print(token.text, token.pos_, token.lemma_, token.is_stop)

# Apple    PROPN   Apple   False
# is       AUX     be      True
# looking  VERB    look    False
# at       ADP     at      True
# buying   VERB    buy     False
```

Compare this with Section 4's separate calls to `word_tokenize`, `pos_tag` and a lemmatizer: spaCy gives you all three, correctly, as attributes of the same object, with no need to specify a part of speech for the lemmatizer to work.

### Named entities

```python
for ent in doc.ents:
    print(ent.text, ent.label_)

# Apple          ORG
# U.K.           GPE
# $1 billion     MONEY
# Tim Cook       PERSON
# London         GPE
```

Every entity here is correctly labelled, including "Apple" as an organisation and "$1 billion" as a monetary amount, categories NLTK's tagger from Section 4 does not attempt at all. `spacy.explain("GPE")` (→ "Countries, cities, states") translates any of these abbreviated labels into plain English.

### Sentences and noun chunks

```python
print(list(doc.sents))
# [Apple is looking at buying a U.K. startup for $1 billion., Tim Cook flew to London.]

print(list(doc.noun_chunks))
# [Apple, a U.K., Tim Cook, London]
```

`doc.sents` splits the text into sentences (using the same parser that tagged the grammar, so it handles abbreviations like "U.K." correctly). `doc.noun_chunks` pulls out noun phrases directly, a shortcut that used to require writing your own rules over POS tags.

### Dependency parsing

Beyond labelling individual words, spaCy also computes the grammatical relationships *between* them, the **dependency parse**:

```python
sentence = nlp("The cat sat on the mat because it was tired")
for token in sentence:
    print(token.text, token.dep_, token.head.text)

# The    det    cat
# cat    nsubj  sat
# sat    ROOT   sat
# on     prep   sat
# the    det    mat
# mat    pobj   on
# because mark  was
# it     nsubj  was
# was    advcl  sat
# tired  acomp  was
```

Each row says how a word relates to another: "cat" is the **nsubj** (nominal subject) of "sat", and "mat" is the **pobj** (object of a preposition) attached to "on". "sat" is the **ROOT**, the main verb the whole sentence hangs from. This structure is what lets spaCy answer questions like "who is doing what to whom", and it can be visualised as a tree with `displacy.render(sentence, style="dep")` in a Jupyter notebook or browser.

### Stopwords and similarity

spaCy keeps its own stopword list, accessible directly:

```python
print(len(nlp.Defaults.stop_words))    # 326
```

It can also compare how similar two pieces of text are, using the vectors built into the pipeline:

```python
doc1 = nlp("I like apples")
doc2 = nlp("I like oranges")
print(doc1.similarity(doc2))    # ≈0.87
```

One caution: the small `sm` pipeline used throughout this section has no true word vectors and estimates similarity from its internal tagging and parsing instead, which spaCy itself will warn you about. For genuinely meaningful similarity scores, use the larger `md` or `lg` pipelines, which ship with real word vectors of the kind described in Section 2.

### NLTK vs spaCy

| | NLTK | spaCy |
|---|---|---|
| Philosophy | a toolbox of separate functions | one pipeline, run once |
| Speed | slower | fast (built for production) |
| Accuracy out of the box | lower (classical algorithms) | higher (trained statistical models) |
| Best for | learning how each NLP step works, linguistics research | building real applications quickly |
| Data | many downloadable corpora and algorithms | fewer, but trained, ready-to-use pipelines |

A reasonable way to think about it: use NLTK to *understand* what tokenizing, tagging and stemming actually do, and use spaCy to actually *do* NLP work in a project.

---

## 6. HuggingFace (Pipeline)

Neither NLTK nor spaCy uses transformers (Section 2). For that, the standard library is **HuggingFace's `transformers`**, which gives direct access to thousands of pre-trained models shared by researchers and companies around the world, through an interface so simple it can feel like cheating.

```python
from transformers import pipeline
```

### The `pipeline()` function

`pipeline()` is a single function that hides the entire process (choosing a model, tokenizing, running it, decoding the result) behind one call. You give it a task name as a string, and it downloads a sensible default pre-trained model for that task the first time you run it:

```python
classifier = pipeline("sentiment-analysis")

print(classifier("I've been waiting for a HuggingFace course my whole life."))
# [{'label': 'POSITIVE', 'score': ≈0.96}]

print(classifier(["This was a waste of time.", "Best day of my life!"]))
# [{'label': 'NEGATIVE', 'score': ≈0.9997},
#  {'label': 'POSITIVE', 'score': ≈0.9999}]
```

The default model behind `"sentiment-analysis"` is a fine-tuned version of DistilBERT (Section 2), so what looks like three lines of code is really a full transformer, pre-trained on vast amounts of text and fine-tuned on labelled reviews, doing the classification.

### Task strings, one per row of Section 3's table

Most of the row of tasks from Section 3 is available as a task string, each returning a sensibly structured result:

```python
# Named Entity Recognition
ner = pipeline("ner", grouped_entities=True)
print(ner("Tim Cook flew to London last week."))
# [{'entity_group': 'PER', 'word': 'Tim Cook', ...},
#  {'entity_group': 'LOC', 'word': 'London', ...}]

# Question Answering
qa = pipeline("question-answering")
result = qa(question="How many moons does Mars have?",
           context="Mars has two small moons, Phobos and Deimos.")
print(result)
# {'answer': 'two', 'score': ≈0.9, 'start': 9, 'end': 12}

# Summarization
summarizer = pipeline("summarization")
print(summarizer("A very long article about renewable energy trends...",
                 max_length=45, min_length=15))
# [{'summary_text': 'A short version of the article...'}]

# Translation (English to French)
translator = pipeline("translation_en_to_fr")
print(translator("I love natural language processing."))
# [{'translation_text': "J'adore le traitement du langage naturel."}]

# Text generation
generator = pipeline("text-generation")
print(generator("In a distant future, humans and AI", max_length=30))
# [{'generated_text': 'In a distant future, humans and AI ...'}]

# Zero-shot classification: classify into labels the model was never trained on
zero_shot = pipeline("zero-shot-classification")
print(zero_shot("This is a course about the Python programming language",
                candidate_labels=["education", "politics", "business"]))
# {'labels': ['education', 'business', 'politics'], 'scores': [≈0.84, ≈0.11, ≈0.05], ...}
```

`grouped_entities=True` merges pieces like "Tim" and "Cook" into a single "Tim Cook" result, instead of returning one row per word-piece. Note how naturally these map onto Section 3's table: one library, one consistent function, and the pipeline names read almost like the task names themselves.

**Zero-shot classification** deserves a second look, because it seems to break the rules of Guideline 39: the model was never trained on "education", "politics" or "business" as categories at all. It works by reframing classification as a question the model *was* trained to answer ("does this text imply the statement 'this example is about education'?"), a trick made possible only by how much general language understanding a large pre-trained transformer already has.

### Choosing a specific model

Leaving the model unspecified, as above, is convenient but unpredictable: the default can change over time, and a warning will tell you so. For real work, name the model explicitly, using its identifier from the HuggingFace Hub (huggingface.co/models):

```python
classifier = pipeline("sentiment-analysis",
                      model="distilbert-base-uncased-finetuned-sst-2-english")
```

The Hub hosts models fine-tuned for particular languages, domains (medical text, legal text, tweets) and tasks, so part of practical HuggingFace work is finding the right pre-trained specialist for your problem rather than training one from nothing, exactly the transfer-learning mindset from Guideline 40.

### Looking beneath the pipeline

`pipeline()` is a convenient wrapper around two objects you can use directly when you need more control: a **tokenizer**, which converts text into the numeric input a transformer expects, and a **model**, which the tokenizer's output is fed into.

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification

model_name = "distilbert-base-uncased-finetuned-sst-2-english"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(model_name)

tokens = tokenizer("I love this!", return_tensors="pt")
print(tokens)
# {'input_ids': tensor([[  101,  1045,  2293,  2023,   999,   102]]), 'attention_mask': ...}

outputs = model(**tokens)
print(outputs.logits)   # raw, un-normalised scores; pipeline() converts these to labels and probabilities for you
```

The `Auto` prefix in `AutoTokenizer` and `AutoModelForSequenceClassification` means the class figures out, from the model's name, exactly which architecture and tokenizer to load, so the same two lines work whether the model behind them is BERT, DistilBERT, RoBERTa or dozens of others. `input_ids` are the token numbers (compare this to the IMDB word numbers from Guideline 40), and `101` and `102` are special tokens marking the start and end of the input that BERT-family models expect.

### NLTK, spaCy, and HuggingFace, side by side

| | NLTK | spaCy | HuggingFace |
|---|---|---|---|
| Era / approach | classical statistical NLP | trained pipelines, production-ready | pre-trained transformers, state of the art |
| Needs the internet | only to download data, once | only to download a pipeline, once | to download model weights, and by default checks for updates |
| Typical use | teaching, exploring NLP concepts | fast, general-purpose NLP in an application | the best possible accuracy, or tasks needing deep language understanding (generation, QA, translation) |
| Runs on | any laptop | any laptop | any laptop for smaller models; larger ones benefit greatly from a GPU |

None of the three replaces the others outright. It is entirely normal for a real project to use spaCy for fast preprocessing and entity extraction, and HuggingFace for the one task, such as summarisation or nuanced sentiment, that genuinely needs a transformer's understanding of language.

---

*End of Guideline 41.*
