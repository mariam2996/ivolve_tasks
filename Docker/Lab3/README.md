# Lab 3 - Creating a Docker Image for a Java Application

## Objective
Containerize a Java calculator application using two different Docker approaches and compare the resulting image sizes.

## Source Code
The application source code was taken from:

`CalculatorMavenApp`

---

## Approach 1 - Build Inside Docker

In this approach, Maven and Java are included in the base image and the application is built inside the Docker image.

### Dockerfile

```dockerfile
FROM maven:3.9.9-eclipse-temurin-17

WORKDIR /app

RUN groupadd --system appgroup && \
    useradd --system --gid appgroup --create-home appuser

COPY . .

RUN chown -R appuser:appgroup /app

USER appuser

RUN mvn clean package

CMD ["java", "-jar", "target/calculator.jar"]
```

### Build the Image

```bash
docker build -t app1 .
```

### Run the Container

```bash
docker run --rm -it --name container1 app1
```

### Verify Non-Root User

```bash
docker run --rm app1 whoami
```

Expected output:

```text
appuser
```

---

## Approach 2 - Build Outside Docker

In this approach, the application is built first and only the generated JAR file is copied into a Java runtime image.

### Dockerfile

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

RUN groupadd --system appgroup && \
    useradd --system --gid appgroup --create-home appuser

COPY calculator.jar /app/calculator.jar

RUN chown -R appuser:appgroup /app

USER appuser

CMD ["java", "-jar", "calculator.jar"]
```

### Build the Image

```bash
docker build -f Dockerfile.app2 -t app2 .
```

### Run the Container

```bash
docker run --rm -it --name container2 app2
```

### Verify Non-Root User

```bash
docker run --rm app2 whoami
```

Expected output:

```text
appuser
```

---

## Image Size Comparison

The two images were compared using:

```bash
docker image ls
```

Results:

- `app1`: approximately **791 MB**
- `app2`: approximately **423 MB**

### Explanation

Approach 1 produces a larger image because it contains the Maven build environment, the JDK, build tools, source code, and the application.

Approach 2 produces a smaller image because it contains only the Java runtime and the packaged JAR file.

Therefore, Approach 2 is more lightweight and better suited for runtime use.

---

## Result

Both Docker images were built and executed successfully using a non-root user. The second approach produced a significantly smaller Docker image.

