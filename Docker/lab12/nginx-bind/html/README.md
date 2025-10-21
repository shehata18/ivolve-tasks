# 🧱 Lab 12: Docker Volume and Bind Mount with Nginx

## 🎯 Objective

Learn how to use **Docker Volumes** and **Bind Mounts** to persist data and serve custom files using **Nginx**.

---

## 🧩 Steps

### 1. Create a Docker Volume for Nginx Logs

```bash
docker volume create nginx_logs
```

Verify the volume:

```bash
docker volume ls
```

Inspect its details:

```bash
docker volume inspect nginx_logs
```

---

### 2. Create a Directory for Bind Mount

```bash
mkdir -p nginx-bind/html
```

---

### 3. Create a Custom HTML File

```bash
echo "Hello from Bind Mount" > nginx-bind/html/index.html
```

---

### 4. Run the Nginx Container

```bash
docker run -d \
  --name nginx-bind-volume \
  -p 8080:80 \
  -v nginx_logs:/var/log/nginx \
  -v $(pwd)/nginx-bind/html:/usr/share/nginx/html \
  nginx
```

**Explanation:**

* `nginx_logs` → Docker-managed volume (stores logs)
* `$(pwd)/nginx-bind/html` → Bind mount (serves HTML from host)
* `-p 8080:80` → Maps host port 8080 to container port 80

---

### 5. Test the Nginx Page

```bash
curl http://localhost:8080
```

**Expected Output:**

```
Hello from Bind Mount
```

---

### 6. Update the HTML File and Re-Test

Edit the HTML file:

```bash
echo "Updated Hello from Bind Mount!" > nginx-bind/html/index.html
```

Test again:

```bash
curl http://localhost:8080
```

**Expected Output:**

```
Updated Hello from Bind Mount!
```

---

### 7. Verify Logs in the Volume

Find the volume path:

```bash
docker volume inspect nginx_logs
```

Check the log file:

```bash
sudo cat /var/lib/docker/volumes/nginx_logs/_data/access.log
```

---

### 8. Cleanup

Stop and remove the container:

```bash
docker stop nginx-bind-volume
docker rm nginx-bind-volume
```

Remove the volume:

```bash
docker volume rm nginx_logs
```

---

## 📘 Summary

In this lab, you:

* Created and used a **Docker volume** to persist Nginx logs
* Used a **bind mount** to serve custom HTML content from your host machine
* Verified that changes on the host were reflected inside the container
* Ensured data persistence using Docker-managed storage

---

✅ **Result:** You now understand how to manage persistent data and live content using Docker Volumes and Bind Mounts.
