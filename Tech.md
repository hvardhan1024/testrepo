# Docker & Kubernetes - Exam Quick Revision Notes

---

## **Teq. 20: Injecting Files into the Image using ADD**

**What it does:** Adds compressed files (tar.gz) from host to container and auto-extracts them.  
**Why use it:** Quick way to copy multiple files/directories in one layer without manual extraction.  
**Key Point:** ADD auto-decompresses `.tar.gz` files; use COPY for simple file transfers.

**Step 1: Create tar file**
```bash
tar -czvf firstfile.tar.gz firstfile
# -c: create, -z: gzip compress, -v: verbose, -f: filename
```

**Step 2: Dockerfile**
```dockerfile
FROM ubuntu:latest
RUN apt-get -y update  # Update package list
ADD dummy.tar.gz .     # Add and auto-extract tar.gz to current dir
```

**Step 3: Build image**
```bash
sudo docker build -t sample-image .
```

**Step 4: List images**
```bash
sudo docker images
```

**Step 5: Run container**
```bash
sudo docker run -it sample-image bash
```

**Step 6: Verify extraction**
```bash
ls  # Check if files are extracted
```

---

## **Teq. 21: Rebuilding without the Cache**

**What it does:** Forces Docker to rebuild all layers from scratch, ignoring cached layers.  
**Why use it:** When you need fresh builds (e.g., updated packages, testing, avoiding stale cache).  
**Key Point:** Slower but ensures no old cached data affects the build.

```bash
sudo docker build --no-cache .
# Rebuilds every layer without using cache
```

---

## **Teq. 22: Busting the Cache**

**What it does:** Compares build times with and without cache to understand cache impact.  
**Why use it:** To analyze build performance and decide when to bust cache.  
**Key Point:** Cache speeds up builds but may cause issues with outdated dependencies.

---

## **Teq. 23: Intelligent Cache-Busting using Build-Args**

**What it does:** Uses ARG with dynamic values (like timestamp) to invalidate cache at specific point.  
**Why use it:** Selectively rebuild from a certain layer onward without `--no-cache`.  
**Key Point:** All layers after ARG line get rebuilt; layers before remain cached.

**Dockerfile:**
```dockerfile
FROM debian
LABEL author "Deepika K deepikak@rvce.edu.in"
LABEL description "This is a demo file to show build-args"
RUN apt-get update && apt-get -y upgrade

ARG CACHEBUST=1  # Cache invalidation point
RUN echo ["Welcome"]  # This and below layers rebuild
```

**Build with cache bust:**
```bash
sudo docker build -t test:2.0 --build-arg CACHEBUST=$(date +%s) .
# $(date +%s) gives current timestamp, changes every second
```

---

## **Teq. 24: Intelligent Cache-Busting using ADD directive**

**What it does:** ADD with remote URL invalidates cache when remote file changes.  
**Why use it:** Auto-rebuild when external resources (repos, files) update.  
**Key Point:** Docker checks remote file's checksum; if changed, cache busts from that layer.

**Dockerfile:**
```dockerfile
FROM debian
LABEL author "Deepika K deepikak@rvce.edu.in"
LABEL description "This is a demo file to show ADD-directive"

RUN apt-get update
RUN echo ["Welcome"]

ADD <Online_File_Link> /location  # Cache busts if remote file changes
```

**Build:**
```bash
sudo docker build -t repo-image:2.3 .
```

**View images:**
```bash
sudo docker images
```

---

## **Teq. 25: Setting the Right Time Zone in the Containers**

**What it does:** Configures container timezone via ENV variables to match your region.  
**Why use it:** Ensures logs, timestamps, and apps use correct local time.  
**Key Point:** `DEBIAN_FRONTEND=noninteractive` prevents timezone prompts during builds.

