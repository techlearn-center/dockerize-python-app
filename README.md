# Dockerize a Python Application

> **What you'll create:** A production-ready Docker container for a Python web API, using best practices that companies actually use.

---

## Learning Objectives

By the end of this challenge, you will be able to:

1. **Explain** what Docker is, why it exists, and the problems it solves
2. **Write** a Dockerfile that packages a Python app into a container
3. **Build** a multi-stage Docker image under 200MB (down from 1.2GB)
4. **Apply** security best practices (non-root user, minimal base image)
5. **Create** a Docker Compose file to run multi-container applications
6. **Use** Docker commands confidently to build, run, and debug containers

---

## Quick Start

```bash
# 1. Fork this repo to your GitHub account

# 2. Clone YOUR fork
git clone https://github.com/YOUR-USERNAME/dockerize-python-app.git
cd dockerize-python-app

# 3. Complete the challenge (see steps below)

# 4. Push and check your score
git add .
git commit -m "Complete Docker challenge"
git push origin main
# → Check GitHub Actions tab for your score!
```

---

## What is This Challenge?

You have a working Python API. Your job is to **containerize it** so it can run anywhere - your laptop, a server, or the cloud.

**By the end, you'll have:**
- A Dockerfile using multi-stage builds
- An image under 200MB (not 1.2GB!)
- Security best practices (non-root user)
- Docker Compose for local development

---

## Do I Need to Know Docker Already?

**No!** This challenge teaches you Docker from scratch. Zero prior Docker knowledge required.

**You need:**
- Basic Python knowledge
- Basic terminal/command line usage

**You'll learn:**
- What containers are and why they matter
- How to write Dockerfiles
- Multi-stage builds for smaller images
- Docker Compose for multi-container apps

---

## What You'll Build

| File | What You Create | Points |
|------|-----------------|--------|
| `Dockerfile` | Multi-stage build for the API | 55 |
| `docker-compose.yml` | Local dev environment with Redis | 15 |
| `.dockerignore` | Exclude unnecessary files | 5 |
| Security & best practices | Non-root user, health checks | 25 |

---

## Step 0: Install Docker

### Why Does Docker Exist? The Problem.

Imagine this scenario:

> You spent two weeks building a Python web app. It works perfectly on your laptop. You hand it to your teammate, and they get:
>
> ```
> ModuleNotFoundError: No module named 'flask'
> ```
>
> They install Flask. Now they get:
>
> ```
> ERROR: Python 3.8 is not compatible with this package (requires >=3.11)
> ```
>
> They upgrade Python. Now the OS is missing a system library. They fix that, and now their database driver version conflicts with yours. **Three hours later, they still can't run your app.**
>
> Then your boss says: "Deploy it to the server." The server runs a different Linux distribution, has a different Python version, different system libraries... and the whole nightmare starts again.

This is called the **"it works on my machine"** problem, and it has plagued software teams for decades.

### The Solution: Containers

Docker solves this by packaging your application **and everything it needs** (Python, libraries, system tools, configuration) into a single, portable unit called a **container**.

```
Without Docker:                          With Docker:
┌──────────────────────────────────┐     ┌──────────────────────────────────┐
│  Your Laptop                     │     │  Your Laptop                     │
│  - Python 3.11                   │     │  ┌────────────────────────────┐  │
│  - Flask 3.0                     │     │  │ Container                  │  │
│  - Ubuntu 22.04                  │     │  │ Python 3.11 + Flask 3.0   │  │
│  - Works!                        │     │  │ + your app + everything   │  │
├──────────────────────────────────┤     │  │ = IDENTICAL everywhere    │  │
│  Teammate's Laptop               │     │  └────────────────────────────┘  │
│  - Python 3.8      ← WRONG      │     ├──────────────────────────────────┤
│  - Flask 2.1       ← WRONG      │     │  Teammate's Laptop               │
│  - macOS           ← DIFFERENT   │     │  ┌────────────────────────────┐  │
│  - BROKEN!                       │     │  │ Same Container             │  │
├──────────────────────────────────┤     │  │ Python 3.11 + Flask 3.0   │  │
│  Production Server               │     │  │ = IDENTICAL               │  │
│  - Python 3.9      ← WRONG      │     │  └────────────────────────────┘  │
│  - CentOS 7        ← DIFFERENT   │     ├──────────────────────────────────┤
│  - BROKEN!                       │     │  Production Server               │
└──────────────────────────────────┘     │  ┌────────────────────────────┐  │
                                         │  │ Same Container             │  │
                                         │  │ Python 3.11 + Flask 3.0   │  │
                                         │  │ = IDENTICAL               │  │
                                         │  └────────────────────────────┘  │
                                         └──────────────────────────────────┘
```

