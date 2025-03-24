Below, I’ve adapted the examples for **navigating to a URL in Playwright with Python** to fit a hypothetical MERN (MongoDB, Express, React, Node.js) application context. I’ll assume the MERN app is a task management system running locally at `http://localhost:3000` (React frontend) with an Express API at `http://localhost:5000`. The examples will reflect realistic navigation scenarios you might encounter in such an app—e.g., landing pages, login flows, dynamic task lists, and API-driven pages. These are tailored for your advanced MERN training participants to see Playwright in action with their full-stack project.

---

## **Conceptual Usages of Navigating to a URL in Playwright with Python (MERN Context)**

For this setup:
- **Frontend**: React app at `http://localhost:3000` (e.g., task management UI).
- **Backend**: Express API at `http://localhost:5000` (e.g., `/api/tasks` endpoint).
- **Features**: Login page, task list, task creation, and a dashboard.

All examples use the synchronous Playwright API, but they can be adapted to async if preferred.

---

### **1. Basic Navigation to the MERN App Homepage**
Navigate to the app’s root URL.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("http://localhost:3000")
    print("Homepage title:", page.title())  # Expected: "Task Manager"
    browser.close()
```

- **Use Case**: Test the initial load of the React app.
- **MERN Context**: Ensures the frontend server is running and renders the homepage.

---

### **2. Navigation with Wait Options**
Wait for specific load states in a dynamic React app.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()

    # Wait for DOM (React renders initial UI)
    page.goto("http://localhost:3000/tasks", wait_until="domcontentloaded")
    print("Tasks page DOM loaded:", page.title())

    # Wait for API calls to finish (e.g., fetching tasks)
    page.goto("http://localhost:3000/dashboard", wait_until="networkidle")
    print("Dashboard fully loaded:", page.title())

    browser.close()
```

- **Options for `wait_until`**:
  - `"domcontentloaded"`: Good for React’s initial render.
  - `"networkidle"`: Ensures API-driven content (e.g., MongoDB task data) loads.
- **MERN Context**: Handles React’s client-side rendering and API fetches.

---

### **3. Navigation with Timeout**
Prevent hanging if the MERN app or API is slow.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    try:
        page.goto("http://localhost:3000/tasks", timeout=3000)  # 3 seconds
        print("Tasks page title:", page.title())
    except Exception as e:
        print("Navigation failed:", str(e))
    browser.close()
```

- **Use Case**: Test responsiveness of the MERN stack under load or failure.
- **MERN Context**: Useful when MongoDB queries or Express responses delay page load.

---

### **4. Navigation with Custom Headers**
Simulate authenticated requests or custom client info.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context(extra_http_headers={
        "Authorization": "Bearer mock-jwt-token",
        "X-App-Version": "1.0.0"
    })
    page = context.new_page()
    page.goto("http://localhost:3000/dashboard")
    print("Dashboard title:", page.title())  # Expected: "Dashboard - Task Manager"
    browser.close()
```

- **Use Case**: Test protected routes requiring JWT tokens from Express.
- **MERN Context**: Mimics a logged-in user accessing a restricted React page.

---

### **5. Navigation with Query Parameters**
Navigate to a filtered task list.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    filter = "completed"
    url = f"http://localhost:3000/tasks?status={filter}"
    page.goto(url)
    print("Filtered tasks title:", page.title())  # Expected: "Tasks - Task Manager"
    browser.close()
```

- **Use Case**: Test React Router or Express query handling.
- **MERN Context**: Verifies frontend routing and backend filtering from MongoDB.

---

### **6. Handling Redirects During Navigation**
Test login redirects in the MERN app.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    response = page.goto("http://localhost:3000/dashboard")  # Redirects to login if not authenticated
    print("Final URL:", response.url)  # Expected: "http://localhost:3000/login"
    print("Status:", response.status)  # Expected: 200 after redirect
    browser.close()
```

- **Use Case**: Validate authentication middleware in Express redirecting to `/login`.
- **MERN Context**: Ensures React Router and Express work together for auth flows.

---

### **7. Navigation with Event Listeners**
Monitor API calls during navigation.

```python
from playwright.sync_api import sync_playwright

def handle_response(response):
    if "api/tasks" in response.url:
        print(f"API call: {response.url}, Status: {response.status}")

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.on("response", handle_response)
    page.goto("http://localhost:3000/tasks")
    print("Tasks page title:", page.title())
    browser.close()
```

