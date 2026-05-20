# 🐳 Docker Architecture 

## 📦 What is a Container?

A Docker container is a lightweight, isolated environment that runs your application. It is built from **layered images**:

- **Linux base image** — the underlying OS layer (e.g., `alpine:3.10`)
- **Application image** — the software layer on top (e.g., `postgres:10.10`)

### Visual Representation

```
┌──────────────────────────┐
│     Postgres:10.10       │  ← Layer: Application image
├──────────────────────────┤
│      alpine:3.10         │  ← Layer: Linux base image
└──────────────────────────┘
```

---

## 🖼️ Docker Images vs Containers

| Concept | Description |
|---|---|
| **Image** | A read-only template used to create containers |
| **Container** | A running instance of an image |

---

## ⚙️ Key Docker Commands

### Check running containers
```bash
docker ps
```
Output includes: `CONTAINER ID`, `IMAGE`, `COMMAND`, `CREATED`, `STATUS`, `PORTS`, `NAME`

### Pull and run an image
```bash
docker run postgres:9.6
```
- If the image is **not found locally**, Docker automatically pulls it from Docker Hub
- Example output:
  ```
  Unable to find image 'postgres:9.6' locally
  9.6: Pulling from library/postgres
  ...layers downloading...
  ```

### Run a second version simultaneously
```bash
docker run postgres:10.10
```
- Docker reuses already-downloaded layers (`Already exists`)
- Only new/different layers are downloaded

---

## 🔄 Layer Caching (How Docker is Efficient)

When pulling a new image version, Docker checks if layers already exist locally:

```
8f91359f1fff: Already exists   ← shared layer, not re-downloaded
c6115f5efcde: Already exists
6de9d066d892: Downloading...   ← new layer, needs download
```

This makes pulling new versions **much faster** because shared layers are reused.

---

## 🗂️ What `docker ps` Shows

```
CONTAINER ID   IMAGE           STATUS          PORTS       NAME
fad0f8456ca7   postgres:9.6    Up 47 seconds   5432/tcp    celess_haibt
```

- **5432/tcp** — default PostgreSQL port exposed by the container

---

## 📝 Key Takeaways

- Containers package your app + its environment into one portable unit
- Images are made of **layers** — base OS + application on top
- Docker Hub hosts public images (like `postgres`, `alpine`, etc.)
- Running `docker run <image>` pulls the image automatically if not local
- Layer caching = faster builds and smaller downloads over time

---

## 🔗 Resources

- [Docker Official Docs](https://docs.docker.com)
- [Docker Hub](https://hub.docker.com)
- [Play with Docker](https://labs.play-with-docker.com)
- 
