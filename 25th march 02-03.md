I can’t directly create or share a `.zip` file as an AI text-based assistant, but I can provide you with the exact file structure and contents that you can easily package into a `.zip` file yourself. Below, I’ll outline the complete directory structure for the MERN task management app and the Playwright tests, including all necessary files. You can copy these into a folder on your local machine, then compress that folder into a `.zip` file using any standard tool (e.g., right-click "Compress to ZIP" on Windows, or `zip -r mern-training.zip mern-training/` on Linux/Mac).

Here’s the step-by-step structure and content:

---

## **Directory Structure for MERN Training Zip**

```
mern-training/
├── backend/
│   ├── package.json
│   ├── server.js
│   └── node_modules/ (generated after npm install)
├── frontend/
│   ├── package.json
│   ├── src/
│   │   ├── App.js
│   │   ├── Login.js
│   │   ├── Tasks.js
│   │   ├── TaskDetails.js
│   │   ├── Dashboard.js
│   │   └── index.js
│   └── node_modules/ (generated after npm install)
├── tests/
│   ├── test_navigation.py
│   ├── requirements.txt
│   └── mern_auth_state.json (generated after running auth setup)
└── README.md
```

---

## **File Contents**

### **1. Backend (Express + MongoDB)**

**`backend/package.json`**
```json
{
  "name": "mern-backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^4.18.2",
    "jsonwebtoken": "^9.0.2",
    "mongoose": "^8.0.0"
  }
}
```

**`backend/server.js`**
```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const jwt = require('jsonwebtoken');

const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect('mongodb://localhost:27017/taskmanager', { useNewUrlParser: true, useUnifiedTopology: true });

const Task = mongoose.model('Task', new mongoose.Schema({
  title: String,
  status: { type: String, enum: ['pending', 'completed'], default: 'pending' },
}));

const User = mongoose.model('User', new mongoose.Schema({
  username: String,
  password: String,
}));

const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ message: 'Unauthorized' });
  try {
    jwt.verify(token, 'secret');
    next();
  } catch (e) {
    res.status(401).json({ message: 'Invalid token' });
  }
};

app.post('/api/login', async (req, res) => {
  const { username, password } = req.body;
  const user = await User.findOne({ username, password });
  if (!user) return res.status(401).json({ message: 'Invalid credentials' });
  const token = jwt.sign({ username }, 'secret', { expiresIn: '1h' });
  res.json({ token });
});

app.get('/api/tasks', authMiddleware, async (req, res) => {
  const { status } = req.query;
  const tasks = await Task.find(status ? { status } : {});
  res.json(tasks);
});

app.get('/api/tasks/:id', authMiddleware, async (req, res) => {
  const task = await Task.findById(req.params.id);
  res.json(task || { message: 'Task not found' });
});

app.listen(5000, () => console.log('Server running on http://localhost:5000'));
```

---

### **2. Frontend (React)**

**`frontend/package.json`**
```json
{
  "name": "mern-frontend",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.22.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build"
  },
  "proxy": "http://localhost:5000"
}
```

**`frontend/src/App.js`**
```javascript
import React, { useState } from 'react';
import { BrowserRouter as Router, Route, Routes, Navigate } from 'react-router-dom';
import Login from './Login';
import Tasks from './Tasks';
import TaskDetails from './TaskDetails';
import Dashboard from './Dashboard';

function App() {
  const [token, setToken] = useState(localStorage.getItem('token'));

  return (
    <Router>
      <Routes>
        <Route path="/" element={<h1>Task Manager</h1>} />
        <Route path="/login" element={<Login setToken={setToken} />} />
        <Route path="/tasks" element={token ? <Tasks token={token} /> : <Navigate to="/login" />} />
        <Route path="/tasks/:id" element={token ? <TaskDetails token={token} /> : <Navigate to="/login" />} />
        <Route path="/dashboard" element={token ? <Dashboard token={token} /> : <Navigate to="/login" />} />
      </Routes>
    </Router>
  );
}

export default App;
```

**`frontend/src/Login.js`**
```javascript
import React from 'react';

function Login({ setToken }) {
  const handleSubmit = async (e) => {
    e.preventDefault();
    const username = e.target.username.value;
    const password = e.target.password.value;
    const res = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username, password }),
    });
    const data = await res.json();
    if (data.token) {
      localStorage.setItem('token', data.token);
      setToken(data.token);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="username" placeholder="Username" />
      <input name="password" type="password" placeholder="Password" />
      <button type="submit">Login</button>
    </form>
  );
}

export default Login;
```

**`frontend/src/Tasks.js`**
```javascript
import React, { useState, useEffect } from 'react';

function Tasks({ token }) {
  const [tasks, setTasks] = useState([]);

  useEffect(() => {
    fetch('/api/tasks', {
      headers: { Authorization: `Bearer ${token}` },
    })
      .then(res => res.json())
      .then(data => setTasks(data));
  }, [token]);

  return (
    <div>
      <h1>Tasks</h1>
      <ul className="task-list">
        {tasks.map(task => (
          <li key={task._id} className="task-item">{task.title} - {task.status}</li>
        ))}
      </ul>
    </div>
  );
}

export default Tasks;
```

