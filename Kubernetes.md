## 1. What is Kubernetes?

- Kubernetes (K8s) is an **open-source container orchestration platform** developed by Google, now maintained by CNCF.
- It automates **deployment, scaling, and management** of containerized applications.
- Helps achieve **high availability, fault tolerance, scalability, and portability** across cloud/on-prem environments.

## 2. When to use Kubernetes?

Use Kubernetes when:

- You need to **manage multiple containers** across multiple nodes.
- Your application requires **scalability** (auto-scaling based on traffic).
- You want **high availability** (self-healing apps, restarts failed containers).
- You need **CI/CD integration** with rolling updates and rollbacks.
- You want to **deploy across hybrid or multi-cloud environments**.
- You need **service discovery, load balancing, and storage orchestration** out of the box.

Not ideal when:

- Small projects with few containers (Docker Compose is simpler).
- Single-node apps that don’t need orchestration.

## 3. What features do orchestration tools offer?

Container orchestration tools (like Kubernetes, Docker Swarm, Nomad) provide:

1. **Automated deployment & scaling** – Rollouts, rollbacks, horizontal scaling.
2. **Service discovery & load balancing** – Containers talk via DNS names/IPs.
3. **Self-healing** – Auto-restart crashed pods, reschedule on healthy nodes.
4. **Resource management** – Efficient CPU/RAM usage, quotas, and limits.
5. **Storage orchestration** – Attach persistent storage dynamically.
6. **Configuration management** – Secrets, config files injected into containers.
7. **Networking** – Ingress, ClusterIP, NodePort for external/internal access.
8. **Multi-cloud & portability** – Same configs run anywhere.

## 4. Kubernetes Components (Key Objects)

### 1. **Pod**

- Smallest deployable unit in K8s.
- Represents **one or more tightly coupled containers** running together.
- Shares:
    - **Network (IP address & port space)**
    - **Storage (volumes)**
- Usually 1 app container + sidecar (e.g., logging/monitoring).

### 2. **Service**

- Stable **network abstraction** for a group of pods.
- Pods are ephemeral (IPs change), Services provide a **constant endpoint**.
- Types:
    - **ClusterIP (default):** Internal-only access.
    - **NodePort:** Exposes app on `<NodeIP>:<Port>`.
    - **LoadBalancer:** Integrates with cloud load balancers.
    - **ExternalName:** Maps to external DNS name.

### 3. **Ingress**

- **API object that manages external access** to Services (HTTP/HTTPS).
- Provides:
    - Path-based routing (`/app1 → service1`, `/app2 → service2`).
    - SSL/TLS termination.
    - Load balancing.
- Requires an **Ingress Controller** (e.g., NGINX, Traefik).

### 4. **ConfigMap**

- Stores **non-sensitive configuration data** (key-value pairs).
- Example: App settings, environment variables, config files.
- Mounted into pods as:
    - Environment variables
    - Config files

### 5. **Secret**

- Stores **sensitive data** (passwords, API tokens, SSH keys).
- Base64-encoded for transport, but should be encrypted at rest.
- Used as env variables or mounted as files in pods.

### 6. **Volumes**

- Provide **persistent storage** to containers.
- Types:
    - **emptyDir** – Temporary, pod’s lifetime only.
    - **hostPath** – Uses node’s filesystem.
    - **PersistentVolume (PV)/PersistentVolumeClaim (PVC):** Decouples storage from pods.
    - **Cloud volumes** – EBS (AWS), GCE Persistent Disk, Azure Disk.

### 7. **Deployment**

- Manages **stateless applications**.
- Provides:
    - **ReplicaSets** – ensures desired # of pod replicas.
    - **Rolling updates** – update pods without downtime.
    - **Rollback** – revert to previous version.
- Best for microservices and scalable apps.

### 8. **StatefulSet**

- Manages **stateful applications** (apps needing stable identity & storage).
- Ensures:
    - **Stable network IDs** (`pod-0`, `pod-1`, …).
    - **Ordered deployment & scaling**.
    - **Persistent storage per pod** (each pod gets its own PVC).
- Used for: Databases (MySQL, Cassandra, Kafka, etc.).

![K8 Components.jpg](attachment:11a9b550-9a45-465a-ac6b-a9fd76a549c2:K8_Components.jpg)

