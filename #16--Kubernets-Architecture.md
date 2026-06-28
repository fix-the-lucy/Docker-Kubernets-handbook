# ☸️ Kubernetes Architecture — Master & Node Processes

> **Topic:** Kubernetes Architecture Deep Dive — Master Processes (API Server, Scheduler, Controller Manager, etcd) and Node Processes (Kubelet, Kube Proxy)

---

## 📋 Topics Covered

| # | Topic |
|---|---|
| 1 | Kubernetes Architecture Overview |
| 2 | How Do You Interact with the Cluster? |
| 3 | Node Processes — Kubelet |
| 4 | Node Processes — Kube Proxy |
| 5 | Master Processes — API Server |
| 6 | Master Processes — Scheduler |
| 7 | Master Processes — Controller Manager |
| 8 | Master Processes — etcd |
| 9 | Adding a New Master/Node Server |

---

## 🏗️ 1. Kubernetes Architecture Overview

A Kubernetes cluster is made up of **2 types of servers**:

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│  Master  │     │  Node 1  │     │  Node 2  │
└──────────┘     └──────────┘     └──────────┘
```

| Server Type | Role |
|---|---|
| **Master** | The "brain" — manages the cluster, makes decisions |
| **Node** (Worker) | Does the **actual work** — runs your Pods/containers |

> A production cluster usually has **multiple Master nodes** (for high availability) and **multiple Worker nodes**.

---

## ❓ 2. How Do You Interact with the Cluster?

Before diving into components, here's the big question this section answers:

**How to:**
- → schedule a Pod?
- → monitor the cluster?
- → re-schedule/re-start a Pod?
- → join a new Node?

> All of this is handled by a set of **background processes** running on Master and Worker nodes.

---

## 🖥️ 3. Node Processes — Kubelet

```
┌─────────────────────────┐
│         Node 1           │
│  ┌────────┐              │
│  │ my-app │              │
│  └────────┘              │
│  ┌────────┐              │
│  │   DB   │              │
│  └────────┘              │
│                          │
│  [container runtime] [Kubelet]
└─────────────────────────┘
```

### What is a Node?
**Node** = a worker machine in the K8s cluster — where your actual Pods run.

### Key Facts:
- Each **Node** has **multiple Pods** running on it
- **3 processes** must be installed on every Node:
  1. **Container runtime** (e.g. Docker) — actually runs the containers
  2. **Kubelet** — talks to both the container and the Node
  3. **Kube Proxy** — handles networking/forwarding
- **Worker Nodes do the actual work** (running apps, DBs, etc.)

### Kubelet:
- **Interacts with both** — the container (via container runtime) and the Node itself
- **Starts the Pod** with a container inside it
- Kubelet is essentially the "agent" on every Node that the Master talks to

```
Master  ──(instructs)──►  Kubelet (on Node)  ──(starts)──►  Pod + Container
```

---

## 🔀 4. Node Processes — Kube Proxy

```
┌──────────────┐                    ┌──────────────┐
│    Node 1     │                    │    Node 2     │
│  ┌────────┐  │                    │  ┌────────┐  │
│  │ my-app │  │                    │  │ my-app │  │
│  └────────┘  │                    │  └────────┘  │
│  ┌────────┐  │   [DB Service]     │  ┌────────┐  │
│  │   DB   │  │◄──────────────────►│  │   DB   │  │
│  └────────┘  │                    │  └────────┘  │
│ [Kube Proxy] │                    │ [Kube Proxy] │
└──────────────┘                    └──────────────┘
```

### Kube Proxy:
- **Forwards the requests** — ensures network traffic correctly reaches the right Pod, even across different Nodes
- Makes a **Service** work seamlessly across the cluster — whether the Pod is on Node 1 or Node 2, requests get routed correctly
- Runs as a process on **every Node**, alongside Kubelet

---

## 👑 5. Master Processes — API Server

```
                    Client
                      │
              ┌───────┴────────┐
            Update            Query
                      │
              ┌───────▼────────┐
              │   API Server   │
              └────────────────┘
                  Master 1
```

### API Server:
- **The gateway** to the cluster — all requests (update or query) go through it first
- It's the **only entry point** for interacting with Kubernetes (whether from `kubectl`, a UI, or another internal process)

### Flow of a request:
```
Some request
     ↓
 API Server
     ↓
validates request
     ↓
..other processes..
     ↓
    Pod
