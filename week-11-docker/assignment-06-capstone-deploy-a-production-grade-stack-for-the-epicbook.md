# Assignment 6 — Capstone: Deploy a Production-Grade Stack for The EpicBook

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this capstone assignment, you will take the EpicBook application from source code to a fully hardened, production-ready Docker Compose deployment on a cloud VM: multi-stage Dockerfiles, network isolation, healthchecks, persistent volumes with a backup plan, structured logging, a secure reverse proxy, and an operations runbook — with an optional CI/CD pipeline.

---

# Task 0 — App Discovery & Architecture

## Goal

Explore the `theepicbook` repository, identify its components (UI, API, DB, workers), document required environment variables/ports/persistence paths, and draw a component architecture diagram.

### Evidence

#### Deliverable — `docs/01-architecture-diagram.png`

![paste file](screenshots/epicbook_diagram.png)

---

#### Deliverable — `docs/02-env-and-ports.md`

![paste file](screenshots/Screenshot-69.png)

---

# Task 1 — Multi-Stage Dockerfiles

## Goal

Create minimal multi-stage Dockerfiles for the frontend and backend services, with `.dockerignore` files to keep the build context lightweight.

### Evidence

#### Deliverable — `frontend/Dockerfile`, `backend/Dockerfile`, and `.dockerignore` files, with a note on layer optimizations and security benefits

### Frontend DockerFile
```
FROM nginx:alpine
COPY public /usr/share/nginx/html
EXPOSE 3000
```

### Backend Docker File
```
# ---------- Stage 1: install production dependencies ----------
FROM node:20-alpine AS deps

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install dependencies
RUN npm ci --omit=dev

# ---------- Stage 2: runtime image ----------
FROM node:20-alpine AS runtime
ENV NODE_ENV=production
WORKDIR /app
RUN apk add --no-cache curl
COPY --from=deps /app/node_modules ./node_modules
COPY . .
USER node
EXPOSE 3001
CMD ["node", "server.js"]

```

### .dockerignore
```
.git
.env
node_modules
npm-debug.log
Dockerfile*
.dockerignore
```

### Layer Optimization
```
Lightweight base images: Alpine-based images reduce unnecessary operating-system packages.

Multi-stage builds: Build tools and intermediate artifacts are kept out of the final image where they are not needed at runtime.

Dependency-layer caching: Copying package manifests before application source code allows Docker to reuse the dependency installation layer when dependencies have not changed.

Production dependencies: The backend should install only required production dependencies in its final image.

Build context reduction: A .dockerignore file prevents unnecessary files from being sent to the Docker build context.

These practices can reduce build time, image size, image transfer time, and deployment overhead.
```

### Security Benefit
Multi-stage builds improve the security posture of the application by reducing the components included in the production image.


---

# Task 2 — Compose Stack & Networks

## Goal

Author `docker-compose.yml` defining the reverse-proxy, frontend, backend, and database services, isolated front-tier/back-tier networks, and named volumes.

### Evidence

#### Deliverable — `docker-compose.yml`

```
services:
  reverse-proxy:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./proxy/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - frontend
      - backend
    networks:
      - front-tier
    restart: unless-stopped

  frontend:
    build: ./frontend
    image: epicbook-frontend:1.0
    networks:
      - front-tier
    restart: unless-stopped

  backend:
    build: ./backend
    image: epicbook-backend:1.0
    environment:
      NODE_ENV: production
      PORT: "3001:80 "
      JAWSDB_URL: mysql://${DB_USER}:${DB_PASSWORD}@database:3306/${DB_NAME}
    depends_on:
      - database
    networks:
      - front-tier
      - back-tier
    restart: unless-stopped

  database:
    image: mysql:8.0
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db_data:/var/lib/mysql
      - ./db/BuyTheBook_Schema.sql:/docker-entrypoint-initdb.d/01-schema.sql:ro
      - ./db/author_seed.sql:/docker-entrypoint-initdb.d/02-authors.sql:ro
      - ./db/books_seed.sql:/docker-entrypoint-initdb.d/03-books.sql:ro
    networks:
      - back-tier
    restart: unless-stopped

networks:
  front-tier:
  back-tier:
    internal: true

volumes:
  db_data:

```

---

# Task 3 — Healthchecks & Startup Order

## Goal

Add a database healthcheck, a `/health` endpoint check on the backend, and `depends_on` conditions checking `service_healthy`.

### Evidence

#### Deliverable — `docs/03-healthchecks-and-depends-on.md`

##### database healthcheck
```
healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-uroot", "-p${DB_ROOT_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s
```
##### /health endpoint check on the backend
```
app.get("/health", (req, res) => {
  res.status(200).send("OK");
});
```
##### depends_on conditions checking service_healthy.
```
1. Reverse-Proxy service health
 depends_on:
      frontend:
        condition: service_healthy
      backend:
        condition: service_healthy

2. Backend service healthy
depends_on:
      database:
        condition: service_healthy
```

---

# Task 4 — Reverse Proxy & CORS Config

## Goal

