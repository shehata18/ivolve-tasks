# Lab 8: Run Java Spring Boot App in a Container

## Objective
Learn how to containerize a Java Spring Boot application using Docker with a Maven base image.

## Prerequisites
- Docker installed on your system
- Git installed
- Basic understanding of Docker and Java

## Lab Steps

### Step 1: Clone the Application Code
```bash
git clone https://github.com/Ibrahim-Adel15/Docker-1.git
cd Docker-1
```

### Step 2: Create the Dockerfile
Create a `Dockerfile` in the project root with the following content:

```dockerfile
FROM maven:3.9-eclipse-temurin-17
WORKDIR /app
COPY . .
RUN mvn package
EXPOSE 8080
CMD ["java", "-jar", "target/demo-0.0.1-SNAPSHOT.jar"]
```

**Dockerfile Explanation:**
- `FROM maven:3.9-eclipse-temurin-17` - Use Maven base image with Java 17
- `WORKDIR /app` - Set working directory inside container
- `COPY . .` - Copy application source code into the container
- `RUN mvn package` - Build the application using Maven
- `EXPOSE 8080` - Expose port 8080 for the application
- `CMD ["java", "-jar", "target/demo-0.0.1-SNAPSHOT.jar"]` - Run the JAR file

### Step 3: Build the Docker Image
```bash
docker build -t app1 .
```

**Note the build time and image size:**
- Build time: ~54 seconds (may vary based on network speed)
- Image size: ~567MB

To check the image:
```bash
docker images app1
```

### Step 4: Run the Container
```bash
docker run -d -p 8080:8080 --name spring-app app1
```

**Command Breakdown:**
- `-d` - Run container in detached mode (background)
- `-p 8080:8080` - Map host port 8080 to container port 8080
- `--name spring-app` - Name the container "spring-app"
- `app1` - Use the app1 image

### Step 5: Test the Application
Wait a few seconds for the application to start, then test it:

```bash
curl http://localhost:8080
```

**Expected Output:**
```
Hello from Dockerized Spring Boot!
```

You can also test in a web browser by navigating to:
```
http://localhost:8080
```

### Step 6: View Container Logs (Optional)
To see the application logs:
```bash
docker logs spring-app
```

To follow logs in real-time:
```bash
docker logs -f spring-app
```

### Step 7: Stop and Delete the Container
```bash
docker stop spring-app
docker rm spring-app
```

## Additional Commands

### View Running Containers
```bash
docker ps
```

### View All Containers (including stopped)
```bash
docker ps -a
```

### Remove the Image
```bash
docker rmi app1
```

### Inspect the Container
```bash
docker inspect spring-app
```

### Execute Commands Inside Running Container
```bash
docker exec -it spring-app bash
```

## Build Metrics

| Metric | Value |
|--------|-------|
| Build Time | ~54 seconds |
| Image Size | 567MB |
| Base Image | maven:3.9-eclipse-temurin-17 |
| Java Version | 17 |
| Maven Version | 3.9 |

## Troubleshooting

### Issue: Connection Refused
**Problem:** `curl: (7) Failed to connect to localhost port 8080: Connection refused`

**Solution:** 
- Wait 5-10 seconds after starting the container for the Spring Boot app to fully start
- Check if the container is running: `docker ps`
- Check container logs: `docker logs spring-app`

### Issue: Port Already in Use
**Problem:** Error binding to port 8080

**Solution:**
- Check if another process is using port 8080: `netstat -tulpn | grep 8080`
- Use a different port mapping: `docker run -d -p 8081:8080 --name spring-app app1`
- Then access via: `curl http://localhost:8081`

### Issue: Build Fails
**Problem:** Maven build fails during image build

**Solution:**
- Ensure you have a stable internet connection (Maven needs to download dependencies)
- Check if the `pom.xml` file is present
- Try building again: `docker build -t app1 .`

## Project Structure
```
Docker-1/
├── Dockerfile
├── pom.xml
├── README.md
└── src/
    └── main/
        └── java/
            └── com/
                └── example/
                    └── demo/
                        └── DemoApplication.java
```

## What You Learned
1. How to create a Dockerfile for a Java Spring Boot application
2. How to use Maven base images with specific Java versions
3. How to build Docker images and understand build metrics
4. How to run containers with port mapping
5. How to test containerized applications
6. How to manage container lifecycle (start, stop, remove)

## Next Steps
- Optimize the Dockerfile using multi-stage builds to reduce image size
- Add environment variables for configuration
- Use Docker Compose for multi-container applications
- Implement health checks in the Dockerfile
- Push the image to Docker Hub or a private registry

## References
- [Docker Documentation](https://docs.docker.com/)
- [Spring Boot Docker Guide](https://spring.io/guides/gs/spring-boot-docker/)
- [Maven Docker Official Image](https://hub.docker.com/_/maven)

