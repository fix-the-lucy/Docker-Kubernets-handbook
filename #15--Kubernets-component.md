# Kubernetes (K8s) Components — Quick Notes

---

## 1. Node and Pod

### Node
- A physical or virtual **machine** in the K8s cluster.
- Runs the actual workloads (pods).

### Pod
- **Smallest unit** of Kubernetes.
- An **abstraction over a container** (you don't interact with Docker directly).
- Usually **1 application per Pod**.
- Each Pod gets its **own IP address**.
- ⚠️ IP changes on restart/re-creation → that's why we need **Services**.

**Example:**
```
Node 1
 ├── Pod: my-app  (IP: 10.0.0.1)
 └── Pod: DB      (IP: 10.0.0.2)
```

---

## 2. Service and Ingress

### Service
- Provides a **permanent/static IP address** attached to a Pod.
- Even if the Pod dies and restarts, the Service IP stays the same.
- Two types:
  - **External Service** → accessible from outside (e.g., `http://my-app-service-ip:port`)
  - **Internal Service** → NOT accessible from outside (e.g., DB service)

**Example:**
```
my-app Pod  ←→  Service (permanent IP)  ←→  User
DB Pod      ←→  Service (internal only, no external access)
```

### Ingress
- Acts as a **domain-name gateway** — routes external traffic using a proper URL.
- Instead of `http://my-app-ip:8080`, users access `https://my-app.com`.

**Example:**
```
User → https://my-app.com → Ingress → Service → Pod
```

---

## 3. ConfigMap and Secret

### ConfigMap
- Stores **external configuration** for your application (non-sensitive data).
- Avoids hardcoding config values inside the app/image.
- Example: database URL

**Example:**
```yaml
DB_URL = mongo-db   ← stored in ConfigMap, read by my-app Pod
```
Without ConfigMap, changing the DB URL would require rebuilding the image, pushing to repo, and re-pulling in the pod.

### Secret
- Like ConfigMap, but for **sensitive data** (passwords, tokens, keys).
- Data is **base64 encoded**.

**Example:**
```yaml
DB_USER = mongo-user   ← stored in Secret
DB_PWD  = mongo-pwd    ← stored in Secret
```

> ⚠️ Secrets are base64 encoded, **not encrypted** by default. Use additional tools (e.g., Vault) for true encryption.

---

## 4. Volumes

- K8s **does not manage data persistence** on its own.
- Volumes attach **storage** (local or remote) to a Pod so data survives restarts.
- Two types:
  - **Local** — storage on the same node
  - **Remote** — external storage (cloud, NFS, etc.)

**Example:**
```
DB Pod → Volumes → local disk (on Node 1)
                 → remote storage (cloud)
```

> Think of it like plugging in an external hard drive to your Pod.

---

## 5. Deployment and StatefulSet

### Deployment
- A **blueprint** for creating Pods (replica management).
- Used for **stateless applications** (e.g., web frontends).
- Handles scaling, rolling updates, and self-healing.

**Example YAML:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2          # 2 copies of the pod
  selector:
    matchLabels:
      app: my-app
  template:
    spec:
      containers:
        - name: my-app
          image: my-image
          ports:
            - containerPort: 8080
```

### StatefulSet
- Used for **stateful applications** or **databases** (e.g., MongoDB, MySQL).
- Manages ordered, stable Pod names and persistent storage.
- More complex than Deployment — often DBs are hosted **outside** the K8s cluster.

**Example:**
```
Node 1           Node 2
my-app (Deployment) ←→  my-app (Deployment)
DB (StatefulSet)    ←→  DB (StatefulSet)
```

---

## 6. Summary — Main K8s Components

| Component     | Purpose |
|---------------|---------|
| **Pod**       | Smallest unit; runs your container |
| **Service**   | Stable IP/DNS for accessing Pods |
| **Ingress**   | Routes external HTTP/S traffic by domain |
| **ConfigMap** | External non-sensitive config (e.g., DB URL) |
| **Secret**    | Sensitive config, base64 encoded (e.g., passwords) |
| **Volumes**   | Persistent storage (local or remote) |
| **Deployment**| Manages stateless app replicas |
| **StatefulSet**| Manages stateful apps/databases |

---

> **Key Rule:** Use `Deployment` for stateless apps, `StatefulSet` for stateful apps or databases.
> 
