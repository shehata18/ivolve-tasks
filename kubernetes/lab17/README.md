# Lab 17: Managing Configuration and Sensitive Data with ConfigMaps and Secrets

## 🎯 Objective

The goal of this lab is to learn how to **manage application configuration** and **store sensitive information** in a secure way using **ConfigMaps** and **Secrets** in Kubernetes.

---

## 🧠 Concept Overview

### 🔹 ConfigMap

A **ConfigMap** is used to store **non-sensitive** configuration data in key-value pairs.
Example: database hostname, port, or user.

### 🔹 Secret

A **Secret** is used to store **sensitive** data, such as passwords or tokens, in **base64 encoded** format.
Kubernetes keeps Secrets encrypted in etcd and makes them available only to authorized pods.

---

## ⚙️ Lab Setup

We will define:

* A **ConfigMap** containing MySQL configuration (non-sensitive data)
* A **Secret** containing MySQL credentials (sensitive data)
* A **Pod** that uses both the ConfigMap and Secret as environment variables

---

## 🧩 Step 1: Create a ConfigMap

Create a file named `mysql-configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  DB_HOST: mysql-service
  DB_USER: ivolve_user
```

Apply it:

```bash
kubectl apply -f mysql-configmap.yaml
```

Verify:

```bash
kubectl get configmap mysql-config -o yaml
```

---

## 🔐 Step 2: Create a Secret

First, encode the passwords using base64:

```bash
echo -n "ivolve123" | base64
echo -n "root123" | base64
```

Then create a file named `mysql-secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  DB_PASSWORD: aXZvbHZlMTIz        # base64 of ivolve123
  MYSQL_ROOT_PASSWORD: cm9vdDEyMw==  # base64 of root123
```

Apply it:

```bash
kubectl apply -f mysql-secret.yaml
```

Verify:

```bash
kubectl get secret mysql-secret -o yaml
```

---

## 🧱 Step 3: Create a Pod that Uses ConfigMap and Secret

Create a file named `app-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: nginx
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: mysql-config
          key: DB_HOST
    - name: DB_USER
      valueFrom:
        configMapKeyRef:
          name: mysql-config
          key: DB_USER
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysql-secret
          key: DB_PASSWORD
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysql-secret
          key: MYSQL_ROOT_PASSWORD
```

Apply it:

```bash
kubectl apply -f app-pod.yaml
```

---

## 🔍 Step 4: Verify the Environment Variables

Check the pod:

```bash
kubectl get pods
kubectl exec -it app-pod -- env | grep DB_
```

You should see:

```
DB_HOST=mysql-service
DB_USER=ivolve_user
DB_PASSWORD=ivolve123
MYSQL_ROOT_PASSWORD=root123
```

---

## 🧹 Step 5: Cleanup

```bash
kubectl delete pod app-pod
kubectl delete configmap mysql-config
kubectl delete secret mysql-secret
```

---

## 🧾 Summary

| Resource          | Purpose                                                        |
| ----------------- | -------------------------------------------------------------- |
| **ConfigMap**     | Store non-sensitive configuration (e.g., DB host, username)    |
| **Secret**        | Store sensitive data (e.g., passwords)                         |
| **env.valueFrom** | Used to inject data into containers from ConfigMaps or Secrets |

---

## ✅ Expected Outcome

By the end of this lab, you will:

* Understand the difference between ConfigMaps and Secrets.
* Know how to securely handle configuration and credentials.
* Successfully inject both into a running pod.
