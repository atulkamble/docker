# Docker Networking, Volumes & Compose 

A beginner-friendly, reproducible lab for Docker Engine on Linux or Docker Desktop. You will connect containers, persist data, and run a Flask + PostgreSQL application with Docker Compose.

## Prerequisites

- Docker Engine running (`docker version`)
- Docker Compose v2 (`docker compose version`)
- Free local ports: **8080** (bind-mount demo) and **5000** (Compose demo)
- A terminal with permission to run Docker commands

> **Safety:** Use these examples for local practice. The example database password is not production-safe, and the Flask built-in server is a development server. Commands that delete volumes permanently remove their data.

## 1. Docker Networking

Docker networks control how containers communicate. On a **user-defined bridge**, containers on the same Docker host resolve one another by container name. Containers on separate user-defined bridges cannot communicate directly unless connected to a common network or reached through published ports.

| Driver | What it does | Typical use |
| --- | --- | --- |
| `bridge` | Private network on one Docker host | Most single-host apps |
| `host` | Shares the host network namespace (supported platforms) | Direct host networking |
| `none` | No external network interface; loopback remains | Network isolation |
| `overlay` | Connects Docker hosts using Swarm | Multi-host workloads |
| `macvlan` | Gives containers MAC addresses on a physical network | LAN integration |
| `ipvlan` | IP-level integration with an external network | Advanced LAN configurations |

### Lab A — Container-to-container communication

**Architecture**

```text
           Docker host
   +--------------------------+
   | User-defined bridge:     |
   | mynetwork                |
   |                          |
   | container1 <--DNS/IP-->  |
   | container2               |
   +--------------------------+
```

**Create and inspect the network**

```bash
docker network create --driver bridge mynetwork
docker network ls
docker network inspect mynetwork
```

**Run two Alpine containers**

```bash
docker run -d --name container1 --network mynetwork alpine:3.22 sleep infinity
docker run -d --name container2 --network mynetwork alpine:3.22 sleep infinity
```

**Test DNS resolution and connectivity**

```bash
docker exec container1 ping -c 4 container2
docker exec container2 ping -c 4 container1
```

Expected: both container names resolve and ping succeeds. IP addresses may vary.

**Connect an existing container to another network (optional)**

```bash
docker network create secondnetwork
docker network connect secondnetwork container1
docker network inspect secondnetwork
docker network disconnect secondnetwork container1
docker network rm secondnetwork
```

**Clean up**

```bash
docker rm -f container1 container2
docker network rm mynetwork
```

**Key distinction:** Docker's default `bridge` does **not** provide automatic container-name DNS like a user-defined bridge does. Published ports (`-p HOST:CONTAINER`) expose a container port through the Docker host; containers on the same bridge usually communicate over the container port without publishing it.

### Host and none modes (optional)

```bash
# Linux: Nginx binds directly to the host network; -p is ignored in host mode.
docker run -d --name host-web --network host nginx:stable
# Clean up before the Compose lab (and ensure host port 80 was free):
docker rm -f host-web

# No external networking; inspect the available interfaces.
docker run --rm --network none alpine:3.22 ip addr
```

Host networking behaves differently or may require enabling it on Docker Desktop.

## 2. Docker Volumes

A **named volume** is managed by Docker and lives independently of a container's writable layer. A **bind mount** maps a specific host path into the container. A **tmpfs mount** keeps temporary data in memory on Linux.

```text
Container A --mount--> [Named volume: myvolume] <--mount-- Container B
                           |
                     Docker-managed data
```

### Lab B — Persist a file after deleting its container

```bash
# 1. Create and inspect the volume.
docker volume create myvolume
docker volume inspect myvolume

# 2. Start a container with the volume mounted at /data.
docker run -d --name volume-writer \
  --mount type=volume,source=myvolume,target=/data \
  alpine:3.22 sleep infinity

# 3. Write a file.
docker exec volume-writer sh -c 'echo "Docker persistent data" > /data/test.txt'

# 4. Delete the container, not the volume.
docker rm -f volume-writer

# 5. Read the same file from a new, temporary container.
docker run --rm \
  --mount type=volume,source=myvolume,target=/data \
  alpine:3.22 cat /data/test.txt
```

Expected output: `Docker persistent data`.

### Lab C — Bind mount a local HTML file

```bash
mkdir -p "$HOME/docker-data"
printf '<h1>Hello Docker</h1>\n' > "$HOME/docker-data/index.html"
docker run -d --name bind-web -p 127.0.0.1:8080:80 \
  --mount type=bind,source="$HOME/docker-data",target=/usr/share/nginx/html,readonly \
  nginx:stable
curl http://localhost:8080
docker rm -f bind-web
```

Expected: `<h1>Hello Docker</h1>`. A bind mount hides any image files already present at its container target while mounted.

**Volume commands**

| Command | Effect |
| --- | --- |
| `docker volume ls` | List volumes |
| `docker volume inspect myvolume` | Inspect a volume |
| `docker volume rm myvolume` | Delete an unused volume and its data |
| `docker volume prune` | Remove unused anonymous volumes by default on current Docker versions |
| `docker volume prune -a` | Also remove unused named volumes |

Do **not** prune volumes that contain needed data. After Lab B, optionally run `docker volume rm myvolume` only if you no longer need the file.

## 3. Docker Compose

Compose defines an application in one `compose.yaml` file. Services on the same Compose network can connect by **service name**. This lab uses Flask as `web`, PostgreSQL as `db`, a private bridge network, and a named volume.

```text
Browser --> 127.0.0.1:5000 --> [web: Flask]
                                  |
                       app-network (bridge)
                                  |
                            [db: Postgres]
                                  |
                       [postgres_data volume]
```

