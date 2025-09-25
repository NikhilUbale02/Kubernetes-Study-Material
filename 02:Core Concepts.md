🔧 Phase 2: Core Concepts

---

📘 Part 0: Foundations Recap (Before Deployments)

- 1.kubectl Basics
  - kubectl get pods → list pods
  - kubectl describe pod <name> → detailed info + events
  - kubectl logs <pod-name> → view container logs
  - kubectl exec -it <pod> -- /bin/sh → exec into container
  - Resource shortnames: po, rs, deploy, svc, ns
  - kubectl create → imperative, one-shot
  - kubectl apply -f → declarative, reconciles state

  ---

- 2.Pod Lifecycle States
  - Pending → Pod accepted, images pulling
  - Running → Containers up & healthy
  - Succeeded → Completed successfully (for Jobs)
  - Failed → Pod terminated with failure
  - CrashLoopBackOff → Container failing → restarting repeatedly
  - restartPolicy: Always (default), OnFailure, Never

---

- 3.Labels & Annotations (Intro)
  - Labels = key-value pairs for grouping/filtering.
  - Annotations = metadata, not for selection.
  🔹 Example:
```bash
metadata:
  name: nginx-pod
  labels:
    app: frontend
    env: dev
  annotations:
    owner: "team-alpha"
```
- Query: kubectl get pods -l app=frontend

---

- 4.Debugging Commands
  - kubectl get events --sort-by=.metadata.creationTimestamp
  - kubectl describe pod <name> → check why it failed
  - kubectl logs <pod>
  - kubectl exec -it <pod> -- /bin/bash

---

## 📘 Part 1: Deployments in Kubernetes

- Concepts
  - ReplicaSet ensures Pods stay alive, but lacks rollout strategy.
  - Deployment manages ReplicaSets, supports:
  - Rolling updates
  - Rollbacks
  - Declarative scaling

- YAML Example (Deployment)
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3                # desired Pod replicas
  selector:                  # how to identify pods
    matchLabels:
      app: nginx
  template:                  # Pod template
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.25
        ports:
        - containerPort: 80
```

---

## 📘 Part 2: Rolling Updates & Rollbacks

- Concepts
  - Default: RollingUpdate → replaces Pods gradually.
  - Strategies:
    - Recreate → kills all Pods, then creates new ones.
    - RollingUpdate → replaces Pods in batches.
- Rollback: kubectl rollout undo deployment/nginx-deployment

---

## 📘 Part 3: ConfigMaps vs Secrets

- ConfigMap = non-sensitive config (env vars, files).
- Secret = sensitive data (base64-encoded).
🔹 ConfigMap Example
```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: "production"
  APP_DEBUG: "false"
```

🔹 Secret Example
```bash
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=   # "admin"
  password: cGFzc3dvcmQ=  # "password"
```

---

## 📘 Part 4: Liveness & Readiness Probes

- Liveness probe → checks if container is alive (restart if fails).
- Readiness probe → checks if container is ready to serve traffic.
🔹 Example (Readiness & Liveness)
```bash
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

---

## 📘 Part 5: Services in Kubernetes

- ClusterIP (default) → internal service
- NodePort → exposes on node’s port
- LoadBalancer → external access via cloud LB
- Headless Service → DNS but no load-balancing

---

## 📘 Part 6: Volumes in Kubernetes

- emptyDir → ephemeral storage
- hostPath → node’s filesystem
- PersistentVolume (PV) + PVC → decoupled storage
- StorageClass → dynamic provisioning

---

## 📘 Part 7: Jobs & CronJobs

- Job = one-off task
- CronJob = scheduled task (like backups)
🔹 Example CronJob
```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup
spec:
  schedule: "0 2 * * *"   # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: alpine
            command: ["sh", "-c", "echo backup db"]
          restartPolicy: OnFailure
```

---

## 📘 Part 8: StatefulSets & Headless Services

- StatefulSet = for stateful apps (DBs).
- Stable Pod names, stable storage.
- Needs Headless Service for DNS.

