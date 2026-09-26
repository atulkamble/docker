# Docker Training 

```
// ECR Practice

1. Fork, Clone and Open Repo
git clone https://github.com/atulkamble/flaskhelloworld.git
cd /flaskhelloworld
code .

2.

aws --version

IAM >> atul (user) >> admin (group)
AdminAccess policy

create access key, secret access key

aws configure >> paste
access key
secret access key
us-east-1
json

git clone https://github.com/atulkamble/flaskhelloworld.git
cd /flaskhelloworld


aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 535002879962.dkr.ecr.us-east-1.amazonaws.com

docker build --platform linux/amd64,linux/arm64 -t 535002879962.dkr.ecr.us-east-1.amazonaws.com/cloudnautic/myapp:latest --push .

docker images

docker run -d -p 5000:5000 535002879962.dkr.ecr.us-east-1.amazonaws.com/cloudnautic/myapp:latest

docker container ls

// network
docker network --help
docker network ls
docker network create mynetwork
docker network inspect mynetwork
docker network ls
docker network rm mynetwork
docker network ls
docker network prune

docker network connect mynetwork mycontainer
docker network disconnect mynetwork mycontainer


// volume
docker volume --help
docker volume ls
docker volume create myvolume
docker volume inspect myvolume
docker volume ls
docker volume rm myvolume
docker volume prune -a

// container commands

docker container create -p 80:80 --name mycontainer nginx
docker container start mycontainer
docker container ls

// run container with volume attached to it
docker container create -p 80:80 -v myvolume --name mycontainer nginx
docker container start mycontainer
docker inspect mycontainer

// run container with volume and network
docker network create mynetwork
docker volume create myvolume
docker container create -p 80:80 -v myvolume --network mynetwork --name mycontainer nginx
docker inspect mycontainer

```

## 1. Modern Application Architecture

### Monolithic Architecture

A monolithic application contains most application components in a **single application/codebase and deployment unit**.

```text
Users
  |
  v
+---------------------------+
|    Monolithic App         |
|---------------------------|
| UI                        |
| Business Logic            |
| Authentication            |
| Product / Order Logic     |
| Database Access           |
+---------------------------+
              |
              v
         +----------+
         | Database |
         +----------+
```

### Points to Remember

* Application components are tightly integrated.
* Usually deployed as one unit.
* Simple to develop and deploy for smaller applications.
* Scaling often means scaling the whole application.
* A failure or bad deployment can affect a large portion of the application.
* As the application grows, releases and maintenance can become more complex.

---

## 2. Microservices Architecture

An application is divided into multiple smaller, independently deployable services.

```text
                    Users
                      |
                      v
               +-------------+
               | API Gateway |
               +-------------+
                      |
        +-------------+-------------+
        |             |             |
        v             v             v
 +-------------+ +----------+ +-------------+
 | User Service| | Product  | |Order Service|
 +-------------+ +----------+ +-------------+
        |             |             |
        v             v             v
      DB-1          DB-2          DB-3
```

### Points to Remember

* Application is divided into smaller services.
* Services can often be developed and deployed independently.
* Different services can use different technologies where appropriate.
* Individual services can be scaled independently.
* Service-to-service communication commonly uses HTTP APIs, gRPC, or messaging.
* Requires additional networking, monitoring, security, and operational management.

### Monolithic vs Microservices

| Feature            | Monolithic             | Microservices        |
| ------------------ | ---------------------- | -------------------- |
| Structure          | Single deployment unit | Multiple services    |
| Deployment         | Whole application      | Individual services  |
| Scaling            | Usually whole app      | Individual services  |
| Complexity         | Lower initially        | Higher operationally |
| Failure isolation  | More limited           | Potentially better   |
| Technology choices | Usually consistent     | Can vary by service  |

---

# 3. Virtualization vs Containerization

## Virtual Machine Architecture

```text
+-----------------------------+
| Application                 |
+-----------------------------+
| Guest Operating System      |
+-----------------------------+
| Virtual Hardware            |
+-----------------------------+
| Hypervisor                  |
+-----------------------------+
| Host OS / Hardware          |
+-----------------------------+
```

Multiple VMs:

