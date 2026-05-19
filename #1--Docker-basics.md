# Docker & Containers — Notes

## What is a Container?

A **container** is a way to package an application along with all its necessary dependencies and configuration into a single portable artifact.

- **Portable** — easily shared and moved between environments
- **Self-contained** — includes everything the app needs to run
- **Efficient** — makes both development and deployment faster and more reliable

> Think of it like a real shipping container: everything is packed inside, standardized, and can be shipped anywhere without worrying about what's at the destination.

---

## Application Development — Before vs After Containers

### ❌ Before Containers (The Problem)

Developers had to manually install specific versions of tools (e.g., PostgreSQL v9.3, Redis v5.0) on their own machines. This caused:

- **Different installation steps** on each OS (Mac, Linux, Windows)
- **Version conflicts** — one developer on Mac, another on Linux = different behavior
- **Many manual steps** where things could go wrong

**Example:**
```
Developer 1 (Mac)  → installs PostgreSQL v9.3 + Redis v5.0 manually
Developer 2 (Linux) → installs PostgreSQL v9.3 + Redis v5.0 manually
          ↕
    cmd x → execution...
    cmd y → execution...
    cmd z → ❌ error executing!   ← something breaks on one OS
```

### ✅ After Containers (The Solution)

With containers, everything is packaged once and runs the same everywhere.

- **One command** to install and start the application
- **No environment setup** needed on each machine
- **Run multiple versions** of the same app side by side without conflict

**Example:**
```
              ┌─────────────────────────────┐
              │         Container            │
              │  [config] [PostgreSQL v9.3]  │
              │         [Start Script]       │
              └────────────┬────────────────┘
                    ┌──────┴──────┐
                    ▼             ▼
          PostgreSQL          PostgreSQL
          Container           Container
          (Developer 1)       (Developer 2)

> cmd: docker run postgres   ← one command, works everywhere ✅
```

---

## Application Deployment — Before vs After Containers

### ❌ Before Containers

Deploying an application meant handing off a JAR file + a textual guide to the Operations team, who then had to:

- Configure the server manually
- Install all external dependencies on the server OS
- Deal with **dependency version mismatches**

```
Developer → [JAR + instructions] → Operations → Server
                                         ↕
                              External dependencies 😤
                              Configuration nightmares
```

### ✅ After Containers

Developers and Operations work **together** to package the app into a container once. Then:

- **No server-side configuration needed** (except installing Docker runtime)
- Container holds: configuration + JAR + all dependencies
- Just deploy the container image to any server

```
┌──────────────────────────────────────┐
│              Container               │
│  [config]  [JAR]  [Dependencies]     │
└──────────────────────────────────────┘
       Developer + Operations 🤝
              ↓
   Java Application Container → Server ✅
```

---

## Where Do Containers Live?

Containers are stored and distributed via a **Container Repository**.

| Type | Description | Example |
|------|-------------|---------|
| **Public** | Open to everyone | Docker Hub |
| **Private** | Restricted access | AWS ECR, GCR |

### Docker Hub
Docker Hub is the most popular **public container repository**. You can search and pull pre-built images for common tools:

```bash
# Pull and run a postgres container
docker run postgres

# Pull a specific version
docker run postgres:9.3

# Other popular images available on Docker Hub:
# redis, nodejs, nginx, mysql, mongo...
```

**Container Repository diagram:**
```
┌─────────────────────────────────────────┐
│          container repository            │
│                                         │
│  [Postgres] [Redis] [Nodejs] [Nginx]    │
│                                         │
└─────────────────────────────────────────┘
```

---

## Quick Summary

| Concept | Key Point |
|--------|-----------|
| Container | Packages app + dependencies + config |
| Before containers | Manual setup, OS-dependent, error-prone |
| After containers (Dev) | One command, consistent across all OS |
| After containers (Deploy) | No server config needed, just Docker runtime |
| Container Repository | Where container images are stored (e.g. Docker Hub) |
| Docker | The most popular container platform |

---

## Key Docker Commands (Cheat Sheet)

```bash
# Pull an image from Docker Hub
docker pull nginx

# Run a container
docker run nginx

# Run in detached mode
docker run -d nginx

# List running containers
docker ps

# Stop a container
docker stop <container_id>

# List all images
docker images
```

---


