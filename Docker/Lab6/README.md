# Lab 6 - Docker Environment Variables and Resource Management

## Objective

Containerize a Python application, configure environment variables using multiple methods, limit container CPU and memory resources, and monitor container resource usage.

## Source Code

Application repository:

`https://github.com/Ibrahim-Adel15/PythonApp.git`

---

## 1. Build the Docker Image

Create a Dockerfile using the `python:3.12-slim` base image.

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

Build the image:

```bash
docker build -t ivolve1 .
```

Verify the image:

```bash
docker image ls ivolve1
```

---

## 2. Scenario 1 - Environment Variables from docker run

Run `container1` and pass the environment variables directly from the command line.

```bash
docker run -d \
  --name container1 \
  -p 5000:5000 \
  -e APP_NAME=iVolveApp1 \
  -e APP_ENV=stg \
  --cpus="1" \
  --memory="256m" \
  ivolve1
```

Verify that the container is running:

```bash
docker ps
```

Verify the application:

```bash
curl http://localhost:5000
```

Verify the environment variables:

```bash
docker exec container1 env | grep APP_
```

Expected output:

```text
APP_NAME=iVolveApp1
APP_ENV=stg
```

Monitor CPU and memory usage:

```bash
docker stats container1
```

The container was limited to:

```text
1 CPU
256 MB RAM
```

---

## 3. Scenario 2 - Environment Variables from a File

Create an environment file:

```bash
nano env.txt
```

Add:

```text
APP_NAME=iVolveApp2
APP_ENV=prod
```

Run a second container using the environment file:

```bash
docker run -d \
  --name container2 \
  -p 5001:5000 \
  --env-file env.txt \
  --cpus="1" \
  --memory="256m" \
  ivolve1
```

Verify the application:

```bash
curl http://localhost:5001
```

Verify the environment variables:

```bash
docker exec container2 env | grep APP_
```

Expected output:

```text
APP_NAME=iVolveApp2
APP_ENV=prod
```

Monitor the container:

```bash
docker stats container2
```

---

## 4. Scenario 3 - Environment Variables in the Dockerfile

Update the Dockerfile and add:

```dockerfile
ENV APP_NAME=iVolveApp3
ENV APP_ENV=dev
```

The Dockerfile becomes:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV APP_NAME=iVolveApp3
ENV APP_ENV=dev

EXPOSE 5000

CMD ["python", "app.py"]
```

Build a new image:

```bash
docker build -t ivolve3 .
```

Run a new container:

```bash
docker run -d \
  --name container3 \
  -p 5002:5000 \
  --cpus="1" \
  --memory="256m" \
  ivolve3
```

Verify the application:

```bash
curl http://localhost:5002
```

Verify the environment variables:

```bash
docker exec container3 env | grep APP_
```

Expected output:

```text
APP_NAME=iVolveApp3
APP_ENV=dev
```

---

## Environment Variable Comparison

| Scenario | Method | Example |
|---|---|---|
| Scenario 1 | Command line | `-e APP_NAME=iVolveApp1` |
| Scenario 2 | Environment file | `--env-file env.txt` |
| Scenario 3 | Dockerfile | `ENV APP_NAME=iVolveApp3` |

Scenario 1 is useful for quickly passing configuration at runtime.

Scenario 2 is cleaner when multiple environment variables are required.

Scenario 3 stores default values inside the Docker image itself.

---

## Resource Management

Docker resource limits were configured using:

```bash
--cpus="1"
--memory="256m"
```

These limits prevent the container from consuming unlimited host resources.

Resource usage was monitored using:

```bash
docker stats <container-name>
```

---

## Result

The Python application was successfully containerized and executed using three different environment variable configuration methods.

The containers were limited to 1 CPU and 256 MB of memory, and their resource usage was monitored using Docker Stats.

This lab demonstrates Docker environment variable management, runtime configuration, and container resource control.
