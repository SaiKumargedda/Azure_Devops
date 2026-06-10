# Azure Repos Complete Guide for Azure DevOps Engineers

## Table of Contents

1. Azure Repos Overview
2. GitFlow Branching Strategy
3. Repository Structure
4. Branch Creation
5. Repository Permissions
6. Branch Policies
7. Pull Requests (PR)
8. PR Validation Pipeline
9. Develop CI Pipeline
10. Release Process
11. Main Branch Pipeline
12. GitVersion Integration
13. Automated Versioning
14. Git Tags
15. Docker Image Versioning
16. ACR Integration
17. AKS Deployment
18. Rollback Strategy
19. Real-Time End-to-End Workflow
20. Azure Repos Interview Questions & Answers

---

# 1. Azure Repos Overview

Azure Repos is a Git-based source control service provided by Azure DevOps.

Used for storing:

* Application Source Code
* Terraform Code
* Dockerfiles
* Kubernetes Manifests
* Azure Pipeline YAML Files
* Shell Scripts
* Documentation

Benefits:

* Version Control
* Collaboration
* Pull Requests
* Branch Policies
* Security Controls
* CI/CD Integration

---

# 2. GitFlow Branching Strategy

Enterprise projects commonly use GitFlow.

```text
main
│
├── develop
│
├── feature/login
├── feature/payment
├── feature/cart
│
├── release/1.0.0
│
└── hotfix/critical-fix
```

Branch Purpose:

| Branch    | Purpose                    |
| --------- | -------------------------- |
| main      | Production Code            |
| develop   | Integration Branch         |
| feature/* | New Features               |
| release/* | Release Preparation        |
| hotfix/*  | Emergency Production Fixes |

---

# 3. Repository Structure

```text
payment-service
│
├── src/
├── pom.xml
├── Dockerfile
├── terraform/
├── k8s/
│
├── GitVersion.yml
│
├── azure-pipelines-pr.yml
├── azure-pipelines-develop.yml
└── azure-pipelines-main.yml
```

---

# 4. Branch Creation

Create develop branch:

```bash
git checkout main
git checkout -b develop
git push origin develop
```

Create feature branch:

```bash
git checkout develop
git checkout -b feature/payment
git push origin feature/payment
```

Create release branch:

```bash
git checkout develop
git checkout -b release/1.0.0
git push origin release/1.0.0
```

Create hotfix branch:

```bash
git checkout main
git checkout -b hotfix/security-fix
git push origin hotfix/security-fix
```

---

# 5. Repository Permissions

## Main Branch

Repos → Branches → Main → Security

```text
Read                     Allow
Contribute               Deny
Force Push               Deny
Delete Branch            Deny
Bypass Policies          Deny
```

Only Pull Requests allowed.

---

## Develop Branch

```text
Read                     Allow
Contribute               Deny
Force Push               Deny
Delete Branch            Deny
```

Only Pull Requests allowed.

---

## Feature Branches

```text
Read                     Allow
Contribute               Allow
```

Developers can commit.

---

# 6. Branch Policies

## Develop Branch Policies

Repos
→ Branches
→ Develop
→ Branch Policies

### Minimum Reviewers

```text
2 Reviewers
```

Example:

```text
Senior Developer
Tech Lead
```

---

### Build Validation

Required:

```text
PR Validation Pipeline
```

---

### Work Item Linking

Enabled

---

### Comment Resolution

Enabled

---

### Merge Strategy

Allow:

```text
Squash Merge
```

Disable:

```text
Merge Commit
```

---

## Main Branch Policies

More strict.

### Minimum Reviewers

```text
3 Reviewers
```

Example:

```text
Lead
Architect
Release Manager
```

---

### Build Validation

Required

---

### Work Item Linking

Required

---

### Comment Resolution

Required

---

### Direct Push

Blocked

---

# 7. Pull Request Process

Developer:

```text
feature/payment
```

Raises PR:

```text
feature/payment
      |
      v
develop
```

Azure DevOps automatically runs:

```text
PR Validation Pipeline
```

Reviewer receives results.

Approves.

Merge allowed.

---

# 8. PR Validation Pipeline

File:

```text
azure-pipelines-pr.yml
```

```yaml
pr:
  branches:
    include:
      - develop
      - main

trigger: none

pool:
  vmImage: ubuntu-latest

steps:

- task: Maven@4
  inputs:
    goals: clean package

- task: SonarQubePrepare@7

- task: SonarQubeAnalyze@7

- script: |
    echo "Run Veracode Scan"

- script: |
    echo "Run Unit Tests"
```

Purpose:

* Build Validation
* SonarQube Validation
* Veracode Validation
* Unit Testing

No deployment.

---

# 9. Develop CI Pipeline

Triggered after merge into develop.

File:

```text
azure-pipelines-develop.yml
```

```yaml
trigger:
  branches:
    include:
      - develop

pr: none

pool:
  vmImage: ubuntu-latest

steps:

- task: Maven@4
  inputs:
    goals: clean package

- publish: target/*.jar
  artifact: drop

- task: Docker@2
  inputs:
    command: buildAndPush
    repository: payment-service
    tags: |
      develop-$(Build.BuildId)
```

Output:

```text
payment-service.jar

payment-service:develop-125
```

Used for:

* QA
* SIT
* UAT

---

# 10. Release Process

After QA signoff:

Create release branch.

```bash
git checkout develop
git checkout -b release/1.0.0
```

Raise PR:

```text
release/1.0.0
        |
        v
main
```

PR Validation Pipeline runs again.

Approvals required.

Merge into main.

---

# 11. Main Branch Pipeline

File:

```text
azure-pipelines-main.yml
```

```yaml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: ubuntu-latest

steps:

- checkout: self
  fetchDepth: 0
```

---

# 12. GitVersion Integration

Install GitVersion Azure DevOps Extension.

Repository File:

```text
GitVersion.yml
```

```yaml
mode: ContinuousDelivery

branches:

  main:
    increment: Patch

  develop:
    increment: Minor

  release:
    increment: Patch

  feature:
    increment: Patch
```

---

# 13. GitVersion Pipeline Configuration

```yaml
- checkout: self
  fetchDepth: 0

- task: gittools.gitversion.setup@0
  inputs:
    versionSpec: '5.x'

- task: gittools.gitversion.execute@0
  inputs:
    useConfigFile: true
    configFilePath: 'GitVersion.yml'

- script: |
    echo $(GitVersion.SemVer)
```

Example Output:

```text
1.0.0
```

---

# 14. Version Maven Artifact

```yaml
- task: Maven@4
  inputs:
    goals: clean package
```

Output:

```text
payment-service-1.0.0.jar
```

---

# 15. Docker Image Versioning

```yaml
- task: Docker@2
  inputs:
    command: buildAndPush
    repository: payment-service
    tags: |
      $(GitVersion.SemVer)
```

Output:

```text
payment-service:1.0.0
```

---

# 16. Create Git Tag

Optional but recommended.

```yaml
- script: |
    git config user.email "devops@company.com"
    git config user.name "AzurePipeline"

    git tag v$(GitVersion.SemVer)

    git push origin v$(GitVersion.SemVer)
```

Example:

```text
v1.0.0
```

---

# 17. Push Image To ACR

Example:

```text
myacr.azurecr.io/payment-service:1.0.0
```

---

# 18. Deploy To AKS

Deployment Manifest:

```yaml
containers:
- name: payment-service
  image: myacr.azurecr.io/payment-service:1.0.0
```

Deploy:

```bash
kubectl apply -f deployment.yaml
```

---

# 19. Rollback Strategy

Current:

```text
payment-service:1.2.0
```

Issue found.

Rollback:

```text
payment-service:1.1.0
```

Update deployment:

```yaml
image: myacr.azurecr.io/payment-service:1.1.0
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

No rebuild required.

---

# 20. End-to-End Real-Time Workflow

```text
Developer
   |
feature/payment
   |
Commit
   |
Push
   |
PR -> develop
   |
PR Validation Pipeline
(Maven + Sonar + Veracode)
   |
Approvals
   |
Merge -> develop
   |
Develop CI Pipeline
   |
Docker Image
payment-service:develop-567
   |
QA/UAT Testing
   |
release/1.0.0
   |
PR -> main
   |
PR Validation Pipeline
   |
Architect Approval
Release Manager Approval
   |
Merge -> main
   |
Main Pipeline
   |
GitVersion
Version=1.0.0
   |
payment-service-1.0.0.jar
   |
payment-service:1.0.0
   |
(Optional) Git Tag v1.0.0
   |
Push ACR
   |
Production Approval
   |
Deploy AKS
```

---

# Azure Repos Interview Questions and Answers

## Q1. What is Azure Repos?

Azure Repos is a Git-based source control service in Azure DevOps used for source code management and collaboration.

---

## Q2. What branching strategy did you use?

GitFlow:

```text
main
develop
feature/*
release/*
hotfix/*
```

---

## Q3. What is a Pull Request?

A Pull Request is a request to merge code from one branch to another after code review and validation.

---

## Q4. What branch policies did you configure?

* Minimum Reviewers
* Build Validation
* Work Item Linking
* Comment Resolution
* Merge Strategy Restrictions

---

## Q5. Difference Between PR Trigger and CI Trigger?

PR Trigger:

```text
Before Merge
Validation Only
```

CI Trigger:

```text
After Merge
Build and Deployment
```

---

## Q6. Why Use Build Validation?

To ensure code compiles successfully and passes SonarQube, Veracode, and unit tests before merging.

---

## Q7. How Did You Prevent Direct Commits To Main?

Configured branch security and branch policies.

---

## Q8. What Is GitVersion?

GitVersion automatically generates semantic versions based on Git history and branching strategy.

---

## Q9. How Did You Version Docker Images?

Used:

```text
$(GitVersion.SemVer)
```

Example:

```text
payment-service:1.0.0
```

---

## Q10. How Did You Handle Rollbacks?

Redeployed a previous Docker image version or Git tag without rebuilding the application.

---

## Q11. Difference Between Branch and Tag?

Branch:

```text
Moves with commits
```

Tag:

```text
Fixed reference to a specific commit
```

---

## Q12. How Did Azure Repos Integrate With Azure Pipelines?

Repository stored source code and YAML pipelines. Pull requests and commits automatically triggered Azure Pipelines for validation, build, and deployment.
