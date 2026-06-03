# 🐳 Docker Volumes Demo — Hands-On Guide

> **Topic:** Docker Volumes in Practice — Named Volumes in Compose, Data Persistence Demo, Volume Locations on Host

---

## 📋 Topics Covered

| # | Topic |
|---|---|
| 1 | Starting Point — No Persistence |
| 2 | Part 1: Define Named Volume in Docker Compose |
| 3 | Part 2: Demo — Data Persists After Restart |
| 4 | Part 3: See Where Volumes Are Located on Host |
| 5 | Docker Volume Locations by OS |
| 6 | Mac Special Case — Linux VM |

---

## ⚠️ 1. Starting Point — No Persistence

**Setup:** Node.js app + MongoDB running via Docker Compose — but **no volumes defined**.

```yaml
# docker-compose.yaml (WITHOUT volume - starting point)
version: '3'
services:
  # my-app:           ← commented out for now
  #   image: ...
  #   ports:
  #     - 3000:3000

  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password

  mongo-express:
    image: mongo-express
    ports:
      - 8080:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
```

**Problem:** Every time you restart or remove the MongoDB container, all data is gone. The app has to start fresh.

---

## 📝 2. Part 1 — Define Named Volume in Docker Compose

### Step 1: Add `volumes` under the service

```yaml
mongodb:
  image: mongo
  ports:
    - 27017:27017
  environment:
    - MONGO_INITDB_ROOT_USERNAME=admin
    - MONGO_INITDB_ROOT_PASSWORD=password
  volumes:
    - mongo-data:/data/db    # ← named volume : container path
```

> **`/data/db`** is the path where MongoDB stores its data inside the container.
> ⚠️ This path **differs for each database:**
> - MongoDB → `/data/db`
> - MySQL → `/var/lib/mysql`
> - PostgreSQL → `/var/lib/postgresql/data`

### Step 2: Declare the named volume at the bottom

```yaml
# At the TOP-LEVEL (same indentation as 'services:')
volumes:
  mongo-data:
    driver: local
```

### Complete `docker-compose.yaml` with volume:

```yaml
version: '3'
services:

  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo-data:/data/db      # ← volume attached here

  mongo-express:
    image: mongo-express
    ports:
      - 8080:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb

volumes:
  mongo-data:                    # ← declared here
    driver: local
```

> **Two rules to remember:**
> 1. Reference volume under the service: `- mongo-data:/data/db`
> 2. Declare volume at bottom: `volumes: mongo-data: driver: local`

---

## ✅ 3. Part 2 — Demo: Data Persists After Restart

### Run the app

```bash
# Start containers (MongoDB + Mongo Express)
docker-compose -f docker-compose.yaml up

# Start Node.js app separately
npm run start
# → app listening on port 3000!
```

### Update data in the app

- Open `localhost:3000` → User Profile page
- Update: Name = **Anna Jones**, Email = **anna.jones@example.com**, Interests = **coding, travel**
- Click **Update Profile** → data saved to MongoDB

### Verify in Mongo Express (`localhost:8080`)

Navigate to: `my-db → users collection`

```
_id              userid   email                    interests        name
5de83452da7c...  1        anna.jones@example.com   coding, travel   Anna Jones
```

### Restart containers → Data still there! ✅

```bash
docker-compose down   # stop and remove containers
docker-compose up     # restart fresh

# Open localhost:3000 → Anna Jones data still shows!
# Open localhost:8080 → Document still in MongoDB!
```

> This proves the volume is working — data survived the container restart. 🎉

---

## 🔍 4. Part 3 — See Where Volumes Are Located on Host

### Step 1: Inspect running containers

```bash
docker ps
# CONTAINER ID   IMAGE           STATUS
# 53eb5c04be0b   mongo           Up 20 minutes   0.0.0.0:27017->27017/tcp
# eb9b354b5e31   mongo-express   Up 20 minutes   0.0.0.0:8080->8081/tcp
```

