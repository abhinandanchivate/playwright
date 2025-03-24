Below is a comprehensive guide showcasing all possible conceptual usages of **navigating to a URL** in Playwright with Python. These examples cover various scenarios, options, and techniques you might encounter when using the `page.goto()` method or related navigation approaches. I'll use real-world websites like `https://www.wikipedia.org` and `https://www.reddit.com` to make the examples practical and relatable for your participants.

---

## **Conceptual Usages of Navigating to a URL in Playwright with Python**

In Playwright, navigation is primarily handled by the `page.goto()` method, which directs a page to a specified URL. However, there are several ways to customize and control this process depending on your use case—whether it's basic navigation, waiting for specific conditions, handling redirects, or integrating with testing frameworks. Below are all the key conceptual approaches, demonstrated with the synchronous API (the async API follows similar principles with `async`/`await`).

---

### **1. Basic Navigation to a URL**
The simplest way to navigate to a URL.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://www.wikipedia.org")
    print("Title:", page.title())  # Expected: "Wikipedia"
    browser.close()
```

- **Use Case**: Quick navigation to a webpage to start interacting with it.
- **Key Point**: `goto()` waits for the page to reach a "load" state by default.

---

### **2. Navigation with Wait Options**
Control how long Playwright waits for the page to load using the `wait_until` parameter.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()

    # Wait until DOM content is loaded
    page.goto("https://www.reddit.com", wait_until="domcontentloaded")
    print("DOM loaded title:", page.title())

    # Wait until all network requests are done
    page.goto("https://www.wikipedia.org", wait_until="networkidle")
    print("Network idle title:", page.title())

    browser.close()
```

- **Options for `wait_until`**:
  - `"load"`: Waits for the `load` event (default).
  - `"domcontentloaded"`: Waits for the DOM to be ready, but not all resources (faster).
  - `"networkidle"`: Waits until no network connections remain for 500ms (slower but thorough).
  - `"commit"`: Waits only for the navigation to commit (earliest point).
- **Use Case**: Fine-tune navigation based on whether you need DOM, full content, or minimal loading.

---

### **3. Navigation with Timeout**
Set a maximum time to wait for navigation to complete.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    try:
        page.goto("https://www.wikipedia.org", timeout=5000)  # 5 seconds timeout
        print("Title:", page.title())
    except Exception as e:
        print("Navigation failed:", str(e))
    browser.close()
```

- **Use Case**: Prevent hanging on slow or unresponsive sites by enforcing a timeout (default is 30 seconds).
- **Key Point**: Raises a `TimeoutError` if navigation exceeds the specified time.

---

### **4. Navigation with Custom Headers**
Pass custom HTTP headers during navigation.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context(extra_http_headers={
        "User-Agent": "MyCustomAgent/1.0",
        "Accept-Language": "en-US"
    })
    page = context.new_page()
    page.goto("https://www.wikipedia.org")
    print("Title:", page.title())  # Wikipedia in English due to header
    browser.close()
```

- **Use Case**: Simulate specific browser behaviors, test localization, or bypass restrictions requiring custom headers.
- **Key Point**: Headers are set at the context level, affecting all pages within it.

---

### **5. Navigation with Query Parameters**
Navigate to a URL with dynamically constructed query parameters.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    search_term = "Python programming"
    url = f"https://en.wikipedia.org/w/index.php?search={search_term}"
    page.goto(url)
    print("Search page title:", page.title())  # Expected: Search results for "Python programming"
    browser.close()
```

- **Use Case**: Test search functionality or parameterized routes in a web app.
- **Key Point**: Build URLs dynamically using Python string formatting.

---

### **6. Handling Redirects During Navigation**
Capture and verify redirects explicitly.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()

    # Navigate to a URL known to redirect (e.g., shortened Wikipedia link)
    response = page.goto("https://en.wikipedia.org/wiki/Main_Page")
    print("Final URL:", response.url)  # Expected: "https://en.wikipedia.org/wiki/Main_Page"
    print("Status:", response.status)  # Expected: 200
    browser.close()
```

- **Use Case**: Test redirect chains or verify final landing pages.
- **Key Point**: `page.goto()` returns a `Response` object with details like `url` and `status`.

---

### **7. Navigation with Event Listeners**
Listen for navigation-related events (e.g., page load, redirects).

```python
from playwright.sync_api import sync_playwright

def handle_response(response):
    print(f"Loaded resource: {response.url} with status {response.status}")

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.on("response", handle_response)  # Log every response
    page.goto("https://www.reddit.com")
    print("Title:", page.title())
    browser.close()
```

