# 🐳 Docker Compose – Deploy with Private Registry & Complete Workflow

> **Topic:** Docker Compose with ECR Image, Multi-Container Deployment, Full CI/CD Workflow Recap

---

## 📋 Topics Covered

| # | Topic |
|---|---|
| 1 | What is Docker Compose? |
| 2 | Docker Compose File Structure |
| 3 | Using Private ECR Image in Compose |
| 4 | Running Multi-Container App |
| 5 | Container Networking in Compose |
| 6 | App Connecting to MongoDB |
| 7 | Complete CI/CD Workflow Recap |

---

## 🗂️ 1. What is Docker Compose?

Instead of running each container manually with `docker run`, **Docker Compose** lets you define all your services in a single YAML file and start them with **one command**.

```
Without Compose:
docker run -d mongo
docker run -d mongo-express
docker run -d my-app

With Compose:
docker-compose -f mongo.yaml up  ✅ (all 3 at once)
```

> Docker Compose also automatically creates a **shared network** so all containers can talk to each other by service name.

---

## 📄 2. Docker Compose File Structure

```yaml
version: '3'          # compose file version
services:             # list of containers to run
  service-name:
    image: ...        # which image to use
    ports:
      - host:container
    environment:
      - KEY=VALUE
```

### Key Fields

| Field | Purpose |
|---|---|
| `version` | Compose file format version |
| `services` | Each service = one container |
| `image` | Docker image to use (local or registry URL) |
| `ports` | Map `host:container` ports |
| `environment` | Set env variables inside container |

---

## ☁️ 3. Using Private ECR Image in Compose

After pushing `my-app` to AWS ECR, reference the **full ECR URL** in your compose file:

```yaml
# mongo.yaml
version: '3'
services:

  my-app:
    image: 664574038682.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.0
    ports:
      - 3000:3000

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

> **Note:** `my-app` image is pulled from **private ECR**, while `mongo` and `mongo-express` are pulled from **public Docker Hub** — both work in the same compose file.

---

## ▶️ 4. Running Multi-Container App

### Start all containers
```bash
docker-compose -f mongo.yaml up
```

**Output:**
```
Creating network "nanajanashia_default" with the default driver
Creating nanajanashia_mongodb_1        ... done
Creating nanajanashia_mongo-express_1  ... done
Creating nanajanashia_my-app_1         ... done
```

All 3 containers start in one command ✅

### Run in background (detached mode)
```bash
docker-compose -f mongo.yaml up -d
```

### Stop and remove all containers
```bash
docker-compose -f mongo.yaml down
```

### Verify running containers
```bash
docker ps
```

---

## 🌐 5. Container Networking in Compose

Docker Compose automatically creates a **shared network** for all services defined in the same file.

```
┌─────────────────────────────────────────────┐
│         nanajanashia_default (network)       │
│                                             │
│  [my-app]  ←──────→  [mongodb]             │
│                           ↑                 │
│                    [mongo-express]           │
└─────────────────────────────────────────────┘
```

- Containers reference each other by **service name** (not IP or localhost)
- `ME_CONFIG_MONGODB_SERVER=mongodb` → mongo-express finds MongoDB by service name `mongodb`
- No manual network setup needed ✅

---

## 🔌 6. App Connecting to MongoDB

In `server.js`, the Node.js app connects using the **MongoDB service name**:

```javascript
// 'mongodb' = service name from docker-compose, NOT localhost
MongoClient.connect("mongodb://admin:password@mongodb:27017",
  function(err, client) {
    if (err) throw err;

    var db = client.db('my-db');

    // Upsert user data
    db.collection("users").updateOne(
      { userid: 1 },
      { $set: userObj },
      { upsert: true },
      function(err, res) {
        client.close();
      }
    );
  }
);
```

### What you'll see running

| Service | URL | What it shows |
|---|---|---|
| my-app | `localhost:3000` | User Profile UI |
| mongo-express | `localhost:8080` | MongoDB GUI |

**User Profile UI** (`localhost:3000`):
- Shows name, email, interests fields
- "Update Profile" button saves data to MongoDB

**Mongo Express** (`localhost:8080`):
- Lists all databases
- Browse `my-db` → `users` collection
- See the saved document:

```json
{
  "_id": ObjectID("5dc5e62f07b99da93e69d3ba"),
  "userid": 1,
  "email": "anna.jones@example.com",
  "interests": "coding",
  "name": "Anna Jones"
}
```

---

## 🔄 7. Complete CI/CD Workflow Recap

This is the full picture of how everything connects:

```
 ┌──────────────────┐
 │  Developer       │  ← pulls base images (mongo, node) from Docker Hub
 │  (Laptop + JS)   │
 └────────┬─────────┘
          │ git push
          ▼
    ┌──────────┐
    │   Git    │
    └────┬─────┘
         │ triggers build
         ▼
   ┌───────────┐
   │    CI     │  (e.g. Jenkins)
   │ (Jenkins) │  ← builds Docker image from Dockerfile + JS code
   └─────┬─────┘
         │ docker push
         ▼
 ┌─────────────────┐
 │ Docker Registry │  (AWS ECR – private)
 │  my-app:1.0     │
 └────────┬────────┘
          │ pull (via docker-compose)
          ▼
   ┌──────────────────────────────┐
   │         Dev Server           │
   │  ┌──────────────────────┐   │
   │  │     mongo.yaml       │   │
   │  │  ├── my-app (ECR)    │   │
   │  │  ├── mongodb         │   │
   │  │  └── mongo-express   │   │
   │  └──────────────────────┘   │
   └──────────────────────────────┘
          │ push updated image
          ▼
   ┌─────────────┐
   │  Docker Hub │  (for public distribution)
   └─────────────┘
```

### Step-by-Step Summary

| Step | Action |
|---|---|
| 1 | Developer writes code, uses local Docker containers for dev |
| 2 | Code pushed to Git |
| 3 | CI server triggers, builds Docker image using Dockerfile |
| 4 | CI pushes built image to **AWS ECR** (private registry) |
| 5 | Dev Server runs `docker-compose up` — pulls my-app from ECR |
| 6 | All 3 containers start on same Docker network |
| 7 | App available at `localhost:3000`, DB GUI at `localhost:8080` |

---

## 📝 Quick Commands Reference

```bash
# Start all services
docker-compose -f mongo.yaml up

# Start in background
docker-compose -f mongo.yaml up -d

# Stop all services
docker-compose -f mongo.yaml down

# View logs
docker-compose -f mongo.yaml logs

# View running containers
docker ps

# Pull latest images before starting
docker-compose -f mongo.yaml pull
```

---

## 💡 Key Takeaways

| Concept | Detail |
|---|---|
| **Docker Compose** | Manage multi-container apps with one YAML file |
| **Private image in Compose** | Use full ECR URL as the `image` value |
| **Automatic networking** | Compose creates a shared network — use service names |
| **One command deploy** | `docker-compose up` starts everything |
| **CI/CD** | Code → Git → CI builds image → ECR → Server pulls & runs |

---