```text
             Physical Server
                    |
               Hypervisor
          __________|__________
         |           |          |
        VM1         VM2        VM3
         |           |          |
      Guest OS    Guest OS   Guest OS
         |           |          |
       App A       App B      App C
```

## Container Architecture

```text
             Physical/Virtual Server
                       |
                    Host OS
                       |
              Container Runtime
            _________|_________
           |         |         |
      Container1 Container2 Container3
           |         |         |
         App A     App B     App C
```

### Points to Remember

**Virtual Machine**

```text
VM = Application + Libraries + Guest OS
```

**Container**

```text
Container = Application + Required Libraries
```

Containers share the host's kernel through OS-level isolation rather than carrying a complete guest OS.

| VM                        | Container                    |
| ------------------------- | ---------------------------- |
| Includes guest OS         | Shares host kernel           |
| Usually larger            | Usually smaller              |
| Boot generally slower     | Starts quickly               |
| Strong VM-level isolation | Process/container isolation  |
| Managed by hypervisor     | Managed by container runtime |

---

# 4. Docker Architecture

```text
                       Docker Host
+--------------------------------------------------+
|                                                  |
| Docker CLI                                       |
|    |                                             |
|    | Docker API                                  |
|    v                                             |
| Docker Engine / dockerd                          |
|    |                                             |
|    +----------+-------------+----------------+   |
|    |          |             |                |   |
|  Images   Containers     Networks         Volumes|
|                                                  |
+--------------------------------------------------+
            |
            | pull / push
            v
     +----------------+
     | Docker Registry|
     | Docker Hub etc.|
     +----------------+
```

### Important Components

**Docker Client**

```bash
docker
```

Used to execute Docker commands.

**Docker Daemon**

```text
dockerd
```

Manages images, containers, networks, and volumes.

**Docker Registry**

Stores and distributes container images.

Examples include Docker Hub and private/cloud registries.

**Docker Image**

Read-only packaged filesystem and metadata used to create containers.

**Docker Container**

A runnable instance of an image.

```text
Dockerfile
    |
    | docker build
    v
Docker Image
    |
    | docker run
    v
Docker Container
```

---

# 5. Docker and CRI

CRI stands for:

```text
Container Runtime Interface
```

It is especially important in Kubernetes.

```text
Kubernetes
    |
 kubelet
    |
   CRI
    |
containerd / CRI-O
    |
Containers
```

### Points to Remember

* Docker is a container development and runtime platform.
* Kubernetes uses the CRI interface to communicate with supported container runtimes.
* containerd and CRI-O are common CRI-compatible runtimes.
* Modern Kubernetes does not require Docker Engine on worker nodes.
* Docker-built OCI-compatible images can still run in Kubernetes.

---

# 6. Installing Docker

## Ubuntu

```bash
sudo apt update
sudo apt install docker.io -y
```

Start Docker:

```bash
sudo systemctl start docker
```

Enable at boot:

```bash
sudo systemctl enable docker
```

Check:

```bash
docker --version
```

```bash
sudo systemctl status docker
```

Test:

```bash
sudo docker run hello-world
```

---

# 7. Docker Post-Installation Configuration

By default, Docker commands may require `sudo`.

Add the current user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Apply the group change in a new login session, or for a quick lab session:

```bash
newgrp docker
```

Test:

```bash
docker ps
```

### Important Security Note

Membership in the `docker` group effectively provides **root-level capabilities on the host**. Only trusted users should be added.

---

# 8. Pulling Images from Docker Hub

Architecture:

```text
Docker Host
    |
    | docker pull nginx
    v
Docker Hub
    |
    v
Local Image
```

Pull:

```bash
docker pull nginx
```

Specify tag:

```bash
docker pull nginx:latest
```

List images:

```bash
docker images
```

or:

```bash
docker image ls
```

---

# 9. Creating Your First Container

Run nginx:

```bash
docker run nginx
```

Run in background:

```bash
docker run -d nginx
```

Give the container a name:

```bash
docker run -d --name myweb nginx
```

Check:

```bash
docker ps
```

Access nginx from host using port mapping:

```bash
docker run -d \
  --name myweb \
  -p 8080:80 \
  nginx
```

Architecture:

```text
Browser
   |
   | localhost:8080
   v
Host Port 8080
   |
   | Port Mapping
   v
Container Port 80
   |
   v
Nginx
```

