# 🐳 Docker Volumes – Data Persistence

> **Topic:** Docker Volumes, Volume Types, Data Persistence for Databases & Stateful Apps

---

## 📋 Topics Covered

| # | Topic |
|---|---|
| 1 | Why Docker Volumes? |
| 2 | The Problem — Data Loss |
| 3 | What is a Docker Volume? |
| 4 | 3 Types of Docker Volumes |
| 5 | Volumes in Docker Compose |

---

## ❓ 1. Why Docker Volumes?

Docker Volumes are used for **data persistence** — keeping data alive even when a container stops or is removed.

Most needed for:
- 🗄️ **Databases** (MongoDB, MySQL, PostgreSQL)
- 📦 **Other Stateful Applications** (anything that stores data)

> Without volumes → **data is lost** every time a container restarts or is removed.

---

## ⚠️ 2. The Problem — Data Loss Without Volumes

By default, containers use a **Virtual File System** — data only lives inside the container.

```
┌─────────────────────────────────┐
│              Host               │
│   ┌─────────────────────────┐   │
│   │       Container         │   │
│   │   /var/lib/mysql/data   │   │  ← Virtual File System
│   └─────────────────────────┘   │         ↓
│                                 │      Database
└─────────────────────────────────┘
```

**❌ Problem:**
- Container restarts → **Data is gone!**
- Container removed → **Data is gone!**

This is a big issue for databases — you don't want to lose all your data on every restart.

---

## ✅ 3. What is a Docker Volume?

A Docker Volume **mounts a folder from the Host file system into the Container's virtual file system**.

```
┌──────────────────────────────────────┐
│                 Host                 │
│   ┌──────────────────────────────┐   │
│   │          Container           │   │
│   │    /var/lib/mysql/data       │   │ ← Virtual File System
│   │            🔌                │   │
│   └──────────────────────────────┘   │
│         /home/mount/data             │ ← Host File System
└──────────────────────────────────────┘
```

> **Key idea:** Data is written to BOTH the container path AND the host path simultaneously. If the container dies, data is **safe on the host**. ✅

---

## 📦 4. Three Types of Docker Volumes

### Type 1 — Anonymous Volumes

Only specify the **container path**. Docker automatically creates a folder on the host with a random hash name.

```bash
docker run -v /var/lib/mysql/data
```

```
Container path:  /var/lib/mysql/data
                       🔌
Host path:  /var/lib/docker/volumes/random-hash/_data
                  (automatically created by Docker)
```

**⚠️ Downside:** Hard to reference later — you don't control the host path name.

---

### Type 2 — Host Volumes

You specify **both** host path and container path. Full control over where data is stored on the host.

```bash
docker run -v /home/mount/data:/var/lib/mysql/data
```

```
Container path:  /var/lib/mysql/data
                       🔌
Host path:       /home/mount/data
                  (you define this!)
```

**✅ Advantage:** You know exactly where your data lives on the host machine.

---

### Type 3 — Named Volumes ⭐ (Most Recommended)

You give the volume a **name** instead of a full path. Docker manages the location on the host, but you can reference it by name.

```bash
docker run -v db-data:/var/lib/mysql/data
```

```
Volume name:     db-data
                    🔌
Container path:  /var/lib/mysql/data
Host location:   /var/lib/docker/volumes/db-data/_data
                  (managed by Docker, referenced by name)
```

**✅ Best practice** — especially in Docker Compose. Easy to reference, Docker handles the path.

---

### Volume Types Comparison

| Type | Command Syntax | Host Path | Best For |
|------|---------------|-----------|---------|
| Anonymous | `-v /container/path` | Random hash | Quick testing |
| Host | `-v /host/path:/container/path` | You define it | Dev environments |
| Named ⭐ | `-v name:/container/path` | Docker manages | Production, Compose |

---

## 🐙 5. Docker Volumes in Docker Compose

### Named Volume in `mongo-docker-compose.yaml`

```yaml
version: '3'
services:

  mongodb:
    image: mongo
    ports:
      - 27017:27017
    volumes:
      - db-data:/var/lib/mysql/data   # ← Named volume

  mongo-express:
    image: mongo-express
    # ...

# Declare all named volumes at the bottom
volumes:
  db-data        # ← volume name declared here
```

**Two things to remember:**
1. Reference the volume inside the service under `volumes:`
2. **Declare** the named volume at the **bottom-level** `volumes:` section

```
services:
  mongodb:
    volumes:
      - db-data:/var/lib/mysql/data    ← use it here

volumes:
  db-data                              ← declare it here
```

> Docker Compose creates the volume once and **reuses it** across container restarts — your data persists! ✅

---

## 💡 Key Takeaways

| Concept | Detail |
|---------|--------|
| **Why volumes?** | Data inside containers is temporary — lost on restart/removal |
| **What is a volume?** | Host folder mounted into container's file system |
| **Anonymous** | Docker auto-generates host path (random hash) |
| **Host** | You specify both host & container paths |
| **Named** ⭐ | Give it a name, Docker manages the path — best practice |
| **In Compose** | Declare at service level + top-level `volumes:` section |

---

## 📝 Quick Commands

```bash
# Run with anonymous volume
docker run -v /var/lib/mysql/data mysql

# Run with host volume
docker run -v /home/mount/data:/var/lib/mysql/data mysql

# Run with named volume
docker run -v db-data:/var/lib/mysql/data mysql

# List all volumes
docker volume ls

# Inspect a volume
docker volume inspect db-data

# Remove a volume
docker volume rm db-data
```

---