**Important:** Inside the Flask container, the database host is `db`, **not** `localhost`. Only the web service publishes a host port. `depends_on: condition: service_healthy` waits for initial database health; the application must still handle later outages.

### Project structure

```text
docker-compose-lab/
├── app.py
├── requirements.txt
├── Dockerfile
└── compose.yaml
```

### Step 1 — Create the project

```bash
mkdir docker-compose-lab
cd docker-compose-lab
```

### Step 2 — Create `app.py`

```python
import os
import psycopg
from flask import Flask, jsonify

app = Flask(__name__)

@app.get("/")
def home():
    with psycopg.connect(
        host=os.environ["DB_HOST"],
        dbname=os.environ["DB_NAME"],
        user=os.environ["DB_USER"],
        password=os.environ["DB_PASSWORD"],
    ) as connection:
        with connection.cursor() as cursor:
            cursor.execute("SELECT version()")
            version = cursor.fetchone()[0]
    return jsonify(message="Flask connected to PostgreSQL", database_version=version)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### Step 3 — Create `requirements.txt`

```text
Flask>=3.1,<4
psycopg[binary]>=3.2,<4
```

### Step 4 — Create `Dockerfile`

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

### Step 5 — Create `compose.yaml`

```yaml
services:
  web:
    build: .
    ports:
      - "127.0.0.1:5000:5000"
    environment:
      DB_HOST: db
      DB_NAME: appdb
      DB_USER: admin
      DB_PASSWORD: examplepassword
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network

  db:
    image: postgres:17
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: examplepassword
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d appdb"]
      interval: 5s
      timeout: 5s
      retries: 10
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  postgres_data:
```

The PostgreSQL 17 image uses `/var/lib/postgresql/data` for the database directory. Do not blindly reuse that mount path when switching to PostgreSQL 18+; its image layout differs. The hardcoded password is **only for this local lab**; use secrets and a production WSGI server for real deployments.

### Step 6 — Validate and run

```bash
docker compose config
docker compose up -d --build
docker compose ps
curl http://localhost:5000
```

Expected JSON contains `"message":"Flask connected to PostgreSQL"` and a PostgreSQL version string. If the first request fails during startup, check `docker compose ps` and `docker compose logs` and retry once healthy.

### Step 7 — Inspect networking and storage

```bash
docker compose exec web getent hosts db
docker compose exec db pg_isready -U admin -d appdb
docker network ls
docker volume ls
docker compose logs --tail=50
```

Compose prefixes network and volume names with the project name unless explicitly configured otherwise.

### Step 8 — Prove database persistence

```bash
# Create a table and insert a record.
docker compose exec -T db psql -U admin -d appdb \
  -c 'CREATE TABLE students (id INT PRIMARY KEY, name TEXT);'
docker compose exec -T db psql -U admin -d appdb \
  -c "INSERT INTO students VALUES (1, 'Atul');"

# Remove containers and the project network, but KEEP the named volume.
docker compose down

# Recreate the application.
docker compose up -d

# The original record should still exist.
docker compose exec -T db psql -U admin -d appdb \
  -c 'SELECT * FROM students;'
```

Expected result:

```text
 id | name
----+-------
  1 | Atul
(1 row)
```

**Warning:** `docker compose down -v` also deletes the project's declared named volume and its database data. Ordinary `docker compose down` does not.

### Everyday Compose commands

| Command | Purpose |
| --- | --- |
| `docker compose config` | Validate/render the configuration |
| `docker compose up -d --build` | Build and start in background |
| `docker compose ps` | Show service containers |
| `docker compose logs -f` | Follow logs |
| `docker compose exec web sh` | Shell in the web container |
| `docker compose restart` | Restart existing containers |
| `docker compose stop` | Stop without removing |
| `docker compose start` | Start stopped containers |
| `docker compose down` | Remove containers and project networks, preserve named volumes |
| `docker compose down -v` | Also remove declared named and attached anonymous volumes; **deletes data** |

## 4. Quick comparison

| Concept | Main purpose | Example |
| --- | --- | --- |
| Network | Communication | Flask reaches PostgreSQL at `db:5432` |
| Volume | Persistence | PostgreSQL data survives container recreation |
| Compose | Orchestration on a Docker host | Starts web + database + network + volume |

## 5. Troubleshooting

**`docker: permission denied`** — Ensure your user can access the Docker daemon; on Linux, Docker group membership grants effectively root-level privileges.

**`port is already allocated`** — Another process is using port 5000 or 8080. Stop it or change the host-side port mapping.

**`could not translate host name "db"`** — Ensure both services are attached to `app-network` and run the application through Compose, not directly on the host.

**`connection refused` on startup** — Check `docker compose ps` and `docker compose logs db`. The health check waits for initial database readiness but does not prevent future disconnections.

**Data disappears after `down -v`** — Expected: `-v` removes the declared named volume. Recover from a backup if one exists.

## 6. Check your understanding

1. Why is a user-defined bridge preferable to the default bridge for multi-container apps?
2. Why does Flask use `db` instead of `localhost` as its database hostname?
3. What happens to a named volume when its container is deleted?
4. What is the difference between a bind mount and a named volume?
5. What does `depends_on: condition: service_healthy` guarantee—and what does it not guarantee?
6. What is the difference between `docker compose down` and `docker compose down -v`?

## Official references

- [Docker network drivers](https://docs.docker.com/engine/network/drivers/)
- [Docker bridge networking](https://docs.docker.com/engine/network/drivers/bridge/)
- [Docker volumes](https://docs.docker.com/engine/storage/volumes/)
- [Docker bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
- [Docker Compose getting started](https://docs.docker.com/compose/gettingstarted/)
- [Compose startup order and health checks](https://docs.docker.com/compose/how-tos/startup-order/)
- [Official PostgreSQL Docker image](https://hub.docker.com/_/postgres)
