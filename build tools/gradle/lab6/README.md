# Lab 6: Building and Packaging Java Applications with Gradle

This lab demonstrates how to build and package Java applications using Gradle build tool.

## Prerequisites Installation

1. Install required packages:
```bash
sudo dnf install java-17-openjdk-devel wget unzip -y
```

2. Download Gradle:
```bash
wget https://services.gradle.org/distributions/gradle-8.10.2-bin.zip -P /tmp
```

3. Install Gradle:
```bash
sudo unzip -d /opt/gradle /tmp/gradle-8.10.2-bin.zip
```

4. Configure Gradle environment:
```bash
echo 'export PATH=$PATH:/opt/gradle/gradle-8.10.2/bin' | sudo tee /etc/profile.d/gradle.sh
sudo chmod +x /etc/profile.d/gradle.sh
source /etc/profile.d/gradle.sh
```

5. Verify Gradle installation:
```bash
gradle -v
```

## Project Setup and Execution

1. Clone the source code repository:
```bash
git clone https://github.com/Ibrahim-Adel15/build1.git
cd build1
```

2. Run the unit tests:
```bash
gradle test
```
This command will execute all unit tests in the project. The test results can be found in `build/reports/tests/test/index.html`.

3. Build the application:
```bash
gradle build
```
This command will:
- Compile the source code
- Run the tests
- Generate the artifact in `build/libs/ivolve-app.jar`

4. Run the application:
```bash
java -jar build/libs/ivolve-app.jar
```
Expected output: `Hello iVolve Trainee`

## Project Structure

The built project will generate the following key directories:
- `build/libs/`: Contains the compiled JAR file
- `build/classes/`: Contains compiled class files
- `build/reports/`: Contains test and analysis reports
- `build/test-results/`: Contains test execution results

## Verification

The application is working correctly if you see the message "Hello iVolve Trainee" when running the JAR file.