## 5. Kubernetes Architecture

### 1. Types of Nodes

A Kubernetes cluster consists of **two types of nodes**:

### a) **Master Node (Control Plane)**

- Responsible for **managing the cluster**.
- Controls scheduling, monitoring, and overall administration.
- Runs critical components including:
    - **API Server** → Entry point for kubectl and client operations.
    - **Controller Manager** → Maintains desired state (replicas, scaling).
    - **Scheduler** → Assigns pods to appropriate worker nodes.
    - **etcd** → Distributed key-value store for all cluster data and configuration.

### b) **Worker Node (Slave Node)**

- Responsible for **running applications**.
- Hosts **multiple pods** on each node.
- Receives and executes instructions from the master node.
- Maintains pods in a **running, healthy, and connected** state.

### 2. Pod Distribution Across Nodes

- Pods, the smallest deployable units in Kubernetes, are distributed **across multiple worker nodes**.
- Each node can host **several pods** based on its available resources.
- The control plane (master node) **distributes workloads evenly** across worker nodes.

### 3. Essential Processes on Every Node

Each **Kubernetes node** (master or worker) runs 3 core processes:

### a) **Container Runtime**

- Responsible for running the containers inside pods.
- Examples: **Docker, containerd, CRI-O**.
- Kubernetes communicates with different runtimes through the **Container Runtime Interface (CRI)**.

### b) **Kubelet**

- Agent running on **every node**.
- Functions:
    - Registers the node with the cluster.
    - Ensures containers in pods run as expected.
    - Reports node & pod status to the Master.
    - Fetches configuration from the API server and implements it locally.

### c) **Kube-Proxy**

- Manages **networking rules** on each node.
- Enables **service discovery & load balancing** for pods.
- Facilitates communication:
    - Pod ↔ Pod within cluster.
    - Pod ↔ Service ↔ External traffic.

- **Master = Brain (control, scheduling, monitoring)**.
- **Worker = Muscles (actual execution of pods)**.
- Each node needs **Container Runtime, Kubelet, and Kube-Proxy** to function.

## Kubernetes Master Node Processes

The **Master Node (Control Plane)** manages the entire Kubernetes cluster.

It runs **four core processes**:

### 1. **API Server**

- The **central entry point** for all interactions with the Kubernetes cluster.
- Receives and processes **requests** from users through `kubectl`, dashboard, or direct API calls.
- Validates and handles REST API requests (such as creating pods, checking pod status, or scaling deployments).
- Functions as a **communication hub**:
    - Connects all other components (scheduler, controller manager, kubelet).
    - Maintains cluster state in **etcd**.
- Serves as the **"front desk"** of the Kubernetes system.

### 2. **Scheduler**

- Responsible for **assigning pods** to nodes.
- Functions:
    - Checks **resource availability** (CPU, memory, GPU).
    - Matches **pod requirements** (labels, affinity/anti-affinity rules, taints/tolerations).
    - Selects the **best node** for a pod.
- Example: When **Node 1 is 30% used** and **Node 2 is 60% used**, the scheduler will prefer Node 1 to balance workloads.
- Doesn't execute pods itself — only **determines placement**.

### 3. **Controller Manager**

- Maintains the **desired state of the cluster** by continuously comparing it to the actual state.
- Manages various controllers (background processes):
    - **Node Controller** → Monitors node health.
    - **Replication Controller** → Maintains the correct number of pod replicas.
    - **Endpoints Controller** → Maps Services to Pods.
    - **Namespace & ServiceAccount Controllers** → Handle cluster resources.
- When issues occur (such as a pod crash), the Controller Manager automatically resolves them by **creating, deleting, or updating resources**.

### 4. **etcd**

- A **distributed, reliable key-value store** that Kubernetes uses to store its **cluster state**.
- Stores:
    - Configurations (pods, deployments, services).
    - Status information (active pods and their node locations).
    - Secrets and metadata.
- Requires **high availability** through replication across multiple master nodes.
- Functions as Kubernetes' **central database and memory**.

- **API Server → Front Door** (accepts commands).
- **Scheduler → Decider** (places pods on nodes).
- **Controller Manager → Maintainer** (ensures state consistency).
- **etcd → Database** (stores cluster state).