### Step 2: Go inside the MongoDB container

```bash
docker exec -it 53eb5c04be0b sh

# Inside container:
ls /data/db
# WiredTiger  WiredTiger.lock  collection-0-...  journal  mongod.lock  storage.bson ...
```

This is the **container's virtual file system** path where MongoDB writes data.

### Step 3: Find the volume on the HOST

```bash
# On Linux (inside Docker VM on Mac):
ls /var/lib/docker/volumes/
# Shows all volumes:
# cfc1ab...ddf22/     ← anonymous volumes (long hashes)
# techworldjsdockerdemoapp_mongo-data/    ← our named volume ✅
# metadata.db

# Navigate into our named volume:
ls /var/lib/docker/volumes/techworldjsdockerdemoapp_mongo-data/_data
# WiredTiger  WiredTiger.lock  collection-0-...  journal  mongod.lock  storage.bson
```

> **Same files!** The `_data` folder on the host mirrors exactly what's in the container's `/data/db`. This is the mount in action. ✅

---

## 📁 5. Docker Volume Locations by OS

Volumes are stored on the **host machine** at different paths depending on your OS:

| OS | Volume Location |
|----|----------------|
| 🪟 **Windows** | `C:\ProgramData\docker\volumes` |
| 🐧 **Linux** | `/var/lib/docker/volumes` |
| 🍎 **Mac** | `/var/lib/docker/volumes` *(inside Linux VM)* |

### Named vs Anonymous volume folders

```
/var/lib/docker/volumes/
├── cfc1ab3d...ddf22/          ← anonymous volume (random hash)
│   └── _data/
├── 931d101e...3f53/           ← another anonymous volume
│   └── _data/
└── techworldjsdockerdemoapp_mongo-data/   ← named volume ✅
    └── _data/
        ├── WiredTiger
        ├── collection-0-...wt
        ├── journal/
        └── storage.bson
```

> Anonymous volumes have random long hash names — hard to identify.
> Named volumes have readable names → much easier to manage. ✅

---

## 🍎 6. Mac Special Case — Linux Virtual Machine

On Mac, if you run `ls /var/lib/docker` directly in the terminal, you'll get:

```bash
ls /var/lib/docker
# ls: /var/lib/docker: No such file or directory
```

**Why?** Docker for Mac doesn't run natively — it creates a **Linux virtual machine** and stores all Docker data (containers, images, volumes) inside that VM.

```
Mac Host
  └── Linux VM (hidden)
        └── /var/lib/docker/volumes/   ← volumes live here
```

To access it, you need to `screen` or SSH into the hidden Docker VM:

```bash
# Access Docker's Linux VM on Mac
screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty

# Then navigate:
ls /var/lib/docker/volumes/
```

> To exit the screen session: press `Ctrl+A` then `K`, then `y` to confirm.

---

## 💡 Key Takeaways

| Concept | Detail |
|---------|--------|
| **No volume = no persistence** | Data lost on every container restart |
| **Named volume** | Best practice — readable name, Docker manages path |
| **Container path** | Differs per DB: MongoDB→`/data/db`, MySQL→`/var/lib/mysql` |
| **Must declare twice** | Under `services → volumes` AND top-level `volumes:` |
| **Host location** | `/var/lib/docker/volumes/<name>/_data` |
| **Anonymous volumes** | Random hash names — hard to manage |
| **Mac** | Volumes inside a hidden Linux VM — not directly on Mac filesystem |

---

## 📝 Quick Commands

```bash
# Start with volumes
docker-compose -f docker-compose.yaml up

# List all volumes
docker volume ls

# Inspect a specific volume (shows mountpoint)
docker volume inspect techworldjsdockerdemoapp_mongo-data

# Go inside a container to verify data path
docker exec -it <container-id> sh
ls /data/db

# Remove volumes when stopping (⚠️ deletes data!)
docker-compose down --volumes
```

---
