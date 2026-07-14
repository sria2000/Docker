# Docker Notes

Personal working notes on Docker fundamentals — images, containers, port binding, and building custom images with a Dockerfile.

---

## 1. Setup

- Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Verify install:
  ```bash
  docker --version
  docker info
  ```

---

## 2. Basic CLI Commands

```bash
docker ps              # list running containers
docker ps -a            # list ALL containers (running + stopped)
docker images           # list local images
docker pull nginx       # pull the "latest" tag of an image
docker pull nginx:1.23  # pull a specific version/tag
```

Example output of `docker images`:

```
IMAGE                             ID             DISK USAGE   CONTENT SIZE   EXTRA
docker/welcome-to-docker:latest   c4d56c24da4f       22.2MB         6.03MB    U
nginx:1.23                        f5747a42e3ad        214MB           57MB
nginx:latest                      b5a9a3cfc86b        241MB           66MB
```
`U` = image currently in use by a container.

> **Tip:** Tags matter. `nginx` with no tag defaults to `nginx:latest`, which can silently change over time. Pin a version (e.g. `nginx:1.23`) for reproducible builds.

---

## 3. Running Containers

```bash
docker run nginx:1.23
```

Check running containers:

```bash
docker ps
```

```
CONTAINER ID   IMAGE                             COMMAND                  CREATED         STATUS         PORTS                                     NAMES
0443c6b82eda   nginx:1.23                        "/docker-entrypoint.…"   9 seconds ago   Up 8 seconds   80/tcp                                    epic_ritchie
dfe7b51a915d   docker/welcome-to-docker:latest   "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes   0.0.0.0:8088->80/tcp, [::]:8088->80/tcp   welcome-to-docker
```

### Run in the background (detached mode)

```bash
docker run -d nginx:1.23
```

### View container logs

```bash
docker logs <container_id>
docker logs -f <container_id>   # follow logs live (tail -f style)
```

### Auto-pull on run

You don't need to run `docker pull` separately — if the image isn't found locally, `docker run` pulls it automatically:

```bash
docker run nginx:1.22-alpine
```

```
Unable to find image 'nginx:1.22-alpine' locally
1.22-alpine: Pulling from library/nginx
034c69766aa3: Pull complete
b0a3a88d1edf: Pull complete
```

### Remove a container / image

```bash
docker rm <container_id>          # remove a stopped container
docker rm -f <container_id>       # force remove a running container
docker rmi <image_id>             # remove an image
docker container prune            # remove all stopped containers
```

### Run and auto-remove on exit

Useful for one-off/throwaway containers:

```bash
docker run --rm nginx:1.23
```

---

## 4. Port Binding

Containers run in an isolated Docker network. To reach an app running inside a container from your machine, you must **bind (publish)** the container's port to a host port.

Example: nginx runs on port 80 inside the container, but `localhost:80` fails unless the port is bound.

```bash
docker stop dfe7b51a915d
```

Bind container port 80 to host port 8000:

```bash
docker run -d -p 8000:80 nginx:1.23
```

- `-d` → run detached
- `-p 8000:80` → map **host** port 8000 to **container** port 80 (`-p <host>:<container>`)

```
CONTAINER ID   IMAGE        COMMAND                  CREATED             STATUS             PORTS                                     NAMES
d6dd4f1ced95   nginx:1.23   "/docker-entrypoint.…"   4 seconds ago       Up 3 seconds       0.0.0.0:8000->80/tcp, [::]:8000->80/tcp   silly_chebyshev
fbab1cd7730e   nginx:1.23   "/docker-entrypoint.…"   About an hour ago   Up About an hour   80/tcp                                    boring_golick
```

Now accessible at `localhost:8000`.

> **Tip:** Publish to all interfaces with `-p 8000:80`, or restrict to localhost only with `-p 127.0.0.1:8000:80`. Use `-P` (capital) to auto-publish all `EXPOSE`d ports to random host ports.

---

## 5. Stopping and Starting Containers

`docker run` **always creates a new container** — it doesn't reuse a previous one. To reuse a container, use `docker start`.

### Show all containers (including stopped)

```bash
docker ps -a
```

```
CONTAINER ID   IMAGE                             COMMAND                  CREATED          STATUS                      PORTS                                     NAMES
d6dd4f1ced95   nginx:1.23                        "/docker-entrypoint.…"   18 minutes ago   Up 18 minutes               0.0.0.0:8000->80/tcp, [::]:8000->80/tcp   silly_chebyshev
1db54527b7e6   nginx:1.22-alpine                 "/docker-entrypoint.…"   25 minutes ago   Exited (0) 24 minutes ago                                             trusting_shockley
f38c6346ba0e   nginx:1.22-alpine                 "/docker-entrypoint.…"   25 minutes ago   Exited (0) 25 minutes ago                                             wizardly_bose
fbab1cd7730e   nginx:1.23                        "/docker-entrypoint.…"   2 hours ago      Up 2 hours                  80/tcp                                    boring_golick
0443c6b82eda   nginx:1.23                        "/docker-entrypoint.…"   2 hours ago      Exited (0) 2 hours ago                                                epic_ritchie
dfe7b51a915d   docker/welcome-to-docker:latest   "/docker-entrypoint.…"   2 hours ago      Exited (0) 20 minutes ago                                             welcome-to-docker
```

### Start a stopped container

```bash
docker start 1db54527b7e6
```

### Stop multiple containers (by name or ID)

```bash
docker stop silly_chebyshev trusting_shockley boring_golick
```

### Restart a container

```bash
docker restart <container_id>
```

### Attach/exec into a running container

