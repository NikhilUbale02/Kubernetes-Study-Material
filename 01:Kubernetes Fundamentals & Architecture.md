🧠 Phase 1: Kubernetes Fundamentals & Architecture

---

📘 Concepts

1. Introduction to Kubernetes
- Kubernetes = container orchestration system.
- Purpose: Automate deployment, scaling, management of containerized apps.
- Core difference vs Docker: Docker = container runtime, K8s = orchestrator.

---

2. Kubernetes Architecture (Control Plane vs Worker Node)
- Control Plane Components
  - API Server → Entry point for all commands (kubectl, API calls).
  - etcd → Key-value store for cluster state.
  - Scheduler → Decides on which node pod should run.
  - Controller Manager → Watches cluster state and makes corrections. 

- Worker Node Components
  - Kubelet → Talks to API server, manages pods on the node.
  - Kube Proxy → Manages networking/routing to services.
  - Container Runtime → (Docker, containerd, CRI-O) runs containers.

---

3. Pods, ReplicaSets, Deployments
- Pod = smallest deployable unit in Kubernetes (wraps 1 or more containers).
- ReplicaSet = ensures a specified number of identical Pods are running.
- Deployment = manages ReplicaSets, supports rolling updates & rollbacks.

---

4. YAML Basics (Line-by-Line)
🔹 Simple Pod YAML
```bash
apiVersion: v1           # API version for Pod object
kind: Pod                # Resource type = Pod
metadata:                # Metadata = name, labels, namespace
  name: my-first-pod     # Unique pod name
  labels:
    app: demo            # Label key/value pair
spec:                    # Specification of pod (what to run)
  containers:            # List of containers in this pod
  - name: nginx-container # Container name
    image: nginx:1.25    # Docker image to use
    ports:
    - containerPort: 80  # Port exposed inside the container
```

---

5. Imperative vs Declarative
- Imperative = tell K8s what to do now.
  - Example: kubectl run nginx --image=nginx
- Declarative = define desired state in YAML, K8s reconciles it.
  - Example: kubectl apply -f pod.yaml

---

6. Namespaces
- Logical partitioning inside a cluster (like virtual clusters).
- Used for multi-team/multi-project isolation.
- Example: kubectl create namespace dev.

---

🛠️ Mini Project
- Goal: Deploy a simple Nginx web server in its own namespace.
  - Create a new namespace my-namespace.
  - Deploy an Nginx Pod in this namespace.
  - Expose Pod on port 80.
  - Verify pod & namespace isolation using kubectl get pods -n my-namespace.

---

🧠 Interview Tips

❓ Q1: Difference between Pod, ReplicaSet, and Deployment?
✅ Answer:
  - Pod → The smallest deployable unit in Kubernetes. It represents one or more tightly coupled containers that share the same network namespace (IP, port space) and can communicate via localhost.

  - ReplicaSet → Ensures that a specified number of replicas (copies) of a Pod are running at all times. If a Pod dies, the ReplicaSet automatically replaces it.

  - Deployment → A higher-level abstraction that manages ReplicaSets. It adds powerful features like rolling updates, rollbacks, and versioned history.

- Analogy:

- Pod = a single worker.

- ReplicaSet = ensures there are always enough workers available.

- Deployment = the manager who decides how workers are hired, fired, promoted, or retrained (updates/rollbacks).

❓ Q2: Why does Kubernetes use etcd?
✅ Answer:
  - etcd is a distributed, reliable key-value store that Kubernetes uses as its source of truth.
  - Stores all cluster data: state of Pods, ConfigMaps, Secrets, Nodes, RBAC policies, etc.
  - Provides strong consistency and fault tolerance using the Raft consensus algorithm.

- Without etcd → Kubernetes would have no memory of desired or current state.
In short → etcd = the “brain & memory” of Kubernetes.

❓ Q3: Can multiple containers run inside a Pod? Why?
✅ Answer:
  - Yes, multiple containers can run inside a single Pod.
  - They share:
    - Same network namespace (IP address, ports).
    - Same storage volumes (if defined).
  - Typical use case = when containers are tightly coupled and must share resources.

Examples:
  - Sidecar pattern: One container runs the app, another logs or proxies traffic.
  - Init containers: Prepares environment before main container starts.
- Why not always do this?
  - In practice, one Pod = one main container (for simplicity, scaling).
  - Multi-container Pods are used for special cases (logging, proxying, adapters).