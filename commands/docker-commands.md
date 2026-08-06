# Docker Commands Used

This document contains the main Docker commands used throughout the FreshCart Week 4 Capstone project.

---

# Build and Start the Application

Build the Docker images and start all services.

```bash
docker compose up --build
```

---

# Start Existing Containers

Start the application without rebuilding the images.

```bash
docker compose up
```

---

# Stop Running Containers

Stop all running containers.

```bash
docker compose down
```

---

# Stop Containers and Remove Volumes

Stop containers and delete the database volume.

> **Warning:** This permanently deletes all PostgreSQL data.

```bash
docker compose down -v
```

---

# View Running Containers

Display all currently running containers.

```bash
docker ps
```

---

# View All Containers

Display both running and stopped containers.

```bash
docker ps -a
```

---

# View Docker Compose Logs

Display logs from all services.

```bash
docker compose logs
```

---

# View Logs for a Specific Service

Checkout API

```bash
docker compose logs checkout-api
```

Storefront

```bash
docker compose logs storefront
```

PostgreSQL

```bash
docker compose logs postgres
```

---

# Rebuild Images

Rebuild all Docker images.

```bash
docker compose build
```

---

# Restart the Application

```bash
docker compose restart
```

---

# List Docker Networks

```bash
docker network ls
```

---

# Inspect the Project Network

```bash
docker network inspect freshcart-network
```

This verifies that all three containers are connected to the same Docker network.

---

# List Docker Volumes

```bash
docker volume ls
```

---

# Inspect the PostgreSQL Volume

```bash
docker volume inspect postgres_data
```

This confirms where Docker stores the PostgreSQL database files.

---

# Build the Checkout API Image

From the `checkout-api` directory:

```bash
docker build -t freshcart-checkout-api .
```

---

# Build the Storefront Image

From the `storefront` directory:

```bash
docker build -t freshcart-storefront .
```

---

# List Docker Images

```bash
docker images
```

---

# Tag the Checkout API Image

```bash
docker tag freshcart-checkout-api:latest kabazbay/freshcart-checkout-api:v1.0.0
```

---

# Push the Image to Docker Hub

```bash
docker push kabazbay/freshcart-checkout-api:v1.0.0
```

---

# Pull the Image from Docker Hub

```bash
docker pull kabazbay/freshcart-checkout-api:v1.0.0
```

---

# Scan the Image with Trivy

```bash
trivy image kabazbay/freshcart-checkout-api:v1.0.0
```

Save the output to a file:

```bash
trivy image kabazbay/freshcart-checkout-api:v1.0.0 > scans/trivy-after.txt
```

---

# Verify the API

Health Check

```bash
curl http://localhost:3000/healthz
```

Expected output:

```json
{"status":"ok"}
```

---

# List Products

```bash
curl http://localhost:3000/api/products
```

This confirms the Checkout API can communicate with the PostgreSQL database.

---

# Open the Application

Storefront

```
http://localhost:8080
```

API Health Endpoint

```
http://localhost:3000/healthz
```

Products Endpoint

```
http://localhost:3000/api/products
```

---

# Cleanup

Stop the application.

```bash
docker compose down
```

Remove unused Docker resources.

```bash
docker system prune
```

Remove unused Docker images.

```bash
docker image prune
```

Remove unused Docker volumes.

```bash
docker volume prune
```

> **Warning:** Only run the prune commands if you understand that unused Docker resources will be deleted.