---

# 10. Docker Commands 101

| Operation          | Command                            |
| ------------------ | ---------------------------------- |
| Version            | `docker --version`                 |
| Information        | `docker info`                      |
| Pull image         | `docker pull nginx`                |
| List images        | `docker images`                    |
| Run container      | `docker run nginx`                 |
| Run background     | `docker run -d nginx`              |
| Running containers | `docker ps`                        |
| All containers     | `docker ps -a`                     |
| Stop               | `docker stop <container>`          |
| Start              | `docker start <container>`         |
| Restart            | `docker restart <container>`       |
| Delete container   | `docker rm <container>`            |
| Force delete       | `docker rm -f <container>`         |
| Delete image       | `docker rmi <image>`               |
| Logs               | `docker logs <container>`          |
| Container details  | `docker inspect <container>`       |
| Enter container    | `docker exec -it <container> bash` |
| Resource usage     | `docker stats`                     |

---

# 11. Containers vs Images

```text
Dockerfile
    |
    | docker build
    v
+---------------+
| Docker Image  |
| pythonapp:v1  |
+---------------+
        |
        | docker run
        v
+-------------------+
| Docker Container  |
| Running Instance  |
+-------------------+
```

### Remember

```text
Image     = Template / packaged artifact
Container = Running or stopped instance created from image
```

Multiple containers can be created from one image:

```text
              nginx:latest
                   |
          +--------+--------+
          |        |        |
          v        v        v
        web1     web2     web3
```

---

# 12. Dockerfile

A Dockerfile contains instructions for building an image.

Example Python application:

`app.py`

```python
print("Hello from Docker")
```

`Dockerfile`

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

Build:

```bash
docker build -t pythonapp:v1 .
```

Check:

```bash
docker images
```

Run:

```bash
docker run pythonapp:v1
```

Expected output:

```text
Hello from Docker
```

### Dockerfile Flow

```text
Dockerfile
   |
docker build
   |
   v
Image
   |
docker run
   |
   v
Container
```

---

# 13. Important Dockerfile Instructions

| Instruction  | Purpose                           |
| ------------ | --------------------------------- |
| `FROM`       | Base image                        |
| `WORKDIR`    | Working directory                 |
| `COPY`       | Copy files                        |
| `ADD`        | Copy with additional capabilities |
| `RUN`        | Execute build-time command        |
| `ENV`        | Environment variable              |
| `EXPOSE`     | Document intended container port  |
| `CMD`        | Default container command         |
| `ENTRYPOINT` | Main executable                   |

### Important Difference

```text
RUN  → executed while building image
CMD  → default command when container starts
```

Also remember: `EXPOSE` **does not publish a port to the host**. Use `docker run -p` for actual port publishing.

---

# 14. Docker Commit

Create container:

```bash
docker run -it --name myubuntu ubuntu bash
```

Inside:

```bash
apt update
apt install curl -y
```

Exit:

```bash
exit
```

Create image from the modified container:

```bash
docker commit myubuntu myubuntu:v1
```

Check:

```bash
docker images
```

### Points to Remember

`docker commit` is useful for learning and troubleshooting, but **Dockerfiles are preferred** for reproducible production image creation.

---

# 15. Central Container Repository

Architecture:

```text
Developer
   |
Docker Build
   |
   v
Local Image
   |
Docker Push
   |
   v
+--------------------+
| Container Registry |
+--------------------+
   |
Docker Pull
   |
   v
Server / CI/CD / K8s
```

Examples:

* Docker Hub
* GitHub Container Registry
* Azure Container Registry
* Amazon Elastic Container Registry
* Google Artifact Registry

---

# 16. Push Image to Docker Hub

Login:

```bash
docker login
```

Build:

```bash
docker build -t myapp:v1 .
```

Tag:

```bash
docker tag myapp:v1 <username>/myapp:v1
```

Push:

```bash
docker push <username>/myapp:v1
```

Another machine can pull:

```bash
docker pull <username>/myapp:v1
```

Run:

```bash
docker run <username>/myapp:v1
```

Flow:

```text
Dockerfile
   ↓
docker build
   ↓
Local Image
   ↓
docker tag
   ↓
docker push
   ↓
Docker Hub
   ↓
docker pull
   ↓
Other Hosts
```