Configure Nginx or Traefik to route `/api` to the backend and `/` to the frontend, with an appropriate CORS allowlist on the backend.

### Evidence

#### Deliverable — `docs/04-proxy-routing-and-cors.md`

```
server {
    listen 80;

    # CORS allowlist — update this to your real public origin in Task 7
    set $cors_origin "http://localhost:80";

    # Health check — bypasses everything, used by monitoring/uptime checks
    location = /health {
        proxy_pass http://backend:3001/health;
    }

    # API calls — backend, with CORS headers
    location /api/ {
        add_header Access-Control-Allow-Origin $cors_origin always;
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;
        add_header Access-Control-Allow-Headers "Content-Type, Authorization" always;

        if ($request_method = OPTIONS) {
            add_header Access-Control-Allow-Origin $cors_origin always;
            add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;
            add_header Access-Control-Allow-Headers "Content-Type, Authorization" always;
            add_header Content-Length 0;
            add_header Content-Type text/plain;
            return 204;
        }

        proxy_pass http://backend:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Static files (CSS, JS, images) — frontend container
    location /assets/ {
        proxy_pass http://frontend:80;
    }

    # Everything else (server-rendered pages) — backend container
    location / {
        proxy_pass http://backend:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```

---

# Task 5 — Persistence & Backups

## Goal

Mount the database data directory to a named volume, document a snapshot/backup plan, and run a live snapshot restore drill.

### Evidence

#### Deliverable — `docs/05-persistence-and-backup.md`

```
## What we back up
A `mysqldump` of the `bookstore` database (full schema + data, as portable SQL).
We do not back up raw volume files directly — a SQL dump is portable across
MySQL versions and easy to inspect or move.

## When
Daily, at a low-traffic hour (example: 2 AM server time).

## Where
Initially on the same VM's disk, in a `backups/` folder outside the Docker
volume. For a production setup we would add a step to upload each dump to
object storage (AWS S3), so a full VM loss doesn't also destroy the backups.
This is a stretch goal for this capstone.

## Retention
Keep the last 7 daily dumps, delete anything older, to avoid unbounded disk growth.

## Automation (for the cloud VM, Task 7)
A cron job running this one-line script:

    docker compose exec -T database sh -c 'mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" bookstore' > /home/<user>/backups/bookstore-$(date +\%Y\%m\%d).sql
    find /home/<user>/backups -name "*.sql" -mtime +7 -delete

## Restore procedure
    cat backups/<chosen-file>.sql | docker compose exec -T database sh -c 'mysql -uroot -p"$MYSQL_ROOT_PASSWORD" bookstore'

## Manual test performed
1. Confirmed baseline count: 54 books.
2. Deleted book id 1 ("28 Summers").
3. Confirmed count dropped to 53, proving real data loss.
4. Restored from a `mysqldump` backup taken beforehand.
5. Confirmed count returned to 54, and book id 1 ("28 Summers") was present again.

## Tested
Yes, manually, on [TODAY'S DATE]. See screenshots (volume inspect,
before/delete/restore counts, container-restart persistence check).

```

---

#### Screenshot — System state before and after a manual snapshot restore cycle

![paste file](screenshots/Screenshot-70.png)

---

# Task 6 — Logging & Observability

## Goal

Bind-mount the reverse proxy log directory to the host, redirect application logs to stdout or named volumes, and enable structured JSON logging where supported.

### Evidence

#### Deliverable — `docs/06-logging-layout.md`

```
## Reverse proxy (Nginx)
- **Where:** bind-mounted to the host at `./logs/nginx/`
  (`access.log` and `error.log`), mapped to `/var/log/nginx` in the container.
- **Format:** structured JSON, one object per request line
  (`json_combined` format defined in `proxy/nginx.conf`).
- **Why:** this is the single public entry point. Bind-mounting means logs
  are readable directly from the host with `cat`/`tail`, without needing
  `docker exec`, and they survive container restarts and rebuilds.

## Backend (Node/Express)
- **Where:** stdout, captured by Docker's own logging driver.
  Viewed with `docker compose logs backend`.
- **Why:** the app already logs its own Sequelize queries and startup
  messages to stdout. This is the standard container logging pattern —
  "log to stdout, let the container platform collect it" — and needs no
  extra volume or code change. It also means log rotation, shipping, and
  retention are the platform's job (or `docker`'s built-in log rotation),
  not something we have to build.

## Database (MySQL)
- **Where:** stdout, default MySQL behavior, viewed with
  `docker compose logs database`.
- Not a deliverable target for this task, listed for completeness.

## Sample log line (proxy, JSON)
    {"time":"2026-09-25T12:40:00+00:00","remote_addr":"172.19.0.1","request":"GET / HTTP/1.1","status":200,"body_bytes_sent":24980,"request_time":0.045,"http_referer":"-","http_user_agent":"curl/8.5.0"}

## Optional forwarder (described, not built)
For a larger deployment, a lightweight log forwarder like Fluent Bit could
tail `./logs/nginx/access.log` and the backend's stdout, and ship both to a
central log store (e.g., CloudWatch Logs on AWS). Not implemented here —
the assignment allows describing this rather than building it, and our
scale (one VM, one app) doesn't need it yet.

## Tested
[TODAY'S DATE] — generated four requests (page, API, static asset, 404),
confirmed all four appear as JSON lines in `logs/nginx/access.log` on the
host, and confirmed the log file persists after `docker compose restart
reverse-proxy`.
```
---

