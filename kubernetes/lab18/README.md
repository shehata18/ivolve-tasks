# Lab 18: Persistent Storage Setup for Application Logging

## 🎯 Objective

The goal of this lab is to configure **persistent storage** in Kubernetes using a **Persistent Volume (PV)** and a **Persistent Volume Claim (PVC)** to ensure that application logs remain available even after Pod restarts or recreation.

---

## 🧩 Lab Concept

By default, data stored inside a Pod is **ephemeral** — it’s lost once the Pod is deleted or restarted.
In this lab, we will:

* Create a **Persistent Volume (PV)** using the **hostPath** type.
* Create a **Persistent Volume Claim (PVC)** that requests 1Gi of storage.
* Mount the PVC inside a Pod that continuously writes logs to a file.
* Verify that data remains persistent even after deleting and recreating the Pod.

---

## ⚙️ Steps to Implement

### Step 1: Create a Persistent Volume (PV)

Define a PV with:

* **Storage size:** 1Gi
* **Storage type:** hostPath
* **Path:** `/mnt/app-logs`
* **Access mode:** `ReadWriteMany`
* **Reclaim policy:** `Retain`

📄 `pv.yml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-logs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/app-logs
```

---

### Step 2: Create a Persistent Volume Claim (PVC)

Request 1Gi of storage and ensure it uses the same access mode and storage class.

📄 `pvc.yml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-logs-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: manual
  resources:
    requests:
      storage: 1Gi
```

---

### Step 3: Create a Pod to Write Logs

The Pod will use a simple `busybox` container that appends timestamps to a log file every 5 seconds.

📄 `app-pod.yml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: log-writer-pod
spec:
  containers:
  - name: log-writer
    image: busybox
    command: ["/bin/sh", "-c"]
    args: ["while true; do date >> /data/app.log; sleep 5; done"]
    volumeMounts:
    - name: app-logs-storage
      mountPath: /data
  volumes:
  - name: app-logs-storage
    persistentVolumeClaim:
      claimName: app-logs-pvc
```

---

### Step 4: Apply the Configurations

```bash
kubectl apply -f pv.yml
kubectl apply -f pvc.yml
kubectl apply -f app-pod.yml
```

Check resources:

```bash
kubectl get pv,pvc,pods
```

---

### Step 5: Verify Logs Inside Pod

```bash
kubectl exec -it log-writer-pod -- cat /data/app.log
```

You should see timestamps being appended every few seconds.

---

### Step 6: Verify Persistence on Node

Enter the Minikube node:

```bash
minikube ssh
cat /mnt/app-logs/app.log
```

If `/mnt/app-logs` exists and has logs, persistence is successful.

---

### Step 7: Test Data Persistence

Delete and recreate the Pod:

```bash
kubectl delete pod log-writer-pod
kubectl apply -f app-pod.yml
```

Then check the file again:

```bash
minikube ssh
cat /mnt/app-logs/app.log
```

✅ Logs should still be there, proving **data persistence**.

---

## 🧠 Key Takeaways

* **Persistent Volumes (PV):** Represent physical storage resources in the cluster.
* **Persistent Volume Claims (PVC):** Requests specific storage resources for Pods.
* **hostPath:** Maps a directory from the node filesystem into the Pod.
* **ReclaimPolicy: Retain:** Keeps data even after the PVC is deleted.

---

## 🧾 Summary

This lab demonstrates how Kubernetes provides **durable and consistent data storage** for applications using Persistent Volumes and Claims.
Even if Pods are deleted or restarted, the application logs remain safely stored on the node.

---