**Dockerfile:**
```dockerfile
FROM ubuntu:latest
ENV TZ=Europe/London              # Set timezone
ENV DEBIAN_FRONTEND=noninteractive  # Avoid interactive prompts
RUN apt-get update && apt-get install -y tzdata  # Install timezone data
```

**Build:**
```bash
sudo docker build -t sample:1.0 .
```

**Run with timezone:**
```bash
sudo docker run -e TZ=Europe/London -it sample:1.0
```

**Check date:**
```bash
date  # Inside container
```

**Exit:**
```bash
exit
```

---

## **Teq. 29: Running GUIs within Docker**

**What it does:** Runs graphical applications (like file managers) inside containers via X11 forwarding.  
**Why use it:** Test GUI apps in isolated environments without full VM overhead.  
**Key Point:** Must share X11 socket and set DISPLAY variable for GUI rendering.

**Give permissions:**
```bash
xhost +local:docker  # Allow Docker to access X11 display
```

**Dockerfile:**
```dockerfile
FROM ubuntu:23.10
ENV DEBIAN_FRONTEND=noninteractive  # No interactive prompts
RUN apt-get update
RUN apt-get install nautilus -y    # Install file manager
CMD ["nautilus"]                   # Run nautilus on startup
```

**Build:**
```bash
sudo docker build -t nautilus-ubuntu .
```

**Run GUI container:**
```bash
sudo docker run -it --rm \
  -e DISPLAY=$DISPLAY \                      # Pass display variable
  -v /tmp/.X11-unix:/tmp/.X11-unix \        # Mount X11 socket
  nautilus-ubuntu:latest
```

---

## **Teq. 30: Inspecting Containers**

**What it does:** Shows detailed JSON metadata about container (network, mounts, config, size).  
**Why use it:** Debug issues, verify settings, check resource usage.  
**Key Point:** Use `--size` flag to see disk usage of container.

**Step 1: Run container**
```bash
sudo docker run -it <image-name:version>
```

**Step 2: Get Container ID**
```bash
sudo docker container ls -a  # List all containers
```

**Step 3: Inspect container**
```bash
sudo docker inspect --size <Container-ID>
# Shows full config + size on disk
```

---

## **Teq. 31: Cleanly Killing Containers**

**What it does:** Forcefully stops a running container by sending SIGKILL signal.  
**Why use it:** When container doesn't respond to `docker stop` (graceful shutdown).  
**Key Point:** `kill` is immediate; `stop` waits for graceful shutdown (10s timeout).

**Step 1: Run container**
```bash
sudo docker run -it <image-name:version>
```

**Step 2: Get Container ID**
```bash
sudo docker container ls -a
```

**Step 3: Kill container**
```bash
sudo docker kill <Container-ID>  # Immediate termination
```

==========================

## **Teq. 32: Using Docker Machine to Provision Docker Hosts**

**What it does:** Docker Machine creates and manages Docker hosts on VMs (VirtualBox, cloud providers).  
**Why use it:** Automate Docker host setup on multiple machines; manage remote Docker environments.  
**Key Point:** Creates VM with Docker pre-installed; useful for testing multi-host setups locally.

**Step 1: Install Docker Machine**
```bash
# Download from: https://github.com/docker/machine/releases
# Install binary to /usr/local/bin/
```

**Step 2: Check version**
```bash
sudo docker-machine --version  # Verify installation
```

**Step 3: List Docker machines**
```bash
sudo docker-machine ls  # Shows all managed Docker hosts (empty initially)
```

**Step 4: Create Docker VM**
```bash
sudo docker-machine create <vm-name>
# Creates VM with Docker engine installed (uses VirtualBox by default)
```

**Step 5: Verify in VirtualBox**
```
Open VirtualBox GUI → Check if VM is created and running
```

**Step 6: List machines again**
```bash
sudo docker-machine ls  # Shows new VM with IP, state, Docker version
```

==========================

## **Teq. 33: Wildcard DNS**