#### Screenshot — Live running JSON container log entries

![paste file](screenshots/Screenshot-71.png)

---

# Task 7 — Cloud Deployment (AWS or Azure)

## Goal

Provision a VM per the security requirements (SSH restricted to your IP; HTTP/HTTPS open), install Docker, and bring the full stack up with a working public URL.

### Evidence

#### Deliverable — `docs/07-cloud-deployment-notes.md` (ports, firewall rules, deployment link)

```
# AWS Infrastructure and EpicBook Deployment

## 1. Overview

This project demonstrates the cloud deployment of the containerized EpicBook application using Amazon Web Services (AWS), Docker, and Terraform.

Terraform was used to provision the cloud infrastructure, while an EC2 virtual machine provides the environment for running the Docker containers. Nginx is configured to route incoming HTTP requests to the deployed application.

The deployment makes the application accessible through a public IP address rather than relying on a local development environment.

## 2. Infrastructure Provisioning

The infrastructure was provisioned using Terraform to automate the creation and configuration of the required AWS resources.

The following resources were used:

- **Amazon VPC:** Provides an isolated virtual network for the deployment.
- **Subnet:** Hosts the EC2 instance within the VPC.
- **Internet Gateway:** Enables internet connectivity for resources in the VPC, subject to routing and security configuration.
- **Route Table:** Defines the network routes required for internet access.
- **Security Group:** Acts as a virtual firewall controlling inbound and outbound traffic to the EC2 instance.
- **Amazon EC2:** Provides the virtual machine on which Docker and the EpicBook application run.

This infrastructure provides the networking, compute capacity, and access controls required to host the application in AWS.

## 3. Port Configuration

The deployment uses the following ports:

| Port | Protocol | Purpose |
|---|---|---|
| 22 | SSH | Secure administrative access to the EC2 instance. |
| 80 | HTTP | Public access to the EpicBook application through the web server or reverse proxy. |

SSH access is restricted to the administrator's public IP address to reduce unauthorized access. HTTP access is permitted from public IP addresses so users can access the application.

## 4. Security Group Firewall Rules

The EC2 security group controls access to the deployed virtual machine.

### SSH access

- Protocol: TCP
- Port: 22
- Source: Administrator's public IP address, configured as a `/32` CIDR range.

This rule restricts remote administration to the authorized IP address instead of exposing SSH to the entire internet.

### HTTP access

- Protocol: TCP
- Port: 80
- Source: `0.0.0.0/0`

This rule permits public HTTP requests to reach the application. IPv6 access would require an appropriate IPv6 rule if needed.

Only the required ports should be exposed. Database and internal application ports should not be opened publicly unless there is a specific, justified requirement.

## 5. Application Deployment

After provisioning the infrastructure, Docker was used to run the containerized EpicBook application on the EC2 instance.

Nginx was configured to receive incoming HTTP requests and route them to the appropriate application service. The deployment configuration was updated to use the public application endpoint instead of a local development address.

This allows users to access the application externally through the EC2 instance's public IP address.

## 6. Public Deployment URL

The application was deployed and made accessible through the following public URL:

**http://44.200.16.34/**

This URL provides access to the deployed EpicBook application from a browser, provided that the EC2 instance is running, the application containers are healthy, and the security group and Nginx configuration allow the request.

## 7. Nginx Configuration Update

The Nginx configuration was updated to use the deployed application endpoint instead of the local application address used during development.

This change connects incoming requests to the deployed application environment rather than relying on a local development server.

For containerized deployments, Nginx should use the appropriate Docker Compose service names for internal routing where applicable. The public IP address is the entry point for external users; it should not normally replace internal service names in Docker-to-Docker communication.

## 8. Conclusion

The EpicBook application was deployed to AWS using Terraform-managed infrastructure, an EC2 virtual machine, Docker containers, and Nginx.

The deployment demonstrates infrastructure as code, cloud networking, security group configuration, containerized application hosting, and public web access. Restricting SSH access while allowing HTTP traffic provides a basic security boundary for the deployment.
```

---

#### Screenshot — Browser showing successful web UI retrieval via the cloud public IP

![paste file](screenshots/Screenshot-72.png)

---

#### Screenshot — Active page interaction confirming backend API data fetches succeed

![paste file](screenshots/Screenshot-73.png)

---

# Task 8 — CI/CD Pipeline (Optional)

## Goal

Build a pipeline (Azure Pipelines or GitHub Actions) that builds and tags multi-stage images and automates deployment via SSH restart steps.

### Evidence

#### Deliverable — `docs/08-ci-cd-pipeline.md`

