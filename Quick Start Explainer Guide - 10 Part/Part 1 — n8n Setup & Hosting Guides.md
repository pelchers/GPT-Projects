# Part 1 — n8n Setup & Hosting Guides

This guide covers two approaches for getting n8n running as your backend automation/orchestration engine:

* **A. Cloud Hosting (e.g., n8n.cloud, Render.com, or other managed host)**
* **B. Self-Hosting (e.g., Hostinger VPS, DigitalOcean, or other private server)**

Each section provides step-by-step instructions for initial setup, security, and preparing n8n to receive requests from your web application.

---

## A. n8n Cloud Hosting Setup

### 1. Choose a Cloud Provider

* Popular options: [n8n.cloud](https://n8n.cloud), [Render.com](https://render.com), [HarperDB Cloud](https://harperdb.io/cloud), \[Any cloud with Docker support]
* Sign up and select a plan (free/paid as required)

### 2. Create an n8n Instance/Workspace

* Use the provider’s UI to launch a new n8n instance
* Note your instance URL (e.g., `https://your-workspace.n8n.cloud`)

### 3. Secure Your Instance

* Set an admin username and a strong password
* Enable HTTPS by default (most managed providers do this)
* Configure allowed domains/origins for webhook/API requests (restrict to your app’s domain for security)

### 4. Prepare for App Integration

* Log in to the n8n editor via your instance URL
* Click **"+ New Workflow"**
* Add an **HTTP Trigger** node as the first step

  * Set the HTTP Method to `POST` (or `GET` if needed)
  * Copy the Webhook URL; you’ll need this for your frontend
* Save and activate the workflow

### 5. Optional: Connect External Databases

* If you want n8n to use a custom Postgres DB (for persistence or advanced workflows), provide the database connection info in the provider’s settings or environment variables
* Managed n8n usually handles this automatically, but advanced users may override it

### 6. (If Required) Set up API Keys or OAuth

* For integrations (Airtable, Notion, Google, etc.), configure credentials under the **Credentials** section
* Use the official n8n guides for specific integrations

---

## B. n8n Self-Hosting Guide (e.g., Hostinger VPS)

### What is Docker Compose and Why Use It?

* **Docker** allows you to package applications (like n8n) and their dependencies into containers that run consistently on any server or OS. This avoids issues where something works on your laptop but fails on your VPS.
* **Docker Compose** is a tool for defining and running multi-container Docker applications. With a simple `docker-compose.yml` file, you can start, stop, and manage the entire stack (including n8n, databases, proxies, etc.) using one command.
* **Benefits:**

  * One-file setup: All configs (ports, env vars, volumes) in a single file
  * Easy to back up, restore, or migrate
  * Updates and restarts are one-command (`docker-compose up -d`)
  * Consistent deployments across environments
* **When to use:** Recommended for nearly all server deployments unless you have a compelling reason to use a traditional install.

### 1. Provision Your Server

* Choose a VPS or dedicated server provider (Hostinger, DigitalOcean, AWS EC2, etc.)
* Minimum recommended: 1 vCPU, 2GB RAM, 20GB storage
* Get your root/admin login details

### 2. Install Docker (Recommended)

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable docker --now
```

### 3. Set Up n8n with Docker Compose

* Create a project folder:

```bash
mkdir ~/n8n && cd ~/n8n
```

* Create a file named `docker-compose.yml`:

```yaml
version: '3.1'
services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - 5678:5678
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=your_admin_username
      - N8N_BASIC_AUTH_PASSWORD=your_strong_password
      - N8N_HOST=your.domain.com
      - N8N_PROTOCOL=https
      # Optional DB persistence
      # - DB_TYPE=postgresdb
      # - DB_POSTGRESDB_HOST=your-db-host
      # - DB_POSTGRESDB_PORT=5432
      # - DB_POSTGRESDB_DATABASE=n8n
      # - DB_POSTGRESDB_USER=dbuser
      # - DB_POSTGRESDB_PASSWORD=dbpass
    volumes:
      - ./n8n_data:/home/node/.n8n
```

* Replace the `N8N_BASIC_AUTH_USER` and `N8N_BASIC_AUTH_PASSWORD` with your credentials
* Set `N8N_HOST` to your domain or IP (optional but recommended for production)

### 4. Run n8n

```bash
docker-compose up -d
```

* n8n will now be accessible at `http://your-server-ip:5678` (or via your configured domain)

### 5. Set Up Domain and SSL

* Point your domain/subdomain’s DNS to your server IP
* Use a tool like [Caddy](https://caddyserver.com/) or [NGINX](https://nginx.org/) as a reverse proxy for HTTPS
* For Let’s Encrypt SSL:

```bash
sudo apt install -y nginx certbot python3-certbot-nginx
sudo certbot --nginx -d your.domain.com
```

* Update your Docker Compose to use `N8N_PROTOCOL=https` if running behind SSL

### 6. Log in and Create Your First Workflow

* Visit your n8n instance in the browser (domain or IP:5678)
* Log in with your admin credentials
* Click **"+ New Workflow"**
* Add an **HTTP Trigger** node for frontend communication
* Save and activate the workflow

### 7. (Optional) Connect to an External/Postgres DB

* Uncomment and fill the `DB_*` environment variables in your Docker Compose file
* Restart your n8n container for changes to take effect

### 8. Integrate Credentials for Third-Party APIs

* In the n8n UI, go to **Credentials**
* Set up connections to Airtable, Notion, Google, etc.
* Use n8n’s built-in instructions for each integration

---

### Traditional (Non-Docker) Install: n8n via Node.js (for Comparison)

#### **When to Consider:**

* Only if you want fine-grained control over every dependency
* When you cannot use Docker on your host
* For development or learning, not usually recommended for production

#### **Step-by-Step:**

1. **Install Node.js (v18+ recommended):**

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

2. **Install n8n globally:**

```bash
sudo npm install n8n -g
```

3. **Start n8n:**

```bash
n8n
```

4. **Configure Environment:**

* Set env variables in your shell or via a `.env` file:

  * `N8N_BASIC_AUTH_ACTIVE=true`
  * `N8N_BASIC_AUTH_USER=your_admin_username`
  * `N8N_BASIC_AUTH_PASSWORD=your_strong_password`
  * `N8N_HOST=your.domain.com`
  * `N8N_PROTOCOL=https`
* For external DBs:

  * `DB_TYPE=postgresdb`
  * Other `DB_*` vars as above

5. **Reverse Proxy and SSL:**

* As with Docker, use NGINX or Caddy for SSL and routing

#### **Pros/Cons vs Docker Compose:**

|                 | Docker Compose           | Traditional Node.js        |
| --------------- | ------------------------ | -------------------------- |
| **Setup**       | One-file, one-command    | Manual, step-by-step       |
| **Updates**     | One command              | Update Node & n8n manually |
| **Isolation**   | Full (no host conflicts) | Possible version issues    |
| **Portability** | Easy (move folder)       | More work (manual setup)   |
| **Backups**     | Easy (volume)            | Manual or scripted         |
| **Security**    | Runs in container        | Runs on host OS            |

---

## Quick Checklist for Both Methods

* [ ] n8n instance deployed (cloud or self-hosted)
* [ ] Secure login and HTTPS enabled
* [ ] HTTP Trigger node created and URL copied
* [ ] Credentials set for required external APIs
* [ ] Ready to connect from your frontend web app

---

*Next: Move to Part 2 — Frontend App Setup when ready.*





# Part 1 Expanded — Docker vs Traditional Setup: Explainer & Best Practices

---

## What is Docker (and Docker Compose)?

**Docker** is a platform that lets you package applications (and all their dependencies, environment variables, and config) into "containers." A container is a self-contained mini-OS that runs your app the same way, everywhere — on your laptop, server, or in the cloud. It solves the "it works on my machine!" problem by shipping your whole environment together.

**Docker Compose** is a tool to define and run multi-container apps (like web server + database + cache) using a single YAML config file (`docker-compose.yml`).

---

## Docker vs Traditional (Bare Metal) Setup

### Traditional Setup

* You install Node.js (or Python, etc.) and all your app's dependencies directly onto the operating system (the "host").
* Configuration (environment variables, file paths, DB addresses) is managed by hand on each server.
* Upgrades or changes can break other apps on the same system.
* Security: If the app is compromised, it can affect the whole server.
* Deployment: Each machine needs to be configured individually, often manually.

### Docker Setup

* You write a `Dockerfile` and/or `docker-compose.yml` that defines everything your app needs — OS, libraries, variables, ports.
* Docker runs your app in its own container, isolated from the rest of the server.
* You can start, stop, copy, move, or duplicate the app with a single command.
* Easy to back up or transfer: just copy the folder and configs.
* Consistent: Same results on any computer, server, or cloud.

---

## Why Use Docker?

* **Isolation:** Each app runs in its own sandbox, no version conflicts.
* **Consistency:** Same setup and behavior everywhere.
* **Portability:** Move to another server or host (e.g., from laptop to Render.com) with zero code change.

  * **What does "zero code change" mean?** You do not need to change any source code, package versions, or system dependencies; the Docker image/container contains everything needed for the app. You may need to adjust environment variables or secrets for each environment (such as different API keys for production and development), but not your code or app dependencies.
* **Reproducibility:** All settings are in one file; spin up a new instance or restore from backup with confidence.
* **One-File Management:** Docker Compose lets you declare multiple services and all env variables in a single file — start everything with `docker-compose up -d`.
* **Rollback/Upgrades:** Stop or update containers with no fuss.

---

## Docker in the Cloud (Render.com Example)

* Services like Render.com let you "bring your own Docker container" or use their one-click n8n deploys.
* Instead of uploading your app and running install scripts, you just push your Docker setup (Dockerfile or `docker-compose.yml`).
* Render reads the config, builds the containers, and runs your app in isolation.
* Cloud env variables are set via the Render dashboard and injected securely (never hard-coded in the image).
* All dependencies and settings are part of the repo, so every deploy is consistent.

---

## Environment Variables and Secrets

* **Never hard-code secrets (like API keys, DB passwords) into your code or Dockerfiles.**
* Use `.env` files or the host/cloud provider's secret management.

### How Environment Variables Work: Traditional vs Docker

* **Traditional Setup:**

  * You usually store environment variables in a `.env` file at your project root.
  * Apps like Node.js load these values automatically with packages like `dotenv`.
  * You can also set variables manually in your shell (e.g., `export API_KEY=...`).

* **Docker Setup:**

  * You can define environment variables directly in the `docker-compose.yml` file under the `environment:` section.
  * However, it’s common to reference variables in the YAML using `${VAR_NAME}`. These values are loaded from a `.env` file in the same folder as your Compose file, or from the host system’s environment.
  * This means: **even with Docker, you usually keep a ********`.env`******** file** (but only on the server/host, never committed to your repo if it has secrets).
  * Why? This approach keeps secrets and settings outside your code and your Docker image, so you can change them without rebuilding the image or exposing secrets in your repository.
  * When deploying to a cloud service (like Render.com), you enter your environment variables via their dashboard (UI), and the platform securely injects them at runtime—again, your Docker image remains generic and safe.

#### **Summary Table**

|                  | Traditional                 | Docker Compose/YAML                |
| ---------------- | --------------------------- | ---------------------------------- |
| Env location     | `.env` file or shell export | `.env` file, Compose YAML, or host |
| Used by app      | Loaded by app (`dotenv`)    | Referenced in Compose as `${VAR}`  |
| In Docker image? | Never (shouldn’t be)        | Never (shouldn’t be)               |
| Cloud deploy     | Enter in dashboard/host     | Enter in dashboard/host            |

**Key Rule:**

> Docker Compose and traditional setups both use `.env` files, but Docker’s approach lets you keep all secrets/config outside the actual image and source code—making it safer for sharing code or open-source work.

### Expanded: Setting Environment Variables in Docker Compose

**What is YAML?**
YAML ("YAML Ain’t Markup Language") is a human-friendly format for writing configuration files. Docker Compose uses a file called `docker-compose.yml` to describe how to build and run all your app’s services and containers. Indentation and colons matter—each level of indentation is a “child” of the line above.

#### 1. **Directly in the Compose YAML**

(Values hardcoded—not recommended for secrets!)

```yaml
services:
  app:
    image: node:20
    environment:
      - NODE_ENV=production
      - API_URL=https://api.example.com
      - API_KEY=hardcoded-value-123    # Not secure for secrets!
```

* These values are visible to anyone with access to your YAML.
* Good for non-secret values (like `NODE_ENV`).

#### 2. **Referencing Variables from a ********`.env`******** File**

(**This is the secure, flexible, recommended way.**)

**In your ********`docker-compose.yml`********:**

```yaml
services:
  app:
    image: node:20
    environment:
      - NODE_ENV=${NODE_ENV}
      - API_URL=${API_URL}
      - API_KEY=${API_KEY}
```

**In your ********`.env`******** file (same folder as ********`docker-compose.yml`********):**

```
NODE_ENV=production
API_URL=https://api.example.com
API_KEY=super-secret-value-123
```

* When you run `docker-compose up`, Docker automatically reads `.env` and fills in the values.
* Swap secrets/configs per environment by changing the `.env` file—no need to change your Compose YAML or Docker image.

#### **Why Use This Pattern?**

* Keeps secrets/configs out of your repo (add `.env` to `.gitignore`)
* Lets you share Compose YAML publicly (e.g., on GitHub) with no risk of exposing secrets
* Makes it easy to use different configs for local, staging, or production by swapping `.env` files

#### **What if I Deploy to a Cloud Platform?**

* Platforms like Render.com, Heroku, Vercel, etc. let you define environment variables in their dashboard/UI.
* These are injected securely at runtime—your Docker image and code stay safe and generic.

#### **How does Docker Compose know to use the ********`.env`******** file?**

* **By default, Docker Compose automatically looks for a ********`.env`******** file in the same directory as your ********`docker-compose.yml`******** file.**

* You don’t need to specify anything extra—if the file exists, Compose loads all variables in it and makes them available for substitution (like `${MY_VAR}`) in your YAML.

* If you want to use a different env file (rare), you can use `--env-file`:

  ```bash
  docker-compose --env-file custom.env up -d
  ```

* **Best practice:**
  Just keep your `.env` file next to your `docker-compose.yml` file, add it to `.gitignore`, and Compose will use it automatically for all variable references.

* Pass values at runtime from the host, CI/CD system, or Render/Heroku dashboard.

* If uploading to a public repo, **exclude your ********`.env`******** files** and never commit secrets!

---

## Starting an App: Traditional vs Docker

### Traditional (Bare Metal)

1. Clone the repo
2. Install dependencies (e.g., `npm install`)
3. Set up environment variables (manually or via `.env`)
4. Start the app (`node app.js`, `n8n`, etc.)

### Docker Compose

1. Clone the repo (should include `docker-compose.yml`)
2. Set up environment variables (usually via a local `.env` or via the hosting provider)
3. Start everything with one command:

```bash
docker-compose up -d
```

4. All containers spin up (app, DB, etc.) — ready to use immediately.

   * **Each service you defined in your ********`docker-compose.yml`******** starts at the same time.**
   * For example: your main app (Node.js server), your database (Postgres, MySQL), and even helper services (like Redis, MailHog, or a task queue) all run together.
   * No need to install dependencies or set up databases manually—each service runs from its own image and config, with connections and ports mapped as defined.
   * Each service has its own isolated container (mini-environment) but can "talk" to other containers via Docker networking.
   * As soon as Docker Compose finishes, your entire stack is running, communicating, and production-ready within seconds (not minutes or hours).

---

### Running Multi-Service Apps (Frontend, Backend, DB) on Render.com

* **On Render.com, you do not typically run a full multi-service stack with ********`docker-compose up -d`******** like you would on your own VPS.**
  Instead, Render.com is designed around deploying each main service (frontend, backend, database) as a *separate Render Service*.

#### **How does this work in practice?**

* **1. One repo, multiple services:**
  If you have a monorepo with frontend, backend, and Docker Compose, you create a separate Render Service for each part:

  * One for the frontend (`web service`)
  * One for the backend (`web service` or `background worker`)
  * One for the database (managed Postgres, or bring-your-own with a Docker service)

* **2. Each service points to the relevant folder in your repo:**

  * For example, in the Render UI, you choose the folder and the Dockerfile (or the root for Docker Compose).
  * \*\*You do not use \*\***`docker-compose up`** directly; Render parses your Dockerfile or runs your specified start command (like `npm run start` for Node apps).

* **3. Start/Run commands:**

  * Each service (frontend, backend) has its own **start command** or **Dockerfile** specified in the Render dashboard.
  * For Node apps:

    * Start command might be `npm run start` or `node server.js` (set in package.json or Render UI).
  * For Dockerfile-based services:

    * Render builds the image from your Dockerfile and runs the CMD specified there.
  * For databases:

    * Render offers a managed Postgres service (no Dockerfile needed—you just connect via credentials), or you can run a DB inside your own container if advanced features are needed.

* **4. Environment variables:**

  * Each Render service has its own set of environment variables, configured in the dashboard for that service.
  * For example: your backend service might have `DATABASE_URL` and `API_KEY`, while your frontend may have its own public-facing variables.

* **5. Routing & networking:**

  * Render automatically provides HTTPS URLs for each web service (e.g., `https://your-backend.onrender.com`).
  * Your frontend can call your backend using this URL, and your backend can connect to the managed Postgres with the provided connection string.

#### **Summary Table: Multi-Service Deployment on Render**

| Service            | Example Render Type  | Start Mechanism          | Folder/Repo Structure    |
| ------------------ | -------------------- | ------------------------ | ------------------------ |
| Frontend           | Web Service          | `npm run start` / Docker | `/frontend` or repo root |
| Backend            | Web Service / Worker | `npm run start` / Docker | `/backend` or repo root  |
| Database           | Managed Postgres     | N/A (managed by Render)  | N/A                      |
| Extra (Redis, etc) | Private Service      | Docker                   | `/redis` or `/infra`     |

#### **Key Differences from VPS/Docker Compose**

* On a VPS, you run `docker-compose up` once and all services start together, talking via Docker networking.
* On Render, each service is a separate deployment; you manage and start each independently via their dashboard/UI, each with its own build/start logic.
* You must configure URLs and environment variables so services know how to reach each other.

#### **Best Practices**

* Organize your repo so each service has its own clear folder and Dockerfile (if needed).
* Set all secrets and service-specific environment variables in the Render dashboard, never in your code or repo.
* Use Render’s managed database service when possible for easier, scalable Postgres hosting.
* For multi-service monorepos, check Render’s docs on [monorepo support](https://render.com/docs/monorepos).

---

### 📁 Example File Tree Structures

#### **A. n8n Backend + Frontend (Monorepo, Dockerized for Compose and Render)**

```
my-app/
├── frontend/
│   ├── Dockerfile         # Builds the frontend (e.g., React/Vite)
│   ├── package.json
│   ├── src/
│   └── ...
├── backend-n8n/
│   ├── Dockerfile         # (Optional) Customizes n8n, or just use official image
│   └── ...
├── infra/
│   ├── docker-compose.yml # Main Compose file for multi-service dev
│   ├── .env               # Shared env variables (never committed)
│   └── README.md
├── README.md
└── .gitignore

```

* **infra/**: This folder holds all infrastructure-level code/config, separating your app code (frontend/backend) from deployment files. This is where you put your Compose files, devops scripts, and anything related to the “plumbing” or environment setup for your stack.

* **What is a Dockerfile?**

  * The `Dockerfile` describes the build steps to create the image for a given service.
  * The Compose file (`docker-compose.yml`) references each service’s Dockerfile (or pulls a public image).

#### **B. Custom Backend (Express, Django, etc) + Frontend (No n8n)**

```
my-app/
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── ...
├── backend/
│   ├── Dockerfile        # Builds custom backend (Node, Python, etc)
│   ├── package.json / requirements.txt
│   └── ...
├── infra/
│   ├── docker-compose.yml
│   ├── .env
│   └── README.md
└── .gitignore

```

* In this version, your backend folder contains your app (Express, FastAPI, Django, etc) and its own Dockerfile.

---

### 🟩 **How Dockerfile and Compose Work Together**

* **Each service** (frontend, backend, n8n) has its own Dockerfile specifying how to build that image.
* **`docker-compose.yml`** (in `infra/`) references these images:

  * It can build them from the Dockerfiles (using `build: ../frontend`), or use an official/prebuilt image (like `n8nio/n8n`).
  * All environment variables, networking, volumes, and port mappings are also declared in the Compose file.
* **Why keep Compose in ********`infra/`********?**

  * This makes your root repo clean and separates “infrastructure as code” from app code.
  * In multi-repo setups, `infra/` can act as the root for deployments (especially on a VPS or with CI/CD).

---

### 🟧 **How does Render.com handle service roots and infra files?**

* **On Render.com:**

  * **Each Render service requires a root path**—this is where it looks for a Dockerfile or start command.
  * If you set your Render “web service” root as `/frontend`, it builds and runs only what’s in the frontend folder (ignoring Compose in `/infra`).
  * **You cannot point a single Render service at a Compose file inside ********`/infra/`******** to spin up the whole stack** (unlike a VPS).
  * Instead, **create a separate Render service for each part (frontend, backend, n8n)**, each with its own root/folder and Dockerfile.

* **Referencing “upwards” from service roots:**

  * Render builds your service in the context of the root folder you specify, so it **cannot reach “up” into parent folders** for files like a Compose in `/infra`.
  * **Best practice:** Keep any code or config that a service needs *inside* that service’s folder, or copy shared files in during the Docker build.

#### **Practical implications:**

* **Local development:**

  * You can use Compose in `infra/` to spin up everything with `docker-compose up`.
* **Render deployment:**

  * You set up separate services for `/frontend`, `/backend-n8n`, `/backend`, etc.
  * If you want the benefits of Compose (shared .env, managed network), you only get that locally, not on Render.
  * Render’s own dashboard acts as your environment variable, secret, and network manager.

---

## Expanded: Docker Portability, Containers, Images, and Hosting Structure

### Portability: What Does “Zero Code Change” Mean?

* In Docker, “portability” means you can copy your app’s folder (with its Docker config and Compose YAML) to any other computer or cloud service that runs Docker, and it will work exactly the same—no changes to code, dependencies, or settings inside the container.
* **Environment variables:** You may need to set or adjust environment variables depending on the new host (for example, API keys, database URLs, or ports). But you never have to change your code or reinstall dependencies—just provide the new values via `.env` file or your cloud provider’s dashboard.
* **Version changes:** If you specify a Docker image version (e.g., `n8nio/n8n:1.28.0`), you always get that version unless you change the tag. This gives you reproducibility across machines.

### What Is a “Container” in Docker?

* A container is like a lightweight, isolated mini-computer that runs your app with its own private OS, libraries, and settings.
* Imagine a glass box: your app runs inside, unaffected by what’s on the rest of the server, and it can’t (normally) mess with the outside world.
* Why use containers?

  * You can run several apps, each with different versions of Node.js, Python, etc., side-by-side, with no conflicts.
  * If something breaks in one container, it doesn’t crash the whole server.
* Key point:

  * You still need to host your backend and frontend as separate containers if they are separate apps, but Docker Compose can manage both together.
  * On Render.com and similar, you typically set up one service per container/repo, but Docker Compose can orchestrate multi-service setups on your own VPS.

### Images, Versions, and “Docker Images” Explained

* A Docker “image” is a snapshot of an app and its environment. Think of it as a ready-to-go app blueprint or template (like a .zip file, but executable).

  * **Compared to Neon Postgres:** Neon is a database service with point-in-time history for your database. Docker images, by contrast, are not live databases—they are blueprints for starting containers (which may then connect to a database like Neon or Postgres running in another container).
  * **Compared to Git:** Git tracks source code and its history over time; you can revert, branch, and merge. Docker images are static—once built, they do not have internal history or branching. If you want to "revert" your app in Docker, you use a different image version (or tag). Changes to your code are usually built into new images, not tracked in the image itself.
  * **Key difference:** Docker images are for reproducible runtime environments; Git is for code, and Neon is for your database (and its history).
  * Example: `n8nio/n8n:1.28.0` is the official n8n image, version 1.28.0.
* Images are versioned (with tags): you can pin to a version (`:1.28.0`) for stability or use `:latest` for the newest, but this can lead to surprises if the latest changes.
* When you “run” an image, Docker spins up a new container from that image.
* Is it like git or a DB with history?

  * Not exactly.
  * Docker images are static (no internal history or branching like git/neon db).
  * You store/use as many images/versions as you like; switching is as simple as changing the version tag in your YAML and re-running.
  * There is no built-in undo/history of your app’s data—data lives outside the image, in volumes or cloud DBs.

### Hosting Backend & Frontend with Docker (On Render, VPS, etc.)

* If frontend and backend are separate apps, you’ll often build a Docker image/container for each.

  * On platforms like Render.com, you usually create a separate “service” for each containerized app (e.g., one for the frontend, one for the backend).
  * On your own VPS with Docker Compose, you can define both services in one file and spin them up together.
* Implication:

  * Docker doesn’t “merge” backend and frontend into one app; it just makes it easy to manage both as isolated, portable services.

---

## Implications for Hosting, Maintenance, and Security (Expanded)

### Upgrades

* Upgrading your app in Docker means:

  * Change the image version/tag in your YAML (or Dockerfile).
  * Pull the new image: `docker-compose pull` (if needed).
  * Restart: `docker-compose up -d` — Docker swaps out the old container for the new one.
* Result:

  * No leftover junk from old installs.
  * Always a clean, reproducible setup.

### Rollback

* To revert to an older version:

  * Change the image tag back in your Compose file.
  * Run `docker-compose up -d` again.
  * Docker runs the specified (older) image—super quick, no “downgrading” headaches.

### Moving to Another Server

* Just copy your project folder (including `docker-compose.yml`, Dockerfiles, etc.) to a new machine.
* Set up Docker, set any needed environment variables, and run the same command.
* No manual dependency install or “works on my machine” issues.

### Cloud Deployment

* Cloud services like Render.com:

  * Accept your Dockerfile or Compose YAML.
  * Handle all OS and dependency install automatically.
  * You manage env vars via their dashboard (never in the image or code).

### Security

* Isolation:

  * Each container is sandboxed: if one is hacked, the host and other containers are much safer.
  * Never run containers as root unless absolutely necessary.
* Base image hygiene:

  * Use official, updated images; rebuild regularly to pull in OS-level security patches.
* Expose only necessary ports:

  * Only open what the app needs; block everything else.
* Secrets management:

  * Never put API keys/passwords in code, Dockerfile, or public repo—always use env vars set securely on the host/cloud.

---

## TL;DR Cheat Sheet

|          | Docker Compose                       | Traditional Node.js             |
| -------- | ------------------------------------ | ------------------------------- |
| Install  | One command (`docker-compose up -d`) | Multiple manual steps           |
| Config   | One YAML file, env vars separate     | .env file or shell, config.json |
| Update   | Change YAML, re-run up               | Manual npm/node update          |
| Move     | Copy folder, re-run                  | Reinstall all dependencies      |
| Security | Container isolation, less risk       | Direct to host OS, more risk    |
| Cloud    | Same as local, just upload repo      | Each cloud needs own config     |

---

## Final Notes for Beginners

* If you’ve never used Docker: Think of it as a "portable app box" that brings all its own tools — you never worry about OS version, system libraries, or conflicting projects again.
* If you’re used to traditional setups: Docker makes complex multi-service apps (like n8n + Postgres + Redis + Caddy) much simpler to manage, back up, move, and upgrade. All your config lives in version control (except for secrets, which must stay private!).
* For team projects or open source: Docker/Compose means everyone gets the *exact* same dev and prod setup.

---

*Use this explainer to decide which path is right for your n8n deployment, and keep it handy as you learn or teach Docker to others!*

---

## 🚀 Example: n8n-Only Docker Compose Stack (Custom Dockerfile Version)

This template demonstrates how to run n8n as its own container using a custom Dockerfile. Use this as a base for advanced customizations (e.g., preloading workflows, extending the n8n image, or adding credentials/scripts).

### `/backend-n8n/Dockerfile`

```Dockerfile
FROM n8nio/n8n:latest

# Example: Add custom workflows or credentials by copying into the image
COPY workflows/ /home/node/.n8n/workflows/
COPY credentials/ /home/node/.n8n/credentials/
# You could add custom scripts or node modules if needed

# Optionally install extra packages or tools
# RUN npm install some-custom-n8n-package

EXPOSE 5678
```

### `/infra/docker-compose.yml`

```yaml
version: "3.8"

services:
  n8n-backend:
    build: ../backend-n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASS}
      - N8N_HOST=localhost
      - N8N_PROTOCOL=http
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

### `/infra/.env`

```
N8N_USER=admin
N8N_PASS=supersecurepassword
```

---





# Docker Project Examples – Specialized Templates

## 1️⃣ Standard Custom Frontend + Backend (Node/Express Example)

### `/infra/docker-compose.yml`

```yaml
version: "3.8"

services:
  frontend:
    build: ../frontend
    ports:
      - "3000:3000"
    environment:
      - VITE_API_URL=${VITE_API_URL}
      - VITE_PUBLIC_KEY=${VITE_PUBLIC_KEY}
    depends_on:
      - backend

  backend:
    build: ../backend
    ports:
      - "5000:5000"
    environment:
      - NODE_ENV=${NODE_ENV}
      - API_KEY=${API_KEY}
      - PORT=${PORT}
      - DATABASE_URL=${DATABASE_URL}
      - JWT_SECRET=${JWT_SECRET}
      # Add any 3rd-party API secrets here

volumes:
  # Optional: for persistent backend data
```

### `/frontend/Dockerfile`

```Dockerfile
FROM node:20 AS build
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 3000
CMD ["nginx", "-g", "daemon off;"]
```

### `/backend/Dockerfile`

```Dockerfile
FROM node:20
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 5000
CMD ["node", "index.js"]
```

### `/infra/.env`

```
# Frontend
VITE_API_URL=http://localhost:5000
VITE_PUBLIC_KEY=public-frontend-value

# Backend
API_KEY=your-backend-secret-key
NODE_ENV=development
PORT=5000
DATABASE_URL=postgres://myuser:mypassword@localhost:5432/mydb
JWT_SECRET=supersecretjwt
# 3rd-party API keys (Mailgun, S3, etc.)
# MAILGUN_API_KEY=...
```

*Variable explanations:*

* `VITE_API_URL`: Frontend uses this to call backend APIs
* `VITE_PUBLIC_KEY`: Publicly exposed value for frontend, safe to expose
* `API_KEY`: Backend-only secret (never in frontend)
* `DATABASE_URL`: Full connection string for Postgres or other DB
* `JWT_SECRET`: For backend JWT authentication
* `PORT`, `NODE_ENV`: Standard runtime config

---

## 2️⃣ Frontend + Backend n8n (Frontend Talks to n8n REST Agent)

### `/infra/docker-compose.yml`

```yaml
version: "3.8"

services:
  frontend:
    build: ../frontend
    ports:
      - "3000:3000"
    environment:
      - VITE_N8N_API_URL=${VITE_N8N_API_URL}
      - VITE_PUBLIC_KEY=${VITE_PUBLIC_KEY}
    depends_on:
      - n8n-backend

  n8n-backend:
    image: n8nio/n8n:latest
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=${N8N_BASIC_AUTH_ACTIVE}
      - N8N_BASIC_AUTH_USER=${N8N_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASS}
      - N8N_HOST=${N8N_HOST}
      - N8N_PROTOCOL=${N8N_PROTOCOL}
      - N8N_PORT=${N8N_PORT}
      # Add any integration API keys here
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

### `/frontend/Dockerfile`

*Same as above.*

### `/infra/.env`

```
# Frontend
VITE_N8N_API_URL=http://localhost:5678
VITE_PUBLIC_KEY=public-frontend-value

# n8n Backend
N8N_BASIC_AUTH_ACTIVE=true
N8N_USER=admin
N8N_PASS=supersecurepassword
N8N_HOST=localhost
N8N_PROTOCOL=http
N8N_PORT=5678
# Example: Airtable/Google integration
# AIRTABLE_API_KEY=...
# GOOGLE_CLIENT_ID=...
# GOOGLE_CLIENT_SECRET=...
```

*Variable explanations:*

* `VITE_N8N_API_URL`: Frontend uses this to call n8n API
* `N8N_USER`, `N8N_PASS`: Secures n8n instance
* `N8N_HOST`, `N8N_PROTOCOL`, `N8N_PORT`: Network config
* Add API keys/secrets for integrations as needed

---

## 3️⃣ n8n Custom Build (Advanced: Custom Dockerfile for n8n Service)

### `/backend-n8n/Dockerfile`

```Dockerfile
FROM n8nio/n8n:latest

# Example: Add custom workflows or credentials by copying into the image
COPY workflows/ /home/node/.n8n/workflows/
COPY credentials/ /home/node/.n8n/credentials/
# You could add custom scripts or node modules if needed

# Optionally install extra packages or tools
# RUN npm install some-custom-n8n-package

EXPOSE 5678
```

### `/infra/docker-compose.yml`

```yaml
version: "3.8"

services:
  n8n-backend:
    build: ../backend-n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=${N8N_BASIC_AUTH_ACTIVE}
      - N8N_BASIC_AUTH_USER=${N8N_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASS}
      - N8N_HOST=${N8N_HOST}
      - N8N_PROTOCOL=${N8N_PROTOCOL}
      - N8N_PORT=${N8N_PORT}
      # If using an external DB:
      # - DB_TYPE=postgresdb
      # - DB_POSTGRESDB_HOST=${DB_POSTGRESDB_HOST}
      # - DB_POSTGRESDB_PORT=${DB_POSTGRESDB_PORT}
      # - DB_POSTGRESDB_DATABASE=${DB_POSTGRESDB_DATABASE}
      # - DB_POSTGRESDB_USER=${DB_POSTGRESDB_USER}
      # - DB_POSTGRESDB_PASSWORD=${DB_POSTGRESDB_PASSWORD}
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

### `/infra/.env`

```
# n8n
N8N_BASIC_AUTH_ACTIVE=true
N8N_USER=admin
N8N_PASS=supersecurepassword
N8N_HOST=localhost
N8N_PROTOCOL=http
N8N_PORT=5678

# Optional: Postgres config for external DB
# DB_POSTGRESDB_HOST=db
# DB_POSTGRESDB_PORT=5432
# DB_POSTGRESDB_DATABASE=n8n
# DB_POSTGRESDB_USER=n8nuser
# DB_POSTGRESDB_PASSWORD=yourpassword
```

*Variable explanations:*

* Same as above, plus Postgres DB vars if used.

---

## 🔑 Usage Tips

---

## 🌐 Example: Expanded `.env` and Variable Management

Below is a more complete example `.env` with common variables for a real-world stack. Remember—never commit secrets to your repository!

```env
# Frontend
VITE_API_URL=http://localhost:5000
VITE_N8N_API_URL=http://localhost:5678

# Backend
API_KEY=your-api-key-here
NODE_ENV=development
PORT=5000
DATABASE_URL=postgres://myuser:mypassword@localhost:5432/mydb
JWT_SECRET=supersecretjwt

# n8n
N8N_USER=admin
N8N_PASS=supersecurepassword
N8N_HOST=localhost
N8N_PROTOCOL=http
N8N_PORT=5678
N8N_BASIC_AUTH_ACTIVE=true
# n8n Postgres (uncomment if using external db)
# DB_TYPE=postgresdb
# DB_POSTGRESDB_HOST=db
# DB_POSTGRESDB_PORT=5432
# DB_POSTGRESDB_DATABASE=n8n
# DB_POSTGRESDB_USER=n8nuser
# DB_POSTGRESDB_PASSWORD=yourpassword

# Example: OAuth/3rd party
# GOOGLE_CLIENT_ID=...
# GOOGLE_CLIENT_SECRET=...
# AIRTABLE_API_KEY=...
```

### 🛡️ Render.com Environment Variable Management

* **On Render, you do not use a **\`\`** file directly for each deployed service.**
* Instead, after creating each service (frontend, backend, n8n), go to the service’s **"Environment"** tab in the dashboard and add the variables there (name and value for each).
* Render injects these into the environment at runtime, so your app, Dockerfile, or Compose can use them as if they were in a `.env` file.
* **Best practice:**

  * Use your local `.env` only for local dev/testing.
  * Manually add all required vars in the Render UI (per service!) for production.
  * Never upload the `.env` file to the repository.
* You can download a sample `.env` for reference, but secrets must always be set via the Render dashboard for security.

---

* Place your `.env` file in the `/infra` directory and never commit with secrets.
* For local dev, run `docker-compose up -d` from the `/infra` folder.
* For Render.com or similar: Deploy each service as a separate Render Service, with each pointing to its own folder and Dockerfile.
* Edit Dockerfiles to suit your language/framework as needed (Python, Go, Next.js, etc.).
