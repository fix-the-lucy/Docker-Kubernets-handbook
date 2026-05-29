# 🐳 Docker Compose

## 📋 Topics Covered

| # | Topic |
|---|---|
| 1 | What is Docker Compose? |
| 2 | docker run vs docker-compose — Side by Side |
| 3 | The Complete `mongo-docker-compose.yaml` File |
| 4 | Docker Compose Commands |
| 5 | What Compose Creates Automatically |
| 6 | Key Takeaways |

---

## 🤔 1. What is Docker Compose?

Running multiple containers with long `docker run` commands is **messy and error-prone**.

**Docker Compose** solves this by letting you define all containers, ports, networks, and environment variables in **one structured YAML file** — then start everything with a single command.

```
Without Compose          →   Multiple long docker run commands
With Compose             →   One docker-compose up command
```

> 💡 Docker Compose **automatically creates a shared network** for all services — no need to manually run `docker network create`.

---

## ⚖️ 2. docker run vs docker-compose — Side by Side

### MongoDB

| docker run command | docker-compose.yaml |
|---|---|
| `docker run -d` | `services:` |
| `--name mongodb` | `mongodb:` (service name) |
| `-p 27017:27017` | `ports: - 27017:27017` |
| `-e MONGO_INITDB_ROOT_USERNAME=admin` | `environment: - MONGO_INITDB_ROOT_USERNAME=admin` |
| `-e MONGO_INITDB_ROOT_PASSWORD=password` | `- MONGO_INITDB_ROOT_PASSWORD=password` |
| `--net mongo-network` | ✅ Auto-created by Compose |
| `mongo` | `image: mongo` |

### Mongo Express

| docker run command | docker-compose.yaml |
|---|---|
| `docker run -d` | `mongo-express:` (service name) |
| `--name mongo-express` | (service name = container name) |
| `-p 8080:8081` | `ports: - 8080:8081` |
| `-e ME_CONFIG_MONGODB_ADMINUSERNAME=admin` | `environment: - ME_CONFIG_MONGODB_ADMINUSERNAME=admin` |
| `-e ME_CONFIG_MONGODB_ADMINPASSWORD=password` | `- ME_CONFIG_MONGODB_ADMINPASSWORD=password` |
| `-e ME_CONFIG_MONGODB_SERVER=mongodb` | `- ME_CONFIG_MONGODB_SERVER=mongodb` |
| `--net mongo-network` | ✅ Auto-created by Compose |
| `mongo-express` | `image: mongo-express` |

---

## 📄 3. The Complete `mongo-docker-compose.yaml`

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

  mongo-express:
    image: mongo-express
    ports:
      - 8080:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
```

> ✅ Save this file as `mongo-docker-compose.yaml` in your project folder.

### YAML Structure Explained

```
version: '3'              ← Compose file version
services:                 ← All containers listed here
  mongodb:                ← Service/container name
    image: mongo          ← Docker image to use
    ports:                ← Port binding (HOST:CONTAINER)
      - 27017:27017
    environment:          ← Environment variables (-e flags)
      - KEY=value
  mongo-express:          ← Second container
    ...
```

---

## ▶️ 4. Docker Compose Commands

### Start all containers
```bash
docker-compose -f mongo-docker-compose.yaml up
```

**What happens:**
```
Creating network "myapp_default" with the default driver  ← auto network!
Creating myapp_mongodb_1        ← MongoDB container
Creating myapp_mongo-express_1  ← Mongo Express container
Attaching to myapp_mongo-express_1, myapp_mongodb_1
```

### Start in background (detached)
```bash
docker-compose -f mongo-docker-compose.yaml up -d
```

### Stop and remove all containers
```bash
docker-compose -f mongo-docker-compose.yaml down
```

> `down` also **removes the network** that was auto-created.

---

## 🌐 5. What Compose Creates Automatically

When you run `docker-compose up`, Docker Compose:

- ✅ Creates a **shared network** (`myapp_default`) automatically
- ✅ Starts **all services** in the correct order
- ✅ Names containers as `<project>_<service>_1` (e.g., `myapp_mongodb_1`)
- ✅ All containers can reach each other **by service name** (e.g., `mongodb`)

```bash
# Verify containers are running
docker ps

# Verify network was auto-created
docker network ls
# You'll see: myapp_default   bridge   local
```

**After docker-compose up:**
```
NAMES                    PORTS
myapp_mongo-express_1    0.0.0.0:8080->8081/tcp
myapp_mongodb_1          0.0.0.0:27017->27017/tcp
```

**After docker-compose down:**
```
NAMES  ← empty, all containers stopped & removed
```

> ⚠️ `docker-compose down` removes containers AND network. `docker-compose up` recreates them fresh.

---

## 📝 6. Key Takeaways

| Concept | Details |
|---|---|
| **What it is** | Tool to run multiple containers from a single YAML file |
| **File name** | `docker-compose.yaml` or `docker-compose.yml` |
| **Network** | Auto-created — no `docker network create` needed |
| **Start** | `docker-compose -f <file> up` |
| **Stop** | `docker-compose -f <file> down` |
| **Service names** | Act as hostnames inside the network |
| **Big advantage** | Replace multiple long `docker run` commands with one file |

---

## 🔁 Quick Comparison Summary

```
Before Compose (manual):
  docker network create mongo-network
  docker run -d --name mongodb -p 27017:27017 -e ... --net mongo-network mongo
  docker run -d --name mongo-express -p 8080:8081 -e ... --net mongo-network mongo-express

After Compose (one command):
  docker-compose -f mongo-docker-compose.yaml up
```

---

## 🔗 Resources

- 📖 [Docker Compose Docs](https://docs.docker.com/compose/)
- 📖 [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- 📺 [Docker Tutorial – TechWorld with Nana](https://youtu.be/tLK9nNFHWH8?si=d5GP-ffP5Qy_iECJ)