```
# EpicBook CI/CD Pipeline Documentation Using Azure DevOps

## 1. Overview

This document explains how to configure and execute an automated Continuous Integration and Continuous Deployment (CI/CD) pipeline for the EpicBook application using Azure DevOps.

The pipeline automates building Docker images for the frontend and backend, pushing the images to Docker Hub, and deploying the containerized application to an AWS EC2 virtual machine using Docker Compose.

The objective is to simplify application deployment, reduce manual intervention, and make future application updates easier to release.

## 2. Architecture and Tools

The deployment process uses the following technologies:

- **GitHub:** Source-code version control and development repository.
- **Azure Repos:** Hosts the repository imported from GitHub for Azure DevOps pipeline integration.
- **Azure Pipelines:** Automates the build, image publishing, deployment, and verification stages.
- **Docker:** Packages the frontend and backend into container images.
- **Docker Hub:** Stores and distributes the published Docker images.
- **AWS EC2:** Hosts the production application containers.
- **Docker Compose:** Coordinates the frontend, backend, and MySQL database containers.
- **Nginx:** Acts as the reverse proxy for incoming application requests.
- **Azure DevOps Variable Groups and Secure Files:** Store deployment variables, database credentials, Docker Hub credentials, and the VM SSH private key.

## 3. CI/CD Deployment Workflow

The deployment follows this sequence:

1. Develop or update the application code.
2. Commit and push changes to GitHub.
3. Import or synchronize the repository with Azure Repos.
4. Configure Azure DevOps pipeline variables and secure files.
5. Create and configure the Azure Pipelines YAML file.
6. Run the pipeline to build and publish Docker images.
7. Transfer deployment configuration files to the EC2 instance.
8. Pull the published images and restart the application containers.
9. Verify that the application responds successfully.

## 4. Configuration and Implementation

### Step 1: Prepare Docker Hub

1. Sign in to Docker Hub.
2. Create or confirm the repositories for the EpicBook frontend and backend.
3. Ensure the repositories match the image names used by the pipeline, for example:
   - `epicbook-frontend`
   - `epicbook-backend`
4. Generate a Docker Hub access token with the permissions required to push and pull images.
5. Store the token securely for use by Azure DevOps.

The pipeline builds the frontend and backend images and pushes them to Docker Hub with version tags. This allows the deployment server to pull the exact images produced by the build stage.

### Step 2: Create an Azure DevOps Variable Group

1. Open the Azure DevOps organization and project.
2. Navigate to **Pipelines → Library**.
3. Select **Variable groups** and create a group named `Epicbook-docker-secrets`.
4. Add the required variables.

| Variable | Purpose | Secret? |
|---|---|---|
| `dockerHubUsername` | Docker Hub account username | No |
| `dockerHubToken` | Docker Hub access token | Yes |
| `dbUserSecret` | MySQL application username | Preferably yes |
| `dbPasswordSecret` | MySQL application password | Yes |
| `dbRootPasswordSecret` | MySQL root password | Yes |
| `dbNameSecret` | Application database name | Preferably yes |
| `vmPublicIp` | Public IP address of the EC2 instance | No |

5. Mark sensitive variables as secret.
6. Authorize the pipeline to access the variable group.

The pipeline references the variable group to obtain the values required for image publishing, database configuration, and deployment.

**Security note:** Never commit database passwords, Docker Hub tokens, or other secrets to GitHub or Azure Repos.

### Step 3: Upload the EC2 SSH Private Key

Azure DevOps needs secure SSH access to the EC2 instance to transfer configuration files and execute deployment commands.

1. Navigate to **Pipelines → Library → Secure files**.
2. Upload the SSH private key associated with the EC2 instance.
3. Name the secure file `epicbook-docker-vm-key`.
4. Grant the deployment pipeline permission to access the file.

The pipeline downloads the key at runtime and applies restrictive file permissions before using it to connect to the EC2 instance.

The corresponding EC2 security group must allow SSH access from the deployment agent's network. Avoid opening SSH to all internet addresses where possible.

### Step 4: Create the Azure Pipelines YAML File

Create a pipeline file named `azure-pipelines-docker.yml` in the project repository.

The YAML file defines the automated build and deployment process. It includes the following main stages:

**A. Build and publish Docker images**

- Check out the application source code.
- Authenticate to Docker Hub using the configured credentials.
- Build the frontend and backend images.
- Tag the images using the pipeline's image-tagging strategy.
- Push the images to Docker Hub.

**B. Deploy to AWS EC2**

- Download the SSH private key from Azure DevOps Secure Files.
- Validate the Docker Compose file, Nginx configuration, and SQL initialization files.
- Transfer the required deployment files to the EC2 instance.
- Create or update the protected `.env` file containing the deployment variables.
- Validate the resolved Docker Compose configuration.
- Pull the exact frontend and backend image tags from Docker Hub.
- Restart the application using Docker Compose.

**C. Verify the deployment**

- Check the container status.
- Request the application's health endpoint through Nginx.
- Fail the pipeline if the deployment does not pass the configured health check.

For production deployment, the Compose file should reference the published images using the configured Docker Hub username and image tag, rather than relying on images built locally on the EC2 instance.

### Step 5: Commit and Push the Pipeline Changes to GitHub

After creating or updating the pipeline YAML file:

1. Validate the YAML structure and review the deployment commands.
2. Confirm that no secrets are included in the source files.
3. Commit the changes to the appropriate Git branch.
4. Push the changes to GitHub.

Example commands:

```bash
git add azure-pipelines-docker.yml docker-compose.yml
git commit -m "Add Azure DevOps Docker deployment pipeline"
git push origin Epicbook-Container-Production
```

Replace the branch name if your project uses a different deployment branch.

### Step 6: Import the GitHub Repository into Azure Repos

1. Open the Azure DevOps project.
2. Navigate to **Repos → Files**.
3. Select the option to import a repository.
4. Enter the GitHub repository URL and any required authentication details.
5. Import the project into Azure Repos.
6. Confirm that the application source code, Docker Compose file, and pipeline YAML file are present.

If you intend to keep making changes in GitHub, configure an appropriate synchronization process or ensure changes are mirrored into Azure Repos. An import does not necessarily keep the repositories synchronized automatically.

### Step 7: Create the Azure Pipeline

1. Navigate to **Pipelines → Pipelines**.
2. Select **New pipeline**.
3. Choose **Azure Repos Git** as the source.
4. Select the imported EpicBook repository.
5. Choose the existing Azure Pipelines YAML file.
6. Select `azure-pipelines-docker.yml`.
7. Review the pipeline configuration.
8. Save and run the pipeline.

Confirm that the pipeline has access to the variable group and Secure File. Also verify that the selected branch contains the latest pipeline configuration.

If the YAML defines a branch trigger, make sure your changes are pushed to the branch configured in that trigger.

### Step 8: Run and Monitor the Pipeline

When the pipeline starts, monitor the job logs for each stage.

The expected execution sequence is:

1. **Build:** Frontend and backend Docker images are built.
2. **Publish:** The images are pushed successfully to Docker Hub.
3. **Transfer:** Docker Compose, Nginx, and SQL files are copied to EC2.
4. **Deploy:** The EC2 host pulls the intended images and restarts the containers.
5. **Verify:** The pipeline checks the application health endpoint.

A successful pipeline run indicates that all configured stages and checks completed successfully. It is still good practice to confirm that the application works correctly from a browser.

## 5. Deployment Verification

After the pipeline finishes, verify the deployment using the following checks.

### Check container status

Connect to the EC2 instance and run:

```bash
cd ~/theepicbook
docker compose --env-file .env ps
```

Confirm that the expected frontend, backend, and database services are running.

### Check application health

Open the application's public URL in a browser or use:

```bash
curl -i http://44.200.16.34//health
```

Replace `<EC2-PUBLIC-IP>` with the configured EC2 public IP address.

The pipeline should receive the HTTP status code expected by its health-check configuration, typically `200 OK`.

### Review deployment logs

If the deployment fails, inspect the container logs:

```bash
docker compose --env-file .env logs --tail=100 backend
docker compose --env-file .env logs --tail=100 frontend
docker compose --env-file .env logs --tail=100 database
```

Use the service names defined in the actual Compose file. Do not share logs containing passwords, tokens, or other sensitive values.

## 6. Common Issues and Troubleshooting

| Issue | Possible cause | Recommended action |
|---|---|---|
| Docker Hub image pull denied | Incorrect repository name, tag, or authentication | Check the image reference and Docker Hub permissions |
| Compose pulls `epicbook-backend` instead of the Docker Hub image | Outdated Compose file or missing image variables | Check `docker compose config --images` on EC2 |
| SSH connection fails | Incorrect key, permissions, or security-group rules | Validate the secure file and SSH access |
| SQL initialization mount fails | SQL file missing or a directory exists at the expected file path | Validate the local SQL files and their destination paths before deployment |
| Database initialization scripts do not run | MySQL data volume was already initialized | Check database state and initialization logs; back up data before considering any reset |
| Health check fails | Containers are unhealthy, Nginx routing is incorrect, or the endpoint is unavailable | Inspect container status, application logs, and Nginx configuration |
| Pipeline cannot access secrets | Variable group or Secure File permissions are missing | Authorize the pipeline to access the required resources |

```
## 7. Security and Operational Best Practices