**What it does:** Uses DNS `dig` command to query wildcard DNS records for a domain.  
**Why use it:** Test DNS configuration; verify if domain supports wildcard subdomains (*.domain.com).  
**Key Point:** Wildcard DNS allows any subdomain to resolve; useful for dynamic subdomain routing.

**Query wildcard DNS:**
```bash
dig a *.rvce.edu.in
# Queries DNS for wildcard A record (any subdomain under rvce.edu.in)
```

**Sample Output Explanation:**
```
;; HEADER: status: NXDOMAIN = No such domain (wildcard not configured)
;; QUESTION: *.rvce.edu.in → What we're asking
;; AUTHORITY SECTION: 
   rvce.edu.in. SOA → Shows authoritative nameserver
   ns1.linode.com = Primary nameserver
   webadmin.rvce.edu.in = Admin contact
;; Query time: 3 msec → Fast response
;; SERVER: 172.16.2.34 → DNS server that answered
```

**Key DNS Record Types:**
- **A record:** Maps domain to IPv4 address
- **SOA:** Start of Authority (shows primary nameserver)
- **NXDOMAIN:** Domain doesn't exist (wildcard not set up)

**Exam Tip:** If wildcard DNS is configured, you'd see an A record with IP; NXDOMAIN means no wildcard support.

==========================

## **Teq. 61: Using the Docker Hub Workflow**

**What it does:** Automated CI/CD pipeline linking Git repos to Docker Hub for auto-builds.  
**Why use it:** Push code → Docker Hub auto-builds image → users get latest version.  
**Key Point:** Changes in Git trigger automatic Docker Hub builds.

**Workflow:**
1. Create GitHub/BitBucket repo
2. Clone repo locally
3. Add Dockerfile + code
4. Commit and push to Git
5. Create Docker Hub repository
6. Link Docker Hub repo to Git repo (in Docker Hub settings)
7. Wait for initial build
8. Push code changes to Git
9. Docker Hub auto-rebuilds image
10. Users pull updated image

---

## **Teq. 66: Running Jenkins Master within Docker Container**

**What it does:** Deploys Jenkins CI/CD server inside a Docker container.  
**Why use it:** Quick Jenkins setup without manual installation; easy to destroy/recreate.  
**Key Point:** Volume mount (`-v`) persists Jenkins data across container restarts.

**Prerequisites:**
- Docker installed
- 256 MB RAM minimum
- 1 GB disk (10 GB recommended)

**Run Jenkins:**
```bash
sudo docker run -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \  # Persist Jenkins data
  jenkins/jenkins:lts-jdk11
# Port 8080: Web UI, Port 50000: Agent communication
```

**View containers:**
```bash
sudo docker containers ls -a
```

**Access Jenkins:**
```
http://localhost:8080
```

---

## **Teq. 70: The Docker Contract - Reducing Friction**

**What it does:** Establishes standard interface between Dev and Ops teams using Docker images.  
**Why use it:** Reduces deployment conflicts ("works on my machine" syndrome).  
**Key Point:** Image = contract. Dev builds it, Ops runs it, no environment debates.

<img width="851" height="465" alt="image" src="https://github.com/user-attachments/assets/513838d9-bc0a-4bce-8d9a-1da87845b34e" />

---

## **Teq. 71: Manually Mirroring Registry Images**

**What it does:** Pull image from one registry, retag it, and push to another registry/namespace.  
**Why use it:** Create backups, move images between registries, organize images under your namespace.  
**Key Point:** Tag format: `registry/username/image:version`

**Step 1: Pull image**
```bash
sudo docker pull deepikakripanithi/ubuntu:latest
```

**Step 2: View images**
```bash
sudo docker images
```

**Step 3: Retag image**
```bash
sudo docker tag deepikakripanithi/ubuntu:latest deepikakripanithi/ubuntu2:1.0
# Creates new tag pointing to same image
```

**Step 4: Login to Docker Hub**
```bash
docker login  # Enter credentials
```

