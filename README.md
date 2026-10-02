# Chronic-wannabees Application - Code Repository

This repository contains the source code, local development configuration, and build instructions with docker for the Chronic-wannabes application (Flask/FastAPI) and database MYSQL on azure.

## 📋 Table of Contents
- [Project Structure](#-project-structure)
- [Architecture](#️-architecture)
- [Local Development (App , Dockercompose)](#-local-developement)
- [CI/CD Workflow &  Architecture](#-cicd-pipeline-workflow)

---

## 🌳 Project Structure

```text
.
├── .gitlab-ci.yml          # CI pipeline to build & push Docker images
├──  application.md        # Application documentation
├── .gitignore              # gitignore file 
├── docker-compose.yml      # Local development multi-container setup
├── Dockerfile              # Container build instructions
└── requirements.txt        # Python dependencies version for the app
└─── app/                   # Python source code
    ├── app.py              # Application entry point
    ├── db.py               # Connexion the app to database 


```

## 🏛️ Architecture

![Architecture Application](https://i.imgur.com/nNRPu3r.jpeg)


# 🌍 Local Developement

This is a simple FastAPI web application that displays a user form with a dynamic typewriter animation and processes the inputs to return a personalized page. It also includes background utility endpoints to instantly check whether the server is online and if the database connection is working properly.

### 🐍 The`Application` 

It is a simple FastAPI website that does three things:

`The Home Page (/)`: Shows a clean form asking for a First Name and Last Name. It features animated typewriter effect that says "Welcome, Cosmopolitan Team !".

`The Welcome Page (/submit)`: When you submit the form, it takes you to a new page that says "Bonjour [Name] ! 👋" with a back button.

The Behind-the-Scenes Checks:

`/health` checks if the website is online.

`/db-test` checks if your database is connected properly.


### 📄 The `docker-compose.yml` 

We use **Docker Compose** to orchestrate both the Python application container and the official MySQL database container locally. This ensures a flawless local parity before deploying to Azure Container Apps.

We create  `docker-compose.yml` file at the root of your project with the following configuration:

```yaml
-compose.yml
365 B
services:
  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_DATABASE: myapp
      MYSQL_ROOT_PASSWORD: password_secret
    ports:
      - "3306:3306"

  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      DB_HOST: db
      DB_USER: root
      DB_PASSWORD: password_secret
      DB_NAME: myapp
    depends_on:
      - db
```

# 🔄 CI/CD Pipeline Workflow

This repository includes a fully automated **GitLab CI/CD** pipeline (`.gitlab-ci.yml`) designed to test , build, and push our application container to microsoft azure. It operates under a decoupled multi-repository architecture, meaning this codebase focus solely on application delivery after the triggering of the infrastructure deployment.

### 👥 Pipeline Stages

```text

test
  ├── GitLab SAST
  └── flake8

build
  └── Validate Docker image

deploy
   ├── Deploy to Staging (manual)
   │     ├── Azure Login (OIDC)
   │     ├── Docker Build
   │     ├── Push image to Azure Container Registry
   │     └── Update Azure Container App
   │
   └── Deploy to Production (automatic on main)
         ├── Azure Login (OIDC)
         ├── Docker Build
         ├── Push image to Azure Container Registry
         └── Update Azure Container App
```

The workflow is divided into three major stages executed sequentially:

1. **Lint & Test:** Analyzes the Python source code using code quality tools flake8 to ensure syntax compliance and adherence to clean coding standards.

2. **The Build** The pipeline packages the application into a box (a Docker image) containing everything it needs to run. At this stage, it simply validates that the box builds successfully without errors.

3. **Build & Push:** Triggers automatically on the `staging` and `main` branches. It builds the specialized multi-stage Docker. 


### 🛡️ Credential and Secret Governance

Following strict security mandates and cloud compliance:
* **No Cloud Secrets Stored:** This pipeline does not store long-lived Azure service principal passwords or client secrets inside GitLab variables.

* **OIDC Authentication:** GitLab CI/CD authenticates dynamically with Microsoft Azure using **OpenID Connect (OIDC)**. Azure Entra ID trusts the temporary, cryptographic JSON Web Token (JWT) issued by GitLab's runner, granting short-lived access exclusively for pushing the Docker image to the ACR.

* **Runtime Secrets:** Database passwords and environment configurations are kept hidden and masked within GitLab's CI environment variables, which are then injected as encrypted variables straight into the Azure Container Apps container runtime.