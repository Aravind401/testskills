# Day 5 — Docker Deep Dive

> **30-Day DevOps → MLOps Study Plan**  
> **Focus:** Docker architecture, images, containers, Dockerfiles, layers, caching, networking, volumes, Compose, security, multi-stage builds, registries, ACR/ECR, Trivy, troubleshooting, and senior-level interview preparation  
> **Recommended time:** 3.5–4 hours

---

## 1. Objectives

By the end of Day 5 you should be able to:

- Explain Docker architecture.
- Explain images vs containers.
- Write production-quality Dockerfiles.
- Understand build context, layers, and caching.
- Create multi-stage builds.
- Understand Docker networking and volumes.
- Use Docker Compose.
- Run containers as non-root.
- Understand Linux capabilities and privileged containers.
- Use health checks and read-only filesystems where appropriate.
- Optimize and scan images.
- Push images to ACR/ECR.
- Troubleshoot common Docker failures.
- Answer senior-level Docker interview questions.

## 2. Schedule

| Time | Topic |
|---|---|
| 00:00–00:30 | Docker architecture |
| 00:30–01:00 | Images, containers, layers |
| 01:00–01:45 | Dockerfiles and builds |
| 01:45–02:20 | Networking and volumes |
| 02:20–02:50 | Multi-stage builds and optimization |
| 02:50–03:20 | Docker Compose |
| 03:20–03:50 | Security and Trivy |
| 03:50–04:20 | Troubleshooting and interview Q&A |

---

# 3. Docker Architecture

```text
Docker CLI
    ↓
Docker API
    ↓
Docker Engine
    ↓
Container Runtime
    ↓
Linux Kernel
```

Important components:

- Docker CLI
- Docker Engine/daemon
- Images
- Containers
- Networks
- Volumes
- Registries

Check:

```bash
docker version
docker info
```

---

# 4. Image vs Container

### Image

An immutable package/template used to create containers.

```text
Application
Runtime
Libraries
Files
   ↓
Docker Image
```

### Container

A runtime instance of an image.

```bash
docker run python:3.12-slim
```

**Interview answer:**

> An image is an immutable package and template. A container is a runtime instance of that image with runtime configuration and a writable container layer.

---

# 5. Registry

Examples:

- Docker Hub
- Azure Container Registry
- Amazon ECR
- GitHub Container Registry

Flow:

```text
docker build
     ↓
Image
     ↓
docker push
     ↓
Registry
     ↓
docker pull
     ↓
Runtime
```

---

# 6. Essential Commands

```bash
docker version
docker info
docker images
docker ps
docker ps -a

docker pull nginx:latest
docker run -d --name web nginx:latest

docker logs web
docker logs -f web
docker exec -it web /bin/sh
docker inspect web
docker top web

docker stop web
docker rm web
docker rmi <image>
```

---

# 7. Dockerfile

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app/ .

RUN useradd --create-home appuser

USER appuser

EXPOSE 8000

