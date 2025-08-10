# 🚀 FastAPI Project Setup Guide with uv

Welcome to the wonderful world of FastAPI! 🎉 If you're here, you're about to embark on a journey to create blazing-fast APIs with Python. Don't worry if you're a complete beginner - we've got your back! This guide will walk you through everything step by step, from installing the tools to running your first API.

## 📋 Table of Contents

1. [What We're Building](#what-were-building)
2. [Prerequisites](#prerequisites)
3. [Installing Python](#installing-python)
4. [Installing uv (The Fast Package Manager)](#installing-uv-the-fast-package-manager)
5. [Setting Up Your Project](#setting-up-your-project)
6. [Installing Dependencies](#installing-dependencies)
7. [Creating Your First FastAPI App](#creating-your-first-fastapi-app)
8. [Running Your Application](#running-your-application)
9. [Testing Your API](#testing-your-api)
10. [Common Issues and Solutions](#common-issues-and-solutions)
11. [Next Steps](#next-steps)

## 🎯 What We're Building

We're going to create a FastAPI application that can handle HTTP requests and return JSON responses. Think of it as the backend brain of a web application - it's where all the magic happens behind the scenes!

**Why FastAPI?**

- It's super fast (hence the name!) ⚡
- Automatically generates interactive API documentation 📖
- Built-in data validation 🛡️
- Easy to learn and use 🎓

**Why uv?**

- It's like npm for Node.js, but for Python and much faster! 🏃‍♂️
- Handles dependencies like a pro 📦
- Creates isolated environments (no more "it works on my machine" problems!) 🏠

## ✅ Prerequisites

Before we dive in, make sure you have:

- A computer (obviously! 😄)
- An internet connection 🌐
- A terminal/command prompt (don't worry, we'll guide you through this!)
- About 30 minutes of your time ⏰

## 🐍 Installing Python

### For Windows Users 🪟

1. **Download Python:**

   - Go to [python.org](https://python.org/downloads/)
   - Click the big yellow "Download Python 3.x.x" button
   - Make sure you get Python 3.8 or newer (FastAPI needs it!)

2. **Install Python:**

   - Run the downloaded installer
   - **IMPORTANT:** Check the box that says "Add Python to PATH" ✅
   - Click "Install Now"
   - Wait for the magic to happen ✨

3. **Verify Installation:**
   - Open Command Prompt (search for "cmd" in Start menu)
   - Type: `python --version`
   - You should see something like `Python 3.11.5`

### For Mac Users 🍎

**Option 1: Using Homebrew (Recommended)**

1. **Install Homebrew first (if you don't have it):**

   - Open Terminal (search for "Terminal" in Spotlight)
   - Paste this command and hit Enter:

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

   - Follow the prompts (it might ask for your password)

2. **Install Python:**

   ```bash
   brew install python
   ```

3. **Verify Installation:**
   ```bash
   python3 --version
   ```

**Option 2: Download from python.org**

- Follow the same steps as Windows users, but download the macOS installer instead

## 🚀 Installing uv (The Fast Package Manager)

Now for the fun part! We're going to install `uv`, which is like a super-powered package manager for Python projects.

### For Windows Users 🪟

Open Command Prompt or PowerShell and run:

```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### For Mac Users 🍎

Open Terminal and run:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Verify uv Installation (Both Platforms):**

```bash
uv --version
```

You should see something like `uv 0.x.x`. If you get a "command not found" error, try closing and reopening your terminal.

## 🏗️ Setting Up Your Project

Time to create your project! This is where the magic begins ✨

### Step 1: Create Your Project Directory

**Windows (Command Prompt):**

```cmd
mkdir my-fastapi-project
cd my-fastapi-project
```

**Mac/Linux (Terminal):**

```bash
mkdir my-fastapi-project
cd my-fastapi-project
```

### Step 2: Initialize Your Project with uv

This creates a new Python project with all the proper structure:

```bash
uv init
```

This command creates:

- `pyproject.toml` - Your project's configuration file (like package.json for Node.js folks)
- A basic project structure
- Virtual environment setup

### Step 3: Activate Your Virtual Environment

Think of a virtual environment as your project's own private Python playground - it keeps all your project's dependencies separate from other projects.

**Windows (CMD):**

```cmd
uv venv
.venv\Scripts\activate
```

**Windows (Powershell):**

```cmd
uv venv
.venv\Scripts\Activate.ps1
```

**Mac/Linux:**

```bash
uv venv
source .venv/bin/activate
```

You'll know it worked when you see `(.venv)` at the beginning of your command prompt! 🎉

## 📦 Installing Dependencies

Now let's install the packages we need. We're keeping it super simple with just the essentials:

### Core Dependencies

```bash
uv add "fastapi[standard]"
```

**What is FastAPI?** It's the web framework that handles HTTP requests and responses.

## 🎨 Creating Your First FastAPI App

Let's create a simple but impressive FastAPI application!

### Step 1: Create the Main Application File

Create a file called `main.py` in your project directory using any code editor like VS Code, Sublime Text, or PyCharm!

### Step 2: Write Your First API

Copy and paste this code into `main.py`:

```python
from fastapi import FastAPI

# Create FastAPI instance
app = FastAPI(
    title="My Awesome FastAPI App",
    description="This is my first FastAPI application! 🚀",
    version="1.0.0"
)

# Root endpoint - your first API endpoint!
@app.get("/")
def read_root():
    return {"message": "Hello World! 🌍"}

# Another simple endpoint
@app.get("/hello/{name}")
def say_hello(name: str):
    return {"message": f"Hello, {name}! 👋"}

# A simple info endpoint
@app.get("/info")
def get_info():
    return {
        "app_name": "My First FastAPI App",
        "version": "1.0.0",
        "status": "running",
        "message": "Welcome to FastAPI! 🚀"
    }
```

**What's happening in this code?**

- We're creating a FastAPI application instance
- We define several simple endpoints that return JSON responses
- The `/hello/{name}` endpoint takes a name from the URL and personalizes the greeting
- Everything returns simple JSON data - no complex validation needed!

## 🏃‍♂️ Running Your Application

Now for the moment of truth! Let's start your API:

### Step 1: Start the Development Server

Make sure you're in your project directory and your virtual environment is activated (you should see `(.venv)` in your prompt).

```bash
fastapi dev main.py
```

**What does this command mean?**

- `fastapi` - The fastapi command to start a server
- `dev` - Run the API application in development mode (used for local environment)
- `main.py` - The entry point of the application where we initialise the app.

### Step 2: Success! 🎉

If everything worked, you should see something like:

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [xxxxx] using WatchFiles
INFO:     Started server process [xxxxx]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

Your API is now running on `http://127.0.0.1:8000` (also known as `localhost:8000`)!

## 🧪 Testing Your API

### Method 1: Using Your Browser

Open your web browser and visit:

- `http://localhost:8000` - Your homepage with the "Hello World" message
- `http://localhost:8000/hello/YourName` - Replace "YourName" with your actual name!
- `http://localhost:8000/info` - App information 🤯

### Method 2: Interactive API Documentation

Visit `http://localhost:8000/docs` in your browser. You'll see:

- All your API endpoints listed
- The ability to test each endpoint directly from the browser
- Automatic request/response examples

Try clicking on any endpoint and hitting the "Try it out" button!

## 🆘 Common Issues and Solutions

### "Command not found" errors

**Problem:** Terminal says `python`, `uv`, or `uvicorn` command not found.

**Solution:**

- Make sure you installed everything correctly
- Try closing and reopening your terminal
- On Windows, make sure you checked "Add Python to PATH" during installation

### Port already in use

**Problem:** Error message about port 8000 already being used.

**Solution:**

- Stop any other servers running on port 8000
- Or use a different port: `uvicorn main:app --reload --port 8080`

### Virtual environment issues

**Problem:** Packages not found or import errors.

**Solution:**

- Make sure your virtual environment is activated (you should see `(.venv)` in your prompt)
- If not activated, run the activation command again

### Import errors

**Problem:** `ModuleNotFoundError` when running your app.

**Solution:**

- Make sure you installed all dependencies with `uv add`
- Check that you're in the correct directory
- Verify your virtual environment is activated

## 🎯 Next Steps

You did it! You've successfully set up a FastAPI project with uv. You now have:

- ✅ A working FastAPI application with simple "Hello World" endpoints
- ✅ Automatic API documentation
- ✅ A proper development environment
- ✅ The foundation for building amazing APIs

In the next lesson, we will learn how to connect this app to a database and performs CRUD operations (Create, Read, Update, Delete) on the data of this database!

Happy coding! 🚀✨

---

_Made with ❤️ for Curtin students learning web development_
