# Azure Boards for DevOps Engineers - Interview Preparation Notes

## What is Azure Boards?

Azure Boards is a work tracking service in Azure DevOps used to manage:

* Epics
* Features
* User Stories
* Tasks
* Bugs
* Backlogs
* Sprints

It helps teams plan, track, and deliver software using Agile methodologies.

---

# Agile Overview

Agile is an iterative software development methodology where work is delivered in small increments called Sprints.

Typical Agile Team:

* Product Owner (PO)
* Scrum Master
* Developers
* QA Engineers
* DevOps Engineers

---

# What is a Sprint?

A Sprint is a fixed-duration development cycle.

Common Sprint Durations:

* 2 Weeks
* 3 Weeks
* 4 Weeks

Most organizations use 2-week sprints.

Example:

Sprint 1:
01-Jun to 14-Jun

Sprint 2:
15-Jun to 28-Jun

---

# Agile Ceremonies

## Sprint Planning

* Team selects work items from backlog
* Story points are estimated
* Work is assigned

## Daily Standup

Typical Questions:

1. What did you do yesterday?
2. What are you working on today?
3. Any blockers?

## Sprint Review

* Demonstration of completed work

## Sprint Retrospective

Discussion on:

* What went well
* What went wrong
* Improvement actions

---

# Azure Boards Hierarchy

Business Requirement
|
v
Epic
|
v
Feature
|
v
User Story
|
v
Task / Bug

---

# Work Item Types

## Epic

Large business objective.

Example:

Build Online Banking Platform

---

## Feature

Major functionality within an Epic.

Example:

Payment Module

---

## User Story

Requirement from end-user perspective.

Example:

As a customer, I want to make online payments.

---

## Task

Implementation work assigned to team members.

Example:

Create AKS Deployment Pipeline

---

## Bug

Defect or issue discovered during testing or production.

Example:

Payment API returning HTTP 500.

---

# Real-Time Example

Epic:
Cloud Migration

Feature:
Deploy Applications to AKS

User Story:
As a user, I want the application deployed on AKS for high availability.

Tasks:

* Create AKS Cluster
* Configure Azure Container Registry
* Create CI Pipeline
* Create CD Pipeline
* Configure Monitoring

---

# Azure Boards Work Item States

## User Story Lifecycle

New
|
v
Active
|
v
Resolved
|
v
Closed

Meaning:

New = Created but not started

Active = Work in progress

Resolved = Development completed

Closed = Fully completed and accepted

---

## Task Lifecycle

Common workflow:

To Do
|
v
Doing
|
v
Done

Some organizations use:

New
|
v
Active
|
v
Closed

---

# Work Item IDs

Whenever a work item is created, Azure Boards automatically generates a unique ID.

Example:

Feature = 100

User Story = 125

Task = 126

Task = 127

Bug = 128

These IDs are generated automatically by Azure DevOps.

No manual creation is required.

---

# Parent-Child Relationship

Epic (100)
|
+-- Feature (110)
|
+-- User Story (125)
|
+-- Task (126)
+-- Task (127)
+-- Task (128)

Completing a task contributes to User Story completion.

Completing User Stories contributes to Feature completion.

Completing Features contributes to Epic completion.

---

# How DevOps Engineers Receive Work

Usually Product Owner creates:

* Epic
* Feature
* User Story

During Sprint Planning, DevOps-related tasks are assigned.

Examples:

* Create Azure DevOps Pipeline
* Configure AKS Deployment
* Configure Application Gateway
* Setup Monitoring
* Create Terraform Infrastructure

Sometimes the entire User Story is assigned to the DevOps Engineer.

Example:

User Story:
Implement CI/CD Pipeline for Inventory Service

DevOps Engineer creates subtasks:

* Build Pipeline
* SonarQube Integration
* ACR Push
* AKS Deployment

---

# Azure Boards UI Concepts

## Backlog View

ID    Work Item                    State

125   Deploy Payment Service       Active

126   Create Build Pipeline        Active

127   Configure ACR               Done

128   Deploy to AKS               Active

---

## Sprint Board

To Do          Doing          Done

Task 126       Task 128       Task 127

