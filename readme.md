# Java to Azure Migration Walkthrough

This guide walks you through migrating a Java application to Azure using GitHub Copilot modernization. It covers assessment, Java/framework upgrades, migration to Azure services, containerization, and deployment.

**What the Modernization Will Do:** Transform your application from outdated technologies to a modern Azure-native solution, including:

- Upgrading from **Java 8 → Java 21**
- Migrating from **Spring Boot 2.x → 3.x**
- Replacing **AWS S3 → Azure Blob Storage**
- Switching from **RabbitMQ → Azure Service Bus**
- Migrating to **Azure Database for PostgreSQL**
- Implementing **managed identity authentication**
- Adding **health checks**
- **Containerizing** the applications for cloud deployment with monitoring

---

## Table of Contents

- [Java to Azure Migration Walkthrough](#java-to-azure-migration-walkthrough)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Current Architecture](#current-architecture)
  - [Environment Setup](#environment-setup)
    - [Prerequisites](#prerequisites)
    - [Option 1: Application-Only Setup (Local Lab Infrastructure)](#option-1-application-only-setup-local-lab-infrastructure)
    - [Option 2: Full Application + Azure Infrastructure Setup](#option-2-full-application--azure-infrastructure-setup)
  - [Running PetClinic Locally](#running-petclinic-locally)
    - [Option A: Quick Start (In-Memory Database)](#option-a-quick-start-in-memory-database)
    - [Option B: With PostgreSQL (Docker)](#option-b-with-postgresql-docker)
      - [Step 1: Free Up Ports](#step-1-free-up-ports)
      - [Step 2: Start PostgreSQL Container](#step-2-start-postgresql-container)
      - [Step 3: Run the Application](#step-3-run-the-application)
      - [Step 4: Explore the Application](#step-4-explore-the-application)
      - [Cleanup](#cleanup)
  - [App Modernization](#app-modernization)
    - [Modernization Prerequisites](#modernization-prerequisites)
      - [GitHub Account \& Copilot Access](#github-account--copilot-access)
      - [Development Environment](#development-environment)
    - [IDE Setup](#ide-setup)
      - [Option 1: Visual Studio Code](#option-1-visual-studio-code)
      - [Option 2: IntelliJ IDEA](#option-2-intellij-idea)
    - [Quick Start Checklist](#quick-start-checklist)
  - [Modernization Stages](#modernization-stages)
    - [Install GitHub Copilot Modernization](#install-github-copilot-modernization)
    - [Stage 1: Assess Application](#stage-1-assess-application)
      - [Run the Assessment](#run-the-assessment)
      - [Understanding Assessment Configuration](#understanding-assessment-configuration)
      - [Review Results](#review-results)
      - [Take Action on Findings](#take-action-on-findings)
  - [Stage 2: Migration Process and Progress Tracking](#stage-2-migration-process-and-progress-tracking)
    - [Version Control Setup](#version-control-setup)
    - [Two-Phase Migration Process](#two-phase-migration-process)
      - [Phase 1: Update Dependencies](#phase-1-update-dependencies)
      - [Phase 2: Configure Application Properties](#phase-2-configure-application-properties)
    - [Automated Validation and Fix Iteration Loop](#automated-validation-and-fix-iteration-loop)
      - [Validation Types](#validation-types)
      - [Automated Error Detection and Resolution](#automated-error-detection-and-resolution)
  - [Stage 3: Generate Containerization Assets with AI](#stage-3-generate-containerization-assets-with-ai)
    - [Initiate Containerization](#initiate-containerization)
    - [Containerization Strategy](#containerization-strategy)
    - [Critical Configuration Areas](#critical-configuration-areas)
      - [1. Labels Section](#1-labels-section)
      - [2. Service Account and Container Configuration](#2-service-account-and-container-configuration)
  - [Stage 4: Build and Deploy to Azure](#stage-4-build-and-deploy-to-azure)
    - [Build and Push Container Image to ACR](#build-and-push-container-image-to-acr)
      - [1. Login to ACR](#1-login-to-acr)
      - [2. Build the Docker Image](#2-build-the-docker-image)
    - [Deploy to AKS](#deploy-to-aks)
      - [1. Apply Kubernetes Manifests](#1-apply-kubernetes-manifests)
      - [2. Monitor Deployment Status](#2-monitor-deployment-status)
  - [Stage 5: Test and Verify Deployment](#stage-5-test-and-verify-deployment)
    - [Test the Deployed Application](#test-the-deployed-application)
      - [1. Port Forward to Access the Application](#1-port-forward-to-access-the-application)
      - [2. Test the Application (in another terminal)](#2-test-the-application-in-another-terminal)
      - [3. Check Pod Logs for Database Connections](#3-check-pod-logs-for-database-connections)
      - [4. Verify Health Endpoints](#4-verify-health-endpoints)
    - [Verify Passwordless Authentication](#verify-passwordless-authentication)
      - [1. Check Environment Variables in the Pod](#1-check-environment-variables-in-the-pod)
      - [2. Verify No Password Environment Variables](#2-verify-no-password-environment-variables)
      - [3. Check Application Logs for Authentication](#3-check-application-logs-for-authentication)
    - [Troubleshooting with Copilot](#troubleshooting-with-copilot)
  - [🎉 Success!](#-success)
    - [What You've Achieved](#what-youve-achieved)
    - [Architecture Highlights](#architecture-highlights)
  - [The application is now successfully deployed to AKS with passwordless authentication to PostgreSQL using Entra ID and workload identity—a modern, secure, and maintainable cloud-native architecture.](#the-application-is-now-successfully-deployed-to-aks-with-passwordless-authentication-to-postgresql-using-entra-id-and-workload-identitya-modern-secure-and-maintainable-cloud-native-architecture)
  - [Troubleshooting](#troubleshooting)
    - [Port 8080 Already in Use](#port-8080-already-in-use)
    - [PostgreSQL Connection Issues](#postgresql-connection-issues)
  - [Archive (Legacy Commands)](#archive-legacy-commands)
    - [Manual Database Configuration](#manual-database-configuration)
    - [Simplified PostgreSQL Profile](#simplified-postgresql-profile)

---

## Overview

In this demo, you will use the GitHub Copilot modernization extension to assess, upgrade, migrate, and finally deploy the project to Azure.

## Current Architecture

<!-- TODO: Add architecture diagram -->

---

## Environment Setup

Clone the repository and set up your environment using one of the two options below.

### Prerequisites

| Tool | Required For |
|------|-------------|
| **JDK 8** | Running the initial application locally |
| **Java 17+** | Running the modernized application |
| **Maven 3.6.0+** | Building the application locally |
| **Docker Desktop** | Running PostgreSQL and containerized services |
| **Azure CLI** | Option 2 (infrastructure provisioning) |
| **WSL** | Option 1 (bash script execution) |

> **Tip:** Ensure Docker Desktop is set to **Linux containers**:
> ```powershell
> & "C:\Program Files\Docker\Docker\DockerCli.exe" -SwitchLinuxEngine
> ```

> **Tip:** If you have a native PostgreSQL service installed, stop it to avoid port conflicts:
> ```powershell
> Stop-Service -Name "postgresql*"
> ```

---

### Option 1: Application-Only Setup (Local Lab Infrastructure)

If you only need to set up the application locally (without provisioning Azure infrastructure), fetch and run the setup script:

**PowerShell:**
```powershell
# Fetch the local lab infrastructure setup script
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/Azure-Samples/aks-labs/refs/heads/main/docs/migration/assets/migrate-to-aks/setup-local-lab-infra.sh" -OutFile "setup-local-lab-infra.sh"
```

**Then run in WSL/Bash:**
```bash
# Run the setup script
wsl bash ./setup-local-lab-infra.sh

# Install JDK 17
wsl bash -c "sudo apt update && sudo apt install -y openjdk-17-jdk"

# Install Docker
wsl bash -c "sudo apt update && sudo apt install -y docker.io && sudo service docker start && sudo usermod -aG docker $USER"
```

---

### Option 2: Full Application + Azure Infrastructure Setup

To set up both the application and provision all required Azure infrastructure (AKS cluster, PostgreSQL, ACR, Key Vault, monitoring, etc.), run the PowerShell setup script:

```powershell
./setup.ps1
```

> **Note:** This script will create Azure resources including a resource group, AKS cluster, PostgreSQL Flexible Server, Azure Container Registry, Key Vault, Log Analytics, Application Insights, and more. Ensure you are logged in to the Azure CLI (`az login`) before running.

---

## Running PetClinic Locally

### Option A: Quick Start (In-Memory Database)

Best for quick testing — no database setup required.

```powershell
cd spring-petclinic
.\mvnw spring-boot:run
```

Access the app at: **http://localhost:8080**

---

### Option B: With PostgreSQL (Docker)

Use this to match the production environment. Requires **Docker Desktop** running with **Linux containers**.

#### Step 1: Free Up Ports

Before starting, ensure ports **5432** and **8080** are not in use by other services.

**Check for port conflicts:**
```powershell
netstat -ano | findstr ":5432 .*LISTENING"
netstat -ano | findstr ":8080 .*LISTENING"
```

If a native PostgreSQL service (e.g., EDB PostgreSQL) is occupying port 5432, stop it:
```powershell
# Requires an elevated (Admin) PowerShell prompt
Stop-Service -Name "postgresql*"
```

If another process is on port 8080 (e.g., Apache httpd bundled with EDB), stop it:
```powershell
# Find the PID from the netstat output above, then:
Stop-Process -Id <PID> -Force
```

> **Tip:** If `Stop-Service` or `Stop-Process` fails with "Access Denied", run the command in an **elevated PowerShell** (Run as Administrator).

#### Step 2: Start PostgreSQL Container

```powershell
docker run -d --name petclinic-postgres `
  -e POSTGRES_USER=petclinic `
  -e POSTGRES_PASSWORD=petclinic `
  -e POSTGRES_DB=petclinic `
  -p 5432:5432 `
  postgres:15
```

Wait a few seconds for the database to initialize:
```powershell
Start-Sleep -Seconds 10
```

**Verify the container is running:**
```powershell
docker ps --filter "name=petclinic-postgres"
```

You should see the container listed with status `Up`.

#### Step 3: Run the Application

```powershell
cd spring-petclinic
.\mvnw spring-boot:run "-Dspring-boot.run.profiles=postgres"
```

The `postgres` profile activates PostgreSQL-specific configuration and seeds the database with sample data on first startup.

> **Note:** The first run downloads Maven dependencies and may take a few minutes. Subsequent runs will be faster.

Access the app at: **http://localhost:8080**

#### Step 4: Explore the Application

Once running, try features like:
- **Find Owners** — search and browse owner records (e.g., George Franklin, Betty Davis)
- **View Owner Details** — see pets and visit history
- **Edit Pet Information** — update pet records
- **Review Veterinarians** — view the vet directory

#### Cleanup

When you're done, stop and remove the PostgreSQL container:
```powershell
docker stop petclinic-postgres
docker rm petclinic-postgres
```

---

## App Modernization

### Modernization Prerequisites

#### GitHub Account & Copilot Access

- A GitHub account with **GitHub Copilot enabled**
- Required plan: **Pro**, **Pro+**, **Business**, or **Enterprise**

#### Development Environment

| Tool | Version | Notes |
|------|---------|-------|
| **Java JDK** | Source + Target versions | Both your current and desired upgrade version |
| **Maven** or **Gradle** | Latest | For building Java projects |
| **Git** | Latest | Project must be Git-managed |

- For Maven-based projects: Access to the public **Maven Central** repository is required

---

### IDE Setup

Choose one of the following IDEs:

#### Option 1: Visual Studio Code

**Required Software:**
- **Visual Studio Code** v1.101 or later
- **GitHub Copilot extension** — [Install from VS Code Marketplace](https://docs.github.com/en/copilot/using-github-copilot/getting-started-with-github-copilot#setting-up-github-copilot-in-visual-studio-code)
  - Sign in to your GitHub account within VS Code
- **GitHub Copilot Modernization extension** — Install from VS Code Marketplace
  - Restart VS Code after installation

**Configuration:**
- In VS Code settings, ensure `chat.extensionTools.enabled` is set to `true`
  > **Note:** This setting might be controlled by your organization.

#### Option 2: IntelliJ IDEA

**Required Software:**
- **IntelliJ IDEA** v2023.3 or later
- **GitHub Copilot plugin** v1.5.59 or later — [Install from JetBrains Marketplace](https://docs.github.com/en/copilot/using-github-copilot/getting-started-with-github-copilot#setting-up-github-copilot-in-intellij-idea)
  - Sign in to your GitHub account within IntelliJ IDEA
- **GitHub Copilot Modernization plugin** — Install from JetBrains Marketplace
  - Restart IntelliJ IDEA after installation
  - If GitHub Copilot is not already installed, installing the Modernization plugin will also install it

**Recommended Configuration:**
1. Open IntelliJ IDEA settings
2. Navigate to **Tools > GitHub Copilot**
3. Enable the following:
   - ✅ **Auto-approve**
   - ✅ **Trust MCP Tool Annotations**

> For more information, see [Configure settings for GitHub Copilot modernization](https://docs.github.com/en/copilot/using-github-copilot/using-github-copilot-for-app-modernization).

---

### Quick Start Checklist

- [ ] GitHub account with active Copilot subscription
- [ ] IDE installed and updated (VS Code 1.101+ OR IntelliJ IDEA 2023.3+)
- [ ] GitHub Copilot extension/plugin installed
- [ ] GitHub Copilot Modernization extension/plugin installed
- [ ] Signed in to GitHub account within IDE
- [ ] IDE restarted after plugin installation
- [ ] Java JDK installed (source and target versions)
- [ ] Maven or Gradle installed
- [ ] Git-managed Java project ready
- [ ] Maven Central repository accessible (for Maven projects)
- [ ] VS Code: `chat.extensionTools.enabled` set to `true` (if using VS Code)
- [ ] IntelliJ: Auto-approve and Trust MCP Tool Annotations enabled (optional but recommended)

---

## Modernization Stages

We will use GitHub Copilot modernization to assess, remediate, and modernize the Spring Boot application in preparation to migrate the workload to AKS.

### Install GitHub Copilot Modernization

1. In VS Code, open the **Extensions** view from the Activity Bar.
2. Search for **GitHub Copilot Modernization** in the marketplace.
3. Click **Install**.
4. After installation completes, a notification will appear in the bottom-right corner of VS Code confirming success.

---

### Stage 1: Assess Application

The first step is to assess the sample Java application. The assessment provides insights into the application's readiness for migration to Azure.

#### Run the Assessment

1. Open VS Code in the `spring-petclinic` directory:
   ```bash
   cd spring-petclinic
   ```
2. In the Activity sidebar, open the **GitHub Copilot Modernization** extension pane.
3. Click **Start Assessment** to trigger the app assessment.
4. Wait for the assessment to complete — this could take several minutes.
5. You can follow progress in the **Output** terminal within VS Code.

#### Understanding Assessment Configuration

The assessment is powered by AppCAT tool. You can customize the assessment by editing `.github/appmod-java/appcat/assessment-config.yaml` with the following options:

| Setting | Options | Description |
|---------|---------|-------------|
| **target** | `azure-aks`, `azure-appservice`, `azure-container-apps` | The Azure compute service you plan to migrate to |
| **capability** | `containerization` | Technology to modernize towards |
| **mode** | `issue-only`, `source-only`, `full` | How deep AppCAT inspects the project |
| **os** | `windows`, `linux` | Target operating system for migration |

**Mode Details:**
- **issue-only** — Analyzes source code to detect issues only
- **source-only** — Fast analysis that examines source code only
- **full** — Inspects source code and scans dependencies (slower, more thorough)

#### Review Results

Once the assessment completes, an **Assessment Report** tab opens with a categorized view of cloud readiness issues and recommended solutions.

**Issue Prioritization:**

| Priority | Color | Description |
|----------|-------|-------------|
| **Mandatory** | 🟣 Purple | Critical issues that must be addressed before migration |
| **Potential** | 🔵 Blue | Performance and optimization opportunities |
| **Optional** | ⚪ Gray | Nice-to-have improvements that can be addressed later |

> Focus on **Mandatory** issues first, then work through **Potential** and **Optional** items as time allows.

**Resolution:** More than **30%** of identified issues can be automatically resolved through code and configuration updates using GitHub Copilot's built-in modernization capabilities.

#### Take Action on Findings

1. Select the **Issues** tab to view proposed solutions and proceed with migration steps.
2. **Example — Migrate to Azure DB for PostgreSQL:**
   - This updates the Java code to work with PostgreSQL Flexible Server using Entra ID authentication.
   - The tool executes the `appmod-run-task` command for `mi-postgresql-spring`, which examines the workspace and initiates the migration task.
   - A comprehensive migration plan is generated outlining the specific changes needed to implement Azure Managed Identity authentication for PostgreSQL connectivity.

## Stage 2: Migration Process and Progress Tracking

After creating the migration plan, the GitHub Copilot Modernization tool orchestrates a systematic migration process with automated validation and error resolution.

### Version Control Setup

The tool automatically:
- Sets up version control for tracking changes
- Creates a new branch for migration changes
- Stashes any uncommitted work to preserve your current state

### Two-Phase Migration Process

The migration follows a structured approach to modernize your Spring Boot application for Azure PostgreSQL with Entra ID authentication:

#### Phase 1: Update Dependencies

**Modify `pom.xml`:**
- Adds Spring Cloud Azure BOM (Bill of Materials)
- Includes PostgreSQL starter dependency
- Updates Spring Cloud Azure version properties

This ensures your application has the necessary libraries for Azure integration and PostgreSQL connectivity.

#### Phase 2: Configure Application Properties

**Update configuration files for managed identity authentication:**
- Modifies `application-postgres.properties` with Entra ID authentication settings
- Replaces traditional username/password authentication with managed identity configuration
- Configures passwordless database connections using Azure managed identities

### Automated Validation and Fix Iteration Loop

After implementing migration changes, the Modernization tool runs a comprehensive validation process to ensure changes are secure, functional, and consistent.

#### Validation Types

The tool performs four types of validation:

| Validation Type | Purpose |
|----------------|---------|
| **CVE Validation** | Scans newly added dependencies for known security vulnerabilities |
| **Build Validation** | Verifies the application compiles and builds successfully after migration changes |
| **Consistency Validation** | Ensures all configuration files are properly updated and aligned |
| **Test Validation** | Executes application tests to verify functionality remains intact |

#### Automated Error Detection and Resolution

The tool includes intelligent error detection capabilities that automatically identify and resolve common migration issues:

- Parses build output to detect compilation errors
- Identifies root causes of test failures
- Applies automated fixes for common migration patterns
- Continues through validation iterations (up to 10 iterations) until the build succeeds

**User Control:**
At any point during the validation process, you can interrupt the automated fixes and manually resolve issues if you prefer to handle specific problems yourself. The tool provides clear feedback on what it's attempting to fix and allows you to take control when needed.

**Result:** This systematic approach ensures your Spring Boot application is successfully modernized for Azure PostgreSQL with Entra ID authentication while maintaining full functionality.

---

## Stage 3: Generate Containerization Assets with AI

Now that the application is modernized, we'll containerize it for deployment to Azure Kubernetes Service (AKS).

### Initiate Containerization

1. In GitHub Copilot Modernization, navigate to: **TASKS → Common Tasks → Containerize Tasks → Containerize Application**
2. The **Containerization Assist MCP Server** will scan your project and develop a containerization strategy

### Containerization Strategy

The tool generates optimized containerization assets:

**Generated Assets:**
- **Dockerfile**: Multi-stage build with optimized base image for reduced size and improved security
- **Kubernetes Deployment**: Configured with Azure workload identity, PostgreSQL secrets, health checks, and resource limits
- **Kubernetes Service**: LoadBalancer configuration for external access

**Expected Output:** Kubernetes manifests created in the `k8s/` directory

### Critical Configuration Areas

After generation, verify and update the following sections in your Kubernetes manifests:

#### 1. Labels Section
```yaml
labels:
  app: spring-petclinic
  version: v1
  azure.workload.identity/use: "true"  # Enable Azure Workload Identity
```

#### 2. Service Account and Container Configuration
```yaml
spec:
  serviceAccountName: sc-account-xxxx-xxxxx-xxxxx-xxxx-xxxx  # ⚠️ Change this
  containers:
  - name: spring-petclinic
    image: acrpetclinic556325.azurecr.io/petclinic:0.0.1  # ⚠️ Change this value
```

**Required Updates:**
- **`serviceAccountName`**: Replace with the service account created during PostgreSQL Service Connector setup
  - Retrieve from your `sc.log` file if needed
- **`image`**: Update with your Azure Container Registry (ACR) login server and image tag
  - Format: `<acr-login-server>/petclinic:0.0.1`

> 💡 **Tip:** If you need to retrieve the Service Connector account name, check your `sc.log` file from the earlier PostgreSQL setup.

Verify Azure PostgreSQL Server is Running

- Before deploying to AKS, ensure your PostgreSQL server is active

```bash
az postgres flexible-server show --resource-group $env:RESOURCE_GROUP --name $env:POSTGRES_SERVER_NAME --query "{State:state, PublicAccess:network.publicNetworkAccess}" -o json

# If State shows "Stopped", start the server
az postgres flexible-server start --resource-group $env:RESOURCE_GROUP --name $env:POSTGRES_SERVER_NAME

# Wait for server to be ready (takes ~30 seconds)
az postgres flexible-server show --resource-group $env:RESOURCE_GROUP --name $env:POSTGRES_SERVER_NAME --query "state" -o tsv

# Verify Firewall Ruless
az postgres flexible-server firewall-rule list --resource-group $env:RESOURCE_GROUP --name $env:POSTGRES_SERVER_NAME -o table
```

---

## Stage 4: Build and Deploy to Azure

### Build and Push Container Image to ACR

Build the containerized application and push it to your Azure Container Registry.

#### 1. Login to ACR
```bash
az acr login --name $env:ACR_NAME
```

#### 2. Build the Docker Image

Build directly in Azure Container Registry (no local Docker daemon required):
```bash
az acr build -t petclinic:0.0.1 . -r $env:ACR_NAME
```

This command:
- Uploads your source code to ACR
- Builds the Docker image in the cloud
- Tags it as `petclinic:0.0.1`
- Stores it in your registry

### Deploy to AKS

Deploy the modernized application to AKS using Service Connector secrets for passwordless authentication with PostgreSQL.

#### 1. Apply Kubernetes Manifests
```bash
kubectl apply -f k8s/petclinic.yml
```

#### 2. Monitor Deployment Status
```bash
kubectl get pods,services,deployments
```

Watch for:
- Pods transitioning to `Running` state
- Service receiving an external IP (if using LoadBalancer)
- Deployment showing desired number of replicas ready

Initial startup takes 60-90 seconds due to:

- Container image pull (~3-5 seconds)
- Spring Boot initialization (~20-25 seconds)
- Database connection and schema initialization (~10-15 seconds)
- Health probe validation (startup probe with 10s delay + retries)

---

## Stage 5: Test and Verify Deployment

### Test the Deployed Application

#### 1. Port Forward to Access the Application
```bash
kubectl port-forward svc/petclinic-service 8080:80
```

#### 2. Test the Application (in another terminal)
```bash
curl http://localhost:8080
```

Expected response: HTML content from the Pet Clinic application

#### 3. Check Pod Logs for Database Connections
```bash
kubectl logs -l app=petclinic
```

Look for successful connection messages to PostgreSQL using Entra ID authentication.

#### 4. Verify Health Endpoints
```bash
curl http://localhost:8080/actuator/health
```

Expected response:
```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP"
    }
  }
}
```

### Verify Passwordless Authentication

Confirm the application is using passwordless authentication with Azure managed identities:

#### 1. Check Environment Variables in the Pod
```bash
# Get first pod with label
POD_NAME=$(kubectl get pods -l app=spring-petclinic -o jsonpath='{.items[0].metadata.name}')

# Check PostgreSQL-related environment variables
kubectl exec $POD_NAME -- env | grep POSTGRES
```

#### 2. Verify No Password Environment Variables
```bash
kubectl exec $POD_NAME -- env | grep -i pass
```

**Expected:** No password-related environment variables should be present

#### 3. Check Application Logs for Authentication
```bash
kubectl logs -l app=spring-petclinic --tail=100 | grep -i "hibernate"
```

Look for successful Hibernate initialization and database connection messages.

### Troubleshooting with Copilot

If you encounter issues, leverage GitHub Copilot for assistance:

**Example Prompt:**
```
I'm deploying this to AKS. I have a PostgreSQL database and my app will be using workload identity.
```

**Advanced Troubleshooting:**
You can reference your Service Connector configuration file directly:
```
Review my deployment configuration. I have Service Connector details in #file:sc.log
```

This provides Copilot with context about your specific Azure resources and configuration.

---

## 🎉 Success!

You've successfully completed a comprehensive application modernization journey, transforming a legacy Spring Boot application into a cloud-native, secure, and scalable solution on Azure.

### What You've Achieved

✅ **Assessed** application cloud readiness using AppCAT analysis  
✅ **Migrated** to Azure PostgreSQL with Entra ID authentication  
✅ **Eliminated** password-based authentication with managed identities  
✅ **Containerized** the application with optimized Docker images  
✅ **Deployed** to Azure Kubernetes Service (AKS)  
✅ **Configured** Azure Workload Identity for secure, passwordless database access  
✅ **Validated** functionality with health checks and testing  

### Architecture Highlights

- **Cloud-Native**: Containerized Spring Boot application running on AKS
- **Secure**: Passwordless authentication using Azure Managed Identities
- **Scalable**: Kubernetes-based orchestration with horizontal scaling capabilities
- **Production-Ready**: Health checks, resource limits, and proper configuration management

The application is now successfully deployed to AKS with passwordless authentication to PostgreSQL using Entra ID and workload identity—a modern, secure, and maintainable cloud-native architecture.
---

## Troubleshooting

### Port 8080 Already in Use

1. Find the process using the port:
   ```powershell
   netstat -ano | findstr :8080
   ```

2. Get process details (replace `<PID>` with the ID from step 1):
   ```powershell
   Get-Process -Id <PID>
   ```

3. Stop the process:
   ```powershell
   Stop-Process -Id <PID> -Force
   ```

**Alternative:** Run PetClinic on a different port:
```powershell
.\mvnw spring-boot:run "-Dspring-boot.run.profiles=postgres" "-Dserver.port=8081"
```

---

### PostgreSQL Connection Issues

**Reset the Docker container:**

```powershell
docker stop petclinic-postgres
docker rm petclinic-postgres
```

Then restart from [Step 1](#step-1-start-postgresql-container).

**Check if native PostgreSQL is running:**

```powershell
netstat -ano | findstr :5432
```

If occupied, stop the native service:
```powershell
Stop-Service -Name "postgresql*"
```

---

## Archive (Legacy Commands)

<details>
<summary>Click to expand alternative configurations</summary>

### Manual Database Configuration

```powershell
.\mvnw clean compile; .\mvnw spring-boot:run `
  "-Dspring-boot.run.arguments=--spring.messages.basename=messages/messages --spring.datasource.url=jdbc:postgresql://localhost/petclinic --spring.sql.init.mode=always --spring.sql.init.schema-locations=classpath:db/postgres/schema.sql --spring.sql.init.data-locations=classpath:db/postgres/data.sql --spring.jpa.hibernate.ddl-auto=none"
```

### Simplified PostgreSQL Profile

```powershell
.\mvnw spring-boot:run "-Dspring-boot.run.arguments=--spring.datasource.url=jdbc:postgresql://localhost/petclinic --spring.sql.init.mode=always --spring.profiles.active=postgres"
```

</details>