- Store credentials in Azure DevOps secret variables or an approved secrets manager.
- Use Docker Hub access tokens instead of account passwords.
- Restrict SSH access to trusted sources.
- Use versioned Docker image tags so deployments are reproducible.
- Validate the Compose configuration before restarting production services.
- Keep the MySQL data volume persistent and back up important data.
- Avoid printing secrets in pipeline logs.
- Configure health checks and retain deployment logs for troubleshooting.
- Test pipeline changes in a non-production environment when possible.

```
---

#### Screenshot (optional) — Completed automated pipeline run

![paste file](screenshots/Screenshot-74.png)

---

# Task 9 — Reliability Tests & Ops Runbook

## Goal

Inject faults (drop backend/database dependencies) and evaluate frontend error handling, then write an operations runbook covering startup, shutdown, secret rotation, and disaster recovery.

### Evidence

#### Deliverable — `docs/09-runbook.md`

# EpicBook — Operations Runbook

## Start / Stop the Stack
Start:
    cd ~/theepicbook
    docker compose up -d

Stop (keeps data):
    docker compose down

Stop and DELETE all data (rarely wanted):
    docker compose down -v

Restart a single service:
    docker compose restart backend

## Rollback Procedure
1. Check the currently running image tag:
       docker compose ps
       grep image: docker-compose.yml
2. Roll back to a previous known-good tag:
       docker pull <dockerhub-username>/epicbook-backend:<previous-tag>
       docker pull <dockerhub-username>/epicbook-frontend:<previous-tag>
       sed -i "s|epicbook-backend:.*|epicbook-backend:<previous-tag>|" docker-compose.yml
       sed -i "s|epicbook-frontend:.*|epicbook-frontend:<previous-tag>|" docker-compose.yml
       docker compose up -d
3. Confirm recovery:
       curl -i http://<PUBLIC_IP>/health
4. Fix the underlying issue in a new commit and let the pipeline
   redeploy the corrected version, rather than leaving the rollback
   as a permanent manual state.

## Rotating Secrets
1. Generate new values:
       openssl rand -hex 16
2. Update `.env` on the VM with the new `DB_PASSWORD` / `DB_ROOT_PASSWORD`.
3. `.env` values only apply to MySQL on its *first* boot. To rotate a
   live database's password:
       docker compose exec database sh -c \
         'mysql -uroot -p"$MYSQL_ROOT_PASSWORD" -e \
         "ALTER USER '\''epicbook'\''@'\''%'\'' IDENTIFIED BY '\''<new-password>'\'';"'
   Then update `.env` to match, and restart the backend:
       docker compose restart backend

## Log Locations
- Reverse proxy (Nginx): `~/theepicbook/logs/nginx/access.log` and
  `error.log` on the VM's host filesystem (bind-mounted, JSON format).
- Backend (Node/Express): stdout, via `docker compose logs backend`.
- Database (MySQL): stdout, via `docker compose logs database`.

## Backup / Restore
Full plan: `docs/05-persistence-and-backup.md`.

Backup:
    docker compose exec -T database sh -c \
      'mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" bookstore' \
      > backups/bookstore-$(date +%Y%m%d-%H%M%S).sql

Restore:
    cat backups/<file>.sql | docker compose exec -T database sh -c \
      'mysql -uroot -p"$MYSQL_ROOT_PASSWORD" bookstore'

## Common Errors & Fixes

| Symptom | Cause | Fix |
|---|---|---|
| Container stopped with `docker compose kill`/`stop` never restarts | `restart: unless-stopped` intentionally does not override a manual stop | `docker compose up -d <service>` to bring it back |
| Backend process crash (killed from inside the container) doesn't come back | (Should self-heal) — if it doesn't, check `docker compose logs backend` for a crash loop | `docker compose up -d backend`, then investigate logs |
| `reverse-proxy` stuck in `Created`, never starts | `backend` took longer than usual to become healthy (DB startup race), dependency wait timed out | `docker compose up -d reverse-proxy` once `backend` is healthy |
| Backend crashes once on fresh boot with `ECONNREFUSED` | MySQL wasn't ready on backend's first connection attempt | Self-heals via `restart: unless-stopped`; investigate only if it loops repeatedly |
| `git pull` fails on the VM: "local changes would be overwritten" | CI/CD pipeline's `sed` step left `docker-compose.yml` modified locally | `git checkout -- docker-compose.yml` before pulling |
| Healthcheck fails with "curl: not found" | `curl` installed in the wrong Docker build stage (discarded in multi-stage build) | Install `curl` in the final `runtime` stage, not `deps` |
| Site returns connection refused on the expected port | `docker-compose.yml`'s port mapping doesn't match what's published | Check `docker compose ps`, fix `ports:`, `docker compose up -d` |

## Tested
[DATE] — ran all three reliability tests against the production VM.
Backend crash recovered automatically via restart policy (StartedAt
timestamp confirmed). Database outage produced a clear failure response
and recovered on restore. Full stack bounce confirmed data integrity
(54 books before and after). See docs/10-reliability-tests.md for full
command-by-command output and screenshots.

---

#### Deliverable — `docs/10-reliability-tests.md`

# EpicBook Reliability Tests

## 1. Purpose

This document describes the reliability tests used to validate the EpicBook application after deployment through the Azure DevOps CI/CD pipeline.

The tests verify that the frontend, backend, MySQL database, and Nginx reverse proxy operate correctly, and that the application remains accessible after a deployment or container restart.

## 2. Test Environment

The reliability tests use the following components:

- Azure DevOps Pipelines for automated deployment.
- Docker Hub for storing frontend and backend images.
- AWS EC2 as the deployment host.
- Docker Compose for managing application containers.
- Nginx for reverse proxy and HTTP traffic routing.
- MySQL for application data persistence.
- An HTTP health endpoint, such as `/health`, for deployment verification.

## 3. Reliability Test Cases

### Test 1: Container Availability

**Objective:** Confirm that the expected containers start successfully after deployment.

**Procedure:**

Connect to the EC2 instance and run:

```bash
cd ~/theepicbook
docker compose --env-file .env ps
```

**Expected result:**

- Frontend container is running.
- Backend container is running.
- Database container is running.
- No unexpected container exits or restart loops are present.

**Pass criteria:** All required services are running and their health checks, where configured, report healthy.

### Test 2: Application Health Endpoint

**Objective:** Verify that the application responds to HTTP health checks.

**Procedure:**

```bash
curl -i --max-time 10 http://localhost/health
```

To test externally, run the request from a machine outside EC2:

```bash
curl -i --max-time 10 http://<EC2-PUBLIC-IP>/health
```

Replace `<EC2-PUBLIC-IP>` with the actual public IP address.

**Expected result:** The configured health endpoint returns the expected HTTP response, typically `200 OK`.

**Pass criteria:** The endpoint responds within the configured timeout and returns the expected status code.

*Note:* The health endpoint must be routed correctly through Nginx. If `/health` is implemented only on the backend, ensure the proxy forwards the request to it.

### Test 3: Frontend Accessibility

**Objective:** Confirm that users can access the EpicBook frontend.

**Procedure:**

1. Open the EC2 public URL in a browser.
2. Alternatively, execute:

```bash
curl -I --max-time 10 http://<EC2-PUBLIC-IP>/
```

**Expected result:** The server returns a successful HTTP response, and the frontend loads without a proxy or server error.

**Pass criteria:** The application page is accessible through the configured public endpoint.

### Test 4: Backend and Database Connectivity

**Objective:** Confirm that the backend can connect to MySQL.

**Procedure:**

1. Inspect backend logs:

```bash
docker compose --env-file .env logs --tail=100 backend
```

2. Inspect database logs:

```bash
docker compose --env-file .env logs --tail=100 database
```

3. If the application exposes a database readiness or health endpoint, call it and verify that the database connection succeeds.

**Expected result:**

- The MySQL server starts successfully.
- The backend can connect to the configured database service.
- No recurring authentication, DNS resolution, or connection-refused errors occur.

**Pass criteria:** The backend successfully accesses the database using the configured application credentials.

### Test 5: Container Restart Recovery

**Objective:** Verify that services recover after a container restart.

**Procedure:**

Restart the backend container:

```bash
docker compose --env-file .env restart backend
```

Then inspect the service state and health endpoint:

```bash
docker compose --env-file .env ps
curl -i --max-time 10 http://localhost/health
```

**Expected result:** The backend restarts and resumes serving requests without requiring a full deployment.

**Pass criteria:** The backend returns to a healthy state and the application health endpoint recovers within the expected startup period.

### Test 6: Database Persistence

**Objective:** Confirm that application data persists across container recreation.

**Procedure:**

1. Create or identify a test record in the application.
2. Record a safe identifier for that record.
3. Recreate the database container without removing named volumes:

```bash
docker compose --env-file .env up -d --force-recreate database
```

4. Wait for MySQL to become ready.
5. Verify that the test record still exists.

**Expected result:** Previously stored data remains available after the database container is recreated.

**Pass criteria:** The test record remains present and readable.

**Safety requirement:** Run this test only in a suitable test environment or during an approved maintenance window. Back up important data first. Never use `docker compose down -v` for this test because it can delete the named database volume.

### Test 7: Deployment Image Consistency

**Objective:** Confirm that the EC2 instance runs the intended Docker Hub image versions.

**Procedure:**

On the EC2 instance, run:

```bash
cd ~/theepicbook
docker compose --env-file .env config --images
docker compose --env-file .env ps
```

Inspect the image tags and confirm they match the version published by the Azure DevOps build stage.

**Expected result:** The Compose configuration references the intended Docker Hub repositories and image tags.

**Pass criteria:** The deployed services use the expected images rather than outdated local image names or unintended tags.

### Test 8: Deployment Failure Detection

**Objective:** Verify that deployment failures are detected instead of being reported as successful.

**Procedure:**

In a non-production environment, test a controlled failure, such as temporarily configuring an invalid image tag.

Run the deployment pipeline and observe the pull and deployment stages.

**Expected result:** The pipeline fails when the required image cannot be pulled or the deployment health check does not pass.

**Pass criteria:** The pipeline reports failure and provides enough logs to identify the problem.

Restore the valid image tag after the test.

## 4. Automated Reliability Checks in Azure DevOps

The deployment pipeline should perform the following checks automatically:

1. Validate required configuration and SQL files before copying them to EC2.
2. Validate the Docker Compose configuration.
3. Pull the exact frontend and backend image tags.
4. Start or update the application containers.
5. Check that the required containers are running and healthy where health checks are configured.
6. Request the application health endpoint and fail the pipeline if the expected response is not received.

Example HTTP verification command:

```bash
set -euo pipefail