- **Use Case**: Debug Express API responses powering the React UI.
- **MERN Context**: Tracks MongoDB data fetching via the `/api/tasks` endpoint.

---

### **8. Navigation with Explicit Waits**
Wait for a task list to render after navigation.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("http://localhost:3000/tasks")
    page.wait_for_selector(".task-list")  # Wait for task list element
    task_count = len(page.query_selector_all(".task-item"))
    print("Number of tasks:", task_count)
    browser.close()
```

- **Use Case**: Ensure React renders dynamic content from MongoDB.
- **MERN Context**: Confirms API data integration into the React UI.

---

### **9. Navigation in a New Tab**
Open task details in a new tab.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context()
    page1 = context.new_page()
    page1.goto("http://localhost:3000/tasks")

    # Open task details in a new tab
    page2 = context.new_page()
    page2.goto("http://localhost:3000/tasks/123")  # Assuming task ID 123
    print("Page1 title:", page1.title())  # "Tasks - Task Manager"
    print("Page2 title:", page2.title())  # "Task Details - Task Manager"
    context.close()
    browser.close()
```

- **Use Case**: Test navigation between task list and detail views.
- **MERN Context**: Simulates React Router navigation within the same session.

---

### **10. Navigation with Authentication**
Navigate as a logged-in user using storage state.

```python
from playwright.sync_api import sync_playwright
import json

# Save login state (run once)
with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context()
    page = context.new_page()
    page.goto("http://localhost:3000/login")
    page.fill("input[name='username']", "testuser")
    page.fill("input[name='password']", "password123")
    page.click("button[type='submit']")
    page.wait_for_url("http://localhost:3000/dashboard")
    context.storage_state(path="mern_auth_state.json")
    browser.close()

# Use authenticated state
with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context(storage_state="mern_auth_state.json")
    page = context.new_page()
    page.goto("http://localhost:3000/dashboard")
    print("Dashboard title:", page.title())  # "Dashboard - Task Manager"
    browser.close()
```

- **Use Case**: Skip login for repeated tests of protected routes.
- **MERN Context**: Integrates Express JWT auth with React UI.

---

### **11. Navigation with Retry Logic**
Retry navigation if the MERN app is temporarily unavailable.

```python
from playwright.sync_api import sync_playwright
import time

def navigate_with_retry(page, url, retries=3):
    for attempt in range(retries):
        try:
            page.goto(url, timeout=3000)
            return True
        except Exception as e:
            print(f"Attempt {attempt + 1} failed: {str(e)}")
            time.sleep(1)
    return False

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    success = navigate_with_retry(page, "http://localhost:3000/tasks")
    print("Navigation successful:", success)
    browser.close()
```

- **Use Case**: Handle Express server restarts or MongoDB connection delays.
- **MERN Context**: Ensures robust testing in development environments.

---

### **12. Navigation in Headless vs. Headful Mode**
Test the MERN app visibly or silently.

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    # Headless mode
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("http://localhost:3000")
    print("Headless title:", page.title())
    browser.close()

    # Headful mode
    browser = p.chromium.launch(headless=False)
    page = browser.new_page()
    page.goto("http://localhost:3000/tasks")
    print("Headful title:", page.title())
    browser.close()
```

- **Use Case**: Debug React UI issues (`headless=False`) or run in CI (`headless=True`).
- **MERN Context**: Validates frontend rendering in different modes.

---

### **13. Navigation with Pytest Integration**
Structure navigation tests for the MERN app.

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

def test_homepage_navigation(page):
    page.goto("http://localhost:3000")
    assert page.title() == "Task Manager"

def test_tasks_navigation(page):
    page.goto("http://localhost:3000/tasks")
    page.wait_for_selector(".task-list")
    assert "Tasks" in page.title()
```

- **Use Case**: Automate testing of MERN app navigation flows.
- **MERN Context**: Verifies React Router and Express API integration.

---

## **Notes for Participants**
- **Setup**: Ensure the MERN app is running locally (`npm start` for React at `:3000`, `node server.js` for Express at `:5000`).
- **Selectors**: Adjust `.task-list`, `.task-item`, or input names based on your app’s DOM structure.
- **Auth**: The login example assumes a basic form; tweak it for your app’s auth flow (e.g., OAuth, JWT).
- **Environment**: Run `pip install playwright` and `playwright install` to set up Playwright.

