# Lab 11: Managing Docker Environment Variables Across Build and Runtime

## 📘 Objective

In this lab, you will learn how to manage environment variables in Docker during both **build** and **runtime** stages.
You’ll containerize a simple **Flask (Python)** application and configure environment variables using three different methods.

---

## 🧩 Steps

### 1️⃣ Clone the Application Code

```bash
git clone https://github.com/Ibrahim-Adel15/Docker-3.git
cd Docker-3
```

---

### 2️⃣ Write the Dockerfile

Create a `Dockerfile` inside the project directory with the following content:

```dockerfile
# Use Python base image
FROM python:3.10-slim

# Set environment variables at build time (production mode)
ENV APP_MODE=production
ENV APP_REGION=canada-west

# Set working directory
WORKDIR /app

# Copy application files
COPY . /app

# Install Flask
RUN pip install flask

# Expose port 8080
EXPOSE 8080

# Run the app
CMD ["python", "app.py"]
```

---

### 3️⃣ Build the Docker Image

```bash
docker build -t env-app .
```

---

### 4️⃣ Run the Container with Environment Variables (via Command)

Set runtime environment variables using the `-e` flag:

```bash
docker run -d -p 8080:8080 \
-e APP_MODE=development \
-e APP_REGION=us-east \
--name env-app-dev env-app
```

✅ *This runs the app in development mode with `us-east` region.*

---

### 5️⃣ Run the Container Using an Environment File

Create a file named `.env` in the project root:

```env
APP_MODE=staging
APP_REGION=us-west
```

Then run the container and pass the environment file:

```bash
docker run -d -p 8081:8080 \
--env-file .env \
--name env-app-staging env-app
```

✅ *This runs the app in staging mode with `us-west` region.*

---

### 6️⃣ Test the Application

Use `curl` or a browser to access the app:

```bash
curl http://localhost:8080
curl http://localhost:8081
```

Each instance should display environment-specific output (based on `app.py`).

---

### 7️⃣ Stop and Remove Containers

```bash
docker stop env-app-dev env-app-staging
docker rm env-app-dev env-app-staging
```

---

## 🧠 Summary

| Method        | Environment Variables       | Description                      |
| ------------- | --------------------------- | -------------------------------- |
| In Dockerfile | `production`, `canada-west` | Default build-time values        |
| In Command    | `development`, `us-east`    | Overridden at runtime using `-e` |
| In File       | `staging`, `us-west`        | Loaded from external `.env` file |

---

## ✅ Expected Outcome

You should successfully:

* Build and run a Python Flask app in Docker.
* Manage environment variables across build and runtime.
* Verify variable values through different methods.

---

**Author:** Mohamed Shehata
**Course:** Docker Labs
**Lab:** #11 — Managing Docker Environment Variables Across Build and Runtime
