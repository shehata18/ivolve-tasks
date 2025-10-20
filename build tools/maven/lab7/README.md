# Lab 7: Building and Packaging Java Applications with Maven

## Overview
This lab demonstrates how to use Apache Maven to build, test, and package a Java application. Maven is a powerful build automation and project management tool used primarily for Java projects.

## Prerequisites
- Java Development Kit (JDK) installed
- Internet connection (for downloading Maven dependencies)
- Git installed

## Lab Objectives
1. Install Apache Maven
2. Clone a Java source code repository
3. Run unit tests using Maven
4. Build and package the application as a JAR file
5. Run and verify the application

---

## Step-by-Step Instructions

### Step 1: Install Maven

First, check if Maven is already installed:

```bash
mvn -v
```

If Maven is not installed, you'll need to install it. On RHEL/CentOS/Rocky Linux:

```bash
sudo yum install maven -y
```

Or on Ubuntu/Debian:

```bash
sudo apt update
sudo apt install maven -y
```

After installation, verify Maven is installed correctly:

```bash
mvn -v
```

**Expected Output:**
```
Apache Maven X.X.X
Maven home: /usr/share/maven
Java version: X.X.X
```

---

### Step 2: Clone the Source Code

Clone the sample Java project repository:

```bash
git clone https://github.com/Ibrahim-Adel15/build2.git
cd build2
```

**Project Structure:**
```
build2/
├── pom.xml                          # Maven Project Object Model file
└── src/
    ├── main/
    │   └── java/
    │       └── com/
    │           └── example/
    │               └── App.java     # Main application
    └── test/
        └── java/
            └── com/
                └── example/
                    └── AppTest.java # Unit tests
```

---

### Step 3: Run Unit Tests

Execute the unit tests using Maven:

```bash
mvn test
```

**Expected Output:**
```
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.AppTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

This command:
- Compiles the source code
- Compiles the test code
- Runs all unit tests in the project

---

### Step 4: Build and Package the Application

Build the application and create a JAR file:

```bash
mvn package
```

**Expected Output:**
```
[INFO] Building jar: /path/to/build2/target/hello-ivolve-1.0-SNAPSHOT.jar
[INFO] BUILD SUCCESS
```

This command:
- Compiles the source code
- Runs unit tests
- Packages the compiled code into a JAR file
- Places the JAR in the `target/` directory

**Verify the JAR file was created:**

```bash
ls target/
```

**Expected Files:**
```
classes/
generated-sources/
generated-test-sources/
hello-ivolve-1.0-SNAPSHOT.jar      # Your packaged application
maven-archiver/
maven-status/
surefire-reports/
test-classes/
```

---

### Step 5: Run the Application

Execute the packaged JAR file:

```bash
java -jar target/hello-ivolve-1.0-SNAPSHOT.jar
```

**Expected Output:**
```
Hello iVolve!
```

---

### Step 6: Verify Application is Working

If you see the "Hello iVolve!" message, your application is working correctly! ✅

---

## Understanding Maven Commands

| Command | Description |
|---------|-------------|
| `mvn clean` | Removes the `target/` directory and all compiled files |
| `mvn compile` | Compiles the source code |
| `mvn test` | Runs unit tests |
| `mvn package` | Creates a JAR/WAR file from the project |
| `mvn install` | Installs the package into the local Maven repository |
| `mvn clean package` | Cleans previous builds and creates a fresh package |

---

## Understanding pom.xml

The `pom.xml` (Project Object Model) file is the core of a Maven project. It contains:

- **Project Information:** Group ID, Artifact ID, Version
- **Dependencies:** External libraries the project needs
- **Build Configuration:** Plugins and build settings
- **Properties:** Java version, encoding, etc.

**Example pom.xml structure:**
```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>hello-ivolve</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    
    <dependencies>
        <!-- Project dependencies -->
    </dependencies>
    
    <build>
        <plugins>
            <!-- Build plugins -->
        </plugins>
    </build>
</project>
```

---

## Maven Build Lifecycle

Maven follows a standard build lifecycle with these main phases:

1. **validate** - Validate the project is correct
2. **compile** - Compile the source code
3. **test** - Test the compiled code using unit tests
4. **package** - Package compiled code into distributable format (JAR/WAR)
5. **verify** - Run integration tests
6. **install** - Install package to local repository
7. **deploy** - Deploy to remote repository

---

## Troubleshooting

### Issue: Maven command not found
**Solution:** Ensure Maven is properly installed and in your PATH

### Issue: Build failures due to missing dependencies
**Solution:** Maven will automatically download dependencies from Maven Central. Ensure you have internet connectivity.

### Issue: Java version mismatch
**Solution:** Check your Java version matches the version specified in `pom.xml`:
```bash
java -version
```

### Issue: Permission denied when running JAR
**Solution:** Ensure the JAR file has execute permissions or run with `java -jar` command

---

## Project Artifact Details

- **Group ID:** `com.example`
- **Artifact ID:** `hello-ivolve`
- **Version:** `1.0-SNAPSHOT`
- **Package Type:** JAR (Java Archive)
- **JAR Location:** `target/hello-ivolve-1.0-SNAPSHOT.jar`

---

## Additional Maven Commands for Practice

```bash
# Clean and rebuild the project
mvn clean package

# Skip tests during build
mvn package -DskipTests

# Run a specific test class
mvn test -Dtest=AppTest

# Display dependency tree
mvn dependency:tree

# Display project information
mvn help:effective-pom
```

---

## Summary

In this lab, you have successfully:
- ✅ Installed Apache Maven
- ✅ Cloned a Java project from Git
- ✅ Executed unit tests using Maven
- ✅ Built and packaged a Java application into a JAR file
- ✅ Ran the packaged application
- ✅ Verified the application works correctly

Maven simplifies Java project management by handling dependencies, build processes, and project structure in a standardized way.

---

## References

- [Apache Maven Official Documentation](https://maven.apache.org/guides/)
- [Maven in 5 Minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)
- [POM Reference](https://maven.apache.org/pom.html)

---

**Lab Completed Successfully! 🎉**

