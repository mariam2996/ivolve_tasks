# Lab 8 - Containerized Node.js and MySQL Stack Using Docker Compose

## Objective

Containerize a Node.js application and MySQL database using Docker Compose, configure networking and persistent storage, verify application-to-database connectivity, and push the application image to Docker Hub.

## Source Code

Application repository:

`https://github.com/Ibrahim-Adel15/iVolveApp.git`

---

## Architecture

```text
Browser
   |
   | Port 3000
   v
Node.js Application
   |
   | ivolve-net
   v
MySQL Database
```

Docker Compose manages both services, their network, environment variables, and persistent volumes.

---

## 1. Application Dockerfile

The Node.js application is built using the provided Dockerfile.

```bash
docker build -t lab8-app .
```

Docker Compose can also build the application automatically using:

```bash
docker compose up -d --build
```

---

## 2. Docker Compose Configuration

The `docker-compose.yml` file contains two services:

- `app`
- `db`

```yaml
services:

  app:
    build: .
    container_name: ivolve-app
    ports:
      - "3000:3000"

    environment:
      DB_HOST: db
      DB_USER: root
      DB_PASSWORD: root123
      DB_NAME: ivolve

    depends_on:
      - db

    networks:
      - ivolve-net

    volumes:
      - app_logs:/app/logs


  db:
    image: mysql:8.0
    container_name: ivolve-db

    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: ivolve

    networks:
      - ivolve-net

    volumes:
      - db_data:/var/lib/mysql


networks:
  ivolve-net:
    driver: bridge


volumes:
  app_logs:
  db_data:
```

---

## 3. Environment Variables

The application uses the following database environment variables:

```text
DB_HOST=db
DB_USER=root
DB_PASSWORD=root123
DB_NAME=ivolve
```

`DB_HOST` is set to `db` because Docker Compose provides internal DNS resolution using the service name.

Therefore, the Node.js application can reach the MySQL service using:

```text
db
```

instead of using a fixed IP address.

---

## 4. Docker Network

A custom Docker network named:

```text
ivolve-net
```

is used for communication between the application and database containers.

Verify the network using:

```bash
docker network inspect lab8_ivolve-net
```

Both containers should appear on the same network:

```text
ivolve-app
ivolve-db
```

---

## 5. Persistent Volumes

Two Docker volumes are used.

### Application Logs

```text
app_logs
```

Mounted to:

```text
/app/logs
```

### MySQL Data

```text
db_data
```

Mounted to:

```text
/var/lib/mysql
```

The MySQL volume ensures that database data remains available even if the database container is removed.

Verify volumes using:

```bash
docker volume ls
```

---

## 6. Start the Application Stack

Start and build all services using:

```bash
docker compose up -d --build
```

Verify the containers:

```bash
docker compose ps
```

Expected services:

```text
ivolve-app
ivolve-db
```

---

## 7. Verify Application Logs

Check application logs:

```bash
docker compose logs app
```

The application should successfully connect to MySQL and start on port `3000`.

---

## 8. Verify Application

Access the application using:

```text
http://localhost:3000
```

Verify the health endpoint:

```bash
curl http://localhost:3000/health
```

Verify the readiness endpoint:

```bash
curl http://localhost:3000/ready
```

---

## 9. Docker Hub Image

The Docker Compose build created the application image:

```text
lab8-app:latest
```

The image was tagged using the Docker Hub username:

```bash
docker tag lab8-app:latest mariamas32/ivolve-app:v1
```

Login to Docker Hub:

```bash
docker login
```

Push the application image:

```bash
docker push mariamas32/ivolve-app:v1
```

The application image is now available from Docker Hub as:

```text
mariamas32/ivolve-app:v1
```

It can be downloaded on another Docker host using:

```bash
docker pull mariamas32/ivolve-app:v1
```

---

## 10. Stop the Stack

The services can be stopped using:

```bash
docker compose down
```

The named volumes remain available unless explicitly removed.

---

## Result

The Node.js application and MySQL database were successfully deployed using Docker Compose.

Docker Compose was used to manage:

- Application and database containers
- Environment variables
- Service networking
- Port mapping
- Persistent application logs
- Persistent MySQL data

The application successfully connected to MySQL, was exposed on port `3000`, and the final application image was tagged and pushed to Docker Hub.
