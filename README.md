*This project has been created as part of the 42 curriculum by oishchen*

# 🌐 Inception

A system administration and infrastructure project designed to broaden knowledge of Docker, system isolation, and service orchestration. This project deploys a secure, multi-container web architecture strictly utilizing Alpine Linux and a custom Makefile execution environment.

## 🏗️ System Architecture & Networking

This infrastructure is completely isolated from the host machine's default network. It utilizes a custom Docker bridge network named `inception` to handle all internal DNS resolution and traffic routing.

### The Virtual Routing (Traffic Flow)
1. **External Access (The Gateway):** * The **NGINX** container is the only service with a port exposed to the host machine (`443:443`). 
   * It strictly accepts encrypted TLSv1.2/1.3 traffic. Any unencrypted HTTP traffic (Port 80) is dropped at the firewall level.
2. **Internal App Routing (FastCGI):**
   * NGINX terminates the SSL connection. When a user requests a `.php` file, NGINX acts as a reverse proxy, packaging the request and forwarding it over the `inception` network to the **WordPress** container on port `9000`.
3. **Internal Data Routing (TCP):**
   * The WordPress PHP-FPM daemon executes the code. To retrieve site data, WordPress resolves the `mariadb` hostname using Docker's internal DNS and connects to the **MariaDB** container on port `3306`.
   * The database is completely sealed off from the host machine; it has no external ports exposed.

## 💾 Storage & Persistence

To ensure complete data persistence across container lifecycles, this project bypasses standard Docker named volumes and uses hybrid bind-mounts physically anchored to the host kernel:
* **Database Volume:** `/home/${USER}/data/mariadb`
* **Website Volume:** `/home/${USER}/data/wordpress`

## 🚀 Deployment Instructions

This project is fully automated via `make`. Do not use manual `docker-compose` commands to manage the lifecycle of these containers.

### Prerequisites
1. Docker and Docker Compose must be installed.
2. Your local `/etc/hosts` file must map the domain `oishchen.42.de` to `127.0.0.1`.

### System Lifecycle Commands

* **Boot the Infrastructure:**
  ```bash
  make


Force Rebuild (Phase 1 Compilation):

Bash
make build
(Forces Docker to re-evaluate the Dockerfiles and build fresh Alpine images).

Graceful Shutdown:

Bash
make down
(Safely stops the containers and preserves the virtual network).

Total Annihilation (Hard Reset):

Bash
make fclean
(WARNING: This stops all containers, prunes the Docker system, and physically deletes all database and website data from the host machine's hard drive).

##Debug:

**make debug**
Outputs a complete health check of the infrastructure, including:

Current container states and exit codes (docker ps -a).

Network and volume registration status.

The last 30 lines of unified system logs across all three microservices.

Live Telemetry:

Bash
**make logs**
Attaches the terminal to the live output streams of all containers. Use this to watch NGINX traffic or PHP worker errors in real-time. (Press Ctrl+C to detach).


### Usefull comands:
1) For the explicit logs:
* docker exec -it wordpress (container_name) -d display_errors=1 /var/www/index.php
* docker logs cnt_name 
2) For the connection:
docker exec -it wordpress php83 /var/www/index.php
3) For the database:
docker exec -it mariadb mysql -u root

### Networks features:

Isolation: Containers in project A cannot accidentally talk to containers in project B.

To inspect the correct network:
> docker network ls
> docker network inspect network_name

To inspect the newtors:
> ip addr show

### CMDs for the DB
SHOW DATABASES;
SHOW TABLES;
USE user_name (check .env)

## Container interaction
In order to go inside a container and run anything , one would need to do: `docker exec -it container_name sh` | `docker exec -it nginx kill -9 1`