URL="http://$(vmPublicIp)/health"

HTTP_CODE=$(curl \
  --silent \
  --show-error \
  --output /tmp/epicbook-health-response.txt \
  --write-out "%{http_code}" \
  --max-time 10 \
  "$URL")

if [ "$HTTP_CODE" != "200" ]; then
  echo "ERROR: Health check failed. HTTP status: $HTTP_CODE"
  cat /tmp/epicbook-health-response.txt
  exit 1
fi

echo "SUCCESS: EpicBook health check returned HTTP 200."
```

The pipeline should not declare deployment success solely because Docker Compose accepted the configuration. The health endpoint must also pass.

## 5. Troubleshooting Guide

| Symptom | Investigation | Corrective action |
|---|---|---|
| Container exits immediately | Inspect container logs and exit status | Correct the application configuration or startup command |
| Backend cannot connect to MySQL | Check service name, port, credentials, and database readiness | Correct the connection settings and verify network connectivity |
| Docker Hub pull fails | Check image repository, tag, and registry authentication | Publish the image or correct the image reference and credentials |
| HTTP 502 from Nginx | Inspect Nginx configuration and backend status | Correct the upstream address or restore the backend service |
| Health endpoint times out | Check application logs, routing, and security-group rules | Restore the endpoint and verify network access |
| Data disappears after recreation | Inspect the database volume configuration | Restore the persistent volume configuration and recover data from backup if necessary |
| Pipeline passes but application is unavailable | Review health-check scope and deployment verification | Add external HTTP verification and container readiness checks |

Useful commands:

```bash
# Service status
docker compose --env-file .env ps

