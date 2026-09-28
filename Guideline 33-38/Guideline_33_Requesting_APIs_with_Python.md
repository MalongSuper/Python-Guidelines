# Guideline 33: Requesting APIs with Python

Up to this point, every program in this guidebook has worked with data that already lived on your machine — variables you typed, files you opened, numbers you generated. But most real software doesn't stop at the edge of your computer. It asks other computers for things: today's weather, a stock price, a list of comments, a translation, a search result. The mechanism behind almost all of that is the **HTTP request** — a small, structured message sent to a server, and a structured message sent back.

An **API** (Application Programming Interface) is simply a defined set of rules a server exposes so that programs, not humans, can ask it for data. Where a website is built for a browser to render, an API is built for code to consume — the response usually comes back as plain text in a predictable format, most often **JSON**. This guideline covers how to make those requests from Python, read what comes back, and — as a bonus — how to pull information out of a plain HTML page when no tidy API exists for it at all.

The core tool for all of this is a third-party library called `requests`. It does not ship with Python by default, so it needs to be installed once per environment:

```bash
pip install requests
```

## 33.1 The `requests` Module

`requests` wraps the underlying complexity of HTTP into a small set of functions, one per HTTP **method** (the "verb" describing what the request is asking the server to do).

```python
import requests
```

| Method | Function | Typical purpose |
|---|---|---|
| GET | `requests.get(url)` | Retrieve data without changing anything on the server |
| POST | `requests.post(url)` | Create a new resource / submit data |
| PUT | `requests.put(url)` | Replace an existing resource entirely |
| PATCH | `requests.patch(url)` | Update part of an existing resource |
| DELETE | `requests.delete(url)` | Remove a resource |
| HEAD | `requests.head(url)` | Like GET, but only the headers are returned, no body |
| OPTIONS | `requests.options(url)` | Ask the server what methods/operations are allowed |

Every one of these functions returns a **Response object**, and that object is where all the useful information lives:

| Attribute / Method | What it gives you |
|---|---|
| `.status_code` | The numeric result of the request (e.g. `200`, `404`) |
| `.ok` | `True` if `.status_code` is under 400, `False` otherwise |
| `.text` | The raw response body, decoded as a string |
| `.content` | The raw response body as bytes (useful for images/files) |
| `.json()` | Parses the body as JSON and returns it as a dict/list |
| `.headers` | A dict-like object of the response's HTTP headers |
| `.url` | The final URL the request actually reached (after redirects) |

A quick status-code vocabulary, since you'll see these constantly:

| Range | Meaning |
|---|---|
| 200–299 | Success |
| 300–399 | Redirection |
| 400–499 | Client error (you asked wrong — bad URL, missing auth, etc.) |
| 500–599 | Server error (the problem is on their end) |

## 33.2 GET Request

A GET request is how you *ask for* data. It's the most common request you'll make, and it's what happens every time a browser loads a page.

```python
import requests

response = requests.get("https://jsonplaceholder.typicode.com/posts/1")

print(response.status_code)   # 200
print(response.text)          # raw JSON as a string
data = response.json()        # the same content, as a Python dict
print(data["title"])
```

It's good practice to check `.status_code` (or `.ok`) before trusting the body of the response — a failed request still returns a Response object, it just won't contain what you expect:

```python
if response.ok:
    data = response.json()
    print(data)
else:
    print(f"Request failed with status {response.status_code}")
```

## 33.3 Query String Parameters

A **query string** is the `?key=value&key2=value2` portion appended to a URL, used to filter, search, or configure what a GET request returns. You could build that string by hand, but `requests` will do it correctly for you — with proper encoding of spaces and special characters — through the `params` argument.

```python
import requests

url = "https://jsonplaceholder.typicode.com/comments"
params = {
    "postId": 1
}

response = requests.get(url, params=params)
print(response.url)     # https://jsonplaceholder.typicode.com/comments?postId=1
print(response.json())
```

Multiple parameters just become additional keys in the dictionary:

```python
params = {
    "postId": 1,
    "_limit": 5
}
```

This is the same idea as arguments in a function call — you're telling the server exactly what you want back, without changing the underlying endpoint.

## 33.4 Request Headers

**Headers** carry metadata about the request itself — not the data you're asking for, but information *about* how you're asking for it. They're passed as a dictionary through the `headers` argument.

```python
import requests

headers = {
    "User-Agent": "python-guidebook-app/1.0",
    "Accept": "application/json"
}

response = requests.get("https://jsonplaceholder.typicode.com/posts/1", headers=headers)
```

A few headers you'll encounter often:

| Header | Purpose |
|---|---|
| `User-Agent` | Identifies the client making the request (some servers block requests with no User-Agent) |
| `Accept` | Tells the server what response format you can handle (e.g. `application/json`) |
| `Content-Type` | Tells the server what format the *request body* you're sending is in |
| `Authorization` | Carries credentials — an API key, a token, etc. (covered in 33.6) |

Response headers work the same way in reverse — `response.headers` lets you inspect what the server sent back, such as `Content-Type` or rate-limit information.

## 33.5 POST, PUT and DELETE (JSON or Form)

Where GET only retrieves data, POST, PUT, and DELETE change something on the server. All three can carry a request **body** — data attached to the request rather than the URL.

**POST** — typically used to create something new:

```python
import requests

url = "https://jsonplaceholder.typicode.com/posts"

# Sending JSON (most common for modern APIs)
payload = {
    "title": "Learning requests",
    "body": "This guideline covers GET, POST, PUT, and DELETE.",
    "userId": 1
}
response = requests.post(url, json=payload)
print(response.status_code)   # 201 Created
print(response.json())
```

Sending the same data as an old-style HTML **form** instead of JSON just means swapping `json=` for `data=`:

```python
response = requests.post(url, data=payload)
```

The difference matters to the server: `json=` automatically sets `Content-Type: application/json` and serializes the dictionary into a JSON string; `data=` sends it as `application/x-www-form-urlencoded`, the same format an HTML `<form>` submits.

**PUT** — replaces an existing resource in full:

```python
url = "https://jsonplaceholder.typicode.com/posts/1"
payload = {
    "id": 1,
    "title": "Updated title",
    "body": "The entire post is replaced with this content.",
    "userId": 1
}
response = requests.put(url, json=payload)
```

**DELETE** — removes a resource, and usually needs nothing but the URL:

```python
url = "https://jsonplaceholder.typicode.com/posts/1"
response = requests.delete(url)
print(response.status_code)   # 200 or 204
```

## 33.6 Authentication

Many APIs won't respond to just anyone — they need to know *who* is asking. `requests` supports a few common authentication patterns.

**Basic authentication** (username/password), passed as a tuple:

```python
response = requests.get(
    "https://api.example.com/data",
    auth=("myusername", "mypassword")
)
```

**API key authentication**, usually passed either as a header or a query parameter (the API's documentation tells you which):

```python
# As a header
headers = {"X-API-Key": "your_api_key_here"}
response = requests.get("https://api.example.com/data", headers=headers)

# As a query parameter
params = {"api_key": "your_api_key_here"}
response = requests.get("https://api.example.com/data", params=params)
```

**Token / Bearer authentication**, the most common pattern for modern APIs (OAuth-style services included), sent through the `Authorization` header:

```python
headers = {
    "Authorization": "Bearer your_access_token_here"
}
response = requests.get("https://api.example.com/data", headers=headers)
```

| Method | Where credentials go | Typical use case |
|---|---|---|
| Basic Auth | `auth=(user, pass)` tuple | Simple internal APIs, some legacy services |
| API Key | Header or query parameter | Weather, maps, and many public data APIs |
| Bearer Token | `Authorization` header | OAuth-protected APIs, most modern SaaS platforms |

A real API key or token is a secret — treat it the way you'd treat a password, and never hard-code it directly into a script you plan to share; keeping it in an environment variable and reading it with `os.environ` is the standard practice.

## 33.7 Send Request Data

Pulling together 33.3 through 33.6, `requests` gives you four distinct ways to attach data to a request, and knowing which one to reach for matters:

| Argument | Goes where | Used for |
|---|---|---|
| `params=` | The URL's query string | Filtering/searching on a GET request |
| `data=` | The request body, form-encoded | Simple key-value form submissions |
| `json=` | The request body, JSON-encoded | Structured data for modern APIs (most common) |
| `files=` | The request body, multipart-encoded | Uploading files |

A file upload example, since it hasn't come up yet:

```python
files = {
    "document": open("report.pdf", "rb")
}
response = requests.post("https://api.example.com/upload", files=files)
```

Combining several of these in one call is completely normal — a single request can carry query parameters, headers, and a JSON body all at once:

```python
response = requests.post(
    "https://api.example.com/items",
    params={"verbose": True},
    headers={"Authorization": "Bearer your_token"},
    json={"name": "New Item", "quantity": 5}
)
```

## 33.8 The BeautifulSoup Module

Not everything you want to pull from the web comes as a tidy JSON API — sometimes the only thing available is a plain HTML page meant for human eyes. **BeautifulSoup** (from the `bs4` package) parses that HTML into a navigable structure so you can pull specific pieces out of it. This practice is commonly called **web scraping**.

```bash
pip install beautifulsoup4
```

Fetching the page is still `requests`' job — BeautifulSoup only understands HTML you hand it:

```python
import requests
from bs4 import BeautifulSoup

response = requests.get("https://example.com")
soup = BeautifulSoup(response.text, "html.parser")
```

The second argument is the **parser** — `"html.parser"` ships with Python and is fine for most tasks; `"lxml"` is faster but must be installed separately (`pip install lxml`).

Once you have a `soup` object, these are the methods you'll use constantly:

| Method / Attribute | What it does |
|---|---|
| `soup.find("tag")` | Returns the *first* matching tag |
| `soup.find_all("tag")` | Returns a list of *all* matching tags |
| `soup.select("css.selector")` | Finds tags using CSS selector syntax |
| `.text` | The visible text inside a tag, with markup stripped |
| `.get("attribute")` | Reads an attribute's value, e.g. `.get("href")` |
| `.find("tag", class_="name")` | Narrows a search by class (note the trailing underscore — `class` is a reserved word) |

A worked example — pulling every link out of a page:

```python
import requests
from bs4 import BeautifulSoup

response = requests.get("https://example.com")
soup = BeautifulSoup(response.text, "html.parser")

for link in soup.find_all("a"):
    href = link.get("href")
    text = link.text.strip()
    print(f"{text}: {href}")
```

And an example narrowing by class, which is the most common real-world pattern — say a page lists articles inside `<h2 class="article-title">` tags:

```python
titles = soup.find_all("h2", class_="article-title")
for title in titles:
    print(title.text.strip())
```

One practical note: scraping only works on content that's already present in the HTML `requests` downloads. Pages that load their content afterward with JavaScript won't show that content to `requests` at all — that's a limitation beyond what this guideline covers, and the usual workaround involves browser-automation tools rather than `requests` and BeautifulSoup.

---

Between the `requests` module and BeautifulSoup, you now have the two halves of nearly all real-world data gathering in Python: structured APIs that hand you clean JSON, and unstructured pages that need to be parsed by hand. Almost every data science project, automation script, or bot you'll build from here eventually touches one of these two tools — often both in the same script, fetching a page and then digging through what came back.