```bash
docker exec -it <container_id> /bin/sh    # or /bin/bash if available
```
Useful for poking around inside a container to debug config, files, or processes.

---

## 6. Naming a Container

Use `--name` to give a container a friendly, memorable name instead of a random one (e.g. `boring_golick`):

```bash
docker run --name sri-web-app -d -p 8000:80 nginx:1.23
```

```
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                                     NAMES
2a8053100a76   nginx:1.23   "/docker-entrypoint.…"   3 seconds ago   Up 2 seconds   0.0.0.0:8000->80/tcp, [::]:8000->80/tcp   sri-web-app
```

View its logs:

```bash
docker logs sri-web-app
```

---

## 7. Docker Registries

- **Public** — Docker Hub
- **Private** — requires authentication, e.g. Amazon ECR, Google Artifact Registry, Azure Container Registry, Nexus, Harbor, etc.

### Registry vs. Repository

- **Docker registry** — a service providing storage for a collection of repositories (e.g. Docker Hub), which can host public or private repositories for your apps.
- **Docker repository** — a collection of related images that share the same name but differ by tag/version (e.g. `nginx:1.22`, `nginx:1.23`, `nginx:latest`).

### Login to a registry

```bash
docker login                       # Docker Hub
docker login <registry-url>        # private registry, e.g. ECR/GCR
```

### Tag and push an image

```bash
docker tag node-app:1.0 myusername/node-app:1.0
docker push myusername/node-app:1.0
```

---

## 8. Dockerfile — Building Your Own Images

A **Dockerfile** is a text document containing instructions to assemble an image. Docker builds the image by reading and executing these instructions in order, layer by layer.

To package an app into a container, you define a "recipe" for building its image — this is the Dockerfile.

- A Dockerfile starts `FROM` a **parent/base image** — an existing image your image is based on.
- Choose the base image based on the tools/runtime you need (Node, Python, Tomcat, Java, etc.). Alpine variants (e.g. `node:19-alpine`) are smaller and lighter than full images.

### Key instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Build the image from a specified base image |
| `WORKDIR` | Set the working directory inside the container for subsequent instructions |
| `COPY` | Copy files/folders from the host into the image |
| `RUN` | Execute a command in a shell inside the container (used at **build** time) |
| `EXPOSE` | Document the port the app listens on (doesn't actually publish it — that's `-p` at runtime) |
| `CMD` | Default command run when the container **starts** |
| `ENV` | Set environment variables inside the container |
| `ARG` | Define build-time variables passed via `--build-arg` |
| `ENTRYPOINT` | Similar to `CMD`, but harder to override — often paired with `CMD` for default args |

> **CMD vs RUN:** `RUN` executes during the image **build** (e.g. installing dependencies). `CMD` defines what runs when a container **starts** from the image.

---

## 9. Example: Node.js Express App in Docker

### 9.1 Create project files

```bash
mkdir sri
cd sri
vi server.js
```

**server.js**

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    res.send("Welcome to my awesome app!");
});

app.listen(3000, function () {
    console.log("app listening on port 3000");
});
```

**package.json**

```bash
vi package.json
```

```json
{
  "name": "my-app",
  "version": "1.0",
  "dependencies": {
    "express": "4.18.2"
  }
}
```

**Dockerfile**

```dockerfile
FROM node:19-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY server.js .

EXPOSE 3000

CMD ["node", "server.js"]
```

> **Best practice:** Copy `package.json` and run `npm install` *before* copying the rest of the source code. Docker caches layers, so if only `server.js` changes, the `npm install` layer is reused instead of re-running — much faster rebuilds.

### 9.2 Build the image

```bash
docker build -t node-app:1.0 .
```

- `-t node-app:1.0` → tag the image with a name and version
- `.` → build context (current directory, where the Dockerfile lives)

Check that it was created:

```bash
docker images
```

```
IMAGE                             ID             DISK USAGE   CONTENT SIZE   EXTRA
docker/welcome-to-docker:latest   c4d56c24da4f       22.2MB         6.03MB    U
nginx:1.22-alpine                 8745c93f1a1c         63MB         16.8MB    U
nginx:1.23                        f5747a42e3ad        214MB           57MB    U
nginx:latest                      b5a9a3cfc86b        241MB           66MB
node-app:1.0                      b2cb1b98138f        262MB         56.2MB
```

### 9.3 Run the container

```bash
docker run -d --name my-node-app -p 3000:3000 node-app:1.0
```

### 9.4 Access the app

Visit **http://localhost:3000**

### 9.5 Rebuild after code changes

```bash
docker stop my-node-app && docker rm my-node-app
docker build -t node-app:1.1 .
docker run -d --name my-node-app -p 3000:3000 node-app:1.1
```

---

## 10. Quick Reference Cheat Sheet

```bash
# Images
docker images                      # list images
docker pull <image>:<tag>          # pull image
docker rmi <image_id>              # remove image
docker build -t <name>:<tag> .     # build image from Dockerfile

# Containers
docker ps                          # running containers
docker ps -a                       # all containers
docker run -d -p <host>:<container> --name <name> <image>
docker start <container>           # start stopped container
docker stop <container>            # stop running container
docker restart <container>         # restart container
docker rm <container>              # remove stopped container
docker logs -f <container>         # follow logs
docker exec -it <container> sh     # shell into container

# Cleanup
docker container prune             # remove all stopped containers
docker image prune                 # remove dangling images
docker system prune -a             # remove all unused data (careful!)

# Registries
docker login
docker tag <local-image> <registry>/<image>:<tag>
docker push <registry>/<image>:<tag>
```