**`frontend/src/TaskDetails.js`**
```javascript
import React, { useState, useEffect } from 'react';
import { useParams } from 'react-router-dom';

function TaskDetails({ token }) {
  const { id } = useParams();
  const [task, setTask] = useState(null);

  useEffect(() => {
    fetch(`/api/tasks/${id}`, {
      headers: { Authorization: `Bearer ${token}` },
    })
      .then(res => res.json())
      .then(data => setTask(data));
  }, [id, token]);

  return task ? <h1>Task: {task.title}</h1> : <p>Loading...</p>;
}

export default TaskDetails;
```

**`frontend/src/Dashboard.js`**
```javascript
import React from 'react';

function Dashboard() {
  return <h1>Dashboard - Task Manager</h1>;
}

export default Dashboard;
```

**`frontend/src/index.js`**
```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```

---

### **3. Playwright Tests**

**`tests/test_navigation.py`**
```python
import pytest
from playwright.sync_api import sync_playwright
import json
import time

# Save auth state (run once)
def save_auth_state():
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

# Fixture for authenticated context
@pytest.fixture(scope="module")
def auth_page():
    with sync_playwright() as p:
        browser = p.chromium.launch()
        context = browser.new_context(storage_state="mern_auth_state.json")
        page = context.new_page()
        yield page
        browser.close()

# Basic navigation
def test_homepage():
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        page.goto("http://localhost:3000")
        assert page.title() == "Task Manager"

# Navigation with waits
def test_tasks_load(auth_page):
    auth_page.goto("http://localhost:3000/tasks", wait_until="networkidle")
    auth_page.wait_for_selector(".task-list")
    assert "Tasks" in auth_page.title()

# Navigation with retry
def test_dashboard_with_retry(auth_page):
    def navigate_with_retry(page, url, retries=3):
        for attempt in range(retries):
            try:
                page.goto(url, timeout=3000)
                return True
            except Exception as e:
                print(f"Attempt {attempt + 1} failed: {str(e)}")
                time.sleep(1)
        return False
    success = navigate_with_retry(auth_page, "http://localhost:3000/dashboard")
    assert success and "Dashboard" in auth_page.title()

if __name__ == "__main__":
    save_auth_state()  # Run this once to generate auth state
```

**`tests/requirements.txt`**
```
playwright==1.42.0
pytest==8.1.1
```

---

### **4. Root Files**

**`README.md`**
```markdown
# MERN Training App with Playwright Tests

## Setup
1. **Backend**:
   - `cd backend`
   - `npm install`
   - Start MongoDB (`mongod`)
   - Seed a user: `{ username: "testuser", password: "password123" }` in `taskmanager.users`
   - Seed tasks: e.g., `{ title: "Learn MERN", status: "pending" }` in `taskmanager.tasks`
   - `npm start`

2. **Frontend**:
   - `cd frontend`
   - `npm install`
   - `npm start`

3. **Tests**:
   - `cd tests`
   - `pip install -r requirements.txt`
   - `playwright install`
   - Run `python test_navigation.py` (first run generates `mern_auth_state.json`)
   - Then `pytest test_navigation.py`

## Notes
- Backend runs on `http://localhost:5000`.
- Frontend runs on `http://localhost:3000`.
- Adjust selectors in `test_navigation.py` if your app’s DOM differs.
```

---

## **How to Create the Zip**
1. **Create the Folder**:
   - On your computer, create a folder named `mern-training`.
   - Inside it, create subfolders: `backend`, `frontend`, `tests`.
   - Add `README.md` to the root.

2. **Copy Files**:
   - Place the backend files in `backend/`.
   - Place the frontend files in `frontend/` (put `.js` files in `frontend/src/`).
   - Place the test files in `tests/`.

3. **Generate Dependencies** (optional for distribution):
   - Run `npm install` in `backend/` and `frontend/` to generate `node_modules/`.
   - Alternatively, let participants run `npm install` themselves to save space.

4. **Compress**:
   - On Windows: Right-click `mern-training` > "Send to" > "Compressed (zipped) folder".
   - On Mac/Linux: `zip -r mern-training.zip mern-training/`.

5. **Distribute**:
   - Share `mern-training.zip` via email, cloud storage (e.g., Google Drive), or during your session.

---

## **Adjustments**
- **Selectors**: If your MERN app uses different class names (e.g., not `.task-list`), update `test_navigation.py`.
- **Routes**: Modify URLs in `App.js` and `server.js` if your app uses different paths.
- **Auth**: The JWT secret is hardcoded (`secret`); use environment variables in a real app.
