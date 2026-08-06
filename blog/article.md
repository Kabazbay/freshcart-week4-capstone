# Containerizing a Real Application with Docker: Building and Securing FreshCart

## Introduction

One common challenge in software development is the phrase, **"It works on my machine."** Applications often behave differently when they are moved from one computer to another because of differences in operating systems, installed software, or dependency versions.

Docker solves this problem by packaging an application together with everything it needs to run. Instead of manually installing dependencies and configuring environments on every machine, developers can run the application inside containers that behave the same way everywhere.

In this project, I containerized a real multi-service application called **FreshCart**. I created optimized Docker images for both services, connected them using Docker Compose, configured a PostgreSQL database with persistent storage, scanned the images for vulnerabilities, and pushed the backend image to Docker Hub. This project helped me understand how containers simplify application deployment while improving consistency, portability, and security.

---

# Project Overview

FreshCart is a simple grocery shopping application consisting of three components:

- A **Storefront** built with Vite and TypeScript.
- A **Checkout API** built with Node.js, Express, and TypeScript.
- A **PostgreSQL database** for storing products and customer orders.

The original repository only contained the application source code. It did not include Dockerfiles, Docker Compose configuration, or container images.

The goal of this project was to containerize the application so that all services could be started with a single command while following Docker best practices.

---

# Solution Architecture

The application consists of three containers running together.

- The **Storefront** serves the user interface through Nginx.
- The **Checkout API** processes requests from the frontend.
- The **PostgreSQL** container stores the application data.

All containers communicate over a shared Docker network, while the PostgreSQL database stores its data inside a Docker named volume to ensure persistence.

![Container Topology Diagram](freshcart/architecture/container-topology.png)

---

# Building Optimized Docker Images

Instead of creating large Docker images containing build tools, compilers, and unnecessary files, I used **multi-stage Docker builds**.

Each Dockerfile has two stages.

## Build Stage

The build stage installs dependencies and compiles the application.

For the Checkout API, this stage compiles the TypeScript files into JavaScript.

For the Storefront, this stage generates static production files using the Vite build process.

## Runtime Stage

The runtime stage starts from a much smaller base image.

Only the files required to run the application are copied into the final image.

This approach removes development dependencies, build tools, and temporary files from production images.

The result is:

- Smaller images
- Faster downloads
- Reduced attack surface
- Better security

![Multi-stage Build Layer Diagram](freshcart/architecture/multi-stage-build-diagram.png)

---

# Improving Build Performance with Docker Layers

Docker builds images one layer at a time.

If layers are ordered correctly, Docker can reuse previously built layers during future builds.

For both services, I copied the package files before copying the application source code.

```Dockerfile
COPY package*.json ./
RUN npm install

COPY . .
```

With this order, Docker only reinstalls dependencies when the package files change.

If I only modify application code, Docker reuses the cached dependency layer and rebuilds only the remaining layers.

This significantly reduces rebuild time during development.

---

# Running Multiple Containers with Docker Compose

Instead of manually starting each container separately, I created a `docker-compose.yml` file.

Running:

```bash
docker compose up --build
```

automatically:

- Creates a Docker network.
- Creates a Docker volume.
- Starts PostgreSQL.
- Loads the database schema and seed data.
- Starts the Checkout API.
- Starts the Storefront.

Docker Compose also allows the services to communicate using their service names instead of IP addresses.

For example, the Checkout API connects to PostgreSQL using:

```
postgres
```

instead of an IP address.

This makes the application much easier to manage.

---

# Persistent Database Storage

Containers are designed to be temporary.

If a PostgreSQL container is deleted, all data inside the container would normally be lost.

To solve this problem, I attached a **Docker named volume** called:

```
postgres_data
```

The database stores its files inside this volume instead of inside the container.

This means the data remains available even if the PostgreSQL container is recreated.

Using a named volume is considered a best practice for databases running in Docker.

---

# Security Improvements

Security was an important part of this project.

## Running Containers as a Non-Root User

By default, containers run as the root user.

Running applications as root increases the impact of a successful attack.

To reduce this risk, both Dockerfiles create and use a dedicated non-root user before starting the application.

This follows the principle of least privilege.

## Minimal Base Images

I selected lightweight base images for both services.

The Checkout API uses:

```
node:20-alpine
```

The Storefront uses:

```
nginx:alpine
```

Smaller images reduce download size, improve startup speed, and reduce the number of installed packages that could contain vulnerabilities.

---

# Vulnerability Scanning

After building the images, I scanned them using **Trivy**.

The initial scan identified vulnerabilities in the container images.

I updated the affected packages and rebuilt the images before performing another scan.

Running vulnerability scans before deployment helps identify known security issues early in the development process.


![Trivy Before Scan](freshcart/images/trivy-before.png)


![Trivy After Scan](freshcart/images/trivy-after.png)

---

# Publishing the Image

After testing the Checkout API image locally, I tagged it with a version number instead of using the `latest` tag.

Example:

```text
kabazbay/freshcart-checkout-api:v1.0.0
```

The image was then pushed to Docker Hub.

Publishing images to a registry allows the same container image to be downloaded and deployed consistently on any machine.

---

# What I Learned

This project helped me understand several important Docker concepts.

I learned the difference between Docker images and containers and why images should be as small as possible.

I learned how multi-stage builds reduce image size by separating the build environment from the runtime environment.

I also learned why Docker layer ordering affects rebuild speed and how Docker caching improves development workflows.

Using Docker Compose showed me how multiple containers can communicate through a shared network while persistent data is stored safely in Docker volumes.

Finally, scanning images with Trivy demonstrated that container security should be considered before deployment rather than after an application reaches production.

---

# Future Improvements

If I continued developing this project, I would:

- Deploy the containers to Kubernetes.
- Build a CI/CD pipeline that automatically builds and pushes images.
- Add automated testing before deployment.
- Add Prometheus and Grafana for monitoring.
- Add Nginx as a reverse proxy with HTTPS.
- Store secrets using a dedicated secrets manager instead of environment variables.

---

# Conclusion

This project transformed FreshCart from a standard source code repository into a fully containerized application that can be built and started using a single command.

Through this project, I gained practical experience creating optimized Docker images, orchestrating multiple containers with Docker Compose, persisting database data with Docker volumes, improving container security, and publishing images to Docker Hub.

The project also reinforced the importance of writing reproducible infrastructure and following container best practices that can scale to larger production environments.