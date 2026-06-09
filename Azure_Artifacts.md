# Azure Artifacts - Complete DevOps Interview Notes

## What is Azure Artifacts?

Azure Artifacts is a package management service in Azure DevOps used to store, version, and share software packages across teams and applications.

Supported package types:

* Maven (Java)
* npm (NodeJS)
* NuGet (.NET)
* Python (PyPI)
* Universal Packages

Think of Azure Artifacts as a private package repository for your organization.

---

# Azure DevOps Services Overview

Azure Boards      → Work Tracking

Azure Repos       → Source Control

Azure Pipelines   → CI/CD

Azure Artifacts   → Package Management

Azure Test Plans  → Test Management

---

# Why Do We Need Azure Artifacts?

Without Azure Artifacts:

Developer Team A
|
v
Build JAR
|
v
Share Manually
|
v
Versioning Problems

Problems:

* No central repository
* Difficult dependency management
* No package version control
* Hard to share reusable libraries

With Azure Artifacts:

Developer Team A
|
v
Build JAR
|
v
Publish to Feed
|
v
Version Controlled Repository
|
v
Multiple Applications Consume It

---

# What is a Feed?

A Feed is a logical container used to store packages.

Example:

company-maven-feed

Structure:

Azure Artifacts
|
v
Feed
|
v
Package
|
v
Version

Example:

company-maven-feed

payment-common-1.0.0.jar

payment-common-1.1.0.jar

payment-common-1.2.0.jar

---

# Real-Time Use Cases

## Use Case 1 - Shared Libraries

Shared Packages:

* security-common.jar
* payment-common.jar
* logging-common.jar

Used By:

* Order Service
* Inventory Service
* Payment Service
* Customer Service

Flow:

Shared Library
|
v
Azure Artifacts Feed
|
v
Consumed by Multiple Applications

This is the most common enterprise use case.

---

## Use Case 2 - Containerized Spring Boot Applications

Flow:

Git Commit
|
v
Maven Build
|
v
Generate JAR
|
v
Docker Build
|
v
Push Image to ACR
|
v
Deploy to AKS

In this case:

Azure Artifacts may not be required.

The JAR is immediately packaged into a Docker image.

---

# Maven Default Output Directory

When Maven runs:

```bash
mvn clean package
```

Maven automatically creates:

target/

Example:

target/
|
+-- app.jar
+-- classes/
+-- test-classes/
+-- surefire-reports/

No special configuration is required.

target/ is Maven's default build output directory.

---

# Publishing Packages to Azure Artifacts

## Step 1 - Create Feed

Azure DevOps
|
v
Artifacts
|
v
New Feed

Example:

company-maven-feed

---

## Step 2 - Configure pom.xml

distributionManagement tells Maven where to publish packages.

```xml
<distributionManagement>
    <repository>
        <id>company-feed</id>
        <url>
        https://pkgs.dev.azure.com/ORG/PROJECT/_packaging/company-maven-feed/maven/v1
        </url>
    </repository>
</distributionManagement>
```

---

## Step 3 - Configure Authentication

settings.xml

```xml
<servers>
    <server>
        <id>company-feed</id>
        <username>AzureDevOps</username>
        <password>PAT_TOKEN</password>
    </server>
</servers>
```

Important:

repository id and server id must match.

---

## Step 4 - Publish Package

Execute:

```bash
mvn clean deploy
```

Flow:

Compile
|
v
Unit Tests
|
v
Package JAR
|
v
Upload to Azure Artifacts Feed

Result:

payment-common-1.0.0.jar

stored in Azure Artifacts.

---

# Consuming Packages from Azure Artifacts

This is the part many interviewers ask about.

---

## Step 1 - Define Dependency

Consumer Application pom.xml:

```xml
<dependency>
    <groupId>com.company</groupId>
    <artifactId>security-common</artifactId>
    <version>1.0.0</version>
</dependency>
```

This tells Maven:

Download security-common version 1.0.0

---

## Step 2 - Configure Repository Location

settings.xml

```xml
<profiles>
    <profile>
        <id>company-feed</id>

        <repositories>
            <repository>
                <id>company-feed</id>
                <url>
                https://pkgs.dev.azure.com/ORG/PROJECT/_packaging/company-feed/maven/v1
                </url>
            </repository>
        </repositories>
    </profile>
</profiles>
```

This tells Maven where to search.

---

## Step 3 - Authentication

settings.xml

```xml
<servers>
    <server>
        <id>company-feed</id>
        <username>AzureDevOps</username>
        <password>PAT_TOKEN</password>
    </server>
</servers>
```

---

# What Happens During Build?

Pipeline executes:

```bash
mvn clean package
```

Flow:

Read pom.xml
|
v
Need security-common:1.0.0
|
v
Check Local Maven Repository (.m2)
|
v
Found?

YES
|
v
Use Local Copy

NO
|
v
Connect to Azure Artifacts
|
v
Authenticate
|
v
Download Package
|
v
Store in .m2 Cache
|
v
Continue Build

---

# Local Maven Repository

Linux:

```bash
~/.m2/repository
```

