# 🐳 Dockerfile – Build a Docker Image from Your App

> **Topic:** What is a Dockerfile, Dockerfile Instructions, Image Layers, Build & Run  


---

## 📋 Topics Covered

| # | Topic |
|---|---|
| 1 | What is a Dockerfile? |
| 2 | Where Dockerfile Fits in the CI/CD Pipeline |
| 3 | Dockerfile Instructions — Explained |
| 4 | Image Layers — How it Stacks |
| 5 | Build the Docker Image |
| 6 | Run & Verify the Container |
| 7 | ⚠️ Important Rule |

---

## 🤔 1. What is a Dockerfile?

A **Dockerfile** is a plain text file with instructions that tells Docker **how to build your own custom image**.

```
Your App Code  +  Dockerfile  =  Docker Image  →  Container
```

> 💡 Think of it as a **blueprint** for your image — it defines the environment, files, and startup command.

Every image on Docker Hub was built using a Dockerfile.

---

## 🔁 2. Where Dockerfile Fits in the CI/CD Pipeline

```
Developer (previous video)          This video
┌──────────────────────┐           ┌──────────────────────────────┐
│  JS App              │  Git →    │  CI (Jenkins)                │
│  + MongoDB container │           │  reads Dockerfile            │
│  (Docker Hub)        │           │  → builds Docker Image       │
└──────────────────────┘           │  → pushes to Docker Registry │
                                   └──────────────────────────────┘
```

> In real life, **Jenkins** (or any CI tool) reads the Dockerfile and runs `docker build` automatically on every code push.

---

## 📄 3. Dockerfile Instructions — Explained

### Blueprint concept (what we want to do → how Dockerfile does it)

| What we want | Dockerfile instruction |
|---|---|
| Install Node.js | `FROM node:13-alpine` |
| Set env variables | `ENV MONGO_DB_USERNAME=admin` |
| Create `/home/app` folder | `RUN mkdir -p /home/app` |
| Copy app files into image | `COPY . /home/app` |
| Start the app | `CMD ["node", "server.js"]` |

### Complete Dockerfile

```dockerfile
FROM node:13-alpine

ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PWD=password

RUN mkdir -p /home/app

COPY . /home/app

CMD ["node", "/home/app/server.js"]
```

### Each Instruction Explained

```dockerfile
FROM node:13-alpine
```
- **Base image** — starts from an existing image (Node.js on Alpine Linux)
- Every Dockerfile must start with `FROM`
- Alpine = tiny Linux distro (~5 MB), keeps image small

```dockerfile
ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PWD=password
```
- Sets **environment variables** inside the image
- Available to the app at runtime
- `\` continues to the next line

```dockerfile
RUN mkdir -p /home/app
```
- **Executes a Linux command** during image build
- Creates the `/home/app` directory inside the container
- `RUN` = runs during **build time**

```dockerfile
COPY . /home/app
```
- Copies files from your **host machine** (`.` = current folder) into the image at `/home/app`
- This is how your app code gets inside the image

```dockerfile
CMD ["node", "/home/app/server.js"]
```
- **Startup command** — what runs when the container starts
- Only **one CMD** allowed per Dockerfile
- `CMD` = runs at **container start time** (not build time)

---

## 🧱 4. Image Layers — How it Stacks

Every `FROM` instruction adds a **layer**. Your app image is built on top of existing images:

```
┌──────────────────┐
│    app:1.0       │  ← Your app (FROM node:13-alpine)
├──────────────────┤
│  node:13-alpine  │  ← Node layer (FROM alpine:3.10)
├──────────────────┤
│   alpine:3.10    │  ← Linux base layer
└──────────────────┘
```

> This is why Docker is efficient — if `node:13-alpine` is already downloaded, it's **reused** and not re-downloaded when building your app image.

---

## 🔨 5. Build the Docker Image

Run this command from the folder containing your `Dockerfile`:

```bash
docker build -t my-app:1.0 .
```

| Part | Meaning |
|---|---|
| `docker build` | Build command |
| `-t my-app:1.0` | Tag: `name:version` for your image |
| `.` | Build context = current directory (where Dockerfile is) |

**Verify the image was created:**
```bash
docker images
# Shows: my-app   1.0   ...
```

---

## ▶️ 6. Run & Verify the Container

### Start the container
```bash
docker run my-app:1.0
```

### Verify it started correctly
```bash
# Check logs
docker logs <container_id>
# Expected: "app listening on port 3000!"

# Enter the container (Alpine uses /bin/sh, NOT /bin/bash)
docker exec -it <container_id> /bin/sh

# Inside container — verify files are there
ls /home/app
# Shows: Dockerfile  index.html  node_modules  server.js  package.json

# Check env variables were set correctly
env
# Shows: MONGO_DB_USERNAME=admin
#        MONGO_DB_PWD=password
#        NODE_VERSION=13.1.0
```

> ⚠️ **Alpine Linux uses `/bin/sh` not `/bin/bash`**  
> Running `docker exec -it <id> /bin/bash` on an Alpine container gives:  
> `no such file or directory` error  
> Use `/bin/sh` instead!

---

## ⚠️ 7. Important Rule

> **When you change the Dockerfile, you MUST rebuild the image!**

```bash
# After any Dockerfile change:
docker build -t my-app:1.0 .   # rebuild
docker run my-app:1.0           # run new image
```

The old running container still uses the old image until you rebuild and rerun.

---

## 📝 Key Takeaways

| Concept | Detail |
|---|---|
| `FROM` | Base image — always first instruction |
| `ENV` | Set environment variables in the image |
| `RUN` | Execute commands **at build time** |
| `COPY` | Copy files from host into image |
| `CMD` | Command to run **when container starts** |
| Alpine images | Use `/bin/sh` not `/bin/bash` |
| Rebuild rule | Change Dockerfile → must rebuild image |
| Jenkins role | Reads Dockerfile and builds image automatically in CI |

---

## 🔁 Full Flow Summary

```bash
# 1. Write your Dockerfile
# 2. Build the image
docker build -t my-app:1.0 .

# 3. Verify image exists
docker images

# 4. Run the container
docker run my-app:1.0

# 5. Check logs
docker logs <container_id>

# 6. Enter container (Alpine = /bin/sh)
docker exec -it <container_id> /bin/sh
ls /home/app
env
```

---

## 🔗 Resources

- 📖 [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- 📖 [docker build docs](https://docs.docker.com/engine/reference/commandline/build/)
- 📺 [Docker Tutorial – TechWorld with Nana](https://youtu.be/tLK9nNFHWH8?si=d5GP-ffP5Qy_iECJ)