---

# 17. Docker Networking

List networks:

```bash
docker network ls
```

Common Docker network drivers:

```text
bridge
host
none
```

### Bridge

Typical network for standalone containers.

```text
Host
 |
Docker Bridge
 |
 +------ Container A
 |
 +------ Container B
```

Create custom network:

```bash
docker network create mynetwork
```

Run containers:

```bash
docker run -d \
  --name web1 \
  --network mynetwork \
  nginx
```

```bash
docker run -d \
  --name web2 \
  --network mynetwork \
  nginx
```

Inspect:

```bash
docker network inspect mynetwork
```

---

# 18. Bridge vs Host vs None

| Network | Description                                                     |
| ------- | --------------------------------------------------------------- |
| bridge  | Containers use Docker-managed networking                        |
| host    | Container uses host network namespace on supported Linux setups |
| none    | Networking disabled                                             |

Commands:

```bash
docker run -d --network bridge nginx
```

```bash
docker run -d --network host nginx
```

```bash
docker run -d --network none nginx
```

For most application labs, prefer a **user-defined bridge network**.

---

# 19. Container-to-Container Communication

```text
+----------------+
| Custom Network |
|                |
| Web Container  |
|       |        |
|       v        |
| DB Container   |
+----------------+
```

Create:

```bash
docker network create app-network
```

Database:

```bash
docker run -d \
  --name database \
  --network app-network \
  mysql:8
```

Web application:

```bash
docker run -d \
  --name web \
  --network app-network \
  nginx
```

Containers on the custom network can use container names for DNS-based communication.

---

# 20. Port Mapping

Syntax:

```bash
docker run -p HOST_PORT:CONTAINER_PORT IMAGE
```

Example:

```bash
docker run -d \
  --name web \
  -p 8080:80 \
  nginx
```

Architecture:

```text
User
 |
 | HTTP :8080
 v
Host Machine
 |
 | 8080 → 80
 v
Docker Container
 |
 | Port 80
 v
Nginx
```

Open:

```text
http://localhost:8080
```

---

# 21. Docker Storage

Container writable data is generally tied to the container lifecycle unless persistent storage is used.

```text
Container
   |
   +---- Writable Container Layer
   |
   +---- Volume
             |
             v
       Persistent Data
```

Common approaches:

```text
Volumes
Bind Mounts
tmpfs mounts
```

---

# 22. Docker Volumes

Create:

```bash
docker volume create mydata
```

List:

```bash
docker volume ls
```

Inspect:

```bash
docker volume inspect mydata
```

Use:

```bash
docker run -d \
  --name myweb \
  -v mydata:/usr/share/nginx/html \
  nginx
```

Architecture:

```text
Container
    |
/usr/share/nginx/html
    |
    v
Docker Volume
    |
    v
Persistent Data
```

Delete:

```bash
docker volume rm mydata
```

---

# 23. Bind Mounts

Bind mounts map a host path directly into a container.

Create:

```bash
mkdir website
```

Create:

```bash
echo "<h1>Hello Docker</h1>" > website/index.html
```

Run:

```bash
docker run -d \
  --name web \
  -p 8080:80 \
  -v "$(pwd)/website:/usr/share/nginx/html:ro" \
  nginx
```

Architecture:

```text
Host
./website
    |
    | Bind Mount
    v
Container
/usr/share/nginx/html
```

### Volume vs Bind Mount

| Volume                                  | Bind Mount                         |
| --------------------------------------- | ---------------------------------- |
| Managed by Docker                       | Uses host filesystem path          |
| Good default for persistent app data    | Good for development/configuration |
| Easier Docker lifecycle management      | Direct host access                 |
| Less dependent on host directory layout | Host-path dependent                |

---

# 24. Docker Storage Best Practices

### Points to Remember

* Don't store important persistent data only in the container writable layer.
* Prefer volumes for application/database persistent data.
* Use bind mounts when direct host-file access is required.
* Use read-only mounts where possible.
* Don't store passwords inside Dockerfiles or images.
* Use secrets/configuration mechanisms appropriate to the deployment platform.
* Back up important persistent volumes.
* Remove unused resources carefully.

Useful commands:

```bash
docker volume ls
```

```bash
docker system df
```

