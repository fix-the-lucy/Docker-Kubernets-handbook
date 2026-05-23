# 🐳 Docker Commands – Logs, Exec & Container Management


## 📋 Topics Covered

| # | Topic |
|---|---|
| 1 | `docker logs` — view container output |
| 2 | `--name` flag — naming your containers |
| 3 | `docker exec -it` — enter a running container |
| 4 | Running & managing multiple containers |
| 5 | `docker ps -a` — all containers history |

---

## 📜 1. `docker logs` — View Container Logs

See what's happening **inside** a running or stopped container without entering it:

```bash
docker logs <container_id>
# OR use the container name:
docker logs <container_name>
```

**Examples from demo:**
```bash
docker logs 51cdac3132f6       # by container ID
docker logs dreamy_bell        # by auto-assigned name
docker logs redis-older        # by custom name
```

> 💡 Useful for **debugging** — see startup messages, errors, and connection status of services like Redis or Postgres.

**Sample output:**
```
Redis is starting...
Redis version=4.0.14
Running mode=standalone, port=6379
Server initialized
Ready to accept connections
```

---

## 🏷️ 2. `--name` Flag — Naming Your Containers

By default Docker assigns a **random name** (e.g., `dreamy_bell`, `heuristic_ardinghelli`).  
Use `--name` to assign a **meaningful name**:

```bash
docker run -d -p6001:6379 --name redis-older redis:4.0
docker run -d -p6000:6379 --name redis-latest redis
```

**Benefits:**
- Easier to reference in commands (`docker logs redis-older`)
- More readable in `docker ps` output
- No need to copy/paste container IDs

**Full command with all flags:**
```bash
docker run -d \
  -p <HOST_PORT>:<CONTAINER_PORT> \
  --name <your-name> \
  <image>:<tag>
```

---

## 🖥️ 3. `docker exec -it` — Enter a Running Container

Opens an **interactive terminal** inside a running container — like SSH-ing into it:

```bash
docker exec -it <container_id_or_name> /bin/bash
```

**Example:**
```bash
docker exec -it cae903a74202 /bin/bash
# Now you're inside the container!
root@cae903a74202:/data#
```

**Once inside, you can run Linux commands:**
```bash
ls          # list files in container
pwd         # print current directory → /data
cd /        # go to root
ls          # see: bin boot data etc home lib...
env         # see environment variables inside container
```

**Environment variables visible inside container:**
```
HOSTNAME=cae903a74202
REDIS_VERSION=5.0.6
HOME=/root
PATH=/usr/local/sbin:/usr/local/bin:...
```

| Flag | Meaning |
|---|---|
| `-i` | Interactive — keep stdin open |
| `-t` | Allocate a pseudo-TTY (terminal) |
| `/bin/bash` | Shell to open inside the container |

> To **exit** the container terminal, type `exit` or press `Ctrl + D`

---

## 🔀 4. Running Multiple Containers Simultaneously

You can run **multiple versions** of the same image at the same time — each on a different host port:

```bash
# Run Redis latest on port 6000
docker run -d -p6000:6379 --name redis-latest redis

# Run Redis 4.0 on port 6001
docker run -d -p6001:6379 --name redis-older redis:4.0
```

**`docker ps` shows both running:**
```
CONTAINER ID   IMAGE       PORTS                    NAMES
cae903a74202   redis       0.0.0.0:6000->6379/tcp   redis-latest
5b7d84a59a7c   redis:4.0   0.0.0.0:6001->6379/tcp   redis-older
```

> Each container is **fully isolated** — same app, different version, different host port.

---

## 📋 5. `docker ps -a` — See All Containers

```bash
docker ps       # only RUNNING containers
docker ps -a    # ALL containers (running + stopped + exited)
```

**Status types you'll see:**
| Status | Meaning |
|---|---|
| `Up X minutes` | Currently running |
| `Exited (0)` | Stopped cleanly |
| `Exited (1)` | Stopped with an error |
| `Created` | Created but never started |

---

## 🔁 Full Demo Workflow

```bash
# 1. Stop old unnamed containers
docker stop cfec85d7bbec

# 2. Run redis:4.0 with a proper name
docker run -d -p6001:6379 --name redis-older redis:4.0

# 3. Stop another old container
docker stop d7a6ef66e6da

# 4. Run latest redis with a proper name
docker run -d -p6000:6379 --name redis-latest redis

# 5. Verify both are running
docker ps

# 6. Check logs of redis-older
docker logs redis-older

# 7. Enter redis-latest container interactively
docker exec -it cae903a74202 /bin/bash

# 8. Inside container — explore
ls
env
exit

# 9. See all containers ever created
docker ps -a

# 10. Restart a stopped container
docker start cae903a74202
```

---

## 📝 Commands Cheatsheet

```bash
# Logs
docker logs <id/name>              # view container logs

# Naming
docker run --name <name> <image>   # assign custom name

# Interactive terminal
docker exec -it <id/name> /bin/bash   # enter container shell

# Container history
docker ps -a                       # all containers (running + stopped)
```

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---|---|
| Running `docker d7a6ef66e6da` (no subcommand) | Always use a subcommand: `docker stop`, `docker logs`, etc. |
| Port conflict when reusing same host port | Stop the old container first, then run the new one |
| Can't find container in `docker ps` | It's stopped — use `docker ps -a` then `docker start <id>` |

---

## 🔗 Resources

- 📺 [Docker Tutorial for Beginners – TechWorld with Nana](https://youtu.be/tLK9nNFHWH8?si=d5GP-ffP5Qy_iECJ)
- 📖 [docker exec docs](https://docs.docker.com/engine/reference/commandline/exec/)
- 📖 [docker logs docs](https://docs.docker.com/engine/reference/commandline/logs/)