**Step 5: Push retagged image**
```bash
sudo docker push deepikakripanithi/ubuntu2:1.0
```

---

## **Teq. 72: Delivering Images Over Constrained Connections**

**What it does:** Multi-stage build creates multiple images from one Dockerfile; they communicate via bridge network.  
**Why use it:** Build different components that need to talk to each other in isolated network.  
**Key Point:** Use `--target` flag to build specific stage; containers can ping each other via IP.

**Dockerfile:**
```dockerfile
FROM alpine as img1
COPY file1.txt .  # Stage 1

FROM alpine as img2
COPY file2.txt .  # Stage 2
```

**Build images:**
```bash
sudo docker build -t image1 --target img1 .  # Build only stage img1
sudo docker build -t image2 --target img2 .  # Build only stage img2
```

**Run containers in detached mode:**
```bash
sudo docker run -it --name image1 -d image1:latest
sudo docker run -it --name image2 -d image2:latest
```

**Get IP addresses:**
```bash
sudo docker network inspect bridge  # Check IPs of both containers
```

**Ping between containers:**
```bash
sudo docker container exec -it image1 /bin/sh
# Inside image1, ping image2's IP
```

---

## **Teq. 73: Sharing Docker Objects as TAR Files**

**What it does:** Export containers as tar files, import them as images, or save/load images directly.  
**Why use it:** Transfer images/containers without registry, share via USB/email, backup purposes.  
**Key Point:** `export` loses image history; `save` preserves all layers.

**Export container:**
```bash
sudo docker export [container-name] > nginx.tar
# Flattens container to single layer
```

**Import as new image:**
```bash
sudo docker import - mynginx < nginx.tar
# Creates image from exported tar
```

**List images:**
```bash
sudo docker images
```

**Save image (preserves layers):**
```bash
sudo docker save -o mynginx1.tar nginx
# Saves complete image with history
```

**Load image:**
```bash
sudo docker load --input mynginx1.tar
# Restores image with all layers
```

---

## **Teq. 74: Informing Your Containers with etcd**

**What it does:** etcd is distributed key-value store for container configuration management.  
**Why use it:** Centralize config across multiple containers; containers read config from etcd.  
**Key Point:** Useful for microservices needing shared config without rebuilding images.

**Run etcd container:**
```bash
sudo docker run -d -p 2379:2379 -p 2380:2380 --name etcd \
  -e ALLOW_NONE_AUTHENTICATION=yes \  # Skip auth for demo
  bitnami/etcd:latest
# Port 2379: Client API, Port 2380: Peer communication
```

**Verify etcd running:**
```bash
sudo docker ps
```

**Set key-value:**
```bash
sudo docker exec etcd etcdctl put /my_name deepika
# Store key=/my_name, value=deepika
```

**Get key-value:**
```bash
sudo docker exec etcd etcdctl get /my_name
# Retrieve value for /my_name
```

---

## **2.2.1: Installing Kubernetes Client (kubectl)**

**What it does:** Installs kubectl CLI tool to interact with Kubernetes clusters.  
**Why use it:** Manage pods, services, deployments via command line.  
**Key Point:** kubectl talks to cluster API server to execute commands.

**Update and install prerequisites:**
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

**Download Kubernetes signing key:**
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

**Add Kubernetes apt repository:**
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
  https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list
```

**Install kubectl:**
```bash
sudo apt-get update
sudo apt-get install -y kubectl
```

**Display cluster info:**
```bash
kubectl cluster-info  # Shows cluster endpoint and services
```

---

## **2.2.3: Setting up Alias and Command-Line Completion for kubectl**

**What it does:** Creates shortcut `k` for `kubectl` and enables autocomplete for faster typing.  
**Why use it:** Saves time during exams/debugging with quick commands.  
**Key Point:** Autocomplete works with alias too.

**Create alias:**
```bash
alias k=kubectl  # Add to ~/.bashrc for persistence
```

**Enable autocomplete permanently:**
```bash
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

