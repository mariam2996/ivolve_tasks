# Lab 4 - Docker Image Optimization and Export/Load

## Objective

Build a Docker image from an existing Dockerfile, optimize it using a multi-stage build, run the application as a non-root user, compare image sizes, and practice exporting and restoring Docker images.

## Source Code

Application repository:

`https://github.com/Ibrahim-Adel15/ImageOptimization-.git`

---

## 1. Build the Original Image

Build the existing Dockerfile and tag the image as `image1`.

```bash
docker build -t image1 .
```

Check the image size:

```bash
docker image ls image1
```

Original image size:

```text
image1: approximately 667 MB
```

---

## 2. Run the Original Container

Run a container named `container1` from `image1`.

```bash
docker run -d \
  --name container1 \
  -p 8081:8080 \
  image1
```

Verify that the container is running:

```bash
docker ps
```

Verify that the application is working:

```bash
curl http://localhost:8081
```

After verification, stop and remove the container:

```bash
docker stop container1
docker rm container1
```

---

## 3. Create an Optimized Multi-Stage Dockerfile

A multi-stage Dockerfile was created to separate the build environment from the final runtime environment.

```dockerfile
# Stage 1: Build
FROM maven:3.9.9-eclipse-temurin-17 AS builder

WORKDIR /app

COPY . .

RUN mvn clean package


# Stage 2: Runtime
FROM eclipse-temurin:17-jre

WORKDIR /app

RUN groupadd --system appgroup && \
    useradd --system --gid appgroup --create-home appuser

COPY --from=builder /app/target/*.jar /app/app.jar

RUN chown -R appuser:appgroup /app

USER appuser

EXPOSE 8080

CMD ["java", "-jar", "/app/app.jar"]
```

The first stage contains Maven and the JDK and is used only to build the application.

The second stage contains only the Java runtime and the generated JAR file.

---

## 4. Build the Optimized Image

Build the optimized image and tag it as `image2`.

```bash
docker build -f Dockerfile.multi -t image2 .
```

Check the image size:

```bash
docker image ls
```

Image sizes:

```text
image1: approximately 667 MB
image2: approximately 498 MB
```

The optimized image is approximately 169 MB smaller than the original image.

---

## 5. Run the Optimized Container

Run a container named `container2`.

```bash
docker run -d \
  --name container2 \
  -p 8082:8080 \
  image2
```

Verify that the application is working:

```bash
curl http://localhost:8082
```

Verify that the application is running as a non-root user:

```bash
docker exec container2 whoami
```

Expected output:

```text
appuser
```

Stop and remove the container:

```bash
docker stop container2
docker rm container2
```

---

## 6. Export the Optimized Image

Save `image2` as a TAR file named `lab-image.tar`.

```bash
docker save -o lab-image.tar image2
```

Verify the TAR file:

```bash
ls -lh lab-image.tar
```

---

## 7. Delete the Optimized Image

Remove `image2` from the local Docker host.

```bash
docker rmi image2
```

Verify that the image was removed:

```bash
docker image ls
```

---

## 8. Restore the Docker Image

Load the image again from the TAR file.

```bash
docker load -i lab-image.tar
```

Expected output:

```text
Loaded image: image2:latest
```

Verify that the restored image exists:

```bash
docker image ls image2
```

---

## Image Size Comparison

| Image | Description | Approximate Size |
|---|---|---:|
| image1 | Original image with build environment | 667 MB |
| image2 | Optimized multi-stage runtime image | 498 MB |

The multi-stage build reduced the final image size by approximately 169 MB.

---

## Result

The original Docker image was successfully built and tested.

A multi-stage Dockerfile was then used to create a smaller optimized image that runs as a non-root user.

The optimized image was successfully exported to a TAR file, deleted from the local Docker host, and restored using `docker load`.

The lab demonstrates Docker image optimization, multi-stage builds, non-root container execution, and Docker image export/import operations.
