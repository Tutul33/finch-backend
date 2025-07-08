# CI/CD Setup Documentation

## Overview

This document outlines the Continuous Integration and Continuous Deployment (CI/CD) setup for the **Finch** project, using **GitHub Actions** as the CI/CD tool. The project is split into two repositories: **frontend** and **backend**, each with its own workflow configuration.

GitHub Actions automates the process of testing, building, linting, and deploying Docker images for both frontend and backend applications upon code changes.

---

## CI/CD Tool: GitHub Actions

GitHub Actions is chosen due to its:

- Native integration with GitHub repositories.
- Support for custom workflows.
- Pre-built actions for Docker, Node.js, etc.
- Secure secret management.
- Easy scalability and minimal configuration overhead.

---

## Workflow Files

### 1. Backend CI Workflow

**File:** `.github/workflows/backend-ci.yml`  
**Trigger:** On push to the `master` branch.

#### Workflow Stages:

| Stage                        | Description                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| Checkout Code               | Clones the repository using `actions/checkout@v4`.                          |
| Set up Docker Buildx        | Prepares Docker Buildx for multi-platform builds.                          |
| Login to Docker Hub         | Uses secrets to log in securely.                                           |
| Build & Push Docker Image   | Builds the Docker image and pushes it to Docker Hub (`finch-backend:latest`). |

---

### 2. Frontend CI Workflow

**File:** `.github/workflows/frontend-ci.yml`  
**Trigger:** On push or pull request to the `master` branch.

#### Workflow Stages:

| Stage                        | Description                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| Checkout Code               | Uses `actions/checkout@v3` to get source files.                             |
| Setup Node.js Environment   | Installs Node.js 16 using `setup-node@v3`.                                  |
| Install Dependencies        | Uses `npm ci` to install dependencies from `package-lock.json`.             |
| Lint Code                   | Runs code linting via `npm run lint`.                                       |
| Run Unit Tests              | Executes unit tests using `npm run test:unit`.                              |
| Build Production Assets     | Compiles frontend assets with `npm run build`.                              |
| Login to Docker Hub         | Logs in using credentials from GitHub Secrets.                              |
| Build & Push Docker Image   | Builds and pushes the Docker image (`finch-frontend:latest`).               |

---

## Secrets Configuration

Both workflows use the following GitHub repository secrets:

- `DOCKER_USERNAME` – Your Docker Hub username.
- `DOCKER_PASSWORD` – Your Docker Hub password or access token.

You can set these in your repository under:  
**Settings → Secrets and variables → Actions → New repository secret**

---

## Triggering Pipelines

### Frontend Workflow

| Trigger       | Description                     |
|---------------|---------------------------------|
| `git push` to `master` | Automatically runs the CI workflow.        |
| `pull_request` to `master` | Automatically triggers testing and linting. |

### Backend Workflow

| Trigger       | Description                     |
|---------------|---------------------------------|
| `git push` to `master` | Automatically runs build and Docker deploy. |

No manual intervention is required once workflows are committed.

---

## Monitoring Pipelines

1. Navigate to your repository on GitHub.
2. Click on the **"Actions"** tab.
3. Select a workflow run to view logs, status, and artifacts.
4. Failed steps will display error logs to help with debugging.

You can also enable **email notifications** or integrate with Slack, Discord, or other tools using GitHub Actions integrations.

---

## Conclusion

This CI/CD setup ensures:

- Automated testing and linting on every commit/PR.
- Reliable builds with reproducible Docker images.
- Streamlined deployment to Docker Hub.
- Early detection of issues with minimal manual work.

For updates or changes to workflows, edit the corresponding YAML files in `.github/workflows/`.

