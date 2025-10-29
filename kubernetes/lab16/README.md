# Lab 16: Namespace Management and Resource Quota Enforcement

## 🎯 Objective

The objective of this lab is to learn how to **create and manage Kubernetes namespaces** and apply **resource quotas** to control resource usage within a namespace.
By setting limits, you ensure that a specific namespace does not consume more resources than allowed — promoting fair resource sharing in multi-user or multi-team environments.

---

## 🧠 Concept Overview

### 🔹 Namespace

A **Namespace** in Kubernetes is a logical partition of the cluster, allowing you to separate and organize resources.
Each namespace provides isolation between resources such as Pods, Services, and Deployments.

Example use cases:

* `dev`, `test`, and `prod` environments.
* Different teams working on the same cluster.

### 🔹 ResourceQuota

A **ResourceQuota** is a policy applied to a namespace that restricts the number or amount of resources (like Pods, CPUs, or Memory) that can be used.

In this lab, we’ll create a quota that **limits the number of pods to 2** inside the namespace `ivolve`.

---

## ⚙️ Lab Steps

### Step 1: Create a Namespace

Create a new namespace called `ivolve`:

```bash
kubectl create namespace ivolve
```

Verify the namespace:

```bash
kubectl get namespaces
```

Expected output:

```
NAME              STATUS   AGE
default           Active   1h
kube-system       Active   1h
ivolve            Active   10s
```

---

### Step 2: Define a ResourceQuota

Create a YAML file named `quota.yml`:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-limit
  namespace: ivolve
spec:
  hard:
    pods: "2"
```

**Explanation:**

| Field                 | Description                                           |
| --------------------- | ----------------------------------------------------- |
| `kind: ResourceQuota` | Defines that this is a resource quota object.         |
| `metadata.namespace`  | Specifies the namespace where the quota applies.      |
| `spec.hard.pods: "2"` | Limits the number of pods to 2 within that namespace. |

---

### Step 3: Apply the ResourceQuota

Apply the configuration:

```bash
kubectl apply -f quota.yml
```

Verify:

```bash
kubectl get resourcequota -n ivolve
```

Expected output:

```
NAME         AGE   HARD   USED
pod-limit    1m    2      0
```

You can also view details:

```bash
kubectl describe resourcequota pod-limit -n ivolve
```

---

### Step 4: Test the Quota Limit

Create two pods within the `ivolve` namespace:

```bash
kubectl run nginx1 --image=nginx -n ivolve
kubectl run nginx2 --image=nginx -n ivolve
```

Now, try to create a third pod:

```bash
kubectl run nginx3 --image=nginx -n ivolve
```

You’ll receive an error like:

```
Error from server (Forbidden): exceeded quota: pod-limit, requested: pods=1, used: pods=2, limited: pods=2
```

This confirms the quota is enforced successfully ✅

---

### Step 5 (Optional): Update or Remove the Quota

To increase the pod limit to 4, edit and reapply the YAML:

```yaml
spec:
  hard:
    pods: "4"
```

Then:

```bash
kubectl apply -f quota.yml
```

To delete the quota:

```bash
kubectl delete resourcequota pod-limit -n ivolve
```

---

## 🧾 Summary

| Command                                              | Description                       |
| ---------------------------------------------------- | --------------------------------- |
| `kubectl create namespace ivolve`                    | Create a new namespace            |
| `kubectl apply -f quota.yml`                         | Apply the resource quota          |
| `kubectl get resourcequota -n ivolve`                | View the quota in a namespace     |
| `kubectl describe resourcequota pod-limit -n ivolve` | Show detailed quota info          |
| `kubectl run nginx1 --image=nginx -n ivolve`         | Create a pod inside the namespace |
| `kubectl delete resourcequota pod-limit -n ivolve`   | Remove the quota                  |

---

## ✅ Expected Outcome

By completing this lab, you will:

* Understand the purpose of namespaces in Kubernetes.
* Learn how to enforce resource limits using ResourceQuotas.
* Successfully restrict the number of pods within a namespace.
* Manage, modify, and remove quotas as needed.
