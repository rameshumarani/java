# Docker Notes

## What is Docker?

Docker is a containerization tool that helps us package an application. The package contains code, dependencies, environment variables, etc.

- A **packaged application** in Docker is called an **Image**
- When we **run** that image, it's called a **Container**

**Image** is a blueprint — it contains what we build and store.
**Container** is a running instance of an image. You can run multiple containers from the same image, just like you'd create multiple instances of a Java object from a class.

> Docker image is like a Java class — it's the definition.
> A container is like an object instance — it's the actual running thing.

When you have just one or two containers, Docker handles it fine. But imagine you're running a hundred containers across multiple servers, and one crashes — you need something to automatically restart it, load balance traffic, manage updates. That's what **Kubernetes** does.

---

## Dockerfile

A Dockerfile is a set of instructions which helps us build the Docker image.

The Dockerfile has instructions for building the image — Java version, copying your jar, setting environment variables for database connectivity, all of that. Docker reads those instructions and creates the image.

---

## Docker Registry

A registry is basically a storage place for Docker images, like a library. The most popular one is **Docker Hub**.

Once you build an image, you can **push** it to a registry so other people or other servers can **pull** it and run it.

- **Public registries** (like Docker Hub) — anyone can access
- **Private registries** — restricted to your team or organization

Push your image up, pull it down wherever you need it.

---

## Docker Compose

Docker Compose helps us run **multiple containers together** and connect them. Separate Dockerfiles build separate Docker images, and Docker Compose ties them all together.

**Example:** Imagine you're building an e-commerce backend with Spring Boot, a MySQL database, and Redis for caching.

- You'd write **three separate Dockerfiles** — one for Spring Boot, one for MySQL, one for Redis
- Each builds its own image
- Then in Docker Compose, you define all three services in **one YAML file**, specify how much memory each needs, which ports they use, and how they connect to each other
- When you run Docker Compose, it starts all three containers and they can talk to each other seamlessly

---

## Docker Networking

Without Docker networking, if you had two containers on different machines, you'd have to manually figure out their IP addresses and hardcode them — messy and brittle.

With Docker networking, containers **automatically discover each other by name**.

**Example:** Your Spring Boot app's connection string is just:

```
jdbc:mysql://mysql:3306/mydb
```

Here `mysql` is the service name defined in Docker Compose. Docker's DNS automatically resolves that to the right container.

### How It Works

1. When your Spring Boot container tries to connect to MySQL, Docker's **internal DNS** checks the request
2. It looks up the container with the **service name** `mysql`
3. Once it finds that service name, it uses **bridge networking** to connect the containers in the same network
4. If the MySQL container restarts and gets a new IP, Docker **updates DNS automatically**
5. Your Spring Boot app doesn't care — it keeps using `mysql` as the hostname, and things keep working

### Network Types

- **Bridge** — connects all containers in the same network (default)
- **Host** — container uses the host machine's network directly instead of its own isolated network
- **Overlay** — used for containers spread across multiple machines

---

## Docker Volumes

When a container stops or crashes, any data inside it — database files, logs, whatever — **disappears**. That's bad for databases especially.

Docker volumes are **persistent storage** that live outside the container. When your MySQL container writes data to a volume, that data survives even if the container gets deleted and recreated.

### Types of Volumes

| Type | Description | Best For |
|------|-------------|----------|
| **Named Volumes** | You give the volume a name and Docker manages the storage location | Production |
| **Bind Mounts** | You link a specific folder on your host machine to a folder inside the container, changes sync both ways | Development |

Named volumes are cleaner for production. Bind mounts are handier during development when you're actively editing files.

---

## Docker Layers

When Docker builds an image from a Dockerfile, **each instruction creates a layer**.

For example:
1. `Install Java` → Layer 1
2. `Copy your jar` → Layer 2
3. `Set environment variables` → Layer 3

Docker stacks these layers on top of each other to create the final image.

The cool part — Docker **caches these layers**. So if you rebuild and only change your jar file, Docker doesn't reinstall Java. It reuses that cached layer and just rebuilds the jar layer.

---

## Multi-Stage Builds

Multi-stage builds let you build your application in one stage, then copy only the final artifact into a smaller stage for running.

**Example:**
- **Stage 1:** Build your Spring Boot jar using Maven (with all the build tools)
- **Stage 2:** Copy just the jar into a minimal Java image

This way your final image is tiny because you're not shipping Maven, git, or other build tools.

### Tips to Keep Images Small

- Use minimal base images like **Alpine** instead of full operating systems
- Remove unnecessary files
- Don't install stuff you don't need

---

## Summary

| Concept | What It Does |
|---------|-------------|
| **Dockerfile** | One Dockerfile builds one image |
| **Image** | Blueprint that runs as one or more containers |
| **Docker Compose** | Orchestrates multiple containers together, defines networking |
| **Networking** | Containers communicate via bridge network using service names |
| **Volumes** | Persist data outside containers so it survives restarts |
| **Layers** | Docker caches unchanged layers for faster builds |
| **Multi-Stage Builds** | Keeps final images small by separating build from runtime |

> **That's the Docker foundation — containers, composition, networking, persistence, and optimization.**
