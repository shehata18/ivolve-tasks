# Lab 9: Run Java Spring Boot App in a Container (Optimized Build)

## Objective
Learn how to containerize a Java Spring Boot application using a lightweight Java runtime image by pre-building the JAR file locally. This approach significantly reduces the Docker image size compared to building within the container.

## Prerequisites
- Docker installed on your system
- Git installed
- Maven installed locally (or use system Maven)
- Java 17 installed
- Basic understanding of Docker and Java

## Lab Overview

This lab demonstrates an optimized approach to containerizing Java applications:
- **Lab 8**: Built the JAR inside the Docker container (Image size: ~567MB)
- **Lab 9**: Build JAR locally, only copy the JAR to container (Image size: ~420MB)

The multi-stage build approach in Lab 9 results in smaller, more efficient images.

## Lab Steps

### Step 1: Clone the Application Code
```bash
git clone https://github.com/Ibrahim-Adel15/Docker-1.git
cd Docker-1
```

### Step 2: Build the JAR File Locally
Before building the Docker image, you need to compile and package the application using Maven:

```bash
mvn clean package
```

This command will:
- Clean any previous build artifacts
- Compile the Java source code
- Run tests (if any)
- Package the application into a JAR file located at `target/demo-0.0.1-SNAPSHOT.jar`

**Expected Output:**
```
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

**Verify the JAR file was created:**
```bash
ls -lh target/demo-0.0.1-SNAPSHOT.jar
```

### Step 3: Create the Dockerfile
Create a `Dockerfile` in the project root with the following content:

```dockerfile
FROM eclipse-temurin:17-jdk-jammy
WORKDIR /app

COPY target/demo-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080

CMD ["java", "-jar", "app.jar"]
```

**Dockerfile Explanation:**
- `FROM eclipse-temurin:17-jdk-jammy` - Use lightweight Java 17 runtime image (Eclipse Temurin on Ubuntu Jammy)
- `WORKDIR /app` - Set working directory inside container
- `COPY target/demo-0.0.1-SNAPSHOT.jar app.jar` - Copy pre-built JAR file into container
- `EXPOSE 8080` - Expose port 8080 for the application
- `CMD ["java", "-jar", "app.jar"]` - Run the JAR file

### Step 4: Build the Docker Image
```bash
docker build -t app2 .
```

**Note the image size:**
The build should complete in just a few seconds since we're only copying files, not building the application.

To check the image size:
```bash
docker images app2
```

**Expected Result:**
- Image size: ~420MB (compared to ~567MB in Lab 8)
- Build time: ~5-10 seconds (much faster than Lab 8's ~54 seconds)

### Step 5: Run the Container
```bash
docker run -d -p 8080:8080 --name spring-app2 app2
```

**Command Breakdown:**
- `-d` - Run container in detached mode (background)
- `-p 8080:8080` - Map host port 8080 to container port 8080
- `--name spring-app2` - Name the container "spring-app2"
- `app2` - Use the app2 image

### Step 6: Test the Application
Wait a few seconds for the application to start, then test it:

```bash
# Wait for the application to start
sleep 5

# Test the application
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

### Step 7: View Container Logs (Optional)
To verify the application started successfully:

```bash
docker logs spring-app2
```

You should see Spring Boot startup logs.

### Step 8: Stop and Delete the Container
```bash
docker stop spring-app2
docker rm spring-app2
```

## Comparison: Lab 8 vs Lab 9

| Metric | Lab 8 (Build in Container) | Lab 9 (Pre-built JAR) |
|--------|----------------------------|----------------------|
| Base Image | maven:3.9-eclipse-temurin-17 | eclipse-temurin:17-jdk-jammy |
| Image Size | ~567MB | ~420MB |
| Build Time | ~54 seconds | ~5-10 seconds |
| Build Approach | Builds JAR inside container | Copies pre-built JAR |
| Pros | Self-contained build | Smaller, faster builds |
| Cons | Larger image, slower builds | Requires local Maven |

**Size Reduction:** ~147MB smaller (26% reduction)

## Additional Commands

### Rebuild the JAR (if you make code changes)
```bash
mvn clean package
docker build -t app2 .
```