### Real-World Analogy: Shipping Containers

Before the 1950s, shipping goods was chaos. Every port loaded cargo differently — bags, barrels, crates of all sizes. Every ship had a different layout. Loading took weeks. Goods were damaged. It was expensive.

Then someone invented the **standardized shipping container** — a metal box with fixed dimensions. It didn't matter what was inside (electronics, food, furniture). Every ship, truck, and train could carry the same box. Loading went from weeks to hours.

**Docker containers are the same idea for software:**

```
Shipping Container                Docker Container
┌─────────────────────────┐      ┌─────────────────────────┐
│ Standard size            │      │ Standard format          │
│ Works on any ship/truck  │      │ Works on any OS/cloud    │
│ Contents don't matter    │      │ Language doesn't matter  │
│ Sealed and isolated      │      │ Isolated from host       │
│ Stack and scale easily   │      │ Scale to thousands       │
└─────────────────────────┘      └─────────────────────────┘
```

### Docker Architecture

When you use Docker, three components work together:

```
You (CLI)              Docker Daemon              Registry (Docker Hub)
┌──────────┐          ┌──────────────┐           ┌──────────────────┐
│ docker   │  ─────►  │ Builds images│  ◄─────►  │ Stores/shares    │
│ build    │          │ Runs         │           │ images publicly  │
│ docker   │          │ containers   │           │ (like GitHub for │
│ run      │          │ Manages      │           │  Docker images)  │
│ docker   │          │ networking   │           │                  │
│ pull     │          │              │           │ python:3.11      │
└──────────┘          └──────────────┘           │ redis:7-alpine   │
                                                 │ nginx:latest     │
                                                 └──────────────────┘
```

- **Docker CLI** — The commands you type (`docker build`, `docker run`)
- **Docker Daemon** — The background service that does the actual work
- **Registry** — A library of pre-built images (like an app store for containers)

### Containers vs Virtual Machines

You may have heard of virtual machines (VMs). Containers are lighter:

```
Virtual Machines                    Containers
┌─────────┐ ┌─────────┐           ┌─────────┐ ┌─────────┐
│  App A  │ │  App B  │           │  App A  │ │  App B  │
├─────────┤ ├─────────┤           ├─────────┤ ├─────────┤
│  Libs   │ │  Libs   │           │  Libs   │ │  Libs   │
├─────────┤ ├─────────┤           └────┬────┘ └────┬────┘
│ Guest OS│ │ Guest OS│                │           │
│ (1-2GB) │ │ (1-2GB) │           ┌────┴───────────┴────┐
├─────────┴─┴─────────┤           │   Docker Engine      │
│     Hypervisor       │           ├──────────────────────┤
├──────────────────────┤           │     Host OS          │
│      Host OS         │           ├──────────────────────┤
├──────────────────────┤           │     Hardware         │
│      Hardware        │           └──────────────────────┘
└──────────────────────┘
                                   No guest OS needed!
Startup: minutes                   Startup: seconds
Size: gigabytes                    Size: megabytes
```

**Key takeaway:** Containers share the host OS kernel, so they're much faster and smaller than VMs.

### Install Docker Desktop

<details>
<summary>Windows (Docker Desktop - Recommended)</summary>

1. Go to [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)
2. Download "Docker Desktop for Windows"
3. Run the installer
4. **Important:** Enable WSL 2 when prompted
5. Restart your computer
6. Open Docker Desktop and wait for "Docker is running"

**Verify:**
```bash
docker --version
# Should show: Docker version 24.x or higher
```

</details>

<details>
<summary>Windows (Chocolatey)</summary>

