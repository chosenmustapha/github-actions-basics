# github-actions-basics
# GitHub Actions CI/CD Fundamentals

A hands-on DevOps project focused on learning how to build automated CI/CD pipelines using GitHub Actions, Docker, and Node.js.

This repository demonstrates the foundational concepts behind modern software delivery pipelines, including:

- Continuous Integration (CI)
- Continuous Deployment (CD)
- Automated testing
- Docker image publishing
- GitHub Container Registry (GHCR)
- Workflow automation
- Secret management

The goal of this project is educational: to understand how GitHub Actions workflows are structured, triggered, and used to automate software engineering tasks in real-world environments.

---

# Architecture Overview

```text
Developer Pushes Code
        │
        ▼
 GitHub Actions Triggered
        │
        ├── Run CI Pipeline
        │     ├── Checkout Code
        │     ├── Setup Node.js
        │     ├── Install Dependencies
        │     └── Run Tests
        │
        ├── Build Docker Image
        │
        ├── Push Image to GHCR
        │
        └── Manual Deployment Workflow
              ├── Verify CI Status
              ├── Pull Latest Image
              └── Restart Application
```

---

# Tech Stack

| Technology | Purpose |
|---|---|
| Node.js | JavaScript runtime |
| GitHub Actions | CI/CD automation |
| Docker | Containerization |
| GHCR | Docker image registry |
| Jest | Unit testing |
| Ubuntu Runner | GitHub-hosted execution environment |

---

# Repository Structure

```bash
.
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy.yml
│       ├── docker_publish.yml
│       ├── main.yml
│       └── security-test.yml
├── Dockerfile
├── index.js
├── index.test.js
├── package.json
└── README.md
```

---

# Application Code

## `index.js`

A simple Node.js module exporting a reusable function.

```javascript
function sum(a, b) {
  return a + b;
}

module.exports = sum;
```

### What This Does

- Creates a function named `sum`
- Accepts two parameters
- Returns the addition of both values
- Exports the function using CommonJS syntax

---

# Unit Testing

## `index.test.js`

```javascript
const sum = require('./index');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(10);
});
```

### Purpose

This file uses Jest to validate application behavior through automated testing.

### Important Observation

The test intentionally fails because:

```javascript
sum(1, 2) // returns 3
```

However, the test expects:

```javascript
10
```

### Why This Is Useful

This demonstrates one of the core purposes of CI pipelines:

- Detecting failed code changes automatically
- Preventing broken code from progressing further in the pipeline
- Providing immediate feedback to developers

If the test fails, the GitHub Actions workflow will also fail.

---

# GitHub Actions Workflows

GitHub Actions workflows are YAML files located inside:

```bash
.github/workflows/
```

Each workflow defines:

- Events that trigger automation
- Jobs to execute
- Steps within each job
- The runner environment

---

# Workflow 1 — `main.yml`

## Purpose

A minimal introductory workflow used to understand basic GitHub Actions syntax and execution flow.

## Workflow Definition

```yaml
name: First Workflow

on: push

jobs:
  first-job:
    runs-on: ubuntu-latest

    steps:
      - name: Print Greeting
        run: echo "Hello from Mustapha"
```

### Key Concepts

- `on: push` triggers automation after code pushes
- `runs-on` defines the runner operating system
- `steps` execute sequentially
- `run` executes shell commands

---

# Workflow 2 — `ci.yml`

## Purpose

Implements a Continuous Integration (CI) pipeline for the Node.js application.

The objective of CI is to automatically validate code whenever changes are pushed to the repository.

## Full Workflow

```yaml
name: Node.js CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 16
      
      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```

## CI Pipeline Breakdown

### Checkout Repository

```yaml
uses: actions/checkout@v4
```

Downloads the repository contents into the runner environment.

---

### Setup Node.js Environment

```yaml
uses: actions/setup-node@v3
```

Installs Node.js version 16 inside the GitHub runner.

---

### Install Dependencies

```yaml
run: npm install
```

Installs all dependencies defined in `package.json`.

---

### Run Automated Tests

```yaml
run: npm test
```

Executes Jest tests to validate application behavior.

If tests fail, the workflow fails.

---

# Workflow 3 — `docker_publish.yml`

## Purpose

Automates Docker image creation and publishing to GitHub Container Registry (GHCR).

This simulates a common production-style container publishing workflow.

## Full Workflow

```yaml
name: Docker Publish

on:
  push:
    branches: ["master", "main"]

permissions:
  contents: read
  packages: write

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Log in to the Container registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}/app:latest
```

