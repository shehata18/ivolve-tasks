# Lab 15: Node Isolation Using Taints in Kubernetes

## 🎯 Objective

The goal of this lab is to understand how to **use taints and tolerations** in Kubernetes to control which pods can be scheduled on specific nodes.
Taints allow you to **isolate nodes** so that only pods with matching tolerations can run on them.

---

## 🧠 Concept Overview

In Kubernetes:

* **Taints** are applied to nodes to repel certain pods.
* **Tolerations** are applied to pods to allow them to be scheduled on nodes with matching taints.

When a node has a taint, **no pod will be scheduled on it** unless the pod specifically tolerates that taint.

**Taint format:**

```
key=value:effect
```

**Effect options:**

* `NoSchedule`: Pod won’t be scheduled unless it tolerates the taint.
* `PreferNoSchedule`: Kubernetes tries to avoid scheduling the pod.
* `NoExecute`: Existing pods without toleration will be evicted.

Example:

```
workload=worker:NoSchedule
```

---

## ⚙️ Lab Steps

### Step 1: Check Cluster Nodes

List all nodes in your Kubernetes cluster:

```bash
kubectl get nodes
```

If you only have one node, add another one:

```bash
minikube node add
```

---

### Step 2: Apply a Taint to the Second Node

```bash
kubectl taint nodes minikube-m02 workload=worker:NoSchedule
```

This command marks the second node so that no pod can run on it unless it has a matching toleration.

---

### Step 3: Verify the Taint

```bash
kubectl describe node minikube-m02 | grep -i taint
```

Expected output:

```
Taints: workload=worker:NoSchedule
```

---

### Step 4: Test Scheduling

Create a regular pod (without toleration):

```bash
kubectl run testpod --image=nginx
```

Check where it’s scheduled:

```bash
kubectl get pod -o wide
```

You’ll notice it’s **not** scheduled on the tainted node.

---

### Step 5: Create a Pod with Toleration

Create a file `toleration-pod.yml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerant-pod
spec:
  tolerations:
  - key: "workload"
    operator: "Equal"
    value: "worker"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
```

Apply it:

```bash
kubectl apply -f toleration-pod.yml
```

Now check:

```bash
kubectl get pod -o wide
```

This pod will be scheduled successfully on the tainted node.

---

### Step 6: Remove the Taint (Cleanup)

If you want to remove the taint:

```bash
kubectl taint nodes minikube-m02 workload=worker:NoSchedule-
```

---

## 🧾 Summary

| Command                                               | Description                        |
| ----------------------------------------------------- | ---------------------------------- |
| `kubectl get nodes`                                   | List all cluster nodes             |
| `kubectl taint nodes node-name key=value:NoSchedule`  | Add a taint to a node              |
| `kubectl describe node node-name`                     | View taints on a node              |
| `kubectl taint nodes node-name key=value:NoSchedule-` | Remove a taint                     |
| `tolerations` in Pod YAML                             | Allow pods to run on tainted nodes |

---

## ✅ Expected Outcome

By the end of this lab, you will:

* Understand the purpose of taints and tolerations.
* Successfully isolate a node using taints.
* Deploy pods that can and cannot tolerate the taint.
* Learn how to remove or modify taints in Kubernetes.
