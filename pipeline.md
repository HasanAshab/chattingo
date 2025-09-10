# Chattingo CI/CD Pipeline Architecture

## Overview
The Chattingo application uses a multi-stage Jenkins CI/CD pipeline with intelligent change detection, parallel processing, and automated deployment. The pipeline consists of three main components: a master orchestrator and two service-specific pipelines for frontend and backend.

## Pipeline Components

### 1. Orchestrator Pipeline (Root Jenkinsfile)
**Trigger**: GitHub push events
**Purpose**: Orchestrates the entire CI/CD process with intelligent change detection

#### Flow:
1. **Change Detection Stage**
   - Compares current commit with previous commit (HEAD~1)
   - Identifies changed files using `git diff --name-only`
   - Sets build flags for frontend and backend services
   - Only builds services that have actual changes

2. **Parallel Service Builds**
   - Triggers frontend pipeline if frontend/ files changed
   - Triggers backend pipeline if backend/ files changed
   - Passes GIT_BRANCH and SHORT_SHA parameters to child pipelines
   - Runs in parallel for maximum efficiency

### 2. Frontend Pipeline (frontend/Jenkinsfile)
**Technology Stack**: Node.js 18, npm, Docker
**Service Name**: frontend
**Docker Image**: chattingo/frontend

#### Stages:
1. **Parameter Validation**
   - Validates required GIT_BRANCH and SHORT_SHA parameters
   - Fails fast if parameters are missing

2. **Dependency Installation**
   - Runs `npm ci` for clean, reproducible builds
   - Uses package-lock.json for exact dependency versions

3. **Parallel CI Checks**
   - **Linting**: `npm run lint` for code style validation
   - **Testing**: `npm test` for unit/integration tests
   - **Security Scanning**: Trivy filesystem vulnerability scan
   - **Dependency Check**: OWASP dependency vulnerability analysis
   - **Code Quality**: SonarQube static code analysis
   - **Docker Build & Scan**: Builds Docker image and scans for vulnerabilities

4. **Production Deployment** (main branch only)
   - **Docker Push**: Pushes images to Docker Hub registry
   - **Compose Update**: Updates chattingo-compose repository with new image tags
   - **VPS Deployment**: Triggers redeployment on production VPS

### 3. Backend Pipeline (backend/Jenkinsfile)
**Technology Stack**: Maven 3.9, Java, Spring Boot, MySQL, Docker
**Service Name**: backend
**Docker Image**: chattingo/backend

#### Stages:
1. **Parameter Validation**
   - Same validation as frontend pipeline

2. **Dependency Installation**
   - Runs `mvn install -DskipTests` for dependency resolution
   - Skips tests during dependency phase for efficiency

3. **Parallel CI Checks**
   - **Linting**: `mvn checkstyle:check` for Java code style
   - **Testing**: Complex test setup with temporary MySQL container
     - Spins up MySQL 8 container on Jenkins network
     - Waits for database readiness
     - Runs Spring Boot tests with test database
     - Cleans up test container after completion
   - **Security Scanning**: Trivy filesystem vulnerability scan
   - **Dependency Check**: OWASP dependency vulnerability analysis
   - **Code Quality**: SonarQube static code analysis
   - **Docker Build & Scan**: Builds Docker image and scans for vulnerabilities

4. **Production Deployment** (main branch only)
   - Same deployment flow as frontend pipeline

## Key Features

### Intelligent Change Detection
- Only builds services with actual code changes
- Reduces build time and resource usage
- Prevents unnecessary deployments

### Parallel Processing
- All CI checks run in parallel within each service
- Frontend and backend pipelines run simultaneously
- Maximizes build efficiency and reduces total pipeline time

### Security-First Approach
- Multiple security scanning layers (Trivy, OWASP)
- Vulnerability scanning for both filesystem and Docker images
- Code quality gates with SonarQube

### Environment-Specific Deployment
- Only deploys to production from main branch
- Uses SHORT_SHA for immutable image tagging
- Maintains latest tag for convenience

### Infrastructure as Code
- Separate chattingo-compose repository for deployment configurations
- Automated updates to deployment manifests
- Version-controlled infrastructure changes

## Deployment Flow

### Development Branches
1. Code pushed to feature branch
2. Master pipeline detects changes
3. Runs CI checks (lint, test, security scan)
4. Builds and scans Docker images
5. Pipeline completes (no deployment)

### Main Branch (Production)
1. Code merged to main branch
2. Master pipeline detects changes
3. Runs full CI pipeline
4. Builds and scans Docker images
5. Pushes images to Docker Hub
6. Updates chattingo-compose repository
7. Triggers VPS redeployment
8. Application goes live

## External Dependencies

### Tools & Services
- **Jenkins**: CI/CD orchestration
- **Docker Hub**: Container registry (chattingo organization)
- **GitHub**: Source code and compose repository
- **VPS**: Production deployment target (72.60.111.27)
- **MySQL**: Database for backend testing and production

### Jenkins Shared Library
- Custom shared library: https://github.com/HasanAshab/Jenkins_SharedLib
- Provides reusable functions for Docker operations
- Standardizes security scanning and deployment processes

### Credentials Management
- `docker`: Docker Hub registry credentials
- `github-ssh-key`: GitHub SSH access for compose repository
- `vps-ssh-key`: VPS SSH access for deployment
- `sonarqube-server`: SonarQube server configuration

## Network Architecture
- Jenkins runs on dedicated network: `jenkins_jenkins_network`
- Test databases connect to Jenkins network
- Production VPS accessible via SSH
- Docker images distributed via Docker Hub

## Monitoring & Cleanup
- Build history retention: 5 builds and artifacts
- Automatic workspace cleanup after each build
- Success/failure notifications with emoji indicators
- Concurrent build prevention for stability