## Workflow Breakdown

### Authenticate to GHCR

```yaml
uses: docker/login-action@v2
```

Authenticates the workflow with GitHub Container Registry.

---

### Build Docker Image

```yaml
uses: docker/build-push-action@v4
```

Builds the Docker image using the local `Dockerfile`.

---

### Push Image to Registry

```yaml
push: true
```

Publishes the image to GHCR.

Example image path:

```bash
ghcr.io/username/repository/app:latest
```

---

# Workflow 4 — `deploy.yml`

## Purpose

Simulates a manual deployment workflow.

This demonstrates the basic structure of a production deployment pipeline.

## Full Workflow

```yaml
name: Deploy To Production

on: workflow_dispatch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Safety Check
        run: echo "verifying that CI passed..."

      - name: Mock SSH Connection
        run: echo "Connecting to production server at 192.168.1.1..."

      - name: Pull New Image
        run: |
          echo "Logging into Registry..."
          echo "Pulling image ghcr.io/myname/app:latest..."

      - name: Restart Application
        run: |
          echo "Stopping old container..."
          echo "Starting new container..."
          echo "Deployment Successful!"
```

## Deployment Breakdown

### `workflow_dispatch`

Allows the workflow to be manually triggered from the GitHub Actions UI.

This is commonly used for:

- Production deployments
- Controlled releases
- Manual approvals

---

### Mock Deployment Steps

The workflow simulates:

- Validating deployment readiness
- Connecting to a production server
- Pulling updated container images
- Restarting application services

---

# Workflow 5 — `security-test.yml`

## Purpose

Demonstrates how GitHub Secrets are securely injected into workflows.

## Full Workflow

```yaml
name: Security Test

on: [push]

jobs:
  test-secret:
    runs-on: ubuntu-latest

    steps:
      - name: Try to print test-secret
        env:
          MY_PASSWORD: ${{ secrets.DB_PASSWORD }}

        run: echo "The password is $MY_PASSWORD"
```

## Secret Management Concepts

### GitHub Secrets

GitHub Secrets securely store sensitive values such as:

- API keys
- Database credentials
- Cloud authentication tokens
- SSH private keys

---

### Injecting Secrets Into Workflows

```yaml
env:
  MY_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

Injects a repository secret into the runner environment.

---

### Security Note

This workflow intentionally prints the secret for educational purposes.

In real-world environments:

- Secrets should never be exposed in logs
- Credentials should be masked
- Least-privilege access should always be followed

---

# Docker Configuration

## `Dockerfile`

```dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

CMD ["node", "index.js"]
```

## Dockerfile Breakdown

### Base Image

```dockerfile
FROM node:16-alpine
```

Uses a lightweight Alpine Linux Node.js image.

---

### Working Directory

```dockerfile
WORKDIR /app
```

Creates and switches into `/app` inside the container.

---

### Install Dependencies

```dockerfile
COPY package*.json ./
RUN npm install
```

Copies dependency files and installs Node.js packages.

Separating these steps improves Docker layer caching.

---

### Copy Source Code

```dockerfile
COPY . .
```

Copies the remaining project files into the container.

---

### Start Container

```dockerfile
CMD ["node", "index.js"]
```

Defines the default startup command when the container launches.

---

# Running the Project Locally

## Clone Repository

```bash
git clone <repository-url>
cd github-actions-basics
```

---

## Install Dependencies

```bash
npm install
```

---

## Run Tests

```bash
npm test
```

Expected behavior:

- The test fails intentionally due to the incorrect assertion.

---

## Build Docker Image

```bash
docker build -t github-actions-basics .
```

---

## Run Docker Container

```bash
docker run github-actions-basics
```

---

# Core Concepts Demonstrated

| Concept | Description |
|---|---|
| Continuous Integration | Automated code validation |
| Continuous Deployment | Automated deployment workflows |
| Docker Containerization | Packaging applications consistently |
| Workflow Automation | Event-driven CI/CD execution |
| GitHub Secrets | Secure credential management |
| Container Registry | Centralized image storage |
| Runner Environments | Isolated execution systems |

---

# Key Learning Outcomes

Through this project, the following concepts are practiced:

- Writing GitHub Actions workflows
- Understanding CI/CD execution flow
- Running automated tests
- Building Docker containers
- Publishing images to registries
- Managing secrets securely
- Understanding workflow triggers
- Simulating deployment pipelines

---