If you have [Chocolatey](https://chocolatey.org/) installed, you can install Docker Desktop with one command:

```powershell
# Run PowerShell as Administrator
choco install docker-desktop -y
```

After installation:
1. Restart your computer
2. Open Docker Desktop
3. Wait for "Docker is running"

**Don't have Chocolatey?** Install it first:
```powershell
# Run PowerShell as Administrator
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Close and reopen PowerShell, then install Docker:
choco install docker-desktop -y
```

**Verify:**
```bash
docker --version
# Should show: Docker version 24.x or higher
```

</details>

<details>
<summary>Mac</summary>

1. Go to [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)
2. Download for your chip:
   - **Apple Silicon (M1/M2/M3):** "Mac with Apple chip"
   - **Intel:** "Mac with Intel chip"
3. Drag to Applications
4. Open Docker Desktop
5. Wait for "Docker is running"

**Verify:**
```bash
docker --version
```

</details>

<details>
<summary>Linux</summary>

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to docker group (so you don't need sudo)
sudo usermod -aG docker $USER

# Log out and back in, then verify:
docker --version
```

</details>

### Test Docker is Working

```bash
docker run hello-world
```

You should see: "Hello from Docker! This message shows that your installation appears to be working correctly."

**What just happened?** Docker pulled a tiny image called `hello-world` from Docker Hub (the registry), created a container from it, ran it, and printed a message. That's the entire Docker workflow in one command.

### Checkpoint: Self-Reflection

Before moving on, make sure you can answer these:

- [ ] **Q1:** In your own words, what problem does Docker solve?
- [ ] **Q2:** What is the difference between a container and a virtual machine?
- [ ] **Q3:** What are the three components of Docker architecture? (CLI, _____, _____)
- [ ] **Q4:** When you ran `docker run hello-world`, what did Docker do behind the scenes?

---

## Step 1: Understand the App

Before containerizing, let's understand what we're working with.

### The Application

Look at `src/app.py` - it's a simple Flask API with these endpoints:

| Endpoint | What It Does |
|----------|-------------|
| `/health` | Returns health status (used by Docker health checks) |
| `/api/greeting/<name>` | Returns a personalized greeting |
| `/api/counter` | Counts visits (uses Redis if available) |
| `/api/info` | Shows system info (useful for debugging containers) |

Here's the core of the app:

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/health')
def health():
    return jsonify({"status": "healthy"})

@app.route('/api/greeting/<name>')
def greeting(name):
    return jsonify({"message": f"Hello, {name}!"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Run It Locally First

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run the app
python src/app.py
```

Test it: http://localhost:5000/health

**Notice what you just did:** You installed Python, created a virtual environment, installed dependencies, and ran the app. That's 4 manual steps. What if your teammate has a different Python version? What if the server doesn't have `venv`? This is exactly what Docker automates.

### Checkpoint: Self-Reflection

- [ ] **Q1:** What would happen if your teammate has Python 3.8 instead of 3.11?
- [ ] **Q2:** How many manual steps did it take to run this app? Could you automate all of them?
- [ ] **Q3:** What does `host='0.0.0.0'` mean? Why not `localhost`? (Hint: think about containers)

---

## Step 2: Understand Docker Concepts

Before writing a Dockerfile, let's understand the key concepts.

### Images vs Containers

Think of it like cooking:

| Concept | What It Is | Analogy |
|---------|-----------|---------|
| **Dockerfile** | Instructions to build an image | A recipe card |
| **Image** | A blueprint/template (read-only) | A recipe |
| **Container** | A running instance of an image | A cooked meal |

You **write** a Dockerfile once, **build** it into an image, then **run** as many containers as you want from that image.

```
Dockerfile          docker build         docker run        docker run
(recipe card)  ──────────────────►  Image  ──────────►  Container 1
                                   (recipe)──────────►  Container 2
                                           ──────────►  Container 3
                                                        (3 meals from
                                                         1 recipe!)
```

**Key insight:** You can run 100 containers from 1 image. Each container is isolated — if one crashes, the others keep running.

### Image Layers: How Docker Builds Efficiently

Every instruction in a Dockerfile creates a **layer**. Docker caches these layers so it doesn't rebuild everything when you make a small change.

```
Dockerfile                           Image Layers
┌──────────────────────────┐        ┌──────────────────────────┐
│ FROM python:3.11-slim    │   ──►  │ Layer 1: Python runtime  │ 120MB (cached)
│ WORKDIR /app             │   ──►  │ Layer 2: Set directory   │ 0MB   (cached)
│ COPY requirements.txt .  │   ──►  │ Layer 3: Copy deps file  │ 1KB   (cached)
│ RUN pip install -r ...   │   ──►  │ Layer 4: Install deps    │ 25MB  (cached)
│ COPY src/ ./src/         │   ──►  │ Layer 5: Copy app code   │ 5KB   (rebuilt)
│ CMD ["python","app.py"]  │   ──►  │ Layer 6: Startup command │ 0MB   (rebuilt)
└──────────────────────────┘        └──────────────────────────┘
```

**Why this matters:** If you only change your source code (`src/`), Docker reuses layers 1-4 from cache and only rebuilds layers 5-6. This makes builds fast (seconds instead of minutes).

**This is why we copy `requirements.txt` BEFORE copying `src/`** — dependencies rarely change, so that layer stays cached.

### Dockerfile Instructions

| Instruction | What It Does | Example |
|-------------|-------------|---------|
| `FROM` | Base image to start from | `FROM python:3.11` |
| `WORKDIR` | Set working directory | `WORKDIR /app` |
| `COPY` | Copy files into image | `COPY . .` |
| `RUN` | Execute a command during build | `RUN pip install flask` |
| `EXPOSE` | Document which port the app uses | `EXPOSE 5000` |
| `CMD` | Command to run when container starts | `CMD ["python", "app.py"]` |
| `ENV` | Set environment variables | `ENV FLASK_ENV=production` |
| `USER` | Switch to a non-root user | `USER appuser` |
| `HEALTHCHECK` | Define how Docker checks if app is healthy | `HEALTHCHECK CMD curl ...` |

### A Simple (Naive) Dockerfile

Here's the simplest Dockerfile that would work:

```dockerfile
# Start from Python base image
FROM python:3.11

# Set working directory
WORKDIR /app

# Copy everything
COPY . .

# Install dependencies
RUN pip install -r requirements.txt

# Document the port
EXPOSE 5000

# Run the app
CMD ["python", "src/app.py"]
```

**This works, but it has serious problems.** Don't worry — you'll discover them yourself in the next step.

### Checkpoint: Self-Reflection

- [ ] **Q1:** What's the difference between an image and a container?
- [ ] **Q2:** Can you run 5 containers from the same image? What happens to each container's data?
- [ ] **Q3:** Why do we copy `requirements.txt` separately before copying `src/`?
- [ ] **Q4:** What does `EXPOSE 5000` actually do? (Trick question — it's documentation only!)

---

## Step 3: Write Your Dockerfile

This is the core of the challenge. We'll do it in three phases so you understand **why** each optimization matters.

### Phase A: Build the Naive Dockerfile First

Open the `Dockerfile` in this project and write the simple version from Step 2:

```dockerfile
FROM python:3.11
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 5000
CMD ["python", "src/app.py"]
```

Now build and inspect it:

```bash
# Build it
docker build -t myapp-naive .

# Check the image size
docker images myapp-naive
```

**You should see something like:**
```
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
myapp-naive   latest    abc123def456   10 seconds ago   1.02 GB
```

**1 GB for a tiny Flask API?!** That's because `python:3.11` is a full Debian Linux image with compilers, development headers, documentation, and tons of stuff your app doesn't need.

Now run it:

```bash
docker run -p 5000:5000 myapp-naive
```

Test: http://localhost:5000/health — it works! But let's check the security:

```bash
# Check who the app runs as
docker run myapp-naive whoami
# Output: root    ← This is a security risk!
```

### Phase B: Identify the Problems

| Problem | Why It's Bad | How to Fix |
|---------|-------------|------------|
| Image is ~1GB | Slow to download, wastes storage, larger attack surface | Use `python:3.11-slim` (120MB vs 900MB) |
| Runs as `root` | If attacker breaks into the container, they have full control | Create and use a non-root user |
| No health check | Docker/Kubernetes can't tell if your app is actually responding | Add `HEALTHCHECK` instruction |
| Copies unnecessary files | `.git/`, `venv/`, `__pycache__` bloat the image | Use `.dockerignore` |
| Single stage | Build tools and source remain in final image | Use multi-stage build |

### Phase C: Write the Optimized Multi-Stage Dockerfile

Now that you've seen the problems, let's fix them all.

**What is Multi-Stage Building?**

Multi-stage builds use multiple `FROM` statements. The first stage installs dependencies (which may need compilers). The second stage copies only what's needed into a clean, slim image.

```
Stage 1 "builder":                 Stage 2 "final":
┌───────────────────────────┐      ┌───────────────────────────┐
│ FROM python:3.11          │      │ FROM python:3.11-slim     │
│                           │      │                           │
│ Full Python + build tools │      │ Slim Python only          │
│ + compilers               │      │ + installed packages      │
│ + dev headers             │ COPY │ + your app code           │
│ + your source code        │ ───► │                           │
│ + installed packages      │      │ Non-root user             │
│                           │      │ Health check              │
│ = ~1.2 GB                 │      │ = ~150 MB                 │
│                           │      │                           │
│ (Thrown away after build!) │      │ (This is what you ship!)  │
└───────────────────────────┘      └───────────────────────────┘
```

### Your Task

Open `Dockerfile` and replace the naive version with a multi-stage build.

**Requirements:**
- [ ] Use `python:3.11-slim` for the final image (not full Python)
- [ ] Final image must be under 200MB
- [ ] Run as non-root user
- [ ] Include a health check
- [ ] Don't copy unnecessary files (use `.dockerignore`)

### Step-by-Step Hints

<details>
<summary>Hint 1: Multi-Stage Structure</summary>

```dockerfile
# Stage 1: Builder
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2: Final
FROM python:3.11-slim
WORKDIR /app

# Copy installed packages from builder
COPY --from=builder /root/.local /root/.local

# Copy application code
COPY src/ ./src/

# ... rest of your Dockerfile
```

</details>

<details>
<summary>Hint 2: Non-Root User</summary>

Running as root is a security risk. Create a dedicated user:

```dockerfile
# Create non-root user
RUN useradd --create-home appuser
USER appuser
```

But wait! If you copy to `/root/.local`, the non-root user can't access it.

**Fix:** Install packages to a location the user can access:

```dockerfile
# In builder stage:
RUN pip install --prefix=/install -r requirements.txt

# In final stage:
COPY --from=builder /install /usr/local
```

</details>

<details>
<summary>Hint 3: Health Check</summary>

Docker can automatically check if your app is healthy:

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1
```

But `curl` isn't installed in slim images! Use Python instead:

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:5000/health')" || exit 1
```

</details>

<details>
<summary>Hint 4: Environment Variables</summary>

Don't hardcode settings. Use environment variables:

```dockerfile
ENV FLASK_APP=src/app.py
ENV FLASK_RUN_HOST=0.0.0.0
ENV FLASK_RUN_PORT=5000
```

</details>

<details>
<summary>Full Solution (only if completely stuck!)</summary>

```dockerfile
# ============================================
# Stage 1: Builder
# ============================================
FROM python:3.11 AS builder

WORKDIR /app

# Install dependencies to a specific location
COPY requirements.txt .
RUN pip install --prefix=/install --no-cache-dir -r requirements.txt

# ============================================
# Stage 2: Final (Production)
# ============================================
FROM python:3.11-slim

# Create non-root user for security
RUN useradd --create-home --shell /bin/bash appuser

WORKDIR /app

# Copy installed packages from builder
COPY --from=builder /install /usr/local

# Copy application code
COPY --chown=appuser:appuser src/ ./src/

# Set environment variables
ENV FLASK_APP=src/app.py
ENV FLASK_RUN_HOST=0.0.0.0
ENV FLASK_RUN_PORT=5000
ENV PYTHONUNBUFFERED=1

# Switch to non-root user
USER appuser

# Document the port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:5000/health')" || exit 1

# Run the application
CMD ["python", "src/app.py"]
```

</details>

### Test Your Dockerfile

```bash
# Build the optimized image
docker build -t myapp .

# Compare sizes!
docker images | grep myapp
# myapp-naive   latest   ...   1.02 GB
# myapp         latest   ...   ~150 MB   ← 85% smaller!

# Run the container
docker run -p 5000:5000 myapp

# Test it
curl http://localhost:5000/health

# Verify it runs as non-root
docker run myapp whoami
# Output: appuser   ← Much better!
```

### Common Mistakes

| Error | Cause | Fix |
|-------|-------|-----|
| `pip: command not found` | Wrong base image | Use `python:3.11-slim`, not `alpine` |
| Permission denied | Running as non-root but files owned by root | Use `COPY --chown=appuser:appuser` |
| Health check fails | `curl` not installed | Use Python urllib instead |
| Image too big | Using full Python image in final stage | Use `python:3.11-slim` in final stage |
| Cache not working | Copying all files before installing deps | Copy `requirements.txt` first, then `src/` |

### Checkpoint: Self-Reflection

- [ ] **Q1:** Why do we use TWO `FROM` statements? What happens to the first stage?
- [ ] **Q2:** Your image went from ~1GB to ~150MB. Where did the 850MB go?
- [ ] **Q3:** Why do we copy `requirements.txt` before copying `src/`? What would break if we didn't?
- [ ] **Q4:** Why is running as root inside a container a security risk?
- [ ] **Q5:** The `HEALTHCHECK` uses Python's `urllib` instead of `curl`. Why?

---

## Step 4: Create .dockerignore

### Why .dockerignore?

When you run `docker build`, Docker sends **ALL files** in the current directory to the Docker daemon (this is called the "build context"). Without a `.dockerignore`, Docker sends your `.git/` folder, `venv/`, `__pycache__/`, and other junk — making builds slower and images larger.

Think of it like `.gitignore`, but for Docker builds.

### Your Task

Create a `.dockerignore` file with these patterns:

```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
venv/
.venv/
ENV/

# IDE
.vscode/
.idea/
*.swp

# Git
.git/
.gitignore

# Docker
Dockerfile
docker-compose*.yml
.dockerignore

# Tests (not needed in production image)
tests/
pytest.ini

# Misc
*.md
*.txt
!requirements.txt
```

**Note:** `!requirements.txt` means "DO include requirements.txt" (exception to the `*.txt` rule).

### Checkpoint: Self-Reflection

- [ ] **Q1:** What is the "build context" and why does it matter?
- [ ] **Q2:** What would happen if `.git/` was included in your Docker image? (Hint: size and security)
- [ ] **Q3:** Why do we exclude `Dockerfile` itself from the build context?

---

## Step 5: Create Docker Compose

### What is Docker Compose?

So far you've been running one container at a time. But real applications have multiple services: a web app, a database, a cache, a message queue...

**Docker Compose** lets you define and run all of them with a single command.

**Without Compose (painful):**
```bash
docker network create mynet
docker run -d --name redis --network mynet redis:7-alpine
docker run -d --name api --network mynet -p 5000:5000 -e REDIS_URL=redis://redis:6379 myapp
# Lots of commands to remember! And what about volumes? Health checks? Restart policies?
```

**With Compose (easy):**
```bash
docker compose up
# One command does everything!
```

### How Container Networking Works

When Docker Compose starts multiple containers, it creates a private network. Containers talk to each other using their **service names** as hostnames:

```
┌──────────────────────────────────────────────────┐
│  Docker Compose Network (automatic)              │
│                                                  │
│  ┌──────────────┐       ┌──────────────┐        │
│  │  api          │       │  redis        │        │
│  │  (your Flask  │──────►│  (cache)      │        │
│  │   app)        │ redis://redis:6379    │        │
│  │              │       │              │        │
│  │  port 5000   │       │  port 6379   │        │
│  └──────┬───────┘       └──────────────┘        │
│         │                                        │
└─────────┼────────────────────────────────────────┘
          │
          │ port 5000 mapped to host
          ▼
    http://localhost:5000
    (your browser)
```

**Key insight:** The API container can reach Redis at `redis://redis:6379` because Docker Compose automatically creates DNS entries using the service name.

### Your Task

Create `docker-compose.yml` that:
- [ ] Runs your API
- [ ] Runs a Redis cache (for future use)
- [ ] Sets up networking between them
- [ ] Uses volumes for development (live reload)

### Step-by-Step Hints

<details>
<summary>Hint 1: Basic Structure</summary>

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=development
```

</details>

<details>
<summary>Hint 2: Add Redis</summary>

```yaml
services:
  api:
    # ... your api config
    depends_on:
      - redis
    environment:
      - REDIS_URL=redis://redis:6379

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

</details>

<details>
<summary>Hint 3: Development Volumes</summary>

For development, mount your code so changes reflect immediately:

```yaml
services:
  api:
    volumes:
      - ./src:/app/src:ro  # Mount source code (read-only)
    environment:
      - FLASK_ENV=development
      - FLASK_DEBUG=1
```

</details>

<details>
<summary>Hint 4: Health Checks in Compose</summary>

```yaml
services:
  api:
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:5000/health')"]
      interval: 30s
      timeout: 3s
      retries: 3
```

</details>

<details>
<summary>Full Solution</summary>

```yaml
version: '3.8'

services:
  # Python API
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=development
      - FLASK_DEBUG=1
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./src:/app/src:ro
    depends_on:
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:5000/health')"]
      interval: 30s
      timeout: 3s
      retries: 3
    restart: unless-stopped

  # Redis cache
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    restart: unless-stopped

volumes:
  redis_data:
```

</details>

### Test Docker Compose

```bash
# Start everything
docker compose up --build

# In another terminal, test the API
curl http://localhost:5000/health
curl http://localhost:5000/api/counter    # Uses Redis!
curl http://localhost:5000/api/counter    # Count increases!

# Stop everything
docker compose down
```

### Checkpoint: Self-Reflection

- [ ] **Q1:** How does the API container know where Redis is? What hostname does it use?
- [ ] **Q2:** What does `depends_on` with `condition: service_healthy` do?
- [ ] **Q3:** What's the difference between a bind mount (`./src:/app/src`) and a named volume (`redis_data:/data`)?
- [ ] **Q4:** What happens to the data in `redis_data` when you run `docker compose down`? What about `docker compose down -v`?

---

## Step 6: Run Tests & Submit

### Local Testing

```bash
python run.py
```

You'll see:
```
  ============================================================
    Dockerize Python App Challenge
  ============================================================

  [Step 1] Dockerfile exists
  [Step 2] Multi-stage build used
  [Step 3] Image size < 200MB (142MB)
  [Step 4] Non-root user configured
  [Step 5] Health check defined
  [Step 6] .dockerignore exists
  [Step 7] docker-compose.yml works

  Overall Progress:
  ████████████████████ 100% (7/7 steps)

  CHALLENGE COMPLETE!
```

### Submit to GitHub

```bash
git add .
git commit -m "Complete Docker challenge"
git push origin main
```

Check the **Actions** tab for your score!

---

## Final Self-Reflection

Before you move on, test your understanding. Try answering these without looking back:

1. **Explain to a non-technical friend** what Docker does and why it matters. Use an analogy.

2. **Your image went from 1GB to 150MB.** Walk through each optimization that contributed to this reduction. Where did the savings come from?

3. **Why does the order of instructions in a Dockerfile matter?** What happens to build time if you put `COPY . .` before `RUN pip install`?

4. **A junior developer on your team commits a Dockerfile that runs as root.** What specific risk does this create? How would you explain it to them?

5. **Your docker-compose.yml has two services: `api` and `redis`.** Without any extra configuration, the API can reach Redis at `redis://redis:6379`. How does Docker make this work? (Hint: DNS)

If you can answer all 5 confidently, you have a solid understanding of Docker fundamentals.

---

## Glossary

| Term | Definition |
|------|-----------|
| **Image** | A read-only template containing your application, dependencies, and OS libraries. Built from a Dockerfile. |
| **Container** | A running instance of an image. Isolated, lightweight, and disposable. |
| **Dockerfile** | A text file with instructions for building a Docker image (like a recipe). |
| **Layer** | Each Dockerfile instruction creates a layer. Layers are cached to speed up rebuilds. |
| **Build Context** | The set of files sent to the Docker daemon when you run `docker build`. Controlled by `.dockerignore`. |
| **Registry** | A storage and distribution service for Docker images. Docker Hub is the default public registry. |
| **Tag** | A label for a specific version of an image (e.g., `python:3.11-slim`, `myapp:v2.0`). |
| **Volume** | Persistent storage that survives container restarts and removals. |
| **Bind Mount** | Maps a host directory into a container (e.g., `./src:/app/src`). Changes are reflected immediately. |
| **Service** | In Docker Compose, a service is one container definition (e.g., `api`, `redis`). |
| **Daemon** | The Docker background process that builds images, runs containers, and manages networking. |
| **Multi-Stage Build** | A Dockerfile technique using multiple `FROM` statements to reduce final image size. |

---

## Key Concepts Summary

| Concept | What It Is | Why It Matters |
|---------|-----------|----------------|
| **Image** | Packaged app + dependencies | Consistent deployments |
| **Container** | Running instance | Isolated, reproducible |
| **Multi-stage** | Build in one image, run in another | Smaller final images |
| **Layers** | Each instruction creates a layer | Caching, efficiency |
| **Non-root** | Don't run as root | Security |
| **Health check** | App self-reporting | Orchestration, monitoring |

---

## Docker Commands Cheat Sheet

```bash
# Build
docker build -t myapp .              # Build image
docker build -t myapp:v1.0 .         # Build with tag
docker build --no-cache -t myapp .   # Build without cache

# Run
docker run myapp                     # Run container
docker run -d myapp                  # Run in background (detached)
docker run -p 5000:5000 myapp        # Map port (host:container)
docker run -e VAR=value myapp        # Set environment variable
docker run -v ./src:/app/src myapp   # Mount volume
docker run --rm myapp                # Auto-remove when stopped
docker run --name my-api myapp       # Give container a name

# Inspect
docker ps                            # List running containers
docker ps -a                         # List all containers (including stopped)
docker logs <id>                     # View container logs
docker logs -f <id>                  # Follow logs in real-time
docker exec -it <id> /bin/bash      # Open shell inside running container
docker inspect <id>                  # Detailed container info

# Manage
docker stop <id>                     # Stop container
docker rm <id>                       # Remove container
docker stop $(docker ps -q)          # Stop all running containers

# Images
docker images                        # List images
docker rmi <image>                   # Remove image
docker image prune                   # Remove unused images

# Compose
docker compose up                    # Start all services
docker compose up -d                 # Start in background
docker compose up --build            # Rebuild images first
docker compose down                  # Stop and remove containers
docker compose down -v               # Also remove volumes
docker compose logs                  # View all logs
docker compose ps                    # List running services
```

---

## What You Can Say in Interviews

> "I containerized a Python application using Docker multi-stage builds, reducing the image size from 1.2GB to 150MB. I implemented security best practices including running as a non-root user, added health checks for orchestration compatibility, and created a Docker Compose configuration for local development with Redis. I understand Docker layers, caching, and how to optimize builds."

**Follow-up questions you can now answer:**
- "What's the difference between `COPY` and `ADD`?" → COPY is simpler and preferred. ADD can extract tars and fetch URLs, but that's rarely needed.
- "Why multi-stage builds?" → To separate build-time dependencies from runtime, reducing image size and attack surface.
- "How do you handle secrets in Docker?" → Never put them in the image. Use environment variables, Docker secrets, or a vault.
- "What's the difference between `CMD` and `ENTRYPOINT`?" → CMD sets the default command (can be overridden). ENTRYPOINT sets the executable (CMD becomes its arguments).

---

## Troubleshooting

<details>
<summary>"docker: command not found"</summary>

Docker Desktop is not installed or not in PATH.

1. Install Docker Desktop (see Step 0)
2. Make sure it's running (look for whale icon)
3. Restart your terminal

</details>

<details>
<summary>"Cannot connect to Docker daemon"</summary>

Docker Desktop is installed but not running.

1. Open Docker Desktop
2. Wait for "Docker is running"
3. Try again

</details>

<details>
<summary>Image is too big (> 200MB)</summary>

Check these:
1. Using `python:3.11-slim` in final stage (not `python:3.11`)?
2. Using `--no-cache-dir` with pip?
3. Not copying unnecessary files?
4. `.dockerignore` configured?
5. Using multi-stage build (not single stage)?

</details>

<details>
<summary>Health check keeps failing</summary>

1. Is the app actually running? Check logs: `docker logs <container_id>`
2. Is the health endpoint correct? `/health` not `/healthz`
3. Is curl installed? Use Python urllib instead
4. Is the `--start-period` long enough for the app to boot?

</details>

<details>
<summary>"Permission denied" errors</summary>

This usually means you're running as a non-root user but files are owned by root.

Fix: Use `COPY --chown=appuser:appuser` when copying files in the Dockerfile.

</details>

<details>
<summary>Container starts but can't connect to Redis</summary>

1. Is Redis running? Check `docker compose ps`
2. Is the `REDIS_URL` environment variable set correctly?
3. Did you use the service name (`redis`) as the hostname, not `localhost`?
4. Does `depends_on` have `condition: service_healthy`?

</details>

---

## What's Next?

You've learned Docker fundamentals. Here's where to go from here:

| Next Step | What You'll Learn | Challenge |
|-----------|------------------|-----------|
| **CI/CD Pipeline** | Automate testing and deployment with Docker | Challenge 1.3 |
| **Kubernetes Basics** | Orchestrate containers at scale | Challenge 2.1 |
| **Docker Networking** | Connect containers across hosts | Deep dive |
| **Docker Registries** | Push images to Docker Hub or private registries | Deep dive |

**Recommended learning path:**
1. **CI/CD Pipeline** (next challenge) — Automate building and deploying your Docker images
2. **Kubernetes** — When one server isn't enough, orchestrate containers across a cluster
3. **Monitoring** — Add observability to your containerized applications