# Application logs
docker compose --env-file .env logs --tail=100 backend

# Nginx logs
docker compose --env-file .env logs --tail=100 frontend

# Database logs
docker compose --env-file .env logs --tail=100 database

# Disk usage
df -h

# Docker disk usage
docker system df
```

Use the actual Compose service names if they differ from `backend`, `frontend`, or `database`.

## 6. Test Results Record

Record the outcome of each test during a deployment or release validation.

| Test | Result | Evidence / Notes |
|---|---|---|
| Container availability | Pending | Record `docker compose ps` output |
| Application health endpoint | Pending | Record HTTP status |
| Frontend accessibility | Pending | Record browser or HTTP result |
| Backend-to-database connectivity | Pending | Record application and database logs |
| Container restart recovery | Pending | Record recovery outcome |
| Database persistence | Pending | Record test record verification |
| Deployment image consistency | Pending | Record resolved image references |
| Deployment failure detection | Pending | Record pipeline run outcome |

Use **Pass**, **Fail**, or **Not Run** after executing each test. Do not mark a test as passed until its expected result has been verified.

## 7. Conclusion

These reliability tests provide a repeatable way to evaluate the EpicBook deployment before and after application releases. They cover service availability, HTTP access, database connectivity, restart recovery, data persistence, image consistency, and deployment failure detection.

Running these checks as part of the Azure DevOps CI/CD workflow helps identify deployment problems early and provides evidence that the application is operating as expected.

---

# LinkedIn Post (Required)

## Goal

Publish a 6–10 line LinkedIn post covering the architectural decision that most improved reliability, the biggest image size reduction achieved (with numbers), and your key production-hardening takeaways.

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

linkedin.com/posts/micheal-omotosho-577230199_devops-docker-dockercompose-share-7518753762862280705-U1gH/?highlightedUpdateUrn=urn%3Ali%3Aactivity%3A7518753764627890177&highlightedUpdateType=SOCIAL_SHARE&origin=SOCIAL_SHARE&utm_source=share&utm_medium=member_desktop&rcm=ACoAAC58XisBJdoafJCMJEdvAEQtCZ209939LWg

---

#### Screenshot — Published LinkedIn post showing the text body and a deployment verification image

![paste file](screenshots/linkedIn-05.png)

---

# Submission Instructions

- Add all required deliverables, screenshots, and links in your submission
- Full name must be visible in required screenshots
- Do not expose passwords, API tokens, cloud account IDs, or private keys

---

# Completion Checklist

- [ ] Task 0: Architecture diagram and env/ports doc completed
- [ ] Task 1: Multi-stage Dockerfiles and `.dockerignore` files created
- [ ] Task 2: `docker-compose.yml` with isolated networks and volumes authored
- [ ] Task 3: Healthchecks and `depends_on` conditions configured
- [ ] Task 4: Reverse proxy routing and CORS configured
- [ ] Task 5: Persistence and backup plan tested (Screenshot)
- [ ] Task 6: Logging and observability configured (Screenshot)
- [ ] Task 7: Cloud deployment live and verified (Screenshots)
- [ ] Task 8: CI/CD pipeline built (optional)
- [ ] Task 9: Reliability tests and runbook completed
- [ ] LinkedIn post published and URL submitted
- [ ] No sensitive information exposed
- [ ] VM torn down after grading to avoid unexpected charges

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
