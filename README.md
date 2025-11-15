# Phase 1 DevSecOps Capstone: 3-Tier Poll Application

This project is a complete, containerised 3-tier web application that demonstrates mastery of all Phase 1 DevOps fundamentals, with a strong emphasis on security.

It is a horizontally-scaled, load-balanced application built on a secure, multi-stage Docker foundation.

## Architecture

This application consists of three services orchestrated by Docker Compose:

1.  **`proxy` (Nginx):** The secure front door. It serves the static HTML/CSS/JS frontend and acts as a **reverse proxy** and **load balancer**, distributing `/api` requests across all three `web` replicas.
2.  **`web` (Python/Flask):** The backend API (scaled to **3 replicas**). It handles business logic, processes votes, and communicates with the database. It runs as a non-root user in a minimal, multi-stage container.
3.  **`db` (PostgreSQL):** The data tier. It stores all vote data. It is completely private and is not exposed to the host, only accessible via the internal Docker network.

### Architecture Diagram

## DevSecOps Features Implemented

* **Linux/Networking (Phase 1A/1C):**
    * **Nginx Reverse Proxy & Load Balancer:** Securely routes traffic based on URL (`/api` vs. `/`) and load balances API requests.
    * **Custom Bridge Network:** A private `app-net` is used, so the `web` and `db` services are *not* exposed to the host machine.
    * **Port Mapping:** The host is only exposed on a single port (`8080`).

* **Git (Phase 1B):**
    * **Secure Workflow:** The `main` branch is protected, forcing all changes through Pull Requests.
    * **Secret Management:** The `.gitignore` file correctly ignores the `.env` secrets file.
    * **History Management:** Resolved a "divergent branches" issue using `git merge --allow-unrelated-histories` to align the local and remote main branches.

* **Docker (Phase 1D):**
    * **Horizontal Scaling:** The `web` service is scaled to 3 replicas using `deploy: replicas: 3` for high availability.
    * **Secure Dockerfile:** The `web` service uses a **multi-stage build** to create a minimal final image.
    * **Non-Root User:** The `web` app runs as a low-privilege `appuser` inside the container.
    * **Build Caching:** The Dockerfile uses `RUN --mount=type=cache` for ultra-fast dependency installation.
    * **Docker Compose:** A single `docker-compose.yml` file defines and orchestrates all 3 services, their networks, volumes, and dependencies.
    * **Persistent Data:** A named **volume** (`db_data`) is used to ensure all poll data survives container restarts.

## Project Journey & Key Challenges

This project involved debugging a series of real-world, interconnected issues.

1. Docker Build Context Error: The nginx container failed to build because its Dockerfile couldn't access files in the ../web directory.
    * **Fix:** The `docker-compose.yml` was modified to use `build: { context: ., dockerfile: ./nginx/Dockerfile }`. This gave the build the entire project as its context.
    * **New Error:** This change broke the COPY paths inside the `nginx/Dockerfile.`
    * **Final Fix:** The `nginx/Dockerfile` `COPY` commands were updated to use paths relative to the new context (e.g., `COPY web/static/index.html ...`).

2. Application File Not Found: The web containers were crashing with [Errno 2] No such file or directory for app.py.
    * **Fix:** This was another context issue. The `web/Dockerfile `was copying `.` (the project root) into its `/app` directory. This was corrected to `COPY web/ .` to copy only the application code into the `WORKDIR`.

## How to Run
1.  **Clone:** `git clone https://github.com/[your-username]/roadmap-phase1-project.git`
2.  **Setup Secrets:** `cd roadmap-phase1-project` and create a `.env` file with the specified content.
3.  **Run:** `docker-compose up --build -d`
4.  **Access:** Open `http://localhost:8080` in your browser.
5.  **Stop:** `docker-compose down`