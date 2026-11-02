# Lab 7 - Multi-Network Microservices, Volumes, and Bind Mounts

## Objective

Containerize and connect three microservices using multiple Docker bridge networks, Docker volumes, and a bind mount.

The application consists of:

- Frontend service
- Backend service
- Notification service

The backend acts as the communication layer between the frontend and notification services.

---

## Source Code

Application repository:

`https://github.com/Ibrahim-Adel15/MicroserviceApp.git`

---

## Architecture

```text
Frontend
   |
   | frontend-backend network
   |
Backend
   |
   | backend-notification network
   |
Notification
```

The backend is connected to both networks.

---

## 1. Build Docker Images

Build the backend image:

```bash
docker build -t backend-image ./backend
```

Build the frontend image:

```bash
docker build -t frontend-image ./frontend
```

Build the notification image:

```bash
docker build -t notification-image ./notification
```

Verify the images:

```bash
docker image ls
```

---

## 2. Create Docker Networks

Create the network between frontend and backend:

```bash
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/24 \
  frontend-backend
```

Create the network between backend and notification:

```bash
docker network create \
  --driver bridge \
  --subnet 172.21.0.0/24 \
  backend-notification
```

---

## 3. Create Docker Volumes

Create a volume for backend logs:

```bash
docker volume create backend-logs
```

Create a volume for notification logs:

```bash
docker volume create notification-logs
```

Verify the volumes:

```bash
docker volume ls
```

---

## 4. Create Backend Bind Mount Directory

Create a directory on the Docker host:

```bash
mkdir -p ~/ivolve_tasks/Docker/Lab7/backend_data
```

This directory is mounted inside the backend container to persist application data.

---

## 5. Run Notification Service

Run the notification container on the `backend-notification` network:

```bash
docker run -d \
  --name notification \
  --network backend-notification \
  -v notification-logs:/app/logs \
  notification-image
```

The notification service runs internally on port:

```text
5001
```

---

## 6. Run Backend Service

Run the backend container on the `frontend-backend` network:

```bash
docker run -d \
  --name backend \
  --network frontend-backend \
  -v backend-logs:/app/logs \
  -v ~/ivolve_tasks/Docker/Lab7/backend_data:/app/data \
  -e NOTIFICATION_URL=http://notification:5001 \
  backend-image
```

Connect the backend container to the second network:

```bash
docker network connect backend-notification backend
```

The backend service runs internally on port:

```text
5000
```

The backend can communicate with:

```text
Frontend through frontend-backend
Notification through backend-notification
```

---

## 7. Run Frontend Service

The frontend application runs internally on port `3000`.

Because port `8080` was already used on the Docker host, port `8081` was used instead.

```bash
docker run -d \
  --name frontend \
  --network frontend-backend \
  -p 8081:3000 \
  -e BACKEND_URL=http://backend:5000 \
  frontend-image
```

Access the application using:

```text
http://localhost:8081
```

---

## 8. Verify Containers

Check that all containers are running:

```bash
docker ps
```

Expected containers:

```text
frontend
backend
notification
```

---

## 9. Verify Docker Networks

Verify the frontend-backend network:

```bash
docker network inspect frontend-backend
```

This network should contain:

```text
frontend
backend
```

Verify the backend-notification network:

```bash
docker network inspect backend-notification
```

This network should contain:

```text
backend
notification
```

The backend container is connected to both networks.

---

## 10. Verify Volumes

List the Docker volumes:

```bash
docker volume ls
```

Expected volumes:

```text
backend-logs
notification-logs
```

---

## 11. Verify Bind Mount

Check the backend data directory:

```bash
ls -la ~/ivolve_tasks/Docker/Lab7/backend_data
```

The bind mount maps:

```text
Host:
~/ivolve_tasks/Docker/Lab7/backend_data

Container:
/app/data
```

---

## 12. Verify Application Flow

Access the frontend:

```text
http://localhost:8081
```

The application communication flow is:

```text
User
 ↓
Frontend
 ↓
Backend
 ↓
Notification
```

The services communicate using Docker DNS names:

```text
frontend → http://backend:5000
backend → http://notification:5001
```

No fixed container IP addresses are required.

---

## Docker Storage Types Used

### Docker Volumes

Used for application logs:

```text
backend-logs
notification-logs
```

Docker manages the storage location.

### Bind Mount

Used for backend application data:

```text
~/ivolve_tasks/Docker/Lab7/backend_data
```

The data is stored directly in a directory on the Docker host.

---

## Result

Three microservices were successfully containerized and connected using two isolated Docker bridge networks.

The backend service was connected to both networks and acted as the communication bridge between the frontend and notification services.

Docker volumes were used to persist service logs, while a bind mount was used to persist backend application data.

The frontend application was successfully published on host port `8081` and connected to the backend service.
