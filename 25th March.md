

## **Examples of Managing Browser Contexts and Pages in Playwright with Python**

These examples use the synchronous Playwright API for clarity, but the asynchronous version (`async`/`await`) follows the same structure. The sites chosen are publicly accessible and recognizable.

### **1. Launching a Browser and Creating Multiple Contexts**
Simulate two independent user sessions by creating separate contexts.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    # Launch the browser
    browser = p.chromium.launch()

    # First context
    context1 = browser.new_context()
    page1 = context1.new_page()
    page1.goto("https://www.wikipedia.org")
    print("Context1 title:", page1.title())  # Expected: "Wikipedia"

    # Second context
    context2 = browser.new_context()
    page2 = context2.new_page()
    page2.goto("https://www.reddit.com")
    print("Context2 title:", page2.title())  # Expected: "Reddit - Dive into anything"

    # Cleanup
    context1.close()
    context2.close()
    browser.close()
```

- **What it does**: Opens Wikipedia in one context and Reddit in another, demonstrating isolated sessions.

---

### **2. Managing Cookies and Sessions Independently**
Show how cookies are isolated between contexts.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()

    # First context
    context1 = browser.new_context()
    page1 = context1.new_page()
    page1.goto("https://www.wikipedia.org")
    context1.add_cookies([{"name": "user", "value": "alice", "domain": ".wikipedia.org", "path": "/"}])
    cookies1 = context1.cookies()
    print("Context1 cookies:", cookies1)  # Shows 'user=alice'

    # Second context
    context2 = browser.new_context()
    page2 = context2.new_page()
    page2.goto("https://www.reddit.com")
    cookies2 = context2.cookies()
    print("Context2 cookies:", cookies2)  # Empty or site-specific cookies only

    # Cleanup
    context1.close()
    context2.close()
    browser.close()
```

- **What it does**: Adds a custom cookie to the Wikipedia context and verifies it’s not present in the Reddit context, highlighting session isolation.

---

### **3. Handling Multiple Pages Within a Single Context**
Demonstrate shared session data across pages in one context.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context()

    # First page
    page1 = context.new_page()
    page1.goto("https://www.wikipedia.org")
    context.add_cookies([{"name": "user", "value": "bob", "domain": ".wikipedia.org", "path": "/"}])

    # Second page in the same context
    page2 = context.new_page()
    page2.goto("https://en.wikipedia.org/wiki/Main_Page")
    cookies2 = page2.context.cookies()
    print("Page2 cookies:", cookies2)  # Shows 'user=bob'

    # Cleanup
    page1.close()
    page2.close()
    context.close()
    browser.close()
```

- **What it does**: Opens Wikipedia’s main portal and its English homepage in the same context, showing that cookies are shared between pages.

---

### **4. Handling New Pages Opened by the Application**
Capture a new tab opened by clicking a link (e.g., a Wikipedia external reference).

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context()

    page = context.new_page()
    page.goto("https://en.wikipedia.org/wiki/Python_(programming_language)")

    # Capture a new page from clicking an external link
    with context.expect_page() as new_page_info:
        page.click("a[title='Official website']")  # Clicks the Python.org link
    new_page = new_page_info.value
    new_page.wait_for_load_state()
    print("New page URL:", new_page.url)  # Expected: "https://www.python.org/"

    # Cleanup
    new_page.close()
    page.close()
    context.close()
    browser.close()
```

- **What it does**: Navigates to the Python Wikipedia page, clicks the “Official website” link (which opens in a new tab), and interacts with the resulting Python.org page.

---

### **5. Using Pytest Fixtures for Test Organization**
Organize tests with reusable fixtures, ideal for training participants on scalable testing practices.

```python
import pytest
from playwright.sync_api import sync_playwright

# Fixture for browser (shared across tests)
@pytest.fixture(scope="session")
def browser():
    with sync_playwright() as p:
        browser = p.chromium.launch()
        yield browser
        browser.close()

# Fixture for context (new per test)
@pytest.fixture
def context(browser):
    context = browser.new_context()
    yield context
    context.close()

# Fixture for page (new per test)
@pytest.fixture
def page(context):
    page = context.new_page()
    yield page
    page.close()

# Sample test
def test_wikipedia_title(page):
    page.goto("https://www.wikipedia.org")
    assert page.title() == "Wikipedia"
```

- **What it does**: Provides a clean setup for testing Wikipedia’s title, with fixtures ensuring resource efficiency and isolation.

---

### **6. Managing Authentication with Storage State**
Simulate a login flow with Reddit (hypothetical, as Reddit requires real credentials) and reuse the state.

```python
from playwright.sync_api import sync_playwright
import json

# Save authenticated state (run this once, hypothetical login)
with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context()
    page = context.new_page()
    page.goto("https://www.reddit.com/login")
    # Hypothetical login (replace with real steps if credentials available)
    page.fill("input[name='username']", "demo_user")
    page.fill("input[name='password']", "demo_pass")
    page.click("button[type='submit']")
    page.wait_for_load_state("networkidle")
    
    # Save storage state
    storage_state = context.storage_state()
    with open("reddit_auth_state.json", "w") as f:
        json.dump(storage_state, f)
    context.close()
    browser.close()

# Use in tests with a fixture
@pytest.fixture
def auth_context(browser):
    with open("reddit_auth_state.json") as f:
        storage_state = json.load(f)
    context = browser.new_context(storage_state=storage_state)
    yield context
    context.close()

def test_authenticated_reddit(auth_context):
    page = auth_context.new_page()
    page.goto("https://www.reddit.com")
    assert "Reddit" in page.title()  # Check for authenticated homepage
```

- **What it does**: Simulates saving a Reddit login state and reusing it to skip login steps in tests. (Note: Real login requires actual credentials; this is a demo structure.)

---