```bash
docker system prune
```

Be careful with prune commands because they can delete unused Docker resources.

---

# 25. Docker Compose

Docker Compose manages **multi-container applications declaratively**.

Instead of:

```text
docker run ...
docker run ...
docker network create ...
docker volume create ...
```

Define the application in:

```text
compose.yaml
```

Modern command:

```bash
docker compose
```

---

# 26. Docker Compose Architecture

Example:

```text
                 User
                  |
               Port 8080
                  |
                  v
          +---------------+
          | Web Container |
          +---------------+
                  |
          Compose Network
                  |
                  v
          +---------------+
          | DB Container  |
          +---------------+
                  |
                  v
              DB Volume
```

---

# 27. Basic Docker Compose File

`compose.yaml`

```yaml
services:

  web:
    image: nginx:latest
    ports:
      - "8080:80"

  database:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: example
```

Start:

```bash
docker compose up
```

Background:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

Stop and remove application containers/networks:

```bash
docker compose down
```

---

# 28. Docker Compose with Build

Directory:

```text
project/
├── Dockerfile
├── app.py
└── compose.yaml
```

Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY . .

CMD ["python", "app.py"]
```

Compose:

```yaml
services:

  app:
    build: .
    container_name: python-app
```

Build:

```bash
docker compose build
```

Start:

```bash
docker compose up -d
```

---

# 29. Docker Compose Networking

Compose normally creates a default application network.

```text
docker compose up
       |
       v
+--------------------------+
| Compose Default Network  |
|                          |
| frontend ---- backend    |
|                 |        |
|              database    |
+--------------------------+
```

Services can communicate using their **service names**.

For example:

```text
database:3306
```

instead of relying on a changing container IP address.

---

# 30. Docker Compose Lifecycle Commands

| Task                   | Command                  |
| ---------------------- | ------------------------ |
| Start                  | `docker compose up`      |
| Background             | `docker compose up -d`   |
| Build                  | `docker compose build`   |
| List                   | `docker compose ps`      |
| Logs                   | `docker compose logs`    |
| Follow logs            | `docker compose logs -f` |
| Stop                   | `docker compose stop`    |
| Start stopped services | `docker compose start`   |
| Restart                | `docker compose restart` |
| Remove stack           | `docker compose down`    |
| Pull images            | `docker compose pull`    |

---

# 31. Complete Docker Lifecycle Architecture

```text
                 Developer
                     |
                     v
               Source Code
                     |
                     v
                Dockerfile
                     |
              docker build
                     |
                     v
              Docker Image
                     |
          +----------+----------+
          |                     |
          v                     v
     docker run            docker push
          |                     |
          v                     v
      Container          Container Registry
                                |
                           docker pull
                                |
                                v
                         Server / CI-CD
                                |
                                v
                            Container
```

For multiple services:

```text
Application
     |
compose.yaml
     |
docker compose up
     |
+----+----------+----------+
|               |          |
v               v          v
Frontend      Backend    Database
   |             |          |
   +-------------+----------+
          Network
                         |
                      Volume
```

# 32. Final Points to Remember

1. **Dockerfile creates an image; an image creates containers.**
2. **Images are immutable packaged artifacts; containers add a writable runtime layer.**
3. `docker build` builds an image.
4. `docker run` creates and starts a container.
5. `docker pull` downloads an image.
6. `docker push` uploads an image to a registry.
7. `-p 8080:80` means **Host 8080 → Container 80**.
8. Custom bridge networks simplify container-to-container communication using names.
9. Volumes provide persistent storage independent of a particular container.
10. Bind mounts map host files/directories directly into containers.
11. Docker Compose is useful for defining and running multi-container applications.
12. Prefer Dockerfiles over `docker commit` for reproducible image creation.
13. Don't hard-code passwords, API keys, or other secrets in images.
14. Keep images small and use trusted/minimal base images where practical.
15. Containers are designed to be replaceable; persistent application state should normally live outside the container's writable layer.

### Quick memory flow

```text
CODE
  ↓
DOCKERFILE
  ↓
IMAGE
  ↓
CONTAINER
  ↓
NETWORK + STORAGE
  ↓
DOCKER COMPOSE
  ↓
MULTI-CONTAINER APPLICATION
```
