🔐 Phase 3: Security & Access

---

📘 Concepts

- 1.Docker Security Best Practices
  - Don’t run containers as root unless required.
  - Use minimal base images (distroless, alpine).
  - Keep images scanned and updated.
  - Principle of least privilege → applies to containers & Kubernetes.

---

- 2.Security Contexts
- Defines privileges and access control settings for Pods/containers.
🔹 Example YAML
```bash
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    securityContext:
      runAsUser: 1000         # non-root UID
      runAsGroup: 3000        # non-root GID
      allowPrivilegeEscalation: false
      capabilities:           # fine-grained Linux permissions
        drop: ["ALL"]
```

---

- 3.Service Accounts
  - Every Pod uses a ServiceAccount to talk to the API Server.
  - By default → default ServiceAccount in its namespace.
  - Custom ServiceAccounts → fine-grained access.
🔹 Example YAML
```bash
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-serviceaccount
  namespace: dev
```

---

- Attach to Pod:
```bash
spec:
  serviceAccountName: my-serviceaccount
```

---

- 4.Role-Based Access Control (RBAC)
  - Defines who can do what on which resource.
  - Roles & ClusterRoles define permissions.
  - RoleBindings & ClusterRoleBindings assign them.

---

- 5.Roles vs ClusterRoles
  - Role → namespace-scoped.
  - ClusterRole → cluster-wide.
🔹 Role Example (namespace level)
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

🔹 RoleBinding Example
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

- 6.ClusterRole + ClusterRoleBinding Example
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-lite
rules:
- apiGroups: ["*"]
  resources: ["nodes", "pods", "deployments"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: bind-alice
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin-lite
  apiGroup: rbac.authorization.k8s.io
```

---

🛠️ Mini Project
- Goal: Secure an app with RBAC & ServiceAccounts
- 1.Create a namespace secure-ns.
- 2.Deploy an Nginx Pod with non-root user (runAsUser: 1000).
- 3.Create a ServiceAccount app-sa and attach it to Pod.
- 4.Create a Role that allows get, list on Pods only.
- 5.Bind Role to the ServiceAccount.
- 6.Verify: Pod using app-sa can list Pods in its namespace but not delete them.

---

🧠 Interview Tips

> Q1: Role vs ClusterRole?
- Role = namespace-scoped.
- ClusterRole = cluster-wide.

> Q2: Why ServiceAccount instead of default?
- Default SA is over-privileged.
- Custom SA → least privilege principle.

> Q3: What’s the purpose of SecurityContext?
- Enforce non-root execution, drop capabilities, prevent privilege escalation.

> Q4: How does RBAC evaluation work?
- Subject (User/SA) → Binding → Role → Rule.
- If no allow rule exists → denied (default deny).

> Q5: How to debug RBAC issues?
- kubectl auth can-i <verb> <resource> --as <user>
Example:
```bash
kubectl auth can-i list pods --as=alice -n dev
```