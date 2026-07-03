# Kubernetes in Practice — Minikube & kubectl

A complete guide covering what Minikube and kubectl are, how clusters are set up, and how to install everything.

---

## 1. What is Minikube?

Minikube is a **CLI tool** that provisions and manages **single-node Kubernetes clusters** optimized for development/testing workflows.

### How it works:
- Creates a **Virtual Box** on your laptop
- A **Node** runs inside that Virtual Box
- Both **Master processes** and **Worker processes** run on that **ONE machine** (unlike production)
- It is a **1 Node K8s cluster** — meant only for testing purposes

> 💡 In production, Master and Worker nodes run on **separate** machines. In Minikube, everything is on one machine to keep it lightweight.

---

## 2. Test/Local Cluster vs Production Cluster

| Feature | Test/Local (Minikube) | Production Cluster |
|---|---|---|
| Nodes | 1 Node | Multiple Nodes |
| Master | Runs on same machine | Separate dedicated machines |
| Worker | Runs on same machine | Separate dedicated machines |
| Purpose | Development & Testing | Real-world, live workloads |
| Setup | Virtual Box on laptop | Separate VMs or physical machines |

### Production Cluster Setup:
- **Multiple Master nodes** — each with: API Server, Scheduler, Controller Manager, etcd
- **Multiple Worker nodes** — each running multiple PODs with Docker + kubelet
- All on **separate virtual or physical machines**

---

## 3. What is kubectl?

`kubectl` is the **command-line tool (CLI)** for interacting with a Kubernetes cluster.

### How it works:
- kubectl talks to the **API Server** on the Master node
- The API Server has 3 types of clients: **UI**, **API**, and **CLI (kubectl)**
- kubectl is the **most powerful** of all 3 clients

### What kubectl does:
- **Enables interaction** with the cluster
- **Creates pods** on nodes
- Manages: `Service`, `Secret`, `ConfigMap`, and more
- Worker processes **enable pods to run** on a node

### kubectl works with BOTH:
- **Minikube cluster** (local/test)
- **Cloud cluster** (production on AWS, Azure, GCP)

> Same `kubectl` commands work on local and cloud clusters.

---

## 4. Minikube Basic Commands

```bash
minikube start       # Starts a local Kubernetes cluster
minikube status      # Gets the status of the local cluster
minikube stop        # Stops a running local cluster
minikube delete      # Deletes the local cluster
minikube dashboard   # Opens the Kubernetes dashboard in browser
```

### Image Commands:
```bash
minikube docker-env  # Sets up Docker env variables
minikube cache       # Add or delete an image from local cache
```

### Config & Management:
```bash
minikube addons      # Modify Kubernetes addons
minikube config      # Modify minikube config
minikube profile     # Get or set current minikube profile
```

---

## 5. Installation — Setting Up Minikube

### Official Docs:
- **Install Minikube:** `https://kubernetes.io/docs/tasks/tools/install-minikube/`
- **Install kubectl:** `https://kubernetes.io/docs/tasks/tools/install-kubectl/`

### Step 1: Install a Hypervisor
Minikube needs a hypervisor to create a virtual machine. Choose one:
- **KVM** (also uses QEMU) — for Linux
- **VirtualBox** — cross-platform (Linux, macOS, Windows)
- **HyperKit** — for macOS

### Step 2: Install HyperKit (macOS example)
```bash
brew update
brew install hyperkit
```

### Step 3: Install Minikube
```bash
brew install minikube
```

### Step 4: Start Minikube Cluster
```bash
minikube start --vm-driver=hyperkit
```

Output will show:
```
minikube v1.6.2 on Darwin 10.14.1
Selecting 'hyperkit' driver from user configuration
Starting existing hyperkit VM for "minikube" ...
Waiting for the host to be provisioned ...
```

> 💡 Tip: Use `minikube start -p <name>` to create a new cluster, or `minikube delete` to delete the current one.

---

## 6. TL;DR Summary

| Tool | What it does |
|---|---|
| **Minikube** | Runs a single-node K8s cluster locally for testing |
| **kubectl** | CLI to interact with any K8s cluster (local or cloud) |
| **Virtual Box / HyperKit** | Hypervisor that Minikube uses to create a VM |
| **API Server** | The gateway that kubectl talks to on the Master node |

- Use **Minikube** to practice Kubernetes locally
- Use **kubectl** to deploy, manage, and monitor your cluster
- For production, switch to a **Cloud cluster** — same kubectl commands apply!