CMD ["python", "app.py"]
```

Important instructions:

```text
FROM
WORKDIR
COPY
ADD
RUN
CMD
ENTRYPOINT
ENV
ARG
EXPOSE
USER
HEALTHCHECK
VOLUME
```

---

# 8. Build Context

Command:

```bash
docker build -t myapp:1.0 .
```

The `.` is the build context.

Docker can access files inside that context.

Avoid sending unnecessary files.

Use:

```text
.dockerignore
```

Example:

```dockerignore
.git
.gitignore
.env
node_modules
__pycache__
*.log
.venv
dist
build
```

Benefits:

- Faster builds
- Smaller context
- Less accidental data exposure

---

# 9. FROM

```dockerfile
FROM python:3.12-slim
```

Choose a small, trusted base appropriate for the application.

Avoid unnecessarily large images.

---

# 10. WORKDIR

```dockerfile
WORKDIR /app
```

Sets the working directory for subsequent instructions.

Prefer this over:

```dockerfile
RUN cd /app
```

---

# 11. COPY

```dockerfile
COPY app.py /app/
```

Copies files from the build context into the image.

---

# 12. RUN

```dockerfile
RUN apt-get update &&     apt-get install -y curl &&     rm -rf /var/lib/apt/lists/*
```

`RUN` executes during image construction and can create an image layer.

---

# 13. CMD vs ENTRYPOINT

### CMD

Default command or arguments:

```dockerfile
CMD ["python", "app.py"]
```

### ENTRYPOINT

Primary executable:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

This effectively runs:

```text
python app.py
```

**Interview answer:**

> ENTRYPOINT defines the primary executable, while CMD supplies default command/arguments and is generally easier to override.

---

# 14. ENV vs ARG

### ENV

Runtime/container environment configuration:

```dockerfile
ENV APP_ENV=production
```

### ARG

Primarily build-time:

```dockerfile
ARG APP_VERSION=1.0
```

Build:

```bash
docker build   --build-arg APP_VERSION=2.0   -t myapp:2.0 .
```

Neither should be treated as a secure secret store.

---

# 15. EXPOSE

```dockerfile
EXPOSE 8000
```

Documents the intended container port.

It does **not** publish the port to the host.

Publish it with:

```bash
docker run -p 8000:8000 myapp:1.0
```

---

# 16. Image Layers

Conceptually:

```text
Application Layer
       ↓
Dependency Layer
       ↓
OS Layer
       ↓
Base Image
```

Docker can reuse unchanged layers.

Good:

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app/ .
```

Poor:

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

The second approach can invalidate dependency caching whenever source files change.

---

# 17. Multi-Stage Builds

Example:

```dockerfile
FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
```

Flow:

```text
Builder
   ↓
Compile
   ↓
Build Output
   ↓
Small Runtime Image
```

Benefits:

- Smaller image
- Fewer packages
- Reduced attack surface
- Faster deployment

---

# 18. Production Python Dockerfile

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1     PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app/ .

RUN useradd --create-home appuser

USER appuser

EXPOSE 8000

CMD ["python", "app.py"]
```

Improve further with:

- Pinned dependencies
- `.dockerignore`
- Health check
- Minimal packages
- No secrets
- Read-only filesystem where practical

---

# 19. Docker Networking

Common network types:

```text
bridge
host
none
overlay
```

Create a user-defined network:

```bash
docker network create app-network
```

Run:

```bash
docker run -d   --name backend   --network app-network   mybackend:1.0
```

Containers on the same user-defined network can communicate using service/container DNS names.

Do not hard-code container IP addresses.

---

# 20. Container-to-Container Communication

Example:

```text
frontend
   |
   ↓
backend:8080
   |
   ↓
database:5432
```

Inside a container:

```text
localhost
```

means the current container, not another container.

Use the service/container name instead.

---

# 21. Port Mapping

```bash
docker run -p 8080:80 nginx
```

Means:

```text
Host:8080
    ↓
Container:80
```

---

# 22. Volumes

Containers are ephemeral by default.

Create persistent storage:

```bash
docker volume create dbdata
```

Use it:

```bash
docker run   -v dbdata:/var/lib/mysql   mysql:8
```

Flow:

```text
Container
   ↓
Volume
   ↓
Persistent Data
```

---

# 23. Bind Mounts

```bash
docker run   -v $(pwd)/config:/app/config   myapp:1.0
```

Concept:

```text
Host Directory
      ↓
Container Directory
```

Useful especially for development and specific host integration.

---

# 24. Volume vs Bind Mount

| Volume | Bind Mount |
|---|---|
| Managed by Docker | Managed by user |
| Good for persistent app data | Good for local development |
| Less host-path coupling | Explicit host path |
| Docker controls storage location | Host filesystem controls location |

---

# 25. Docker Compose

Example:

```yaml
services:

  frontend:
    image: myfrontend:1.0
    ports:
      - "8080:80"

  backend:
    image: mybackend:1.0
    ports:
      - "5000:5000"

  database:
    image: postgres:16
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: local-password
```

Do not commit real production credentials.

Commands:

```bash
docker compose up -d
docker compose down
docker compose logs -f
```

---

# 26. Health Checks

Example:

```dockerfile
HEALTHCHECK   --interval=30s   --timeout=5s   --start-period=10s   --retries=3   CMD curl -f http://localhost:8080/health || exit 1
```

A good health check should test meaningful application health.

---

# 27. Docker Security

Important controls:

```text
Non-root user
       ↓
Drop unnecessary capabilities
       ↓
No privileged container
       ↓
Read-only filesystem where practical
       ↓
Minimal trusted image
       ↓
Image scanning
       ↓
Pinned dependencies
       ↓
No secrets in image
```

---

# 28. Non-Root Containers

Example:

```dockerfile
RUN useradd --create-home appuser
USER appuser
```

Check:

```bash
docker run --rm myapp:1.0 id
```

Running as non-root reduces the impact of some container compromises.

---

# 29. Linux Capabilities

Capabilities split some traditional root privileges into smaller permissions.

Concept:

```bash
docker run   --cap-drop=ALL   myapp:1.0
```

Only add capabilities that are genuinely required.

---

# 30. Privileged Containers

Avoid:

```bash
docker run --privileged ...
```

unless there is a strong technical requirement.

Privileged containers significantly weaken isolation and can expose additional host-level capabilities.

---

# 31. Read-Only Filesystem

Where practical, make the root filesystem read-only.

In Kubernetes:

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

If the application needs temporary writes, provide a dedicated writable volume.

---

# 32. Secrets

Never bake secrets into:

```dockerfile
ENV DB_PASSWORD=secret123
```

Use:

- Azure Key Vault
- AWS Secrets Manager
- Kubernetes Secrets
- Docker/Compose secret mechanisms where appropriate
- Other platform-native secret stores

Treat build arguments as non-secret unless your build system explicitly provides a secure secret-mount mechanism.

---

# 33. Trivy

Scan an image:

```bash
trivy image myapp:1.0
```

Focus on high/critical findings:

```bash
trivy image   --severity HIGH,CRITICAL   myapp:1.0
```

CI flow:

```text
Docker Build
     ↓
Trivy
     ↓
Policy
 ├── Fail
 └── Continue
```

Define the vulnerability policy explicitly.

---

# 34. Image Optimization

Best practices:

1. Use a minimal trusted base.
2. Use multi-stage builds.
3. Use `.dockerignore`.
4. Optimize layer ordering.
5. Remove unnecessary packages and caches.
6. Avoid debugging tools in production.
7. Pin important dependencies.
8. Run as non-root.
9. Scan images.
10. Keep images focused on one application/service.

---

# 35. Bad Dockerfile

```dockerfile
FROM ubuntu:latest

RUN apt update

RUN apt install -y python3 python3-pip curl git vim wget

COPY . .

RUN pip install -r requirements.txt

ENV PASSWORD=secret123

CMD ["python3", "app.py"]
```

Problems:

- Large base
- Mutable `latest`
- Too many packages
- Poor cache strategy
- Secret in image
- Entire context copied
- Root user
- No health check
- No scanning

---

# 36. Better Dockerfile

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1     PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app/ .

RUN useradd --create-home appuser

USER appuser

EXPOSE 8000

CMD ["python", "app.py"]
```

---

# 37. Image Tagging

Avoid relying only on:

```text
myapp:latest
```

Prefer:

```text
myapp:1.5.2
myapp:git-a83f21c
myapp:build-1054
```

A useful model:

```text
Git Commit
   ↓
Build
   ↓
Image Tag
   ↓
Registry
   ↓
Deployment
```

Every production image should be traceable to source and build information.

---

# 38. ACR / ECR

Azure:

```text
Azure DevOps
     ↓
ACR
     ↓
AKS / App Service / Container Apps
```

AWS:

```text
CI/CD
  ↓
ECR
  ↓
ECS / EKS
```

Core Docker operations remain:

```bash
docker build
docker tag
docker login
docker push
docker pull
```

Registry authentication and cloud integration differ.

---

# 39. Docker in CI/CD

```text
Git
 ↓
Build
 ↓
Unit Test
 ↓
Static Analysis
 ↓
Docker Build
 ↓
Trivy
 ↓
Registry
 ↓
Deploy
 ↓
Health Check
 ↓
Monitor
```

Example:

```yaml
steps:

- script: |
    docker build -t myapp:$(Build.BuildId) .
  displayName: "Docker Build"

- script: |
    trivy image --severity HIGH,CRITICAL myapp:$(Build.BuildId)
  displayName: "Container Scan"
```

---

# 40. Complete Hands-On Lab

## Objective

Build a Python image, run it, inspect it, test networking and storage, run as non-root, optimize it, and scan it.

### Step 1 — Project

```text
docker-day5/
├── app.py
├── requirements.txt
├── Dockerfile
└── .dockerignore
```

`app.py`:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer

class Handler(BaseHTTPRequestHandler):

    def do_GET(self):
        if self.path == "/health":
            self.send_response(200)
            self.end_headers()
            self.wfile.write(b"healthy")
            return

        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"Hello Docker Day 5")

server = HTTPServer(("0.0.0.0", 8000), Handler)
server.serve_forever()
```

`requirements.txt` can be empty for this exercise.

### Step 2 — Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

RUN useradd --create-home appuser

USER appuser

EXPOSE 8000

CMD ["python", "app.py"]
```

### Step 3 — Build

```bash
docker build -t docker-day5:1.0 .
docker images
```

### Step 4 — Run

```bash
docker run -d   --name docker-day5   -p 8000:8000   docker-day5:1.0
```

Test:

```bash
curl http://localhost:8000
curl http://localhost:8000/health
```

### Step 5 — Inspect

```bash
docker ps
docker logs docker-day5
docker inspect docker-day5
docker top docker-day5
docker exec docker-day5 id
```

### Step 6 — Networking

```bash
docker network create day5-network
docker network inspect day5-network
```

### Step 7 — Volume

```bash
docker volume create day5-data

docker run --rm   -v day5-data:/data   alpine   sh -c 'echo "persistent data" > /data/test.txt'

docker run --rm   -v day5-data:/data   alpine   cat /data/test.txt
```

### Step 8 — `.dockerignore`

```dockerignore
.git
.gitignore
.env
*.log
__pycache__
.venv
tests
```

Build again:

```bash
docker build -t docker-day5:1.1 .
```

### Step 9 — Scan

```bash
trivy image docker-day5:1.1
```

Then:

```bash
trivy image   --severity HIGH,CRITICAL   docker-day5:1.1
```

Review findings against your organization's policy.

### Step 10 — Cleanup

```bash
docker stop docker-day5
docker rm docker-day5
```

---

# 41. Troubleshooting

## Container Exits Immediately

Check:

```bash
docker ps -a
docker logs <container>
```

Possible causes:

- Application crash
- Incorrect CMD
- Missing dependency
- Wrong path
- Invalid configuration

## Port Already in Use

```bash
docker ps
```

Use another host port:

```bash
docker run -p 8001:8000 myapp:1.0
```

## Build Is Slow

Check:

- Build context
- `.dockerignore`
- Layer ordering
- Dependency installation
- Cache usage
- Base image size

## Container Cannot Reach Another Container

Check:

```text
Same network?
Correct service/container name?
Correct container port?
Application listening on 0.0.0.0?
Network policy/firewall?
```

Remember:

```text
localhost
↓
Current container
```

---

# 42. Docker vs VM

| Container | Virtual Machine |
|---|---|
| Shares host kernel | Has guest OS/kernel |
| Usually starts quickly | Usually slower |
| Lightweight | More resource intensive |
| Process-level isolation | Full OS virtualization |
| Smaller footprint | Larger footprint |

Containers and VMs solve related but different problems.

---

# 43. Docker vs Kubernetes

Docker:

```text
Build
Run
Package
Containerize
```

Kubernetes:

```text
Schedule
Scale
Heal
Network
Roll out
Manage containers
```

Typical flow:

```text
Dockerfile
 ↓
Docker Image
 ↓
Registry
 ↓
Kubernetes
 ↓
Deployment
```

---

# 44. Interview Q&A

## Q1. What is Docker?

> Docker is a container platform used to package applications and dependencies into images and run them as isolated containers.

## Q2. Image vs container?

> An image is an immutable package/template. A container is a runtime instance with runtime configuration and a writable container layer.

## Q3. What is a Docker layer?

> A layer is an immutable filesystem change created during image construction. Unchanged layers can be reused through caching.

## Q4. How do you optimize Docker builds?

> Use small trusted base images, `.dockerignore`, good layer ordering, dependency caching, multi-stage builds, and remove unnecessary packages.

## Q5. What is a multi-stage build?

> It uses multiple build stages so build dependencies can be excluded from the final runtime image.

## Q6. CMD vs ENTRYPOINT?

> ENTRYPOINT defines the primary executable; CMD supplies default command/arguments.

## Q7. COPY vs ADD?

> COPY is simpler and explicit. ADD has additional behavior such as archive extraction. Prefer COPY unless ADD's extra behavior is intentionally required.

## Q8. ARG vs ENV?

> ARG is primarily build-time; ENV becomes part of image/container environment configuration. Neither should be treated as a secret store.

## Q9. Why not use localhost between containers?

> `localhost` inside a container refers to that same container. Containers should use service/container DNS names over a shared network.

## Q10. What is a volume?

> Docker-managed persistent storage that survives container replacement.

## Q11. Volume vs bind mount?

> Volumes are Docker-managed and are generally better for persistent application data. Bind mounts map explicit host paths and are common in development.

## Q12. Why run containers as non-root?

> It reduces the privileges available to the application and limits the impact of some container compromises.

## Q13. What are Linux capabilities?

> Capabilities divide traditional root privileges into smaller permissions so unnecessary privileges can be removed.

## Q14. What is a privileged container?

> A container with broad additional host-level capabilities. It weakens isolation and should be avoided unless justified.

## Q15. How do you secure Docker images?

> Use minimal trusted bases, non-root users, pinned dependencies, multi-stage builds, `.dockerignore`, vulnerability scanning, no secrets in images, and controlled runtime privileges.

## Q16. Why is `latest` dangerous?

> It is mutable, making deployments less reproducible and harder to trace.

## Q17. How do you troubleshoot a container that exits?

> Check `docker ps -a`, logs, inspect configuration, command, dependencies, environment, file paths, and startup behavior.

## Q18. How do you reduce image size?

> Minimal base, multi-stage builds, `.dockerignore`, fewer packages, cache cleanup, and copying only required files.

## Q19. How does Docker fit into CI/CD?

> CI builds/tests the application, creates an image, scans it, pushes a versioned image to a registry, and CD deploys that exact image.

## Q20. How should production images be tagged?

> Use immutable identifiers such as semantic versions, Git SHAs, or CI build IDs and retain enough metadata to trace the image to source and build.

## Q21. What is Docker Compose?

> Compose defines and runs multi-container applications using declarative YAML.

## Q22. Docker vs Kubernetes?

> Docker focuses on packaging and running containers. Kubernetes orchestrates containers across a cluster with scheduling, scaling, networking, rollout, and self-healing.

## Q23. How do you scan images?

> Use a scanner such as Trivy, define severity/policy thresholds, and integrate scanning into CI before promotion.

## Q24. How do you handle secrets?

> Never bake secrets into Dockerfiles or images. Use an external secret manager or platform-native runtime secret mechanism.

## Q25. How would you troubleshoot a production container?

> Check deployment status, container state, logs, health checks, configuration, resource usage, networking, image/version, dependencies, and monitoring before making changes.

---

# 45. Senior Scenarios

## Scenario 1 — Image Is 2 GB

Investigate:

```text
Base Image
 ↓
Packages
 ↓
Build Dependencies
 ↓
Source
 ↓
Cache
```

Solutions:

- Slim trusted base
- `.dockerignore`
- Multi-stage build
- Remove build dependencies
- Clean caches
- Copy only required files

## Scenario 2 — Container Runs as Root

Use:

```dockerfile
RUN useradd --create-home appuser
USER appuser
```

Verify:

```bash
docker exec <container> id
```

## Scenario 3 — Secret in Image

If a credential was committed/baked into an image:

1. Treat it as compromised.
2. Rotate/revoke it.
3. Remove it from source/build.
4. Review image history and registry exposure.
5. Rebuild.
6. Use runtime secret injection.
7. Add secret scanning.

## Scenario 4 — Database Connection Fails

Check:

```text
Database
 ↓
Network
 ↓
DNS/service name
 ↓
Port
 ↓
Application bind address
 ↓
Credentials
 ↓
Firewall
```

With Compose, use:

```text
database:5432
```

rather than:

```text
localhost:5432
```

## Scenario 5 — Build Is Slow

Poor:

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

Better:

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app/ .
```

---

# 46. Day 5 Checklist

- [ ] I understand Docker architecture.
- [ ] I understand image vs container.
- [ ] I can run and inspect containers.
- [ ] I can read container logs.
- [ ] I understand Dockerfile instructions.
- [ ] I understand build context.
- [ ] I can create `.dockerignore`.
- [ ] I understand layers and caching.
- [ ] I can write a multi-stage Dockerfile.
- [ ] I understand networking.
- [ ] I understand volumes and bind mounts.
- [ ] I can use Docker Compose.
- [ ] I understand non-root containers.
- [ ] I understand Linux capabilities.
- [ ] I understand privileged containers.
- [ ] I understand read-only filesystems.
- [ ] I can scan an image with Trivy.
- [ ] I understand ACR/ECR integration.
- [ ] I can troubleshoot startup failures.
- [ ] I can troubleshoot networking.
- [ ] I completed the lab.
- [ ] I answered at least 20 interview questions aloud.

---

# 47. Homework

- [ ] Build a Python Docker image.
- [ ] Run it as non-root.
- [ ] Add a health endpoint.
- [ ] Add `.dockerignore`.
- [ ] Practice Docker networking.
- [ ] Practice a named volume.
- [ ] Create a Docker Compose application.
- [ ] Build a multi-stage image.
- [ ] Run Trivy.
- [ ] Tag an image with version + Git/build identifier.
- [ ] Push it to ACR or ECR if available.
- [ ] Explain Docker security aloud.
- [ ] Record yourself answering 5 Docker interview questions.

---

# 48. Final Day 5 Challenge

Explain this without looking at your notes:

```text
Developer
    ↓
Git
    ↓
Dockerfile
    ↓
docker build
    ↓
Docker Image
    ↓
Trivy
    ↓
Registry
    ↓
Kubernetes / ECS / Container Apps
    ↓
Container
    ↓
Health Check
    ↓
Monitoring
```

Answer:

1. What is an image?
2. What is a container?
3. What is a layer?
4. How does caching work?
5. Why use multi-stage builds?
6. Why use non-root?
7. What are Linux capabilities?
8. Why should secrets not be baked into images?
9. How do containers communicate?
10. Volume vs bind mount?
11. How would you reduce a 2 GB image?
12. How would you troubleshoot an exiting container?
13. How would you troubleshoot networking?
14. How would you scan an image?
15. How would you trace an image back to a Git commit?

---

# 49. Success Criteria

You are ready for **Day 6 — Kubernetes Fundamentals** when you can confidently explain:

```text
Dockerfile
   ↓
docker build
   ↓
Image
   ↓
Registry
   ↓
Container Runtime
   ↓
Container
   ↓
Network
   ↓
Volume
   ↓
Health Check
```

And you can write a reasonably secure Dockerfile without copying one from memory.

---

# Next Day

## Day 6 — Kubernetes Fundamentals

Topics:

- Kubernetes architecture
- Control plane
- Worker nodes
- API Server
- Scheduler
- Controller Manager
- etcd
- Kubelet
- Container runtime
- Pods
- ReplicaSets
- Deployments
- Namespaces
- Labels and selectors
- Services
- ClusterIP
- NodePort
- LoadBalancer
- ConfigMaps
- Secrets
- Resource requests and limits
- Probes
- YAML manifests
- kubectl commands
- Minikube practical lab
- Kubernetes troubleshooting
- 25+ Kubernetes interview questions
