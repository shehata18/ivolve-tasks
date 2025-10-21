# 🧪 Lab 10: Multi-Stage Build for Java Spring Boot App

This lab demonstrates how to build and run a **Java Spring Boot** application using a **multi-stage Docker build**.
The first stage uses Maven to build the JAR, and the second stage runs it on a lightweight Java 17 base image.

---

## 🧰 Prerequisites

* Docker installed and running
* Java 17 and Maven (optional, for verification)
* Internet connection

---

## ⚙️ Steps to Complete

### 1. Clone the Application Code

```bash
git clone https://github.com/Ibrahim-Adel15/Docker-1.git
cd Docker-1
```

---

### 2. Create Dockerfile

Create a file named **Dockerfile** and add the following:

```dockerfile
# ---------- Stage 1: Build the Application ----------
FROM maven:3.9.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

# ---------- Stage 2: Run the Application ----------
FROM eclipse-temurin:17-jdk-jammy
WORKDIR /app
COPY --from=build /app/target/demo-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

### 3. Build the Docker Image

```bash
docker build -t app3 .
```

> Note the build time and final image size after the build finishes.

---

### 4. Run the Container

```bash
docker run -d --name app3-container -p 8080:8080 app3
```

---

### 5. Test the Application

Check if the app is working:

```bash
curl http://localhost:8080
```

Or open in a browser (replace `<VM-IP>` with your VM IP):

```
http://<VM-IP>:8080
```

Expected Output:

```
Hello from Dockerized Spring Boot!
```

---

### 6. Stop and Delete the Container

```bash
docker stop app3-container
docker rm app3-container
```

(Optional) Remove the image:

```bash
docker rmi app3
```

---

## ✅ Summary

| Step  | Command                           | Description      |
| ----- | --------------------------------- | ---------------- |
| Clone | `git clone ...`                   | Get source code  |
| Build | `docker build -t app3 .`          | Create image     |
| Run   | `docker run -d -p 8080:8080 app3` | Start container  |
| Test  | `curl localhost:8080`             | Verify response  |
| Clean | `docker stop && docker rm`        | Remove container |

---

### 💡 Notes

* Multi-stage builds reduce image size by removing build tools (Maven) from the final image.
* Only the compiled JAR and Java runtime remain in the final container.
