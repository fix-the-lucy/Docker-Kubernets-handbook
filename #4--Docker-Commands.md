# 🐳 Docker Basic Commands – Learning Notes


## 📋 Topics Covered

| # | Topic |
|---|---|
| 1 | Container vs Image |
| 2 | Version and Tag |
| 3 | Port Binding (Host vs Container) |
| 4 | `docker pull` |
| 5 | `docker run` & options |
| 6 | `docker stop` / `docker start` |
| 7 | `docker ps` |
| 8 | `docker exec -it` |
| 9 | `docker logs` |

---

## 🖼️ 1. Container vs Image

| Term | Description |
|---|---|
| **Image** | The packaged template (read-only). Downloaded from Docker Hub. |
| **Container** | A **running instance** of an image. Created from `docker run`. |

> 💡 One image can create **many containers** simultaneously.

---

## 🏷️ 2. Version and Tag

Specify a version using a **tag** with the `:` syntax:

```bash
docker run redis          # pulls latest version
docker run redis:4.0      # pulls specific version (tag = 4.0)
docker run postgres:9.6   # specific postgres version
```

> If no tag is specified, Docker uses **`:latest`** by default.

---

## 🔌 3. Port Binding — Container Port vs Host Port

Containers have their **own internal ports**, separate from your laptop (host). You must **bind** a host port to a container port to access it from outside.

```
Host Machine
┌──────────────────────────────────────────────┐
│  Port 5000  ──►  Port 5000  [ Container ]    │
│  Port 3000  ──►  Port 3000  [ Container ]    │
│  Port 3001  ──►  Port 3000  [ Container ]    │  ← different host port, same container port
└──────────────────────────────────────────────┘
```

**Syntax:**
```bash
docker run -p <HOST_PORT>:<CONTAINER_PORT> <image>
```

**Examples:**
```bash
docker run -p6000:6379 redis        # host 6000 → container 6379
docker run -p6001:6379 redis:4.0    # host 6001 → container 6379
```

> ⚠️ **Error: port already allocated** — means the host port is already in use by another container. Use a different host port.

---

## 📥 4. `docker pull`

Downloads an image from Docker Hub **without** running it:

```bash
docker pull redis
docker pull redis:4.0
```

---

## ▶️ 5. `docker run` & Options

Starts a **new container** from an image:

```bash
docker run redis               # runs in foreground (blocks terminal)
docker run -d redis            # -d = detached mode (runs in background)
docker run -p6000:6379 redis   # with port binding
docker run -d -p6000:6379 redis  # detached + port binding
```

| Flag | Meaning |
|---|---|
| `-d` | Detached — runs in background |
| `-p` | Port binding `HOST:CONTAINER` |

> `docker run` always **creates a new container**. Use `docker start` to restart an existing one.

---

## ⏹️ 6. `docker stop`

Stops a **running** container (container still exists, just stopped):

```bash
docker stop <container_id>
# Example:
docker stop 8381867e8242
```

---

## ▶️ 7. `docker start`

Restarts an **existing stopped** container (keeps same config/data):

```bash
docker start <container_id>
# Example:
docker start 8381867e8242
```

> `docker start` vs `docker run`:
> - `run` → creates a **brand new** container
> - `start` → restarts an **existing** stopped container

---

## 📋 8. `docker ps`

Lists containers:

```bash
docker ps         # only RUNNING containers
docker ps -a      # ALL containers (including stopped)
```

**Sample output:**
```
CONTAINER ID   IMAGE        STATUS          PORTS                    NAMES
51cdac3132f6   redis:4.0    Up 4 seconds    0.0.0.0:6001->6379/tcp   dreamy_bell
d7a6ef66e6da   redis        Up 1 minute     0.0.0.0:6000->6379/tcp   heuristic_ardinghelli
```

> Docker auto-assigns a random **NAME** (e.g., `dreamy_bell`) if you don't specify one.

---

## 🔁 9. Full Demo Flow (Redis Example)

```bash
# 1. Run Redis in background with port binding
docker run -d -p6000:6379 redis

# 2. Run a second Redis (different version, different host port)
docker run -d -p6001:6379 redis:4.0

# 3. Check both are running
docker ps

# 4. Stop one
docker stop 8381867e8242

# 5. Check all containers (including stopped)
docker ps -a

# 6. Restart the stopped one
docker start 8381867e8242
```

---

## ⚠️ Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| `port is already allocated` | Host port already used by another container | Use a different host port e.g. `-p6001:6379` |
| `docker run requires at least 1 argument` | Missing image name | Add image name: `docker run -p6000:6379 redis` |
| Container not showing in `docker ps` | Container is stopped | Use `docker ps -a` to see all |

---

## 📝 Quick Reference Cheatsheet

```bash
docker pull <image>              # Download image
docker run <image>               # Create & start container
docker run -d <image>            # Run in background
docker run -p HOST:CONT <image>  # With port binding
docker ps                        # List running containers
docker ps -a                     # List all containers
docker stop <id>                 # Stop a container
docker start <id>                # Restart stopped container
```

---

## 🔗 Resources

- 📖 [Docker Docs – docker run](https://docs.docker.com/engine/reference/commandline/run/)
- 🐋 [Docker Hub](https://hub.docker.com)