Windows:

```text
C:\Users\<username>\.m2\repository
```

Example:

```text
~/.m2/repository/com/company/security-common/1.0.0
```

Future builds use the local cache.

---

# Azure Artifacts + Docker Workflow

Enterprise Example:

Azure Artifacts Feed

* security-common
* logging-common
* payment-common

|
v

Maven Build

|
v

Download Dependencies

|
v

Build app.jar

|
v

Docker Build

|
v

Push Image to ACR

|
v

Deploy to AKS

Important:

Docker does NOT directly download packages from Azure Artifacts.

Maven downloads dependencies first.

Docker packages the final application.

---

# How Docker Gets the JAR

Pipeline:

```bash
mvn clean package
```

Generates:

```text
target/app.jar
```

Dockerfile:

```dockerfile
FROM eclipse-temurin:17-jre

COPY target/app.jar app.jar

ENTRYPOINT ["java","-jar","app.jar"]
```

Docker copies the JAR from the pipeline workspace.

No Azure Artifacts interaction occurs during this step.

---

# Can Docker Build from Azure Artifacts?

Yes, but it is less common.

Flow:

Build Pipeline
|
v
mvn clean deploy
|
v
Publish JAR to Azure Artifacts

Later Pipeline
|
v
Download JAR
|
v
Docker Build
|
v
Push Image to ACR

Example:

```bash
mvn dependency:get
```

Download package first.

Then:

```dockerfile
COPY downloads/app.jar app.jar
```

This pattern is less common than building directly from the current workspace.

---

# Azure Artifacts vs Build Artifacts

## Azure Artifacts

Purpose:

Package Repository

Examples:

* Maven Packages
* npm Packages
* NuGet Packages
* Shared Libraries

Characteristics:

* Persistent
* Versioned
* Reusable

---

## Pipeline Build Artifacts

Purpose:

Pass files between pipeline stages

Examples:

* app.jar
* terraform.tfplan
* deployment.zip

Characteristics:

* Temporary
* Pipeline-specific

---

# Azure Artifacts vs Azure Container Registry

Azure Artifacts:

* Maven Packages
* npm Packages
* NuGet Packages
* Shared Libraries

Azure Container Registry:

* Docker Images
* OCI Images

Interview Point:

Docker Images belong in ACR, not Azure Artifacts.

---

# Real-Time DevSecOps Pipeline

Stage 1 - Build

* Maven Compile
* Unit Tests
* Integration Tests

Stage 2 - Code Quality

* SonarQube Scan
* Quality Gate

Stage 3 - Security

* Veracode
* Dependency Scanning

Stage 4 - Package

* Generate JAR

Stage 5 - Containerization

* Docker Build

Stage 6 - Container Security

* Prisma Scan

Stage 7 - Registry

* Push to ACR

Stage 8 - Deployment

* AKS Deployment

Flow:

Git Commit
|
v
Maven Build
|
v
Unit Tests
|
v
SonarQube
|
v
Veracode
|
v
Generate JAR
|
v
Docker Build
|
v
Prisma Scan
|
v
Push to ACR
|
v
Deploy to AKS

---

# Common Interview Questions

## Q1. What is Azure Artifacts?

Azure Artifacts is a package management service used to store, version, and share Maven, npm, NuGet, Python, and Universal packages.

---

## Q2. What is a Feed?

A Feed is a logical container that stores packages and versions.

---

## Q3. How do you publish Maven artifacts?

Configure:

* distributionManagement
* settings.xml

Then execute:

```bash
mvn clean deploy
```

---

## Q4. How do you consume packages from Azure Artifacts?

1. Add dependency in pom.xml
2. Configure feed repository
3. Configure authentication
4. Run Maven build

Maven downloads packages automatically.

---

## Q5. Does Docker download dependencies from Azure Artifacts?

No.

Maven downloads dependencies.

Docker packages the generated application artifact.

---

## Q6. Where are Docker Images stored?

Azure Container Registry (ACR).

---

## Q7. Does Azure Artifacts store Docker Images?

No.

Azure Artifacts stores packages.

ACR stores container images.

---

## Q8. Can Azure Artifacts and Docker be used together?

Yes.

Azure Artifacts provides dependencies.

Maven builds application.

Docker packages application.

ACR stores image.

---

## Q9. Did you use Azure Artifacts in your project?

Answer:

Azure Artifacts was used for hosting internal Maven packages and shared Java libraries. During builds, Maven downloaded required dependencies from Azure Artifacts. The application was then packaged into a Docker image and pushed to Azure Container Registry for deployment to AKS.

---

# Interview Summary

Must Know:

* Feed
* Maven Package Publishing
* distributionManagement
* settings.xml
* mvn deploy
* Dependency Download Flow
* Local Maven Repository (.m2)
* Azure Artifacts vs ACR
* Shared Library Use Cases

Most Important Interview Point:

Azure Artifacts manages reusable packages.

ACR manages deployable Docker images.

In modern AKS microservice projects, ACR is used more frequently than Azure Artifacts, but Azure Artifacts remains important when applications share internal Maven libraries.
