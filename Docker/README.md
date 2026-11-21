# Cloud DevOps Project - Microservices Architecture

##  Project Overview

This repository contains an end-to-end cloud-native microservices web application. The platform provides user authentication (sign up & login) and displays a comprehensive **DevOps Roadmap** upon successful authentication.

The architecture is distributed across independent microservices containerized with Docker and orchestrated locally using Docker Compose.

---

##  System Architecture

```text
                           +----------------------+
                           |   Client / Browser   |
                           +----------+-----------+
                                      |
                                      | HTTP (:3001)
                                      v
                           +----------------------+
                           |   Frontend Service   |
                           |     (Node.js/EJS)    |
                           +-----+----------+-----+
                                 |          |
            HTTP POST (:5000)    |          |   HTTP GET (:8080)
            +--------------------+          +---------------------+
            |                                                     |
            v                                                     v
+-----------------------+                             +------------------------+
|     Auth Service      |                             |    Roadmap Service     |
|    (Python / Flask)   |                             |  (Java / Spring Boot)  |
+-----------+-----------+                             +------------------------+
            |
            | TCP (:3306)
            v
+-----------------------+
|    MySQL Database     |
|   (Relational Data)   |
+-----------------------+
```

---

##  Microservices Specification

| Service | Technology Stack | Container Port | Host Port | Responsibility |
| :--- | :--- | :--- | :--- | :--- |
| **Frontend** | Node.js (Express & EJS) | `3000` | `3001`| Web UI, session handling, calls backend APIs |
| **Auth Service** | Python 3.12 (Flask) | `5000` | `5000` | User registration & authentication with bcrypt |
| **Roadmap Service** | Java 21 (Spring Boot) | `8080` | `8080` | Provides DevOps roadmap content API |
| **Database** | MySQL 8.0 | `3306` | `3306` | Persistent relational storage for user accounts |

---

##  Local Setup & Testing (Task 2)

### 1. Prerequisites
Ensure you have the following installed locally:
- [Git](https://git-scm.com/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (running with Linux containers)

### 2. Clone the Repository
```bash
git clone https://github.com/Ibrahim-Adel15/iVolveFinalProject.git CloudDevOpsProject
cd CloudDevOpsProject
```

### 3. Run with Docker Compose
To build and start all containers in detached mode:
```bash
docker compose up --build -d
```

### 4. Verify Running Services
Check container health and statuses:
```bash
docker compose ps
```

### 5. Access the Application
Open your web browser and navigate to:
- **Frontend Web UI:** `http://localhost:3001`
- **Auth Service Health Check:** `http://localhost:5000/health`

---

##  Testing Scenarios

1. **Sign Up:**
   - Go to `http://localhost:3001/signup`.
   - Create a user account (minimum 3 characters username, 8 characters password).
2. **Login:**
   - Log in with your registered credentials.
3. **Roadmap Verification:**
   - Upon successful login, the application redirects to `/roadmap`.
   - Verify that roadmap modules are loaded via the `roadmap-service`.

---

### Stop and Clean Up
To stop the application and clean up containers:
```bash
docker compose down
```
To remove persistent database volumes as well:
```bash
docker compose down -v
```
