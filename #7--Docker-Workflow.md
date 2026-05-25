# 🐳 Docker Workflow 


## 📋 Topics Covered

| # | Topic |
|---|---|
| 1 | Workflow Overview |
| 2 | Stage 1 — Development |
| 3 | Stage 2 — Continuous Delivery (CI) |
| 4 | Stage 3 — Deployment |
| 5 | Full Pipeline Diagram |

---

## 🔄 Workflow Overview

Docker fits into **3 stages** of a real development pipeline:

```
Development  →  Continuous Delivery (CI)  →  Deployment
```

---

## 💻 Stage 1 — Development

The developer works locally on an app (e.g., a JavaScript app with MongoDB).

**Without Docker:**
- Every team member installs MongoDB differently
- "Works on my machine" problems are common
- Version mismatches cause bugs

**With Docker:**
- Pull MongoDB image from Docker Hub — no local install needed
- Everyone on the team uses the exact same environment

```bash
# Pull MongoDB instead of installing it locally
docker pull mongo

# Run it as a container
docker run -d -p27017:27017 --name mongodb mongo
```

```
Developer's Laptop
┌──────────────────────────────────┐
│  JS App (your code)              │
│  MongoDB Container (Docker Hub)  │ ← pulled, not installed
└──────────────────────────────────┘
```

> ✅ No installation conflicts. Same version for the whole team.

---

## 🔁 Stage 2 — Continuous Delivery (CI)

Once the developer pushes code to **Git**, a **CI tool** (like Jenkins) takes over automatically.

**What CI does:**
1. Pulls the latest code from Git
2. Builds the JS application
3. Packages it as a **Docker image**
4. Pushes the new image to a **Docker repository** (private or Docker Hub)

```
Git Push  →  CI (Jenkins)  →  Build App  →  Create Docker Image  →  Push to Repo
```

```bash
# CI builds your app into a Docker image
docker build -t my-js-app:1.0 .

# CI pushes it to a Docker repository
docker push my-registry/my-js-app:1.0
```

```
┌─────────┐     ┌──────────┐     ┌────────────────────┐     ┌──────────────────┐
│   Git   │────►│    CI    │────►│  Build Docker Image│────►│  Docker Registry │
│ (push)  │     │(Jenkins) │     │  (JS App image)    │     │  (private/hub)   │
└─────────┘     └──────────┘     └────────────────────┘     └──────────────────┘
```

> ✅ Every code push automatically produces a fresh, tested Docker image.

---

## 🚀 Stage 3 — Deployment

The **Dev/Prod Server** pulls both images from their repositories and runs them together.

**Images pulled by the server:**
- Your app image (from your private Docker repository)
- MongoDB image (from Docker Hub)

```bash
# On the server — pull both images
docker pull my-registry/my-js-app:1.0
docker pull mongo

# Run both containers
docker run -d --name js-app my-registry/my-js-app:1.0
docker run -d --name mongodb mongo
```

```
Dev/Prod Server
┌──────────────────────────────────────┐
│  JS App Container   ←  Docker Repo   │
│  MongoDB Container  ←  Docker Hub    │
└──────────────────────────────────────┘
```

> ✅ No manual setup on the server. Just `docker pull` and `docker run`.

---

## 🗺️ Full Pipeline Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                    DOCKER IN PRACTICE                            │
│                                                                  │
│  Developer Laptop                                                │
│  ┌──────────────────┐                                           │
│  │  JS App + MongoDB│──► Git ──► CI (Jenkins)                  │
│  │  (via Docker Hub)│              │                            │
│  └──────────────────┘              │                            │
│                                    ▼                            │
│                         Build JS App Docker Image               │
│                                    │                            │
│                                    ▼                            │
│                         Push to Docker Repository               │
│                                    │                            │
│                                    ▼                            │
│                         Dev/Prod Server                         │
│                         ┌──────────────────────┐               │
│                         │ JS App  ← Docker Repo│               │
│                         │ MongoDB ← Docker Hub │               │
│                         └──────────────────────┘               │
└──────────────────────────────────────────────────────────────────┘
```

---

## 💡 Why This Workflow is Powerful

| Benefit | Explanation |
|---|---|
| ✅ Consistent environments | Dev, CI, and Prod all use the exact same image |
| ✅ No "works on my machine" | Everyone uses the same container |
| ✅ Fast deployments | Server just pulls and runs — no manual config |
| ✅ Easy rollback | Just pull and run the previous image version |
| ✅ Scalable | Spin up multiple containers of the same image instantly |

---

## 📝 Key Takeaways

- **Development** → Use Docker to pull dependencies (like MongoDB) instead of installing them
- **CI** → Automatically builds your app into a Docker image on every Git push
- **Deployment** → Server pulls both your app image and DB image and runs them as containers
- The whole flow means **one consistent environment** from your laptop to production

---

## 🔗 Resources

- 📺 [Docker Tutorial for Beginners – TechWorld with Nana](https://youtu.be/tLK9nNFHWH8?si=d5GP-ffP5Qy_iECJ)
- 📖 [Docker CI/CD Overview](https://docs.docker.com/build/ci/)
- 📖 [Docker Hub](https://hub.docker.com)
