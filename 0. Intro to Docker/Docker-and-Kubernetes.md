# Docker & Kubernetes Fundamentals

### Brief Overview

This note covers **Docker & Kubernetes fundamentals** and was created from [Docker Containers and Kubernetes Fundamentals](https://www.youtube.com/watch?v=kTp5xUtcalw) – Full Hands‑On Course YouTube video. It walks through container creation, orchestration, and the key concepts needed to build and deploy micro‑services in a cloud‑native environment.

It covers Docker basics, Kubernetes architecture, deployment strategies, and hands‑on labs.

### Key Points

- Master **Docker CLI**, image building, and volume management.
- Understand Kubernetes concepts: **Pods**, **ReplicaSets**, **Deployments**, **Services**, and **Autoscaling**.
- Apply practical labs: **microservices**, **rolling updates**, **blue/green** deployments, and **HPA**.
- Learn how to set up and use **monitoring**, **logging**, and **secrets**.

---

## Setup & Prerequisites

- Required hardware: **Windows 10+**, **macOS**, or **Linux** laptop/PC.
- Recommended tools:
  - **Visual Studio Code** + Docker extension (free, cross‑platform).
  - **Docker Desktop** with Kubernetes enabled (Windows/macOS) or Docker + Kubernetes installed via distro docs (Linux).
  - Optional: Docker Hub account, Git (or download zip).
- No prior Docker/Kubernetes knowledge needed; suitable for developers, DevOps, IT pros, or technical managers.

## Microservices Architecture

_“A variant of Service‑Oriented Architecture that structures an application as a collection of loosely coupled, fine‑grained services exposing lightweight APIs (e.g., HTTP, gRPC).”_

### Monolithic vs. Microservices

| Aspect           | Monolithic                                          | Microservices                                                   |
| ---------------- | --------------------------------------------------- | --------------------------------------------------------------- |
| Structure        | Single compiled unit, all code in one address space | Multiple independent services, each with its own responsibility |
| Deployment       | One unit deployed to a server/VM                    | Each service deployed independently                             |
| Scaling          | Scale whole app (adds whole VM)                     | Scale individual services as needed                             |
| Technology stack | Uniform across whole app                            | Teams can choose languages/technologies per service             |
| Data storage     | Shared database                                     | Separate databases per service possible                         |

### Transforming a Monolith

- **Strangler Pattern** (Martin Fowler):
  1. Place a **facade** that routes requests to the legacy code.
  2. Incrementally extract functionality into new microservices.
  3. Route calls from the facade to the new services.
  4. Remove legacy code and facade once fully migrated.

### Anti‑patterns

- Treating microservices as a “magic dust” – expecting instant benefits.
- Introducing all changes at once → **complexity explosion** and potential system failures.
- Neglecting **DevOps**, **CI/CD**, and testing pipelines.
- Missing metrics to validate each migration step.

### Benefits & Drawbacks

**Benefits**

- Failure isolation – one service down does not crash the whole system.
- Reduced vendor lock‑in (open‑source tech).
- Smaller codebases → faster development, easier understanding.
- Independent scaling per service.

**Drawbacks**

- Added operational complexity (multiple services, networking, data consistency).
- Need for robust testing, monitoring, and retry strategies (service mesh helps).
- Potential latency from inter‑service API calls.
- Security surface grows – more services can communicate.

### Cloud‑Native

_“An approach to building and running applications that fully exploits cloud computing models, using containers, service meshes, microservices, immutable infrastructure, and declarative APIs.”_

### Core Characteristics

- **Containers** & **service meshes** for runtime.
- **Microservices** for modularity.
- **Immutable infrastructure** – replace, don’t patch.
- **Declarative APIs** – desired state expressed, system converges automatically.
- **Observability** – metrics, logs, tracing.
- **Open‑source ecosystem** (CNCF projects).

### Cloud‑Native Trail Map – First Six Steps

1. **Containerize** your application.
2. **Deploy & monitor** containers (basic visibility).
3. Implement **CI/CD** pipelines for automated delivery.
4. Adopt an **orchestrator** (e.g., Kubernetes).
5. Add **observability** tools (metrics, tracing).
6. Introduce **service mesh** for advanced networking features.

You may adopt steps incrementally; simultaneous implementation is discouraged.

### CNCF Landscape

| Maturity Level | Description                               |
| -------------- | ----------------------------------------- |
| Sandbox        | Early‑stage projects, experimental.       |
| Incubating     | Growing projects, gaining traction.       |
| Graduated      | Mature, production‑ready, widely adopted. |

**Examples**

- **Graduated:** Kubernetes, Helm, Jaeger.
- **Incubating:** Linkerd, gRPC.

The CNCF website (cncf.io) provides project pages with repository links, star counts, activity metrics, and official Twitter handles for staying updated.

### Containers Fundamentals

_“A container is a unit of deployment that bundles application code, runtime, libraries, and system tools, ensuring it runs the same everywhere.”_

### VM vs. Container

| Feature            | Virtual Machine                      | Container                                        |
| ------------------ | ------------------------------------ | ------------------------------------------------ |
| Boot process       | BIOS → OS boot (5–10 min).           | Uses host kernel, starts in seconds.             |
| Resource footprint | 12 GB RAM, 500 GB disk (typical).    | ~100 MB disk, 64–100 MB RAM.                     |
| Isolation          | Full OS isolation.                   | Process‑level isolation (shared kernel).         |
| Portability        | Heavier, less portable.              | Highly portable across environments.             |
| Use case           | Long‑running, heavyweight workloads. | Short‑lived tests, microservices, rapid scaling. |

### Image Layers

- Images are built as **layers** (base OS → customizations → app).
- Docker caches layers; repeated pulls only download missing layers.
- Only the **top layer** is writable; lower layers are read‑only.

### Registry & Orchestrator

- **Container registry**: centralized storage for images (e.g., Docker Hub, cloud provider registries).
- **Orchestrator**: manages scaling, health, and networking of containers (self‑hosted or managed Kubernetes clusters).

### Docker Platform

- **Docker** supplies:
  - Runtime for Windows, macOS, Linux.
  - CLI for building/managing containers.
  - Dockerfile format for image builds.

- **Docker Desktop** (free) – primary way to run Docker locally.
  - On Windows 10 2004+ can leverage **WSL 2** (Linux kernel inside Windows) for faster performance.
  - UI provides: container list, image list, settings (WSL 2 toggle, Kubernetes enable), troubleshooting (restart).
- Login to Docker Desktop uses the same credentials as Docker Hub.

### Docker CLI Basics

- **docker info** – displays system details (containers, images, resources).
- **docker version** – shows version numbers of client and server components.
- **docker login** – authenticates to a container registry (defaults to Docker Hub).

### Running Containers ▶

| Command                                                                | Purpose                                                  |
| ---------------------------------------------------------------------- | -------------------------------------------------------- |
| docker pull <image>                                                    | Download image (layers cached).                          |
| docker run [‑d] [‑‑name <cname>] -p <hostPort>:<containerPort> <image> | Start container (‑d runs detached).                      |
| docker ps                                                              | List running containers. Add ‑a to include stopped ones. |
| docker stop <cname>                                                    | Gracefully stop a running container.                     |
| docker kill <cname>                                                    | Force‑stop a stuck container.                            |
| docker image inspect <image>                                           | Show image metadata (debugging).                         |
| docker exec -it <cname> <cmd>                                          | Run a command inside a running container (e.g., bash).   |
| docker rm <cname>                                                      | Remove a stopped container from memory.                  |
| docker rmi <image>                                                     | Delete a local image.                                    |

- **Image name** = identifier in the registry.
- **Container name** = runtime instance; used for stop, exec, rm.
- Use --name to assign a readable container name; otherwise Docker auto‑generates one.
- Resource limits can be set on docker run (e.g., --memory, --cpus).

---

All commands above are foundational; further options are explored later in the course.

🧹 Docker CLI Cleanup & Management

List stopped containers → docker ps -a (filter for Exited status)

Remove stopped containers → docker rm $(docker ps -aq -f status=exited)

List cached images → docker images

Delete an image → docker rmi <image_name>

System prune → docker system prune

Deletes all unused images, containers, networks, and build cache. Use with caution.

▶️ Running & Managing Containers

Launching an NGINX container

docker run -d --name webserver -p 8080:80 nginx

-d runs in detached mode (returns prompt).

--name webserver assigns a readable name.

-p 8080:80 maps host port 8080 → container port 80.

Verifying the container

docker ps shows the running container, its name, image, and port mapping.

Access via a browser at http://localhost:8080 – the NGINX welcome page appears.

Inspecting the container filesystem

docker exec -it webserver bash opens an interactive shell as root inside the container.

Use standard Linux commands (ls, cat, less, etc.) for debugging.

Stopping & removing the container

Stop: docker stop webserver (uses the container name, not the image name).

Remove from memory: docker rm webserver (container disappears from docker ps -a).

Cleaning up the image

Image remains after container removal: docker images still lists nginx.

Remove the image: docker rmi nginx (deletes all associated layers).

🏗️ Building Docker Images & Dockerfiles

Basic Dockerfile structure

FROM <base_image>
COPY <src> <dest>

FROM specifies the base image (e.g., nginx:alpine).

COPY transfers files from the build context into the image.

More complex example (Node.js app)

Instruction

Purpose

FROM node:14-alpine

Use a lightweight Node base image.

RUN npm install -g npm@latest

Update npm inside the image.

COPY . /src

Copy application source code.

WORKDIR /src

Set working directory.

RUN npm install

Install dependencies.

EXPOSE 8080

Declare the container’s listening port.

CMD ["node", "app.js"]

Define the startup command.

Building the image

docker build -t mynodeapp:1.0 .

-t assigns name:tag (e.g., mynodeapp:1.0).

. indicates the Dockerfile is in the current directory.

Tagging images

docker tag <source_image> <repository>/<name>:<tag>

If no repository is provided, Docker Hub is assumed.

🖥️ Using Visual Studio Code for Docker

Install the Docker extension from Microsoft (≈ 6 M installs).

Extension features:

Generate Dockerfiles (Ctrl+Shift+P → Docker: Add Docker Files to Workspace).

Create .dockerignore automatically.

UI panel listing Images, Containers, and Volumes.

Right‑click actions: Run, Stop, Inspect, Remove, Push, Pull, Tag.

Build an image from VS Code:

Command palette → Docker: Build Image (uses the same docker build under the hood).

Run a container from VS Code:

Command palette → Docker: Run → select the built image, specify port mapping (e.g., 3000).

Access logs, attach a shell, or open the app in a browser directly from the UI.

💾 Data Persistence with Volumes

Containers are ephemeral; data written inside disappears when the container is removed.

Volumes map external storage (host folder or cloud storage) into a container path, ensuring data survives container restarts or crashes.

Creating and using a volume

docker volume create mydata
docker run -d --name webapp -v mydata:/app/data nginx

-v mydata:/app/data mounts the volume at /app/data inside the container.

Verifying persistence

docker exec -it webapp bash → create a file in /app/data.

Stop & remove the container, then start a new one with the same volume; the file remains.

📦 Managing Docker Volumes

Command

Description

docker volume create <name>

Create a new volume.

docker volume ls

List all volumes.

docker volume inspect <name>

Show detailed information (mount point, usage).

docker volume rm <name>

Delete a volume (must be unused).

docker volume prune

Remove all dangling volumes (not attached to any container).

Important: A volume cannot be removed while any container is still using it. Stop and docker rm those containers first.

🗂️ YAML Fundamentals

YAML = “YAML Ain’t Markup Language”, human‑readable data serialization.

Basic key‑value: key: value (note the required space after :).

Nested structures use two‑space indentation.

Lists (block style):

items:

- first
- second

Flow style (JSON‑like) exists but is rarely used in Docker/Kubernetes docs.

Use an online YAML linter (e.g., yamllint.com) to catch spacing errors.

🐳 Docker Compose Essentials

Docker Compose lets you define multi‑container applications in a single docker-compose.yml.

Introduced as a CLI plugin (docker compose) in version 2 (written in Go, no Python dependency).

Typical docker-compose.yml structure

services:
web:
image: mywebapp:latest
ports: - "5000:5000"
redis:
image: redis:alpine

Service names (web, redis) act as hostnames within the internal network.

When to use Compose

Small workloads, local development, testing, or cloud services that natively support Compose (e.g., Azure App Service, AWS ECS).

📋 Docker Compose Cheat Sheet

Command

Purpose

docker compose build

Build images defined in the compose file.

docker compose up [-d]

Start all services (add -d for detached mode).

docker compose down

Stop containers and remove them, networks, and default volumes.

docker compose stop

Stop containers but keep them in memory.

docker compose ps

List running services.

docker compose rm

Remove stopped containers.

docker compose logs [service]

Show logs for a service.

docker compose exec <service> <cmd>

Run a command inside a running service (e.g., bash).

docker compose cp <service>:<src> <dest>

Copy files from a container.

docker compose cp <src> <service>:<dest>

Copy files to a container.

docker compose -p <project_name> up

Run a second instance of the same compose file under a different project name.

docker compose ls

List active compose projects.

🧪 Lab: Python Front‑End + Redis with Docker Compose

docker-compose.yml (excerpt)

services:
web:
build: .
ports: - "5000:5000"
redis:
image: redis:alpine

Web service builds a custom image from the local Dockerfile (Python app).

Redis service pulls the official redis:alpine image.

Workflow

Build images: docker compose build (optional, up builds automatically).

Launch: docker compose up -d.

Verify: Open http://localhost:5000 in a browser; the app responds.

Inspect running containers: docker compose ps.

Stop & clean: docker compose down.

🔧 Additional Docker Commands Referenced

Detach flag: -d (run in background).

Port mapping: -p <hostPort>:<containerPort>.

Name assignment: --name <container_name>.

Volume mount: -v <volume_name>:<container_path> or bind‑mount -v /host/path:/container/path.

Inspect: docker inspect <name_or_id> (shows configuration, mount points, etc.).

These notes capture the core Docker CLI operations, image creation, VS Code integration, data persistence, YAML basics, and Docker Compose workflow needed to manage containerized applications end‑to‑end.

📦 Docker Compose Project Management

📋 Listing containers & logs

docker compose ps – shows running containers for the current compose project.

docker compose logs -f <service> – streams live logs of a specific service; press Ctrl C to stop.

Logs are useful to verify that a service started correctly and to monitor runtime behavior.

🔀 Multiple compose projects & port conflicts

By default, Docker Compose uses the folder name as the project identifier.

To run a second instance, supply a custom name:

docker compose -p test up -d

If the same host port is already bound (e.g., 5000), Docker reports “bind for local address 5000 failed: port is already allocated.”

Resolve by editing the ports mapping in docker‑compose.yml, e.g., change 5000:5000 to 5001:5000, then redeploy.

🧹 Cleaning up projects

Remove a project’s containers, network, and default volumes:

docker compose down

When a custom project name was used, specify it:

docker compose -p test down

After down, docker compose ls should show only remaining projects.

docker ps and docker volume ls can confirm that no containers or stray volumes remain; volumes are not removed automatically and must be deleted manually (docker volume rm <name>).

🛠️ Advanced Docker Compose Features

🌐 Networks (public ↔ private)

Define named networks in the networks section.

Assign services to one or more networks to control connectivity:

Service

Networks attached

Visibility

frontend

public

Can reach services on public only.

backend

public, private

Bridges frontend and db.

db

private

Only reachable from services on private.

Service names act as DNS hostnames within the same network.

📂 Volumes (named & service‑scoped)

Named volumes are declared under volumes: and can be shared across services.

Service‑scoped volumes are defined inline (- ./data:/app/data) and are not shareable.

Example of a read‑only bind mount:

volumes:

- mydata:/var/lib/data:ro

⚙️ Resource limits

Use the deploy.resources block (shown in yellow in the demo) to set reservations and limits:

deploy:
resources:
reservations:
cpus: "0.25"
memory: "20M"
limits:
cpus: "0.5"
memory: "150M"

Reservations guarantee resources; limits enforce a hard ceiling.

🌱 Environment variables & secrets

Inline definition:

environment:

- NODE_ENV=production
- API_KEY=${API_KEY}

Override at runtime with -e flags (docker compose run -e VAR=value).

.env file: Docker Compose automatically loads key/value pairs from a file named .env in the same directory.

Secrets (CNCF‑style) are defined under secrets: and referenced by services; values are sourced from files (e.g., password.txt).

⏳ Service start order (depends_on)

Ensure a dependent service starts only after its prerequisite:

services:
app:
depends_on: - db

Compose will wait for db to become healthy before launching app.

🔄 Restart policies

Control container behavior on host reboot or failure:

Policy

Description

no (default)

Container stays stopped after host restart.

always

Container restarts indefinitely until explicitly removed.

on-failure

Restarts only if the container exits with a non‑zero status.

unless-stopped

Restarts on reboot unless the user stopped it manually.

restart: always

📤 Container Registries

🔐 Authentication

Log in to a registry (Docker Hub, Azure Container Registry, etc.) with:

docker login

If already logged in, Docker reports the existing session.

🏷️ Tagging images

Tag must include the registry/repository prefix (e.g., myuser/myrepo:1.0).

docker tag local-image myuser/myrepo:1.0

📤 Push & 📥 Pull workflow

Push a tagged image to the remote registry:

docker push myuser/myrepo:1.0

Pull an image by its full name:

docker pull myuser/myrepo:1.0

Public images are accessible to anyone; private repositories require explicit permission.

🗑️ Deleting images & repositories

Remove a local image: docker rmi <image>.

Delete a remote repository via the provider’s UI (e.g., Docker Hub Settings → Delete repository).

☸️ Introduction to Kubernetes

📚 What Kubernetes Is

Kubernetes (often abbreviated “k8s”) is a vendor‑neutral, open‑source platform for deploying, managing, and scaling containerized workloads.

Originated at Google (successor to Borg & Omega) and donated to the CNCF in 2015.

Provides service discovery, load balancing, storage orchestration, rollout/rollback, health monitoring, configuration, and secret management.

Does not supply application‑level services (databases, message buses, caches).

🏗️ Architecture Overview

Component

Role

Control plane (master node)

Runs API server, scheduler, controller manager; stores desired state.

Worker nodes

Host the pods (the smallest deployable unit).

Pod

One or more containers sharing network namespace and storage.

Cluster

Set of worker nodes managed by a single control plane.

💻 Running Kubernetes Locally

Tool

OS support

Notes

Docker Desktop

Windows 10 2004+, macOS

Enables a single‑node cluster; integrates with WSL 2 on Windows.

Minikube

Windows, macOS, Linux

Creates a VM (VirtualBox, Hyper‑V, or Docker) to host a multi‑node cluster.

Kind (Kubernetes IN Docker)

Windows, macOS, Linux

Runs clusters as Docker containers; useful for testing multi‑node topologies.

Docker Desktop limits you to one node but is sufficient for most dev scenarios.

Minikube and Kind can emulate multiple workers for more complex tests (e.g., node affinity).

🛠️ kubectl CLI & Contexts

kubectl talks to the API server; configuration lives in ~/.kube/config.

A context groups a cluster, user, and namespace.

Command

Purpose

kubectl config current-context

Show the active context.

kubectl config get-contexts

List all configured contexts.

kubectl config use-context <name>

Switch to a different context.

kubectl config delete-context <name>

Remove a context from the config file.

kubectl config rename-context <old> <new>

Rename a context for clarity.

Shortcut tool kubectx simplifies context switching (kubectx demo).

📜 Declarative vs Imperative Resource Creation

Imperative: series of kubectl commands (e.g., kubectl run, kubectl create deployment). Good for quick tests.

Declarative: define resources in YAML files and apply them:

apiVersion: apps/v1
kind: Deployment
metadata:
name: web
spec:
replicas: 3
selector:
matchLabels:
app: web
template:
metadata:
labels:
app: web
spec:
containers: - name: web
image: myuser/web:1.0
ports: - containerPort: 80

Apply with kubectl apply -f deployment.yaml.

Declarative files are version‑controlled, reproducible, and align with GitOps practices.

📄 YAML Generation & Resource Creation

YAML – Human‑readable markup language used to declare Kubernetes objects.

Copy from docs – Browse https://kubernetes.io/docs, locate the desired object, click the copy icon.

VS Code template – Create a new .yaml file, press Ctrl Space and pick the generated manifest template.

kubectl dry‑run – Generate a manifest from the CLI without applying it:

kubectl create deployment myapp --image=nginx \
 --dry-run=client -o yaml > myapp.yaml

--dry-run=client tells kubectl to validate locally.

-o yaml outputs the manifest in YAML format.

Redirect (>) writes the result to a file.

🚀 Deploying an NGINX Container

Imperative (command‑line)

kubectl create deployment mynginx1 --image=nginx
kubectl get deployments # list deployments

Declarative (YAML)

apiVersion: apps/v1
kind: Deployment
metadata:
name: mynginx2
spec:
replicas: 1
selector:
matchLabels:
app: nginx
template:
metadata:
labels:
app: nginx
spec:
containers: - name: nginx
image: nginx

kubectl create -f mynginx2.yaml
kubectl get deployments # shows both deployments

Cleanup

kubectl delete deployment mynginx1
kubectl delete deployment mynginx2

🏷️ Namespaces

Namespace – A logical partition within a cluster that groups related resources, acting like a folder.

Purpose – Separate environments (e.g., dev, test, prod) or isolate test workloads.

Default – Kubernetes always creates a default namespace.

Namespace manifest

apiVersion: v1
kind: Namespace
metadata:
name: prod

Use the namespace in other resources by adding namespace: prod under metadata.

Resource quotas (optional)

Apply a ResourceQuota object to limit CPU, memory, etc., within a namespace.

Common commands

Command

Description

kubectl get namespaces (or kubectl get ns)

List all namespaces

kubectl create ns <name>

Create a namespace

kubectl delete ns <name>

Delete a namespace (removes all child objects)

kubectl config set-context --current --namespace=<name>

Set the current context’s default namespace

kubectl get pods --all-namespaces (-A)

List pods across every namespace

🖥️ Master Node (Control Plane)

Component

Role

API Server

Exposes the REST API; only entry point for kubectl and other clients.

etcd

Distributed key‑value store; holds the cluster’s desired state (single source of truth).

Controller Manager

Runs controllers that reconcile actual state with desired state (e.g., Deployment controller).

Cloud Controller Manager

Interfaces with cloud provider APIs (load balancers, storage, node lifecycle).

Scheduler

Watches for unscheduled Pods and assigns them to suitable Nodes based on constraints.

Add‑ons

Optional extensions (e.g., DNS, metrics server) installed on the control plane.

Application containers do not run on the master node in typical production setups.

🖧 Worker Nodes

Node – Physical or virtual machine that hosts Pods.

Installed services (automatically when a node joins):

Container runtime – Implements the CRI (e.g., containerd, CRI‑O).

kubelet – Ensures Pods described in the API are running and healthy.

kube‑proxy – Manages network rules; all pod traffic passes through it.

Runtime note

Prior to Kubernetes 1.19 Docker’s moby shim was used; from 1.19 Docker is no longer a supported runtime.

Images still run unchanged; direct Docker commands on a node are replaced by crictl.

Node pools

A node pool is a group of VMs with identical size/shape.

A cluster may contain multiple pools (e.g., one without GPUs, another with GPUs).

📊 Inspecting Nodes

kubectl get nodes # summary of all nodes
kubectl describe node <node-name> # detailed information

Typical fields shown by describe:

Name, Roles (master/worker), Labels, CreationTimestamp

Capacity: CPU, memory, Pods (max number of Pods the node can host)

Allocatable resources, Operating System, Architecture

System Info (kernel version, Docker/container runtime version)

🐳 Pods Overview

Pod – The smallest deployable unit in Kubernetes, encapsulating one or more containers that share networking and storage.

Shared IP and port space across containers in the same Pod.

Volumes can be mounted by any container in the Pod.

Ephemeral – If a Pod crashes, a new Pod with a fresh IP is created; the original is never updated.

Scaling – Performed by adding more Pods, not by adding containers to an existing Pod.

Typical use‑case: one main container (application logic) plus optional sidecar containers (logging, proxy, etc.).

🔄 Pod Lifecycle

Creation flow

kubectl create → API server writes object to etcd.

Scheduler selects a Node and updates the object.

kubelet on the chosen Node creates the container(s).

Pod status is updated in etcd.

Deletion flow

kubectl delete → API server marks Pod for termination (default graceful period 30 s).

kubelet sends SIGTERM, then SIGKILL after the grace period.

Pod status changes to Terminating → finally removed from etcd.

Common Pod phases

Phase

Meaning

Pending

Scheduled but not yet running (e.g., waiting for resources).

Running

All containers started; at least one is still running.

Succeeded

All containers terminated with exit code 0.

Failed

At least one container terminated with non‑zero exit code.

Unknown

Status cannot be obtained from the node.

CrashLoopBackOff

Container repeatedly fails; kubelet backs off before restarting.

🛠️ Defining & Running Pods

Declarative (YAML)

apiVersion: v1
kind: Pod
metadata:
name: myapp
labels:
app: myapp
spec:
containers:

- name: nginx
  image: nginx
  ports:
  - containerPort: 80
    env:
  - name: DBCON
    value: "my-connection-string"

kubectl create -f myapp.yaml

Imperative (CLI)

kubectl run mynginx --image=nginx
kubectl run mybusybox --image=busybox -it -- /bin/sh

kubectl get pods – list Pods.

kubectl get pods -o wide – extra columns (node, IP).

kubectl get pod mynginx -o yaml > mynginx.yaml – export current definition.

kubectl describe pod <name> – detailed status, events, and container info.

kubectl exec -it <pod> -- /bin/sh – open interactive shell inside a running container.

kubectl delete pod <name> or kubectl delete -f <file>.yaml – remove a Pod.

🧩 Init Containers

Init container – A special container that runs to completion before any app containers start, used for setup tasks.

Executed sequentially; each must finish successfully before the next begins.

Typical uses: fetch configuration files, run database migrations, wait for dependent services.

Restart policy is Never; if an init container fails, kubelet retries until it succeeds (or the Pod’s restart policy disallows it).

Defined in the Pod spec under initContainers: (separate from the regular containers: list).

🔎 Selectors & Labels

Labels – User‑defined key/value pairs attached to objects (e.g., app: myapp, type: frontend).

Selectors – Query mechanism that matches objects based on label criteria.

Example: Node selector

apiVersion: v1
kind: Pod
metadata:
name: fast-pod
spec:
nodeSelector:
type: super-fast
containers:

- name: app
  image: myapp

The scheduler places fast-pod only on Nodes that have the label type=super-fast.

Conceptually similar to SELECT \* FROM nodes WHERE type='super-fast'.

🎯 Selector Concept & Service Linking

🔖 Labels & Selectors

Label – a key/value pair attached to Kubernetes objects
Selector – a query that matches objects based on their labels

Pod definition (myapp.yaml) includes two labels:

app: my-app

type: front-n

Service definition (myservice.yaml) uses the same selector:

selector:
app: my-app
type: front-n

Both objects must share identical label values for the selector to bind the Service to the Pod.

🧪 Testing the Selector

Deploy the Pod:

kubectl apply -f myapp.yaml

Deploy the Service:

kubectl apply -f myservice.yaml

Verify the Pod IP:

kubectl get pod -o wide

Check the Service’s endpoints (the Service’s Endpoint object lists the Pod IPs it routes to):

kubectl get ep myservice

Port‑forward to the Service and browse http://localhost:8080 to confirm traffic reaches the NGINX container.

⚠️ Mismatched Labels Break the Link

Editing myapp.yaml to change app: my-app → app: my-app2 and re‑applying the Pod removes the endpoint entry (ENDPOINTS column shows none).

Port‑forward fails because the Service no longer selects any Pod.

Takeaway: Both the Service selector and the Pod labels must match exactly.

🐳 Multi‑Container Pods

📐 Common Patterns

Pattern

Role of the helper container

Sidecar

Provides auxiliary functions (e.g., log shipping) without modifying the main app code.

Adapter

Transforms complex output from the main container into a format understood by external services.

Ambassador

Acts as a proxy; the main app sends data to it, and the ambassador forwards the data to a specific backend (e.g., a NoSQL store).

These patterns keep the primary application code clean and portable across cloud providers.

🛠️ Defining a Multi‑Container Pod (YAML)

apiVersion: v1
kind: Pod
metadata:
name: two-containers
spec:
containers:

- name: my-nginx
  image: nginx
  ports:
  - containerPort: 80
- name: my-busybox
  image: busybox
  command: ["sleep","3600"] # keep the container alive for an hour
  ports:
  - containerPort: 81

📋 Common Commands

Command

Purpose

kubectl create -f two-containers.yaml

Create the multi‑container Pod.

kubectl get pod two-containers -o wide

Show status and the single IP shared by both containers.

kubectl describe pod two-containers

Detailed view, including events for each container.

kubectl exec -it two-containers -c my-busybox -- /bin/sh

Open a shell inside the busybox container (note the -c flag to select the container).

kubectl logs two-containers -c my-nginx

Stream logs from the nginx container.

Why localhost works inside the busybox container: both containers share the same network namespace; busybox can reach nginx on localhost:80 without extra configuration.

🌐 Kubernetes Networking Basics

Flat Network – Every Pod receives its own IP address and can address any other Pod directly.

Pod IP – Ephemeral, assigned per Pod; shared by all containers in that Pod.

Service IP – Stable, virtual IP that load‑balances across selected Pods.

Communication Rules

Source → Destination

Path

Container ↔ Container in the same Pod

localhost:<port> (shared network namespace)

Pod ↔ Pod (different Pods)

Direct Pod IP (no localhost)

External client → Service

Through the Service IP (or a cloud load balancer for external traffic)

Node ↔ Pod

Node can reach any Pod IP directly

Services provide a persistent endpoint, while Pods are recreated with new IPs on failure.

📦 Workload Overview

Workload – An application component that runs on Kubernetes; every workload ultimately creates one or more Pods.

Workload Type

Primary Function

Pod

Atomic unit; single or multi‑container.

ReplicaSet

Guarantees a specified number of Pod replicas (self‑healing).

Deployment

Manages a ReplicaSet; adds rolling updates and rollbacks.

DaemonSet

Ensures one Pod runs on every node (or a subset).

StatefulSet

Provides stable network IDs and ordered startup for stateful apps.

Job / CronJob

Executes short‑lived tasks to completion (one‑off or scheduled).

📈 ReplicaSets

Purpose

Maintains the desired replica count.

Replaces failed Pods automatically.

YAML Skeleton

apiVersion: apps/v1
kind: ReplicaSet
metadata:
name: rs-example
spec:
replicas: 3
selector:
matchLabels:
app: nginx
template: # Pod template (same structure as a standalone Pod)
metadata:
labels:
app: nginx
spec:
containers: - name: nginx
image: nginx:alpine
ports: - containerPort: 80

Cheat Sheet

Command

Description

kubectl apply -f rs.yaml

Create or update the ReplicaSet.

kubectl get rs

List ReplicaSets.

kubectl describe rs <name>

Detailed status and events.

kubectl delete rs <name>

Remove the ReplicaSet (and its Pods).

🚀 Deployments

Advantages Over ReplicaSets

Rolling updates – replace Pods gradually.

Rollback – revert to a previous revision.

Handles the underlying ReplicaSet automatically.

YAML Highlights

apiVersion: apps/v1
kind: Deployment
metadata:
name: deploy-example
spec:
replicas: 3
revisionHistoryLimit: 3
strategy:
type: RollingUpdate # default; can be Recreate
selector:
matchLabels:
app: nginx
template:
metadata:
labels:
app: nginx
spec:
containers: - name: nginx
image: nginx:alpine
ports: - containerPort: 80

Cheat Sheet

Command

Description

kubectl create deployment <name> --image=nginx --replicas=3

Imperative creation.

kubectl apply -f deploy.yaml

Declarative creation/update.

kubectl get deploy

List Deployments.

kubectl describe deploy <name>

Show rollout status, strategy, events.

kubectl delete deploy <name>

Delete the Deployment (and its ReplicaSet).

🖥️ DaemonSets

Role

Guarantees one Pod per node (useful for log collection, node monitoring, etc.).

Example YAML (with toleration to avoid the master node)

apiVersion: apps/v1
kind: DaemonSet
metadata:
name: daemonset-example
spec:
selector:
matchLabels:
app: busybox
template:
metadata:
labels:
app: busybox
spec:
tolerations: - key: node-role.kubernetes.io/master
operator: Exists
effect: NoSchedule
containers: - name: busybox
image: busybox
command: ["sleep","3600"]

Cheat Sheet

Command

Description

kubectl apply -f ds.yaml

Create the DaemonSet.

kubectl get ds

List DaemonSets.

kubectl describe ds <name>

View pod distribution across nodes.

kubectl delete ds <name>

Remove the DaemonSet (pods are deleted automatically).

📊 StatefulSets

Core Concepts

Provides stable, sticky identities (<statefulset-name>-<ordinal>) for each Pod.

Pods are created sequentially and terminated in reverse order.

Requires a headless Service (clusterIP: None) to enable direct DNS resolution of each Pod’s name.

Typical YAML (headless Service + StatefulSet)

apiVersion: v1
kind: Service
metadata:
name: mysql-headless
spec:
clusterIP: None
selector:
app: mysql

---

apiVersion: apps/v1
kind: StatefulSet
metadata:
name: mysql
spec:
serviceName: mysql-headless # links to the headless Service
replicas: 3
selector:
matchLabels:
app: mysql
template:
metadata:
labels:
app: mysql
spec:
containers: - name: mysql
image: mysql:5.7
ports: - containerPort: 3306
volumeMounts: - name: mysql-pvc
mountPath: /var/lib/mysql
volumeClaimTemplates:

- metadata:
  name: mysql-pvc
  spec:
  accessModes: ["ReadWriteOnce"]
  resources:
  requests:
  storage: 10Gi

Each Pod can be reached via DNS mysql-0.mysql-headless, mysql-1.mysql-headless, etc.

Persistent Volume Claims (PVCs) created by the volumeClaimTemplates are not deleted when the StatefulSet is removed; manual cleanup is required.

Cheat Sheet

Command

Description

kubectl apply -f sts.yaml

Create the headless Service and StatefulSet.

kubectl get sts

List StatefulSets.

kubectl describe sts <name>

Show ordering, PVC templates, events.

kubectl delete sts <name>

Delete the StatefulSet (PVCs persist).

kubectl delete pvc -l app=mysql

Remove associated PVCs when no longer needed.

📦 StatefulSets Deep Dive

StatefulSet – A workload API object that manages stateful applications, providing each pod with a stable, unique network identity and persistent storage.

Creation & Observation

Apply the manifest:

kubectl apply -f statefulset.yaml

List pods (ordered creation):

kubectl get pods -w

First pod (‑0) appears, then ‑1, then ‑2; ages confirm sequential startup.

Deletion follows reverse order (‑2 → ‑1 → ‑0).

Pod‑to‑PVC Mapping

Each replica gets its own PersistentVolumeClaim (PVC) named after the pod (e.g., scs-0, scs-1, scs-2).

Verify with:

kubectl get pvc

Detailed view of a specific PVC:

kubectl describe pvc scs-2

Persistence Test

Open a shell in replica 2:

kubectl exec -it scs-2 -- /bin/sh

Create a file in the mounted volume (/var/www):

echo "Hello from scs-2" > /var/www/hello.txt

Replace the default nginx page:

cd /usr/share/nginx/html
cat > index.html <<EOF
Hello from replica 2
EOF

Exit the shell, delete the pod:

kubectl delete pod scs-2

The pod is recreated automatically; the previously created hello.txt and modified index.html remain, confirming storage persistence.

Access Across Replicas

From another replica (e.g., scs-0), reach the web page served by scs-2 using the DNS name:

http://scs-2.myservice:80

myservice is the headless Service name that enables intra‑cluster DNS.

Cleanup

Delete the StatefulSet (does not remove PVCs):

kubectl delete -f statefulset.yaml

Manually remove the orphaned PVCs:

kubectl delete pvc -l app=nginx

🏃 Jobs & CronJobs

Jobs Overview

Job – A controller that creates one or more pods to perform a finite task and tracks successful completions.

Key fields: parallelism, completions, activeDeadlineSeconds, restartPolicy: Never.

Pods are not kept alive after the job finishes; they remain for log inspection.

Commands Cheat Sheet

Command

Purpose

kubectl create job <name> --image=busybox -- /bin/sh -c "echo hello"

Imperative job creation

kubectl apply -f job.yaml

Declarative creation

kubectl get jobs

List jobs

kubectl describe job <name>

Detailed status, completions

kubectl logs job/<name>

View pod logs

kubectl delete job <name>

Remove job (pods stay until GC)

kubectl delete -f job.yaml

Delete via manifest

Lab Example (one‑off job)

apiVersion: batch/v1
kind: Job
metadata:
name: hello
spec:
template:
spec:
restartPolicy: Never
containers: - name: busybox
image: busybox
command: ["sh", "-c", "echo hello"]

kubectl apply -f hello-job.yaml
kubectl get jobs # shows completions=1, duration≈2s
kubectl describe job hello
kubectl logs job/hello
kubectl delete -f hello-job.yaml

CronJobs Overview

CronJob – A scheduled Job that runs periodically according to a cron‑style expression (UTC).

History limits: successfulJobsHistoryLimit (default 3) and failedJobsHistoryLimit.

Set suspend: true to pause scheduling.

Commands Cheat Sheet

Command

Purpose

kubectl create cronjob <name> --image=busybox --schedule="_/1 _ \* \* \*" -- /bin/sh -c "echo tick"

Imperative CronJob

kubectl apply -f cronjob.yaml

Declarative creation

kubectl get cronjobs (or cj)

List scheduled jobs

kubectl describe cronjob <name>

View schedule, history limits

kubectl delete cronjob <name>

Remove schedule

kubectl delete -f cronjob.yaml

Delete via manifest

Lab Example (run every minute)

apiVersion: batch/v1
kind: CronJob
metadata:
name: hello-cron
spec:
schedule: "_/1 _ \* \* \*" # every minute
jobTemplate:
spec:
template:
spec:
restartPolicy: Never
containers: - name: busybox
image: busybox
command: ["sh", "-c", "echo hello from cron"]

kubectl apply -f hello-cron.yaml
kubectl get cronjobs
kubectl describe cronjob hello-cron

# after a few minutes:

kubectl get jobs # shows recent job pods
kubectl logs job/hello-cron-<timestamp>
kubectl delete -f hello-cron.yaml

🔄 Rolling Updates

RollingUpdate – A deployment strategy that updates pods incrementally, maintaining service availability.

Strategies Compared

Strategy

Behavior

Recreate

Terminates all existing pods, then creates new ones; brief downtime possible.

RollingUpdate (default)

Replaces pods gradually; configurable maxSurge and maxUnavailable.

maxSurge – how many extra pods may be created beyond the desired replica count.

maxUnavailable – how many pods may be down during the update.

If omitted, both default to 25 % of the replica count.

Deployment Manifest (rolling update)

apiVersion: apps/v1
kind: Deployment
metadata:
name: lo-app
spec:
replicas: 3
strategy:
type: RollingUpdate
rollingUpdate:
maxSurge: 1
maxUnavailable: 1
selector:
matchLabels:
app: lo
template:
metadata:
labels:
app: lo
spec:
containers: - name: lo
image: lo-app:1.0

Update Workflow

Modify the image tag to lo-app:2.0 and re‑apply:

kubectl apply -f lo-app.yaml

Monitor progress:

kubectl rollout status deployment lo-app

View history:

kubectl rollout history deployment lo-app

Roll back to previous revision (default previous):

kubectl rollout undo deployment lo-app

To target a specific revision, add --to-revision=<n>.

Observations from Lab

New pods (version 2) appear with a green status while old pods (version 1) terminate.

Two ReplicaSets coexist during the transition; the older set is retained according to revisionHistoryLimit (default 10).

🎨 Blue/Green Deployments

Blue/Green – A release pattern that runs two complete sets of an application (blue = current, green = new) and switches traffic by updating a Service selector.

Mechanics

Deploy v1 (blue) with label app=lov1.

Deploy v2 (green) with label app=lov2.

Service initially selects app=lov1.

apiVersion: v1
kind: Service
metadata:
name: hello-svc
spec:
selector:
app: lov1
ports:

- port: 8080
  targetPort: 80

When ready, modify the Service selector to app: lov2 and apply. Traffic now flows to the green version.

Pros & Cons

Advantage

Drawback

Zero‑downtime switch (if both versions are compatible)

Requires duplicate resources; double the cluster footprint

Simple rollback (re‑switch selector)

Potential schema/database incompatibilities still need handling

Lab Steps

Deploy hello-v1.yaml (label lov1).

Deploy hello-v2.yaml (label lov2).

Deploy hello-svc.yaml pointing at lov1.

Verify version 1 via port‑forward (kubectl port-forward svc/hello-svc 8080:8080).

Update Service selector to lov2 (kubectl apply -f hello-svc.yaml).

Verify version 2 appears.

Clean up all three resources with a single kubectl delete -f <file> command.

🌐 Services Recap

Why Services Exist

Pods have ephemeral IPs; when a pod dies its IP disappears, breaking direct pod‑to‑pod communication. Services provide stable IP/DNS and optional load‑balancing.

Service Types

Type

Visibility

Port Range

Typical Use

ClusterIP (default)

Internal to the cluster

N/A

In‑cluster communication

NodePort

Internal + External (node IP)

30000‑32767

Simple external exposure without a cloud load balancer

LoadBalancer

External (cloud provider LB)

N/A

Production‑grade external access (cloud only)

ClusterIP Details

ClusterIP – A virtual IP reachable only within the cluster; routes traffic to selected pods.

port: Service’s listening port.

targetPort: Port on the pod containers.

apiVersion: v1
kind: Service
metadata:
name: example-svc
spec:
selector:
app: example
env: prod
ports:

- port: 80 # service port
  targetPort: 8080 # container port

Imperative Commands

kubectl expose pod <pod-name> --port=80 --target-port=8080 --name=example-svc
kubectl expose deployment <deploy-name> --port=80 --target-port=8080

kubectl get svc -o wide
kubectl describe svc example-svc
kubectl delete svc example-svc

NodePort Details

NodePort

🌐 NodePort Services

NodePort – A Service type that exposes a pod on each node’s IP at a static port (default range 30000‑32767), allowing external traffic to reach the cluster.

Kubernetes does not let you set the node‑port number directly in the Service spec; if omitted, the system picks a random port in the allowed range.

Imperative exposure of an existing deployment:

kubectl expose deployment <deployment-name> \
 --port <service-port> \
 --target-port <container-port> \
 --type NodePort \
 --name <service-name>

Declarative YAML (example nodeport.yaml):

apiVersion: v1
kind: Service
metadata:
name: nginx-nodeport
spec:
selector:
app: nginx
type: NodePort
ports: - port: 80 # Service port
targetPort: 80 # Pod container port
nodePort: 32410 # Explicitly request this port (must be 30000‑32767)

Apply the manifest: kubectl apply -f nodeport.yaml

List services with additional columns: kubectl get svc -o wide

Detailed view: kubectl describe svc nginx-nodeport

Delete via file: kubectl delete -f nodeport.yaml or by name: kubectl delete svc nginx-nodeport

Accessing the service

On Docker Desktop, the node’s IP maps to localhost; use http://localhost:<nodePort> (e.g., http://localhost:32410).

On a cloud provider, retrieve the node’s external IP: kubectl get nodes -o wide → use <external‑IP>:<nodePort>.

📊 Service Concepts Recap

Service – A stable abstraction that provides a durable IP address and DNS name, routing traffic to Pods selected by label selectors.

Problem solved: Pods have ephemeral IPs; when a pod is recreated its IP changes, breaking direct pod‑to‑pod calls.

Selectors match Pods by label key/value pairs (e.g., zone: prod and version: v1). Only Pods whose labels satisfy all selector criteria are included.

Service types

Type

Visibility

Typical Use

ClusterIP (default)

Internal to the cluster

In‑cluster communication

NodePort

Internal + External (node IP)

Simple external exposure

LoadBalancer

External (cloud LB)

Production‑grade external access

Ingress

External (HTTP/HTTPS routing)

Layer 7 traffic management

ClusterIP offers a virtual IP reachable only inside the cluster.

NodePort exposes the service on each node’s IP at a static port.

LoadBalancer provisions a cloud provider’s L4 (TCP/UDP) load balancer.

Ingress operates at L7 (HTTP/HTTPS), allowing path‑based or host‑based routing rules.

🔀 LoadBalancer Services

LoadBalancer – A Service type that creates a cloud provider’s external load balancer (L4) and assigns a public IP.

Docker Desktop emulates a LoadBalancer locally, enabling testing without a cloud provider.

Declarative YAML (example lb.yaml):

apiVersion: v1
kind: Service
metadata:
name: nginx-lb
spec:
selector:
app: nginx
type: LoadBalancer
ports: - port: 8080 # Service port exposed externally
targetPort: 80 # Container port

Deploy: kubectl apply -f lb.yaml

Retrieve the external IP (or local placeholder) with: kubectl get svc -o wide → column EXTERNAL‑IP.

Test: open a browser to http://localhost:8080 (Docker Desktop) or to the cloud LB’s public IP.

Cleanup: kubectl delete -f lb.yaml.

📁 Persistent Storage Overview

Persistence – Mechanism that decouples data from the container lifecycle, ensuring data survives pod termination.

Containers are ephemeral; any data written inside disappears when the container stops.

Volumes let containers mount external storage (cloud disks, network filesystems, host directories).

Two provisioning models:

Model

Description

Static

Administrator pre‑creates a PersistentVolume (PV); users claim it with a PersistentVolumeClaim (PVC).

Dynamic

Users request storage via a StorageClass; the cluster automatically creates a PV to satisfy the PVC.

📦 Static Provisioning: PV & PVC

Core Objects

PersistentVolume (PV) – Cluster‑wide storage resource defined by the admin (capacity, access mode, reclaim policy).

PersistentVolumeClaim (PVC) – User request for storage that binds to a suitable PV.

Typical Workflow

Select a cloud provider’s storage plugin (e.g., hostPath for local testing).

Create a PV with desired capacity and access mode.

Create a PVC that matches the PV’s accessModes and requests a size ≤ PV capacity.

Reference the PVC in a Pod’s volumes section; mount it via volumeMounts.

YAML Examples

PersistentVolume (pv.yaml)

apiVersion: v1
kind: PersistentVolume
metadata:
name: pv001
spec:
capacity:
storage: 10Mi
accessModes: - ReadWriteOnce
persistentVolumeReclaimPolicy: Retain # keep data after PVC deletion
hostPath:
path: /data/pv001

PersistentVolumeClaim (pvc.yaml)

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: myclaim
spec:
accessModes: - ReadWriteOnce
resources:
requests:
storage: 10Mi

Pod using the PVC (busybox-pod.yaml)

apiVersion: v1
kind: Pod
metadata:
name: busybox-pv
spec:
containers: - name: busybox
image: busybox
command: ["sleep","3600"]
volumeMounts: - name: mydata
mountPath: /demo
volumes: - name: mydata
persistentVolumeClaim:
claimName: myclaim

Important Properties

ReclaimPolicy

Delete (default): PV is removed when the PVC is deleted → data lost.

Retain: PV remains, preserving data; manual cleanup required.

Access Modes

ReadWriteOnce – mounted read‑write by a single node (others read‑only).

ReadOnlyMany – read‑only by many nodes.

ReadWriteMany – read‑write by many nodes.

PV & PVC Lifecycle States

State

Meaning

Available

PV exists, not bound to any PVC.

Bound

PV is linked to a PVC; in use.

Released

PVC deleted, PV still holds data, awaiting reclamation.

Failed

Provisioning or binding error.

Cheat Sheet (commands)

Action

Command

Create PV/PVC from YAML

kubectl apply -f <file>.yaml

List PVs

kubectl get pv

List PVCs

kubectl get pvc

Describe PV

kubectl describe pv <pv-name>

Describe PVC

kubectl describe pvc <pvc-name>

Delete by file

kubectl delete -f <file>.yaml

Delete by name

kubectl delete pv <pv-name> / kubectl delete pvc <pvc-name>

⚡ Dynamic Provisioning: StorageClass

StorageClass – A cluster‑wide description of a storage provisioner (cloud driver) that enables on‑demand PV creation when a PVC references it.

No need to pre‑allocate capacity; multiple PVCs can be satisfied from the same class.

The provisioner field specifies the driver (e.g., kubernetes.io/aws-ebs, kubernetes.io/gce-pd).

Additional parameters tune the underlying storage (e.g., type: gp2).

YAML Example (sc.yaml)

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
type: gp2
fsType: ext4
reclaimPolicy: Delete # default; change to Retain if needed

PVC Using a StorageClass

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: dynamic-claim
spec:
storageClassName: fast-ssd
accessModes: - ReadWriteOnce
resources:
requests:
storage: 5Gi

The PVC triggers the provisioner to create a PV matching the request.

Benefits vs. Static PV

Aspect

Static PV

Dynamic (StorageClass)

Admin effort

Pre‑create PVs

No manual PV creation

Capacity utilization

Fixed; unused space may be wasted

Allocated per claim

Scalability

Limited by pre‑provisioned PVs

Unlimited (subject to provider limits)

Flexibility

Requires admin to match sizes & access modes

PVC defines size & mode at request time

Dynamic Provisioning Cheat Sheet

Action

Command

Create StorageClass

kubectl apply -f sc.yaml

List StorageClasses

kubectl get sc

Describe StorageClass

kubectl describe sc <name>

Delete StorageClass

kubectl delete sc <name>

Create PVC referencing class

kubectl apply -f pvc-dynamic.yaml

Inspect bound PV

kubectl get pv (shows CLAIM column)

🗂️ ConfigMaps

ConfigMap – A key‑value store for non‑confidential configuration data, decoupled from pod specifications.

Creation methods

Imperative: kubectl create configmap <name> --from-literal=key1=value1 --from-literal=key2=value2

From file/folder: kubectl create configmap <name> --from-file=path/

Declarative YAML.

YAML Example (cm-example.yaml)

apiVersion: v1
kind: ConfigMap
metadata:
name: cm-example
data:
state: Michigan
city: Detroit

Using ConfigMap in a Pod

Environment variable injection

env:

- name: CITY
  valueFrom:
  configMapKeyRef:
  name: cm-example
  key: city

Volume mount (updates reflected automatically)

volumes:

- name: config-volume
  configMap:
  name: cm-example

Mount point in container: mountPath: /etc/config → each key appears as a file.

🔐 Secrets – Accessing & Managing Sensitive Data

Secret – a Kubernetes object that stores small amounts of confidential data (e.g., passwords, tokens) in base‑64‑encoded form.

Viewing a secret

kubectl get secret <name> -o yaml → outputs the full manifest, including the decoded data values.

kubectl describe secret <name> only shows the keys; the actual values remain hidden.

Typical workflow

Create a secret (e.g., kubectl create secret generic my‑creds --from-literal=username=admin --from-literal=password=secret).

Deploy a pod (e.g., a BusyBox) that reads the secret as environment variables.

Inside the pod, echo PASSWORD display the decoded values.

Delete the secret and the pod when finished.

kubectl get secret my‑creds -o yaml
kubectl describe secret my‑creds
kubectl delete secret my‑creds
kubectl delete pod busybox‑demo

📊 Probes – Monitoring Container Health

Probe – a mechanism that lets Kubernetes periodically check the state of a container and act accordingly.

Probe type

Primary purpose

Typical use

Startup

Determines when the container has finished its initialization phase.

Prevents traffic before the app is ready.

Readiness

Indicates whether the container is ready to accept traffic.

Stops routing to a pod that is temporarily unavailable.

Liveness

Detects if the container is still alive; a failure triggers a restart.

Recovers from deadlocks or unrecoverable errors.

Probe configuration basics

livenessProbe:
exec:
command: ["cat", "/tmp/healthy"]
initialDelaySeconds: 15
periodSeconds: 20
failureThreshold: 3

initialDelaySeconds – wait time before the first probe.

periodSeconds – interval between subsequent probes.

failureThreshold – number of consecutive failures before the action is taken.

Action types

Action

How it works

exec

Runs a command inside the container (e.g., cat /tmp/healthy).

tcpSocket

Attempts to open a TCP connection on the specified port.

httpGet

Performs an HTTP GET request to the defined path and port.

Interaction between Readiness & Liveness

Readiness failure → pod remains Running but is removed from service load‑balancing.

Liveness failure → pod is restarted by the kubelet.

🧪 Lab – Liveness Probe in Action

Pod manifest defines a liveness probe that runs cat /tmp/healthy.

Container startup script creates /tmp/healthy, sleeps 15 s, then deletes the file.

Probe settings: initialDelaySeconds: 5, periodSeconds: 5, failureThreshold: 2.

livenessProbe:
exec:
command: ["cat", "/tmp/healthy"]
initialDelaySeconds: 5
periodSeconds: 5
failureThreshold: 2

Observed behavior

After the file is removed, the probe fails twice → kubelet kills the pod.

A new pod is created automatically (as part of the deployment), and the cycle repeats.

Cleanup:

kubectl delete pod liveness-example

🖥️ Observability Dashboards

Dashboard

Deployment mode

Key features

Kubernetes Dashboard

In‑cluster add‑on (web UI)

Resource browsing, inline YAML edit, limited exposure → potential attack surface.

Lens

Local desktop IDE (Mac/Windows/Linux)

Cluster overview, built‑in terminal, live editing, multi‑cluster support.

K9s

Terminal‑based UI (cross‑platform)

Ultra‑light, fast navigation, log view, port‑forward, resource actions via keyboard shortcuts.

Quick usage tips

Kubernetes Dashboard – install via kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml. Access through kubectl proxy and navigate to http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/.

Lens – download from https://k8slens.dev, add clusters by pointing to ~/.kube/config. Use the Workloads → Pods view to see replica sets, edit manifests, view logs, and open a terminal directly from the UI.

K9s – install via package manager (brew install k9s, choco install k9s, etc.). Launch with k9s.

:svc → list services.

s on a selected pod → open a shell.

f → port‑forward.

Ctrl‑d → delete the selected resource.

All three tools provide the same information you would retrieve with kubectl commands, but with a visual or interactive interface.

📈 Horizontal Pod Autoscaler (HPA) – Automatic Scaling

HPA – a controller that adjusts the number of pod replicas for a Deployment (or other scalable resource) based on observed CPU/memory usage or custom metrics.

Prerequisites

Metrics Server must be deployed in the kube-system namespace.

Pods must define resource requests (and optionally limits) for CPU/memory.

resources:
requests:
cpu: "100m"
memory: "64Mi"
limits:
cpu: "200m"
memory: "128Mi"

HPA manifest example

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
name: hpa-example
spec:
scaleTargetRef:
apiVersion: apps/v1
kind: Deployment
name: webserver
minReplicas: 1
maxReplicas: 4
metrics:

- type: Resource
  resource:
  name: cpu
  target:
  type: Utilization
  averageUtilization: 50

Scale‑up delay – default 3 min.

Scale‑down delay – default 5 min.

HPA command cheat sheet

Action

Command

Create HPA (imperative)

kubectl autoscale deployment <name> --cpu-percent=50 --min=1 --max=4

Create HPA (YAML)

kubectl apply -f hpa.yaml

View HPA status

kubectl get hpa <name>

Delete HPA

kubectl delete hpa <name>

🧪 Lab – Scaling with HPA & Metrics Server

Install Metrics Server (Docker Desktop needs the --kubelet-insecure-tls flag).

kubectl apply -f components.yaml # edit to add -–kubelet-insecure-tls

Deploy a simple web server deployment (hpa-example) with 3 replicas.

Create the HPA targeting the deployment, CPU threshold 50 %, min 1, max 4.

kubectl autoscale deployment hpa-example --cpu-percent=50 --min=1 --max=4

Launch a BusyBox pod that continuously curls the web server, generating CPU load.

Observe scaling in K9s (or kubectl get hpa) – replica count rises to 4 as average CPU exceeds 50 %.

Stop the load generator; after the scale‑down delay, replicas shrink back to 1.

Cleanup steps:

kubectl delete deployment hpa-example
kubectl delete hpa hpa-example
kubectl delete pod busybox‑loadgen
kubectl delete -f components.yaml # removes Metrics Server

🛠️ Summary of Key Commands Mentioned

Category

Command

Secrets – view full data

kubectl get secret <name> -o yaml

Secrets – describe keys only

kubectl describe secret <name>

Probes – apply pod manifest

kubectl apply -f pod‑with‑probes.yaml

Dashboard – start Lens terminal

Lens UI button → “Terminal”

K9s – open shell in pod

s (select pod) → Enter

HPA – create via CLI

kubectl autoscale deployment <name> --cpu-percent=50 --min=1 --max=4

Metrics Server – deploy

kubectl apply -f components.yaml (with insecure‑tls flag)

General – delete resource

kubectl delete <type> <name>

These notes capture secret handling, container health probes, observability dashboards, and automatic scaling—all essential for building robust, production‑grade Kubernetes workloads.
