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
  1. Create a new namespace my-namespace.
  2. Deploy an Nginx Pod in this namespace.
  3. Expose Pod on port 80.
  4. Verify pod & namespace isolation using kubectl get pods -n my-namespace.