# Docker Build and Deployment Pipeline

## Overview

This Jenkins pipeline automates the process of building, testing, securing, and deploying Docker images. It provides a streamlined CI/CD workflow with multiple security scanning options and deployment capabilities.

## Pipeline Stages

### 1. Build Docker Image

Builds a Docker image based on specified parameters and configurations defined in `DOCKER_BUILD_INFORMATION.txt`.

**Key Features:**

- Memory-constrained build process to optimize resource usage
- Support for custom base images via the `DOCKER_FROM_IMAGE` parameter
- Comprehensive error handling and reporting

### 2. Unit Test Docker Image

Performs basic validation testing of the Docker image by:

- Running the container with appropriate environment variables
- Performing a health check via the container's API endpoint
- Validating version and status information

### 3. Push Docker Image

Publishes the Docker image to Docker Hub with:

- Tag specified by `SET_DOCKER_REPOSITORY_TAG` parameter
- Additional "latest" tag for easy access to current version

### 4. Docker CVE Tests

Parallel security scanning of the Docker image using multiple security tools:

#### a. Docker Scout CVE

- Scans for Common Vulnerabilities and Exposures (CVEs)
- Generates reports in SARIF and Markdown formats

#### b. Xray Scan

- Integration with JFrog Xray for additional vulnerability scanning
- Supports failing the build based on security findings

#### c. SonarQube Analysis

- Static code analysis for quality and security issues
- Integration with SonarQube server for detailed reports

#### d. GitGuardian Scan

- Detection of secrets, credentials, and sensitive information
- Comprehensive reporting with optional secret display

#### e. GitLeaks Scan

- Additional layer of secret detection
- Baseline comparison for identifying new issues

### 5. Continuous Deployment

- Utilizes ArgoCD for GitOps-based Kubernetes deployment
- Sets up repository connections and application configurations
- Applies Kubernetes manifests with Kustomize and Helm

## Environment Variables

```groovy
STAGE_SECURITY_TESTS_DOCKER_SCOUT = "true"
STAGE_SECURITY_TESTS_XRAY_SCAN = "false"
STAGE_SECURITY_TESTS_SONARQUBE = "false"
STAGE_SECURITY_TESTS_GITGUARDIAN = "false"
STAGE_SECURITY_TESTS_GITLEAKS = "false"
DOCKER_BUILD_MEMORY_CACHE = "100M"
API_PORT = "5000"
SERVER_IP = "SERVER"
```

## Pipeline Options

- **Timeout**: 10 minutes for the overall build
- **Skip Stages After Unstable**: Prevents execution of further stages if a stage is unstable
- **ANSI Color**: Enables colored console output for better readability

## Required Parameters

- `USE_STAGE_BUILD_DOCKER_IMAGE`: Enable/disable Docker image building
- `USE_STAGE_DOCKER_UNITEST`: Enable/disable unit testing
- `USE_STAGE_PUSH_DOCKER_IMAGE`: Enable/disable image pushing to registry
- `USE_STAGE_DOCKER_CVE_SCAN`: Enable/disable security scanning
- `USE_STAGE_CONTINUOUS_DEPLOYMENT`: Enable/disable ArgoCD deployment
- `RUN_JOB_NODE_NAME`: Jenkins node to run the job on
- `JOB_WORKSPACE`: Workspace path for the job
- `SET_DOCKER_REPOSITORY_TAG`: Tag to use for the Docker image
- `JOB_BUILD_NUMBER`: Upstream build number for display

## Required Credentials

- `Docker-Hub-Login-Credentials`: Username/password for Docker Hub
- `GitGuardian-Access-Credentials`: API key for GitGuardian
- `SonarQube-Access-Credentials`: Access token for SonarQube

## External Dependencies

- Docker CLI
- Docker Scout CLI
- SonarQube Scanner
- GitGuardian CLI (`ggshield`)
- GitLeaks
- kubectl
- kustomize
- argocd CLI

## Usage Examples

### Basic Build and Test

```groovy
pipeline {
    // ... pipeline configuration ...
    parameters {
        booleanParam(name: 'USE_STAGE_BUILD_DOCKER_IMAGE', defaultValue: true)
        booleanParam(name: 'USE_STAGE_DOCKER_UNITEST', defaultValue: true)
        booleanParam(name: 'USE_STAGE_PUSH_DOCKER_IMAGE', defaultValue: false)
        booleanParam(name: 'USE_STAGE_DOCKER_CVE_SCAN', defaultValue: false)
        booleanParam(name: 'USE_STAGE_CONTINUOUS_DEPLOYMENT', defaultValue: false)
        string(name: 'RUN_JOB_NODE_NAME', defaultValue: 'master')
        string(name: 'JOB_WORKSPACE', defaultValue: '/var/jenkins_home/workspace/my-project')
        string(name: 'SET_DOCKER_REPOSITORY_TAG', defaultValue: '1.0.0')
        string(name: 'JOB_BUILD_NUMBER', defaultValue: '1')
    }
}
```

### Full CI/CD Pipeline

```groovy
pipeline {
    // ... pipeline configuration ...
    parameters {
        booleanParam(name: 'USE_STAGE_BUILD_DOCKER_IMAGE', defaultValue: true)
        booleanParam(name: 'USE_STAGE_DOCKER_UNITEST', defaultValue: true)
        booleanParam(name: 'USE_STAGE_PUSH_DOCKER_IMAGE', defaultValue: true)
        booleanParam(name: 'USE_STAGE_DOCKER_CVE_SCAN', defaultValue: true)
        booleanParam(name: 'USE_STAGE_CONTINUOUS_DEPLOYMENT', defaultValue: true)
        string(name: 'RUN_JOB_NODE_NAME', defaultValue: 'master')
        string(name: 'JOB_WORKSPACE', defaultValue: '/var/jenkins_home/workspace/my-project')
        string(name: 'SET_DOCKER_REPOSITORY_TAG', defaultValue: '1.0.0')
        string(name: 'JOB_BUILD_NUMBER', defaultValue: '1')
    }
}
```

## Required Files

- `DOCKER_BUILD_INFORMATION.txt`: Contains Docker build configuration
- `Dockerfile`: Located in the directory specified by `DOCKER_BUILD_FOLDER`
- ArgoCD configuration files in the `argocd` and `jobProject` directories

## Post-Build Actions

The pipeline includes comprehensive post-build notifications and badge updates based on build status:

- Aborted: Yellow badge with "Build Aborted" text
- Unstable: Yellow badge with "Build Unstable" text
- Failure: Red badge with "Build Failure" text
- Success: Green badge with "Build Success" text
