*This document was generated with the help of AI, consciously, by me, oishchen, cos i genuinly do not know why do we have README, USER_DOC and DEV_DOC, they are all the same c'mon*

# ⚙️ Inception - Developer Documentation

This document outlines the architecture, setup requirements, and development workflow for the Inception infrastructure.

## 🛠️ 1. Prerequisites & Host Setup

Before building the project, the host machine must meet the following requirements:
* **Software:** Docker, Docker Compose, and Make must be installed.
* **DNS Resolution:** The domain `oishchen.42.de` must resolve to localhost.
  * *Action:* Add `127.0.0.1 oishchen.42.de` to the host's `/etc/hosts` file.

## 🏗️ 2. Setup and Compilation

The project uses a custom Makefile to orchestrate Docker Compose commands and manage physical volume directories.

### Makefile Commands:
* `make` / `make all`: Executes the `make_dir.sh` script to ensure host volume directories exist, then boots the stack in detached mode (`up -d`).
* `make build`: Forces a complete re-compilation of the Alpine Linux Dockerfiles before booting. Use this after altering a `Dockerfile` or `nginx.conf`.
* `make down`: Gracefully stops the containers and tears down the `inception` bridge network.
* `make re`: Executes `make down` followed by `make build`.
* `make clean`: Stops containers, prunes the Docker system, and physically deletes the contents of the host data directories.
* `make fclean`: Ruthlessly stops all containers, forces a total system prune (including networks and volumes), and wipes all host data.

## 🐳 3. Docker Compose Architecture

The stack is defined in `srcs/docker-compose.yml` and consists of three microservices communicating over a custom bridge network (`inception`):

1. **NGINX:** The sole entry point. Binds to host port `443`. Routes `.php` requests to the WordPress container via FastCGI on port `9000`.
2. **WordPress (PHP-FPM):** Contains the PHP 8.3 daemon. Executes application logic. Connects to MariaDB internally on port `3306`.
3. **MariaDB:** The SQL engine. Isolated entirely from the host (no exposed ports). Initializes the database using credentials from the `.env` file.

## 💾 4. Data Persistence (Volumes)

To comply with stateless container principles, all persistent data is anchored directly to the host kernel using bind mounts.

* **Database Files:** Bound to `/home/${USER}/data/mariadb`.
* **Web Application Files:** Bound to `/home/${USER}/data/wordpress`.

*Developer Note:* The default Docker named volumes are bypassed in favor of explicit `driver_opts: o: bind` directives to ensure strict control over physical block storage location.