**Autocomplete with alias:**
```bash
alias k=kubectl
complete -o default -F __start_kubectl k  # Enable autocomplete for 'k'
```

---

## **2.3.1: Deploying Node.js App on Kubernetes**

**What it does:** Creates a pod running Node.js container from specified image.  
**Why use it:** Deploy apps on Kubernetes; pod is smallest deployable unit.  
**Key Point:** Pod wraps one or more containers; here we use single container.

**Start Minikube:**
```bash
minikube start  # Start local K8s cluster
```

**Deploy app:**
```bash
kubectl run kubia --image=luksa/kubia --port=8080
# Creates pod named 'kubia' with Node.js app on port 8080
```

**List pods:**
```bash
kubectl get pods  # Check pod status (Pending/Running/Error)
```

**List pods again:**
```bash
kubectl get pods  # Verify status changed to Running
```

---

## **2.3.2: Accessing the Web Application**

**What it does:** Exposes pod via LoadBalancer service to access app externally.  
**Why use it:** Pods have internal IPs only; service provides stable external access.  
**Key Point:** LoadBalancer type assigns external IP (in cloud or Minikube tunnel).

**Create pod:**
```bash
kubectl run kubia --image=luksa/kubia --port=8080
```

**Expose pod via service:**
```bash
kubectl expose pod kubia --type=LoadBalancer --name kubia-http
# Creates service 'kubia-http' that routes traffic to pod
```

**List services:**
```bash
kubectl get services  # Check if service created
```

**Check external IP:**
```bash
kubectl get svc  # Wait for EXTERNAL-IP assignment
```

**Access app:**
```bash
curl http://<EXTERNAL-IP>:8080  # Test app via browser or curl
```



<img width="784" height="352" alt="image" src="https://github.com/user-attachments/assets/c851055e-f53f-4755-a676-960ebba96639" />



---

## **2.3.5: Examining What Nodes the App is Running On**

**What it does:** Shows which cluster node hosts your pod and pod's internal IP.  
**Why use it:** Debug networking, understand pod placement, verify node distribution.  
**Key Point:** `-o wide` shows extra columns (IP, Node); `describe` gives full details.

**Display pod details:**
```bash
kubectl get pods -o wide  # Shows Pod IP and Node name
```

**Detailed pod inspection:**
```bash
kubectl describe pod <name>  # Full pod config, events, status
```

---

## **2.3.6: Introducing the Kubernetes Dashboard**

**What it does:** Launches web-based UI to manage Kubernetes cluster visually.  
**Why use it:** Easier than CLI for viewing resources, logs, and debugging.  
**Key Point:** Minikube includes built-in dashboard; production uses separate deployment.

**Access dashboard:**
```bash
minikube dashboard  # Opens dashboard in browser automatically
```

---

## **Quick Exam Tips**

1. **Dockerfile:** Remember `FROM`, `RUN`, `COPY`, `ADD`, `CMD`, `EXPOSE`, `ENV`, `ARG`
2. **Cache:** Use `--no-cache` to force rebuild; ARG for selective cache bust
3. **Networking:** Bridge network for container-to-container communication
4. **Volumes:** `-v` for persistence; data survives container deletion
5. **kubectl:** `get`, `describe`, `logs`, `exec`, `delete` are most used
6. **Services:** ClusterIP (internal), NodePort (fixed port), LoadBalancer (external)
7. **Export vs Save:** Export = container snapshot; Save = image with layers
8. **etcd:** Distributed config store; use `put`/`get` for key-values
9. **Minikube:** Local single-node cluster for testing K8s
10. **Pods:** Smallest unit in K8s; can contain multiple containers

---

**Remember:** Read the question carefully, identify the technique number, recall the command syntax, and write concise answers. Good luck! 🚀