- **Events**:
  - `"response"`: Fired for every resource loaded.
  - `"request"`: Fired for every request initiated.
  - `"load"`: Fired when the page fully loads.
- **Use Case**: Debug navigation issues or monitor network activity.

---

### **8. Navigation with Explicit Waits**
Wait for specific conditions after navigation (e.g., an element to appear).

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://en.wikipedia.org/wiki/Python_(programming_language)")
    page.wait_for_selector("h1#firstHeading")  # Wait for the page title
    heading = page.text_content("h1#firstHeading")
    print("Heading:", heading)  # Expected: "Python (programming language)"
    browser.close()
```

- **Use Case**: Ensure critical elements are present before proceeding (common in dynamic SPAs).
- **Key Point**: Use `wait_for_selector()`, `wait_for_load_state()`, or similar methods post-navigation.

---

### **9. Navigation in a New Tab**
Open a URL in a new tab within the same context.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context()
    page1 = context.new_page()
    page1.goto("https://www.wikipedia.org")

    # Open a new tab
    page2 = context.new_page()
    page2.goto("https://www.reddit.com")
    print("Page1 title:", page1.title())  # "Wikipedia"
    print("Page2 title:", page2.title())  # "Reddit - Dive into anything"

    context.close()
    browser.close()
```

- **Use Case**: Simulate multi-tab browsing within a single user session.
- **Key Point**: Pages in the same context share cookies and session data.

---

### **10. Navigation with Authentication**
Navigate to a protected page using pre-authenticated context.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context(http_credentials={"username": "user", "password": "pass"})
    page = context.new_page()
    page.goto("https://httpbin.org/basic-auth/user/pass")  # Public test site
    print("Content:", page.text_content("body"))  # Expected: Authenticated response
    browser.close()
```

- **Use Case**: Test pages behind HTTP Basic Auth or simulate logged-in states.
- **Key Point**: Use `http_credentials` in the context or load a storage state for complex auth.

---

### **11. Navigation with Retry Logic**
Retry navigation on failure (e.g., network issues).

```python
from playwright.sync_api import sync_playwright
import time

def navigate_with_retry(page, url, retries=3):
    for attempt in range(retries):
        try:
            page.goto(url, timeout=5000)
            return True
        except Exception as e:
            print(f"Attempt {attempt + 1} failed: {str(e)}")
            time.sleep(1)  # Wait before retry
    return False

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    success = navigate_with_retry(page, "https://www.wikipedia.org")
    print("Navigation successful:", success)
    browser.close()
```

- **Use Case**: Handle flaky networks or intermittent server issues in tests.
- **Key Point**: Custom logic can wrap `goto()` for robustness.

---

### **12. Navigation in Headless vs. Headful Mode**
Control whether the browser runs visibly or in the background.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    # Headless mode (default)
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://www.wikipedia.org")
    print("Headless title:", page.title())
    browser.close()

    # Headful mode (visible browser)
    browser = p.chromium.launch(headless=False)
    page = browser.new_page()
    page.goto("https://www.reddit.com")
    print("Headful title:", page.title())
    browser.close()
```

- **Use Case**: Debug with `headless=False` or run silently with `headless=True` in CI/CD.
- **Key Point**: Toggle visibility based on debugging or performance needs.

---

### **13. Navigation with Pytest Integration**
Use navigation in a test suite with fixtures.

```python
import pytest
from playwright.sync_api import sync_playwright

@pytest.fixture(scope="module")
def page():
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        yield page
        browser.close()

def test_wikipedia_navigation(page):
    page.goto("https://www.wikipedia.org")
    assert page.title() == "Wikipedia"

def test_reddit_navigation(page):
    page.goto("https://www.reddit.com")
    assert "Reddit" in page.title()
```

- **Use Case**: Structure navigation tests for a MERN app or public site in a reusable way.
- **Key Point**: Fixtures ensure consistent setup and teardown.

---

## **Key Takeaways**
- **Flexibility**: `page.goto()` supports timeouts, wait conditions, and response handling.
- **Context Awareness**: Navigation behavior depends on the browser context (e.g., cookies, headers).
- **Robustness**: Add retries, event listeners, or waits for dynamic pages.
- **Testing**: Integrate with Pytest or custom logic for scalable workflows.
- **Real-World Applicability**: These examples work with live sites like Wikipedia and Reddit, or can be adapted to a MERN app.
