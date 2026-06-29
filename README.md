# 🛒 Emart App — Containerized Microservices on AWS EC2 

> A full-stack **e-commerce web application** built with a microservices architecture, fully containerized using **Docker & Docker Compose**, and deployed on an **AWS EC2 instance**.
 
---

## 📋 Table of Contents

- [Overview](#-overview) 
- [Architecture](#-architecture)
- [Services](#-services)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Prerequisites](#-prerequisites)
- [Run Locally with Docker Compose](#-run-locally-with-docker-compose)
- [Deploy on AWS EC2](#-deploy-on-aws-ec2)
- [Nginx API Gateway](#-nginx-api-gateway)
- [Service Ports](#-service-ports)
- [Environment Variables](#-environment-variables)

---

## 🌐 Overview

**Emart App** is a containerized e-commerce platform that uses a **microservices architecture** to separate the frontend, backend APIs, and databases into independent Docker containers.

The entire application is orchestrated using **Docker Compose** and was built and deployed on an **AWS EC2 instance**. Nginx acts as a reverse proxy, routing all incoming traffic to the appropriate service through a single entry point on port `80`.

---

## 🏗️ Architecture

```
  Browser
    │
    ▼
┌─────────────────────────────────────┐
│           AWS EC2 Instance          │
│                                     │
│  ┌──────────────────────────────┐   │
│  │     NGINX  (Port 80)         │   │
│  │       API Gateway            │   │
│  └────┬──────────┬──────────────┘   │
│       │          │                  │
│  /    │    /api  │    /webapi       │
│       ▼          ▼          ▼       │
│  ┌─────────┐ ┌──────────┐ ┌──────┐ │
│  │ Angular │ │ Node.js  │ │ Java │ │
│  │ Client  │ │ Emart API│ │ Book │ │
│  │  :4200  │ │  :5000   │ │ API  │ │
│  └─────────┘ └────┬─────┘ │:9000 │ │
│                   │       └──┬───┘ │
│              ┌────▼───┐  ┌───▼───┐ │
│              │MongoDB │  │ MySQL │ │
│              │:27017  │  │ :3306 │ │
│              └────────┘  └───────┘ │
└─────────────────────────────────────┘
```

---

## 🔧 Services

The application runs **6 containers**, each handling a distinct role:

| Service | Container Name | Technology | Port | Description |
|---|---|---|---|---|
| **Angular Client** | `client` | Angular 12 + Nginx | `4200` | Frontend single-page application |
| **Emart API** | `api` | Node.js 14 + Express | `5000` | REST API for users & shop |
| **Book API** | `webapi` | Java 8 + Spring Boot | `9000` | REST API for book management |
| **NGINX Gateway** | `nginx` | Nginx | `80` | Reverse proxy / API gateway |
| **MongoDB** | `emongo` | MongoDB 4 | `27017` | Database for Emart API |
| **MySQL** | `emartdb` | MySQL 8.0.33 | `3306` | Database for Book API |

---

## ✅ Prerequisites

Ensure the following are installed on your machine or EC2 instance:

- [Docker](https://docs.docker.com/get-docker/) `v20.10+`
- [Docker Compose](https://docs.docker.com/compose/install/) `v2.0+`
- [Git](https://git-scm.com/)

---

## 🚀 Run Locally with Docker Compose

### 1. Clone the Repository

```bash
git clone https://github.com/hurairahh/Emart-App-containerization-project.git
cd emartapp-main
```

### 2. Build and Start All Containers

```bash
docker-compose up --build
```

This single command will:

1. Build the **Angular client** image (Node.js build → Nginx)
2. Build the **Node.js Emart API** image
3. Build the **Java Book API** image (Maven build → JRE)
4. Pull the official **Nginx**, **MongoDB**, and **MySQL** images
5. Start all 6 containers in the correct dependency order

### 3. Access the Application

| URL | Description |
|---|---|
| `http://localhost` | Full application via Nginx (port 80) |
| `http://localhost:4200` | Angular frontend (direct) |
| `http://localhost:5000` | Node.js Emart API (direct) |
| `http://localhost:9000` | Java Book API (direct) |

### 4. Stop the Application

```bash
docker-compose down
```

To also remove database volumes (**⚠️ deletes all data**):

```bash
docker-compose down -v
```

---

## ☁️ Deploy on AWS EC2

### Step 1 — Launch an EC2 Instance

- **AMI:** Ubuntu 22.04 LTS (recommended)
- **Instance type:** `t3.medium` or higher (at least 2 vCPU, 4 GB RAM)
- **Security Group — open the following inbound ports:**

| Port | Protocol | Purpose |
|---|---|---|
| `22` | TCP | SSH access |
| `80` | TCP | Nginx / Application |
| `4200` | TCP | Angular client (optional direct access) |
| `5000` | TCP | Node.js API (optional direct access) |
| `9000` | TCP | Java Book API (optional direct access) |

### Step 2 — Install Docker on EC2

SSH into your instance and run:

```bash
# Update packages
sudo apt-get update -y

# Install Docker
sudo apt-get install -y docker.io

# Install Docker Compose
sudo apt-get install -y docker-compose

# Add current user to docker group (no sudo needed)
sudo usermod -aG docker $USER
newgrp docker
```

### Step 3 — Clone the Repository

```bash
git clone https://github.com/hurairahh/Emart-App-containerization-project.git
cd emartapp-main
```

### Step 4 — Build and Run

```bash
docker-compose up --build -d
```

The `-d` flag runs all containers in **detached mode** (background).

### Step 5 — Access the Application

Open your browser and navigate to your EC2 instance's public IP:

```
http://<EC2-Public-IP>
```

### Useful Commands on EC2

```bash
# Check running containers
docker ps

# View logs for a specific service
docker-compose logs -f api
docker-compose logs -f webapi
docker-compose logs -f client

# Restart all services
docker-compose restart

# Stop and remove all containers
docker-compose down
```

---

## 🌐 Nginx API Gateway

Nginx listens on **port 80** and proxies requests to the correct container based on the URL path.

**Config file:** [`nginx/default.conf`](./nginx/default.conf)

| Request Path | Proxied To | Service |
|---|---|---|
| `/` | `http://client:4200` | Angular Frontend |
| `/api` | `http://api:5000` | Node.js Emart API |
| `/webapi` | `http://webapi:9000` | Java Book API |

---

## 🔌 Service Ports

| Container | Image | Host Port | Container Port |
|---|---|---|---|
| `nginx` | `nginx:latest` | `80` | `80` |
| `client` | Built locally | `4200` | `4200` |
| `api` | Built locally | `5000` | `5000` |
| `webapi` | Built locally | `9000` | `9000` |
| `emongo` | `mongo:4` | `27017` | `27017` |
| `emartdb` | `mysql:8.0.33` | `3306` | `3306` |

---

## 🔑 Environment Variables

These are configured directly in [`docker-compose.yaml`](./docker-compose.yaml):

### MongoDB (`emongo`)

| Variable | Value |
|---|---|
| `MONGO_INITDB_DATABASE` | `epoc` |

### MySQL (`emartdb`)

| Variable | Value |
|---|---|
| `MYSQL_ROOT_PASSWORD` | `emartdbpass` |
| `MYSQL_DATABASE` | `books` |

> ⚠️ **Note:** For production use, avoid hardcoding credentials. Use environment files (`.env`) or a secrets management tool.

---

<div align="center">
  <strong>Built with ❤️ — Docker | Angular | Node.js | Java Spring Boot | Nginx | AWS EC2</strong>
</div>