Task 129       Story 125      Task 130

---

# Story Points

Used to estimate effort and complexity.

Common Fibonacci Scale:

1 = Very Small

2 = Small

3 = Small

5 = Medium

8 = Large

13 = Very Large

21 = Extremely Large

---

# Backlog vs Sprint Backlog

Backlog:

Complete list of pending work.

Sprint Backlog:

Selected work items planned for the current sprint.

---

# Burndown Chart

Shows remaining work during the sprint.

Used to track sprint progress.

---

# Velocity Chart

Shows average story points completed per sprint.

Used for future sprint planning.

---

# Linking Azure Boards with Azure Repos

Example:

Work Item ID = 125

Developer or DevOps Engineer commits:

git commit -m "AB#125 Added AKS deployment manifests"

Result:

Work Item #125
|
v
Git Commit

Azure Boards automatically creates the relationship.

---

# Linking Pull Requests

Example:

PR Title:

Deploy Payment Service to AKS AB#125

or

Link work item manually from Pull Request UI.

Result:

Work Item #125
|
v
Pull Request

---

# Linking CI/CD Pipelines

Most projects DO NOT pass Work Item IDs inside pipeline YAML.

Instead, linkage happens automatically.

Flow:

Work Item
|
v
Commit (AB#125)
|
v
Pull Request
|
v
CI Build
|
v
CD Deployment

Azure DevOps automatically tracks the relationship.

---

# Traceability Flow

Azure Board Work Item
|
v
Git Commit
|
v
Pull Request
|
v
Build Pipeline
|
v
Release Pipeline
|
v
Deployment

This provides complete traceability from requirement to production deployment.

---

# Azure Boards and CI/CD Integration

Typical Flow:

User Story #125

|

Code Commit

|

Pull Request

|

Build Pipeline

|

Docker Image Build

|

Push to ACR

|

Deploy to AKS

|

Testing

|

Close User Story

---

# How I Use Azure Boards as a DevOps Engineer

We follow Agile Scrum with 2-week sprints.

During Sprint Planning, DevOps-related work such as:

* CI/CD Pipeline Enhancements
* AKS Deployments
* Terraform Changes
* Monitoring Setup
* Security Remediation
* Production Support Activities

are assigned through Azure Boards.

I update work item status daily, participate in standups, and link commits and pull requests to work items.

Azure Boards integrates with Azure Repos and Azure Pipelines, providing end-to-end traceability from requirements to deployment.

---

# Frequently Asked Interview Questions

Q. What is Azure Boards?

A. Azure Boards is a work tracking service used to manage Epics, Features, User Stories, Tasks, Bugs, Backlogs, and Sprints.

---

Q. What are Work Items?

A. Work items are units of work such as Epics, Features, User Stories, Tasks, and Bugs.

---

Q. What is the hierarchy in Azure Boards?

A.

Epic
|
Feature
|
User Story
|
Task/Bug

---

Q. What is a Sprint?

A. A fixed-duration iteration used to deliver work incrementally.

---

Q. What are Story Points?

A. Relative effort estimates used during Sprint Planning.

---

Q. How do you link commits to Azure Boards?

A.

git commit -m "AB#125 Implemented AKS deployment"

---

Q. How do you link work items to CI/CD pipelines?

A. Usually through commits and pull requests. Azure DevOps automatically tracks the relationship between work items, builds, releases, and deployments.

---

Q. What is Backlog?

A. Prioritized list of pending work.

---

Q. What is Sprint Backlog?

A. Selected work items planned for the current sprint.

---

Q. As a DevOps Engineer, what Azure Boards knowledge is expected?

A.

Must Know:

* Agile
* Scrum
* Sprint
* User Stories
* Tasks
* Bugs
* Work Item Lifecycle
* Backlog
* Sprint Backlog
* Azure Boards Integration with Repos and Pipelines
* Commit Linking using AB#WorkItemID

Nice to Know:

* Epics
* Features
* Story Points
* Burndown Charts
* Velocity Charts
* Azure Boards Administration

Most interviewers focus more on CI/CD, AKS, Terraform, Azure Infrastructure, Monitoring, and Release Management than advanced Azure Boards administration.