```

> Every single action in K8s — creating a Pod, querying status, updating a Deployment — **starts at the API Server**, which validates it before passing it along.

---

## 📅 6. Master Processes — Scheduler

```
Client → API Server → Scheduler
```

### Scheduler:
- Decides **where** (which Node) a new Pod should be placed
- **Just decides** — it does NOT actually create/start the Pod (that's Kubelet's job)

### Flow: "Schedule new Pod"
```
Schedule new Pod
       ↓
   API Server
       ↓
   Scheduler
       ↓
"Where to put the Pod?"
```

### How it decides:
```
Node 1 (30% used)   Node 2 (60% used)
   [Pod][Pod]          [Pod][Pod][Pod]
```

> Scheduler looks at resource usage across Nodes (e.g. Node 1 = 30% used, Node 2 = 60% used) and picks the **best-fit Node** — usually the one with **more free capacity**.

---

## 🎛️ 7. Master Processes — Controller Manager

```
Controller Manager
       ↓
   Scheduler
       ↓
   Kubelet
```

### Controller Manager:
- **Detects cluster state changes** — like a Pod going down/crashing
- Example: If a Pod on Node 1 dies (❌), the Controller Manager **notices** the change

### Flow when a Pod crashes:
```
Pod crashes (❌ on Node 1)
       ↓
Controller Manager detects it
       ↓
   Scheduler (decides where to reschedule)
       ↓
   Kubelet (starts the new Pod)
```

> Controller Manager is what makes Kubernetes **self-healing** — if something dies, it notices and triggers a reschedule.

---

## 🗄️ 8. Master Processes — etcd

```
┌──────────────────┐
│   Cluster Brain    │  ← etcd's role
│   👑                │
│  Key Value Store   │  ← how it stores data
└──────────────────┘
```

### etcd:
- The **"cluster brain"** — a **key-value store** that holds the entire cluster's state
- Stores info like: which Nodes exist, which Pods are running where, current configuration, etc.
- All Master processes (API Server, Scheduler, Controller Manager) **read/write to etcd** to know the current state of the cluster

### Complete Master Process Stack:
```
┌────────────────────┐
│     API Server       │  ← entry point for all requests
├────────────────────┤
│     Scheduler        │  ← decides where Pods go
├────────────────────┤
│  Controller Manager  │  ← detects state changes, self-healing
├────────────────────┤
│       etcd           │  ← stores the cluster's actual state
└────────────────────┘
        Master 1
```

---

## ➕ 9. Adding a New Master/Node Server

```
        ┌─────────┐ ┌─────────┐ ┌─────────┐
        │ Master  │ │ Master  │ │ Master  │
        └─────────┘ └─────────┘ └─────────┘
              │           │           │
   ┌──────────┴───────────┴───────────┴──────────┐
   │       │       │       │       │       │     │
[Node] [Node] [Node] [Node] [Node] [Node]
```

### Steps to add a new Master or Node:

1. **Get a new bare server** (VM or physical machine)
2. **Install all the required processes:**
   - For a **Master** → API Server, Scheduler, Controller Manager, etcd
   - For a **Node (Worker)** → Container runtime, Kubelet, Kube Proxy
3. **Add it to the cluster** — it joins and starts participating

> Multiple Master nodes = **High Availability**. If one Master goes down, others keep the cluster running.

---

## 🧠 Complete Picture: How a Pod Gets Created

```
1. Client sends request → API Server
2. API Server validates the request
3. Scheduler decides which Node has capacity
4. Controller Manager monitors the process
5. etcd stores the new desired state
6. Kubelet (on the chosen Node) starts the Pod
7. Kube Proxy sets up networking for the Pod
```

---

## 💡 Key Takeaways

| Process | Location | Role |
|---|---|---|
| **API Server** | Master | Entry point — validates all requests |
| **Scheduler** | Master | Decides which Node a Pod should run on |
| **Controller Manager** | Master | Detects changes, triggers self-healing |
| **etcd** | Master | Key-value store — the cluster's "brain"/source of truth |
| **Kubelet** | Node | Starts/manages Pods on that Node, talks to Master |
| **Kube Proxy** | Node | Forwards network requests to the right Pod |
| **Container Runtime** | Node | Actually runs the containers (e.g. Docker) |

### Summary Table: Master vs Node

| | Master | Node (Worker) |
|---|---|---|
| Role | Brain / Control plane | Does the actual work |
| Key processes | API Server, Scheduler, Controller Manager, etcd | Kubelet, Kube Proxy, Container Runtime |
| Runs your apps? | ❌ No | ✅ Yes |
| Can have multiple? | ✅ Yes (for HA) | ✅ Yes (for scale) |

---

*Notes based on: Kubernetes Architecture — Node & Master Processes (TechWorld with Nana)*
