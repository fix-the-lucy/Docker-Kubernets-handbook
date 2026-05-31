# 🐳 Docker Registry & AWS ECR – Push Images to Private Repository

> **Topic:** Docker Private Registry, Image Naming, AWS ECR Setup, docker tag & push  

---

## 📋 Topics Covered

| # | Topic |
|---|---|
| 1 | What is a Docker Registry? |
| 2 | Registry Options |
| 3 | Image Naming Convention |
| 4 | Create Private Repo on AWS ECR |
| 5 | Login to AWS from Terminal |
| 6 | Build & Tag the Image |
| 7 | Push Image to ECR |
| 8 | Push a New Version |

---

## 🗄️ 1. What is a Docker Registry?

A **Docker Registry** is a storage service for Docker images. You push your built images to a registry so servers can pull and run them.

```
Developer builds image → pushes to Registry → Server pulls from Registry → runs as container
```

> Docker Hub is the **public** registry. For your own apps, you need a **private** registry.

---

## 🏪 2. Registry Options

| Registry | Type | Notes |
|---|---|---|
| **Docker Hub** | Public | Free for public images |
| **AWS ECR** | Private | Amazon Elastic Container Registry |
| **DigitalOcean** | Private | Simple, developer-friendly |
| **Nexus** | Private | Self-hosted option |

> In this demo we use **AWS ECR** (Elastic Container Registry).

---

## 🏷️ 3. Image Naming Convention

Every image in a registry follows this format:

```
registryDomain/imageName:tag
```

### Docker Hub (public)
```bash
docker pull mongo:4.2
# same as:
docker pull docker.io/library/mongo:4.2
```

### AWS ECR (private)
```bash
docker pull 520697001743.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.0
```

> The long prefix (`664574038682.dkr.ecr.eu-central-1.amazonaws.com`) is your **ECR registry URL** = account ID + region.

**Key rule:** One repository = one image, but **multiple tags/versions**:
```
my-app:1.0   ← version 1
my-app:1.1   ← version 2
my-app:2.0   ← major update
```

---

## ☁️ 4. Create Private Repo on AWS ECR

1. Go to **AWS Console** → search **ECR** → Elastic Container Registry
2. Click **Repositories** → **Create repository**
3. Enter your repo name (e.g., `my-app`)
4. Leave defaults → Click **Create repository**

Your repository URL will look like:
```
664574038682.dkr.ecr.eu-central-1.amazonaws.com/my-app
```

> ✅ ECR stores all versions (tags) of **one image** in one repository.  
> Each image version = different tag (`1.0`, `1.1`, `2.0` etc.)

---

## 🔑 5. Login to AWS from Terminal

### Pre-requisites
- AWS CLI installed
- AWS credentials configured (`aws configure`)

### Login command
```bash
$(aws ecr get-login --no-include-email --region eu-central-1)
```

This command:
1. Gets a temporary Docker login token from AWS
2. Automatically runs `docker login` with that token
3. Authenticates your local Docker to the ECR registry

> ✅ After this, `docker push` to ECR will work.

---

## 🔨 6. Build & Tag the Image

### Step 1 — Build the image locally
```bash
docker build -t my-app:1.0 .
```

### Step 2 — Verify it exists
```bash
docker images
# Shows: my-app   1.0   83fdc778a892   116 MB
```

### Step 3 — Tag it with the full ECR registry URL
```bash
docker tag my-app:1.0 664574038682.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.0
```

**What `docker tag` does:**
- Creates a new name (alias) for the same image
- The image ID stays the same — it's just a new label
- Both names point to the **same image**

```bash
docker images
# Now you see TWO entries with same IMAGE ID:
# my-app                                          1.0   83fdc778a892   116 MB
# 664574038682.dkr.ecr...amazonaws.com/my-app     1.0   83fdc778a892   116 MB
```

---

## 📤 7. Push Image to ECR

```bash
docker push 664574038682.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.0
```

**Output:**
```
The push refers to repository [664574038682.dkr.ecr.eu-central-1.amazonaws.com/my-app]
5678c8aa26b3: Pushed
bb2a17dfd6c2: Pushed
099773e542db: Pushed
...
1.0: digest: sha256:fc8aeac852... size: 1576
```

**Verify in AWS ECR Console:**
- Go to ECR → Repositories → `my-app` → **Images**
- You'll see `1.0` with size ~44.66 MB

---

## 🆕 8. Push a New Version (1.1)

When you update your app, rebuild with a new tag:

```bash
# Build new version
docker build -t my-app:1.1 .

# Tag with registry URL
docker tag my-app:1.1 664574038682.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.1

# Push
docker push 664574038682.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.1
```

**Smart caching — only changed layers push:**
```
eaf1692e831f: Pushing [=======>] 10.97 MB   ← only this changed
ca0c58954a65: Pushed
099773e542db: Layer already exists           ← reused from v1.0!
9efd3ca0eab1: Layer already exists
a721b64d51de: Layer already exists
```

**In AWS ECR — now you see both versions:**
```
Image tag    Image URI                                        Size
1.1          664574038682.dkr.ecr...amazonaws.com/my-app:1.1  44.66 MB
1.0          664574038682.dkr.ecr...amazonaws.com/my-app:1.0  44.66 MB
```

---

## 📝 Full Workflow Summary

```bash
# 1. Login to AWS ECR
$(aws ecr get-login --no-include-email --region eu-central-1)

# 2. Build your image
docker build -t my-app:1.0 .

# 3. Tag it for ECR
docker tag my-app:1.0 <account-id>.dkr.ecr.<region>.amazonaws.com/my-app:1.0

# 4. Push to ECR
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/my-app:1.0

# 5. For a new version:
docker build -t my-app:1.1 .
docker tag my-app:1.1 <account-id>.dkr.ecr.<region>.amazonaws.com/my-app:1.1
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/my-app:1.1
```

---

## 💡 Key Takeaways

| Concept | Detail |
|---|---|
| **Registry** | Storage for Docker images (public or private) |
| **ECR** | AWS private registry — stores your app images |
| **Image naming** | `registryDomain/imageName:tag` |
| **`docker tag`** | Renames/aliases an image for a specific registry |
| **`docker push`** | Uploads image to the registry |
| **Layer caching** | Only changed layers are re-pushed — saves time and bandwidth |
| **One repo = one app** | All versions stored as different tags in one repo |

---

## 🔗 Resources

- 📖 [AWS ECR Docs](https://docs.aws.amazon.com/ecr/)
- 📖 [docker tag docs](https://docs.docker.com/engine/reference/commandline/tag/)
- 📖 [docker push docs](https://docs.docker.com/engine/reference/commandline/push/)
- 📺 [Docker Tutorial – TechWorld with Nana](https://youtu.be/tLK9nNFHWH8?si=d5GP-ffP5Qy_iECJ)