---

## 📘 Part 9: Resource Requests & Limits

- Request = minimum guaranteed
- Limit = maximum allowed
- Helps scheduler place Pods properly.

---

🛠️ Mini Project

- Deploy a NodeJS app with:
  - ConfigMap for environment config.
  - Secret for DB credentials.
  - Liveness & Readiness probes.
  - Service (ClusterIP + NodePort).
  - Rollout new version → test rollback.

---

🧠 Interview Tips

❓ Q1: Deployment vs ReplicaSet
- Short:
  - ReplicaSet → ensures fixed number of Pods.
  - Deployment → manages ReplicaSets + adds rollouts & rollbacks.

- Expanded:
  - A ReplicaSet only ensures that a specified number of Pod replicas are running at any given time. It does not support updates or versioning.
  - A Deployment sits on top of ReplicaSets. It allows rolling updates, rollbacks, and keeps track of revision history.
  - In practice → you almost always use Deployments, not bare ReplicaSets.

  ---

  ❓ Q2: Difference between Liveness & Readiness probe
- Short:
  - Liveness probe → checks if container is alive; restarts if it fails.
  - Readiness probe → checks if container is ready to accept traffic.

- Expanded:
  - Liveness probe is about self-healing. If the probe fails, Kubelet kills and restarts the container.
  - Readiness probe is about service availability. If the probe fails, the Pod stays alive but is removed from Service endpoints so traffic won’t be sent to it.
  - Example:
    - Liveness = “Is the app still running?”
    - Readiness = “Is the app warmed up and ready to serve requests?”

---

❓ Q3: ConfigMap vs Secret? Why base64 in Secret?
- Short:
  - ConfigMap = stores non-sensitive config.
  - Secret = stores sensitive info, base64-encoded.

- Expanded:
  - ConfigMap is used for non-sensitive data like feature flags, app modes, config files.
  - Secret is used for sensitive data like passwords, tokens, SSH keys.
  - Kubernetes requires Secret data to be stored as base64-encoded strings in YAML.
    - Base64 is not encryption — just encoding.
    - Purpose: ensures all data fits the JSON/YAML spec safely, even binary.
    - Real security comes from restricting RBAC and enabling secret management integrations (Vault, KMS).

---

❓ Q4: How does scheduler pick a node?
- Short:
  - Scheduler matches Pod requirements with Node resources.

- Expanded:
  - The Kube-Scheduler places Pods on Nodes in two steps:
    - Filtering → Removes nodes that don’t meet Pod’s requirements (CPU/Memory requests, taints/tolerations, affinity rules, node selectors).
    - Scoring → Ranks remaining nodes based on resource availability, affinity preference, and policies.
  - Scheduler then assigns the Pod to the best-scoring node.
  - If no node fits, the Pod stays in Pending state.

---

❓ Q5: How would you debug a failing Pod rollout?

- Short:
  - Check rollout status, events, logs, and describe Pod.

- Expanded (Step-by-Step):
  - 1.Check rollout status:
```bash
kubectl rollout status deployment <name>
```

  - 2.Check events:
```bash
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

  - 3.Check container logs:
```bash
kubectl logs <pod-name>
```
  
  - 4.Exec into container:
```bash
kubectl exec -it <pod> -- /bin/sh
```

  - 5.Rollback if needed:
```bash
kubectl rollout undo deployment <name>
```

---

🔑 Hidden Gems

- kubectl rollout history deployment <name> → shows version history.
- kubectl explain deployment.spec.strategy → deep dive on rollout strategy.
- Use Headless Services with StatefulSets → stable DNS names.
- For Secrets: kubectl create secret generic is faster than writing YAML.

---

🏡 Homework (5 Tasks)

- Create a Deployment of Nginx with 3 replicas.
- Perform a rolling update to a new version, then rollback.
- Write a ConfigMap + Secret, inject into Pod as env vars.
- Add liveness + readiness probes to your app.
- Create a CronJob that prints “Hello Kubernetes” every minute.