### Run with Custom Port Mapping
```bash
docker run -d -p 8081:8080 --name spring-app2 app2
curl http://localhost:8081
```

### View Running Containers
```bash
docker ps
```

### View All Containers
```bash
docker ps -a
```

### Remove the Image
```bash
docker rmi app2
```

### Inspect the Container
```bash
docker inspect spring-app2
```

### Execute Commands Inside Running Container
```bash
docker exec -it spring-app2 bash
```

### Check Container Resource Usage
```bash
docker stats spring-app2
```

## Troubleshooting

### Issue: JAR File Not Found During Build
**Problem:** `COPY failed: file not found in build context`

**Solution:**
- Ensure you built the JAR file first: `mvn clean package`
- Verify the JAR exists: `ls -lh target/demo-0.0.1-SNAPSHOT.jar`
- Check that you're running `docker build` from the project root directory

### Issue: Connection Refused
**Problem:** `curl: (7) Failed to connect to localhost port 8080: Connection refused`

**Solution:**
- Wait 5-10 seconds after starting the container
- Check if container is running: `docker ps`
- Check logs for errors: `docker logs spring-app2`

### Issue: Port Already in Use
**Problem:** Error binding to port 8080

**Solution:**
- Check what's using port 8080: `netstat -tulpn | grep 8080` or `lsof -i :8080`
- Stop other containers using port 8080
- Use a different port: `docker run -d -p 8081:8080 --name spring-app2 app2`

### Issue: Maven Build Fails
**Problem:** Maven package command fails

**Solution:**
- Ensure Maven is installed: `mvn --version`
- Ensure Java 17 is installed: `java --version`
- Check internet connection (Maven downloads dependencies)
- Try: `mvn clean install -U` to force update dependencies

## Multi-Stage Build (Advanced)

For even better optimization, you can use a multi-stage Dockerfile that builds the JAR in the container but doesn't include Maven in the final image:

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jre-jammy
WORKDIR /app
COPY --from=builder /app/target/demo-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

**Benefits:**
- Even smaller image (~300MB) using JRE instead of JDK
- Self-contained build process
- No need for local Maven

## Project Structure
```
Docker-1/
├── Dockerfile
├── pom.xml
├── README.md
├── src/
│   └── main/
│       └── java/
│           └── com/
│               └── example/
│                   └── demo/
│                       └── DemoApplication.java
└── target/
    └── demo-0.0.1-SNAPSHOT.jar  (created after mvn package)
```

## What You Learned
1. How to build a Java application locally using Maven
2. How to create optimized Docker images by separating build and runtime
3. The difference between build-time and runtime dependencies
4. How to reduce Docker image size significantly
5. How to choose appropriate base images (JDK vs JRE)
6. Performance benefits of pre-building applications
7. Container lifecycle management

## Best Practices Demonstrated
✅ Use specific, lightweight base images  
✅ Separate build and runtime environments  
✅ Minimize image layers  
✅ Use appropriate Java runtime (JDK for build, JRE for runtime)  
✅ Expose only necessary ports  
✅ Use meaningful container and image names  

## Next Steps
- Implement multi-stage builds to combine benefits of both approaches
- Use JRE instead of JDK for even smaller runtime images
- Add health checks to the Dockerfile
- Implement .dockerignore to exclude unnecessary files
- Use Docker Compose for multi-container applications
- Push images to Docker Hub or private registry
- Implement CI/CD pipelines for automated builds

## References
- [Eclipse Temurin Docker Images](https://hub.docker.com/_/eclipse-temurin)
- [Docker Multi-Stage Builds](https://docs.docker.com/build/building/multi-stage/)
- [Spring Boot Docker Guide](https://spring.io/guides/gs/spring-boot-docker/)
- [Maven Docker Guide](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)

## Quick Command Reference

```bash
# Build JAR
mvn clean package

# Build Docker image
docker build -t app2 .

# Run container
docker run -d -p 8080:8080 --name spring-app2 app2

# Test application
curl http://localhost:8080

# View logs
docker logs spring-app2

# Stop and remove
docker stop spring-app2 && docker rm spring-app2

# Check image size
docker images app2
```

