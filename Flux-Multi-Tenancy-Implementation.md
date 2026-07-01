# Flux Multi-Tenancy Implementation Documentation

> **Project Name:** CI/CD Pipeline with Flux GitOps Multi-Tenancy
> **Repository:** testing-proj-gitops
> **Application:** React Application
> **Container Registry:** Docker Hub
> **GitOps Tool:** Flux CD
> **Kubernetes Distribution:** Minikube
> **CI Tool:** GitHub Actions

---

# 1. Introduction

## 1.1 Overview

This document explains the complete implementation of **Flux Multi-Tenancy** in our GitOps project.

Initially, the project was designed to deploy a single React application into the **default Kubernetes namespace** using Flux CD. Every code change followed an automated GitOps pipeline:

```
Developer
      │
      ▼
GitHub Repository
      │
      ▼
GitHub Actions
      │
      ▼
Docker Hub
      │
      ▼
Flux Image Automation
      │
      ▼
GitOps Repository
      │
      ▼
Flux CD
      │
      ▼
Kubernetes Cluster
```

Although this architecture worked perfectly for a single deployment, it did not support deploying different environments or multiple tenants independently.

To solve this limitation, **Flux Multi-Tenancy** was introduced.

Instead of deploying only one application in the default namespace, the same application is now deployed independently into multiple namespaces.

In this project, the following tenants were created:

* Default
* Development (dev)
* Production (prod)

Each tenant has its own Kubernetes resources while sharing the same GitOps workflow.

---

# 1.2 What is Multi-Tenancy?

Multi-Tenancy is a deployment architecture where multiple independent environments (called tenants) share the same Kubernetes cluster while remaining logically isolated from each other.

Instead of creating multiple Kubernetes clusters, a single cluster contains multiple namespaces.

Example:

```
Single Kubernetes Cluster

├── default
│      React App
│
├── dev
│      React App
│
└── prod
       React App
```

Each namespace behaves like an independent environment.

Resources inside one namespace do not interfere with resources in another namespace.

---

# 1.3 Why Multi-Tenancy?

Without Multi-Tenancy:

```
Cluster

└── default
      └── React App
```

Problems:

* Only one deployment exists.
* No environment separation.
* No isolated testing.
* Production and development cannot be managed independently.
* Difficult to demonstrate enterprise GitOps architecture.

---

With Multi-Tenancy:

```
Cluster

├── default
│
├── dev
│
└── prod
```

Advantages:

* Environment isolation
* Independent deployments
* Independent scaling
* Easy testing
* Enterprise-ready architecture
* Better GitOps organization

---

# 1.4 Objectives

The primary objectives of this implementation are:

* Learn Flux Multi-Tenancy
* Deploy multiple environments in one cluster
* Keep GitOps fully automated
* Use one Git repository to manage all tenants
* Automatically update Docker image tags using Flux Image Automation

---

# 1.5 Existing Project Workflow

Before implementing Multi-Tenancy, the workflow was:

```
Developer
      │
      ▼
Git Push
      │
      ▼
GitHub Actions
      │
      ▼
Docker Build
      │
      ▼
Docker Push
      │
      ▼
Flux Image Repository
      │
      ▼
Flux Image Policy
      │
      ▼
Flux Image Automation
      │
      ▼
GitOps Repository
      │
      ▼
Flux Sync
      │
      ▼
Default Namespace
```

Only the **default namespace** was managed.

---

# 1.6 New Workflow

After Multi-Tenancy:

```
Developer
      │
      ▼
GitHub
      │
      ▼
GitHub Actions
      │
      ▼
Docker Hub
      │
      ▼
Flux Image Repository
      │
      ▼
Flux Image Policy
      │
      ▼
Flux Image Automation
      │
      ▼
GitOps Repository
      │
      ▼
Flux Kustomization
      │
      ▼
Minikube Cluster

 ├── default
 ├── dev
 └── prod
```

Every namespace now receives the latest application automatically.

---

# 2. Architecture

## 2.1 Overall Architecture

```
                    +----------------------+
                    |     Developer        |
                    +----------+-----------+
                               |
                               |
                               ▼
                  +-------------------------+
                  | GitHub Source Repository|
                  | Testing_proj_CICD       |
                  +-----------+-------------+
                              |
                              |
                              ▼
                  +-------------------------+
                  | GitHub Actions CI/CD    |
                  +-----------+-------------+
                              |
                              |
                     Build Docker Image
                              |
                              ▼
                 +---------------------------+
                 | Docker Hub Registry       |
                 +-------------+-------------+
                               |
                               |
                               ▼
                +----------------------------+
                | Flux Image Repository      |
                +-------------+--------------+
                              |
                              ▼
                +----------------------------+
                | Flux Image Policy          |
                +-------------+--------------+
                              |
                              ▼
                +----------------------------+
                | Image Update Automation    |
                +-------------+--------------+
                              |
                Update deployment YAML
                              |
                              ▼
              +-------------------------------+
              | GitOps Repository             |
              | testing-proj-gitops           |
              +---------------+---------------+
                              |
                              ▼
                    Flux Kustomization
                              |
                              ▼
                    Kubernetes Cluster

        +-----------------------------------------+
        |               Minikube                  |
        |                                         |
        |   Namespace : default                   |
        |   Namespace : dev                       |
        |   Namespace : prod                      |
        +-----------------------------------------+
```

---

## 2.2 Components

### GitHub Actions

Responsible for

* Building Docker Image
* Tag Generation
* Docker Hub Push

---

### Docker Hub

Stores container images.

Example:

```
uditdataxis12345/testing-proj-cicd
```

---

### Flux Image Repository

Continuously scans Docker Hub.

Example:

```
Every 1 minute
```

It detects newly pushed image tags.

---

### Flux Image Policy

Chooses which image tag should be deployed.

Example:

```
Latest Timestamp
```

---

### Flux Image Automation

Automatically updates

```
deployment.yaml
```

inside GitOps Repository.

---

### GitOps Repository

Contains Kubernetes manifests.

Current repository:

```
testing-proj-gitops
```

---

### Flux Kustomization

Reads Kubernetes manifests from GitHub and applies them to the cluster.

---

### Kubernetes Cluster

Runs the application.

Namespaces:

```
default

dev

prod
```

---

# 3. Folder Structure (Before / After)

## 3.1 Before Multi-Tenancy

```
testing-proj-gitops
│
├── clusters
│   └── my-cluster
│       │
│       ├── apps
│       │   └── react-app
│       │       ├── deployment.yaml
│       │       ├── service.yaml
│       │       └── kustomization.yaml
│       │
│       ├── flux-system
│       │
│       ├── image-automation.yaml
│       │
│       └── kustomization.yaml
```

Only one application existed.

Everything was deployed into

```
default namespace
```

---

## 3.2 After Multi-Tenancy

```
testing-proj-gitops

clusters
└── my-cluster
    │
    ├── apps
    │   └── react-app
    │
    ├── tenants
    │   ├── dev
    │   │   ├── deployment.yaml
    │   │   ├── service.yaml
    │   │   └── kustomization.yaml
    │   │
    │   └── prod
    │       ├── deployment.yaml
    │       ├── service.yaml
    │       └── kustomization.yaml
    │
    ├── flux-system
    │
    ├── image-automation.yaml
    │
    └── kustomization.yaml
```

---

## 3.3 Why Tenants Folder?

Instead of placing every deployment inside **apps**, we created a dedicated **tenants** directory.

Reason:

* Better organization
* Environment separation
* Independent deployment
* Easy scalability

Future example:

```
tenants

├── dev
├── prod
├── qa
├── uat
├── customer-a
├── customer-b
├── customer-c
```

This structure is commonly used in enterprise GitOps repositories.

---

# 4. Complete Implementation

## Step 1 — Existing Deployment

Initially, only one deployment existed:

```
default namespace
```

Resources:

* Deployment
* Service

---

## Step 2 — Create Tenant Folder

A new folder named

```
tenants
```

was created inside

```
clusters/my-cluster
```

Inside it, two tenants were created:

```
dev

prod
```

---

## Step 3 — Create Dev Tenant

Inside

```
clusters/my-cluster/tenants/dev
```

the following files were added:

```
deployment.yaml

service.yaml

kustomization.yaml
```

The deployment uses

* Namespace: dev
* Replica: 1
* Separate Service

---

## Step 4 — Create Production Tenant

Inside

```
clusters/my-cluster/tenants/prod
```

the following files were created:

```
deployment.yaml

service.yaml

kustomization.yaml
```

Production uses

```
Replica = 3
```

to simulate a production environment.

---

## Step 5 — Update Root Kustomization

Before:

```
apps/react-app
```

was the only resource.

After:

```
apps/react-app

tenants/dev

tenants/prod
```

were added.

Flux now applies all three deployments.

---

## Step 6 — Enable Image Automation

Initially Image Automation updated only

```
apps/react-app
```

Later, it was modified so that Flux scans the entire

```
clusters/my-cluster
```

directory.

As a result, image tags are automatically updated inside:

* apps/react-app
* tenants/dev
* tenants/prod

---

## Step 7 — Configure GitHub Authentication

Image Automation needs permission to push commits.

A GitHub Personal Access Token (PAT) was created.

The Kubernetes Secret was updated.

Flux was configured to authenticate against GitHub.

---

## Step 8 — Automatic GitOps Flow

After everything was configured:

```
Code Push

↓

GitHub Actions

↓

Docker Image Build

↓

Docker Hub Push

↓

Flux detects image

↓

Flux updates deployment YAML

↓

Flux commits to GitHub

↓

Flux reconciles

↓

Kubernetes updates deployments

↓

Application updated automatically
```

---

# 5. Detailed Explanation of Every File

This section explains the purpose of every important file used in the project, why it was created, and how it contributes to the overall GitOps workflow.

---

# 5.1 clusters/my-cluster/kustomization.yaml

## Purpose

This is the **Root Kustomization** file of the Kubernetes cluster.

Flux starts reading Kubernetes manifests from this file.

It acts as the entry point of the complete GitOps repository.

---

## Current Configuration

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ./apps/react-app
  - ./image-automation.yaml
  - ./tenants/dev
  - ./tenants/prod
```

---

## Explanation

This file tells Flux to deploy four resources.

### apps/react-app

Deploys the application inside the default namespace.

---

### image-automation.yaml

Creates:

* ImageRepository
* ImagePolicy
* ImageUpdateAutomation

---

### tenants/dev

Deploys application inside the Development namespace.

---

### tenants/prod

Deploys application inside the Production namespace.

---

## Why Required?

Without this file Flux would never know which Kubernetes manifests should be applied.

It is the root of the entire deployment tree.

---

# 5.2 apps/react-app/deployment.yaml

## Purpose

Deploys the application inside the **default namespace**.

---

## Responsibilities

* Creates Deployment
* Pulls Docker image
* Starts Pods
* Uses DockerHub credentials
* Uses latest image selected by Flux

---

## Important Fields

### metadata.name

```yaml
name: react-app
```

Deployment name.

---

### namespace

```yaml
namespace: default
```

Application will be deployed inside the default namespace.

---

### replicas

```yaml
replicas: 1
```

Only one pod runs.

---

### image

```yaml
image: uditdataxis12345/testing-proj-cicd:<tag>
```

Docker image.

Flux automatically updates this value.

---

### Flux Setter

```yaml
# {"$imagepolicy": "flux-system:react-app"}
```

This comment is extremely important.

Flux Image Automation searches for this setter and replaces only the image tag.

Without this comment automatic image updates will never work.

---

### imagePullSecrets

```yaml
dockerhub-credentials
```

Used for pulling Docker images from DockerHub.

---

# 5.3 apps/react-app/service.yaml

## Purpose

Exposes the application inside Kubernetes.

---

### Type

```yaml
NodePort
```

Application becomes accessible outside the cluster.

---

### NodePort

Example

```yaml
30007
```

Application URL

```
http://<Minikube-IP>:30007
```

---

# 5.4 apps/react-app/kustomization.yaml

## Purpose

Groups all application resources together.

```yaml
resources:
- deployment.yaml
- service.yaml
```

Without this file Kustomize cannot build manifests.

---

# 5.5 tenants/dev/deployment.yaml

## Purpose

Deploys the application for Development Environment.

---

## Namespace

```yaml
namespace: dev
```

Resources remain isolated.

---

## Replica

```yaml
replicas: 1
```

Development requires only one pod.

---

## Image

Uses the same Docker image as production.

Flux updates image automatically.

---

## Labels

```yaml
app: react-app-dev
```

Unique labels prevent conflicts with production.

---

# 5.6 tenants/dev/service.yaml

Purpose

Expose Development application.

NodePort

```yaml
30008
```

---

# 5.7 tenants/dev/kustomization.yaml

Purpose

Groups Development resources.

```yaml
resources:
- deployment.yaml
- service.yaml
```

Also injects

```yaml
namespace: dev
```

automatically.

---

# 5.8 tenants/prod/deployment.yaml

Purpose

Deploy Production environment.

---

Replica

```yaml
replicas: 3
```

Production should be highly available.

---

Labels

```yaml
app: react-app-prod
```

Keeps Production isolated.

---

# 5.9 tenants/prod/service.yaml

Purpose

Expose Production application.

NodePort

```yaml
30009
```

---

# 5.10 tenants/prod/kustomization.yaml

Purpose

Groups all Production resources.

```yaml
resources:
- deployment.yaml
- service.yaml
```

---

# 5.11 image-automation.yaml

This is the most important file in the GitOps repository.

It contains three Flux resources.

---

## ImageRepository

Purpose

Continuously scans DockerHub.

Example

```
DockerHub

↓

Latest Image Tag
```

---

## ImagePolicy

Purpose

Selects which image should be deployed.

Our policy

```
Latest Timestamp
```

---

## ImageUpdateAutomation

Purpose

Updates deployment.yaml automatically.

Workflow

```
Latest Docker Image

↓

Find Setter

↓

Replace Tag

↓

Commit

↓

Push GitHub
```

---

## Update Path

Initially

```yaml
path: ./clusters/my-cluster/apps/react-app
```

Problem

Only default deployment updated.

---

Final

```yaml
path: ./clusters/my-cluster
```

Now Flux updates

* default
* dev
* prod

automatically.

---

# 5.12 flux-system/gotk-sync.yaml

Purpose

Bootstrap file created by Flux.

Contains

* GitRepository
* Root Kustomization

Flux continuously watches this repository.

---

# 5.13 flux-system/gotk-components.yaml

Contains all Flux Controllers.

Examples

* Source Controller
* Kustomize Controller
* Notification Controller
* Helm Controller
* Image Automation Controller
* Image Reflector Controller

Generated automatically during bootstrap.

Never modify manually.

---

# 6. GitHub Changes

While implementing Multi-Tenancy several changes were required in GitHub.

---

# 6.1 CI Repository

Repository

```
Testing_proj_CICD
```

Responsibilities

* Build React Application
* Build Docker Image
* Push Docker Image

---

# 6.2 GitOps Repository

Repository

```
testing-proj-gitops
```

Responsibilities

* Store Kubernetes YAML
* Store Flux manifests
* Store Tenant manifests
* Receive automatic commits from Flux

---

# 6.3 GitHub Actions Workflow

Workflow

```
.github/workflows/cd.yml
```

Responsibilities

* Checkout repository
* Login DockerHub
* Generate Timestamp Tag
* Build Docker Image
* Push Docker Image

---

Generated Tag Example

```
20260624071242-7e8df276
```

This unique tag prevents image caching.

---

# 6.4 GitHub Personal Access Token

Flux Image Automation requires write permission.

A Personal Access Token (PAT) was created.

Permission

```
Repository Contents

Read

Write
```

---

# 6.5 Why PAT?

Flux performs

```
Git Commit

↓

Git Push
```

automatically.

Without authentication GitHub rejects write requests.

---

# 6.6 Automatic GitHub Commit

After successful image detection Flux automatically creates

```
ci: update react-app image [skip ci]
```

commit.

Purpose

Avoid manual deployment updates.

---

# 7. Flux Changes

Initially Flux managed only one deployment.

After Multi-Tenancy several changes were introduced.

---

# 7.1 Root Kustomization

Before

```
apps/react-app
```

After

```
apps/react-app

tenants/dev

tenants/prod
```

Flux now deploys three environments.

---

# 7.2 Flux Bootstrap

Already existed.

No change required.

Flux continues watching

```
clusters/my-cluster
```

---

# 7.3 GitRepository

Repository changed from old GitLab configuration to GitHub.

Flux now watches

```
testing-proj-gitops
```

---

# 7.4 Reconciliation

Whenever Git changes

Flux performs

```
Pull

↓

Compare

↓

Apply

↓

Update Cluster
```

No manual deployment required.

---

# 7.5 Inventory

Flux internally maintains inventory.

Tracks

* Deployments
* Services
* ImagePolicy
* ImageRepository
* ImageAutomation

Useful for pruning deleted resources.

---

# 8. Image Automation

Image Automation is the core feature of this project.

Without it deployments would need manual image updates.

---

## Complete Flow

```
Developer

↓

Git Push

↓

GitHub Actions

↓

Docker Build

↓

DockerHub

↓

Image Repository

↓

Image Policy

↓

Image Update Automation

↓

Git Commit

↓

GitHub

↓

Flux Reconcile

↓

Kubernetes Deployment

↓

Pods Restart
```

---

## ImageRepository

Scans DockerHub.

Example

```
Every 1 minute
```

---

## ImagePolicy

Selects

```
Latest Timestamp
```

image.

---

## ImageUpdateAutomation

Reads ImagePolicy.

Searches

```
$imagepolicy
```

setter.

Updates deployment files.

Commits to GitHub.

---

## Why Setters?

Instead of replacing the whole YAML,

Flux only replaces

```
Image Tag
```

Everything else remains unchanged.

---

## Files Updated Automatically

```
apps/react-app/deployment.yaml

tenants/dev/deployment.yaml

tenants/prod/deployment.yaml
```

---

## Final Result

Whenever a developer pushes code,

everything happens automatically:

```
Developer

↓

GitHub Actions

↓

Docker Image

↓

DockerHub

↓

Flux detects image

↓

Flux updates deployment.yaml

↓

GitHub commit

↓

Flux Sync

↓

Cluster Updated

↓

Application Running
```

No manual deployment, image editing, or Kubernetes update is required anymore.

This completes the end-to-end GitOps automation for the project.

# 9. Flux Multi-Tenancy Setup

This section explains the complete implementation of Multi-Tenancy inside the existing Flux GitOps project.

Unlike Hub-and-Spoke architecture, this project uses **Standalone Flux Architecture**, where a single Flux instance manages multiple tenants (namespaces) inside a single Kubernetes cluster.

---

# 9.1 Existing Architecture

Initially the project contained only one application deployed into the **default namespace**.

```text
Minikube Cluster

└── default
      ├── Deployment
      └── Service
```

This deployment worked correctly but did not provide any environment separation.

---

# 9.2 Goal of Multi-Tenancy

The objective was to deploy the same application into multiple isolated namespaces while using:

* One Kubernetes Cluster
* One Flux Installation
* One GitOps Repository
* One Docker Image
* One Image Automation

Target Architecture:

```text
Minikube Cluster

├── default
│     React Application
│
├── dev
│     React Application
│
└── prod
      React Application
```

---

# 9.3 Why Namespace-Based Multi-Tenancy?

There are two common approaches.

## Option 1

Separate Kubernetes Cluster

```text
Cluster-1

Cluster-2

Cluster-3
```

Advantages

* Strong Isolation

Disadvantages

* High Infrastructure Cost
* Multiple Flux Installations
* Complex Management

---

## Option 2

Single Cluster

Multiple Namespaces

```text
Minikube

├── default
├── dev
└── prod
```

Advantages

* Lightweight
* Easy to manage
* Enterprise GitOps Practice
* Less Resource Consumption

This project uses this approach.

---

# 9.4 Tenant Directory Structure

A new directory named

```text
tenants
```

was created inside

```text
clusters/my-cluster
```

Final structure

```text
clusters
└── my-cluster
      │
      ├── apps
      │
      ├── tenants
      │     ├── dev
      │     │      deployment.yaml
      │     │      service.yaml
      │     │      kustomization.yaml
      │     │
      │     └── prod
      │            deployment.yaml
      │            service.yaml
      │            kustomization.yaml
      │
      ├── flux-system
      │
      ├── image-automation.yaml
      │
      └── kustomization.yaml
```

---

# 9.5 Development Tenant

Namespace

```text
dev
```

Resources

* Deployment
* Service
* Kustomization

Replica Count

```yaml
replicas: 1
```

Reason

Development environment requires only one Pod.

---

# 9.6 Production Tenant

Namespace

```text
prod
```

Resources

* Deployment
* Service
* Kustomization

Replica Count

```yaml
replicas: 3
```

Reason

Production requires High Availability.

---

# 9.7 Root Kustomization Changes

Before

```yaml
resources:
- ./apps/react-app
```

After

```yaml
resources:
- ./apps/react-app
- ./image-automation.yaml
- ./tenants/dev
- ./tenants/prod
```

Now Flux deploys all environments automatically.

---

# 9.8 Image Automation

Initially Image Automation updated only

```text
apps/react-app
```

Later it was modified to

```yaml
path: ./clusters/my-cluster
```

Now Flux updates

* Default
* Dev
* Production

simultaneously.

---

# 9.9 Final Multi-Tenant Deployment

```text
Minikube Cluster

default namespace

    React App
         │
         ▼

dev namespace

    React App
         │
         ▼

prod namespace

    React App
```

Each namespace has

* Separate Deployment
* Separate Service
* Separate Pods
* Separate Replica Count

while sharing

* Same Cluster
* Same Docker Image
* Same GitOps Repository
* Same Flux Controllers

---

# 10. Verification

After implementation, multiple verification steps were performed.

---

# 10.1 Verify Namespaces

Command

```bash
kubectl get ns
```

Expected Output

```text
default

dev

prod

flux-system
```

Purpose

Confirms namespace creation.

---

# 10.2 Verify Deployments

```bash
kubectl get deployment -A
```

Expected

```text
default

react-app

dev

react-app-dev

prod

react-app-prod
```

Purpose

Ensures all deployments are created.

---

# 10.3 Verify Pods

```bash
kubectl get pods -A
```

Expected

Pods should exist inside

* default
* dev
* prod

---

# 10.4 Verify Services

```bash
kubectl get svc -A
```

Expected

```text
react-app-service

react-app-dev-service

react-app-prod-service
```

---

# 10.5 Verify Flux

```bash
flux get all
```

Expected

```text
GitRepository

ImageRepository

ImagePolicy

ImageUpdateAutomation

Kustomization
```

READY

```text
True
```

---

# 10.6 Verify Image Automation

```bash
flux get image all
```

Expected

Latest Docker image tag.

---

# 10.7 Verify Image Update

```bash
grep image:
clusters/my-cluster/apps/react-app/deployment.yaml

grep image:
clusters/my-cluster/tenants/dev/deployment.yaml

grep image:
clusters/my-cluster/tenants/prod/deployment.yaml
```

Expected

All three deployment files should contain the latest Docker image tag.

---

# 10.8 Verify Running Image

```bash
kubectl get deployment react-app-dev \
-n dev \
-o=jsonpath='{.spec.template.spec.containers[0].image}'
```

Repeat for

* default
* prod

Purpose

Confirms Kubernetes is running the latest image.

---

# 10.9 Verify Flux Commit

Open GitHub Repository.

Expected Commit

```text
ci: update react-app image [skip ci]
```

Purpose

Confirms Image Automation is working.

---

# 10.10 Verify URLs

Default

```text
minikube service react-app-service --url
```

Development

```text
minikube service react-app-dev-service -n dev --url
```

Production

```text
minikube service react-app-prod-service -n prod --url
```

All three applications should open successfully.

---

# 11. Frequently Used Commands

---

## Flux Commands

Reconcile Source

```bash
flux reconcile source git flux-system
```

Reconcile Kustomization

```bash
flux reconcile kustomization flux-system --with-source
```

Reconcile Image Automation

```bash
flux reconcile image update react-app \
-n flux-system
```

---

## Flux Status

```bash
flux get all
```

```bash
flux get kustomizations
```

```bash
flux get images all
```

```bash
flux get sources git
```

---

## Kubernetes Commands

Namespaces

```bash
kubectl get ns
```

Pods

```bash
kubectl get pods -A
```

Deployments

```bash
kubectl get deployment -A
```

Services

```bash
kubectl get svc -A
```

Describe Deployment

```bash
kubectl describe deployment react-app-dev -n dev
```

Describe Kustomization

```bash
kubectl describe kustomization flux-system \
-n flux-system
```

---

## Kustomize

```bash
kubectl kustomize clusters/my-cluster
```

Useful for validating manifests before deployment.

---

## Manual Apply

```bash
kubectl apply -k clusters/my-cluster
```

Useful during testing.

---

## Rollout Status

```bash
kubectl rollout status deployment/react-app-dev \
-n dev
```

---

## Logs

Flux Logs

```bash
kubectl logs deployment/image-automation-controller \
-n flux-system
```

Kustomize Logs

```bash
kubectl logs deployment/kustomize-controller \
-n flux-system
```

---

# 12. Complete Troubleshooting Guide

This project encountered several real-world issues while implementing Multi-Tenancy.

The following sections describe each problem, its root cause, and the final solution.

---

# Issue 1

## Problem

```text
kustomization path not found
```

Example

```text
stat /tmp/kustomization.../tenants/dev
```

### Cause

Tenant directory was created outside the Flux root path.

Flux only scans resources inside

```text
clusters/my-cluster
```

### Solution

Move

```text
tenants
```

inside

```text
clusters/my-cluster
```

---

# Issue 2

## Problem

```text
No resources found in dev namespace
```

### Cause

Tenant directories were not included inside

```yaml
kustomization.yaml
```

### Solution

Add

```yaml
resources:
- ./tenants/dev
- ./tenants/prod
```

---

# Issue 3

## Problem

Only default namespace updated.

Development and Production never received new images.

### Cause

ImageUpdateAutomation path

```yaml
./clusters/my-cluster/apps/react-app
```

only watched one folder.

### Solution

Update

```yaml
path:
./clusters/my-cluster
```

---

# Issue 4

## Problem

```text
context deadline exceeded
```

during

```bash
flux reconcile source git
```

### Cause

Flux was still trying to authenticate using old GitLab credentials.

### Solution

Update GitRepository.

Replace GitLab Secret.

Point Flux to GitHub.

---

# Issue 5

## Problem

```text
authentication required

No anonymous write access
```

### Cause

Flux Image Automation had no GitHub authentication.

### Solution

Create GitHub PAT.

Create Kubernetes Secret.

Configure GitRepository to use GitHub credentials.

---

# Issue 6

## Problem

```text
Image tag never updated
```

### Cause

Flux could not push commits to GitHub.

### Solution

Fix GitHub authentication.

After authentication Flux automatically created

```text
ci: update react-app image [skip ci]
```

commit.

---

# Issue 7

## Problem

```text
Node 20 Deprecated

Docker QEMU Error
```

### Cause

Old GitHub Actions version.

### Solution

Upgrade GitHub Actions versions.

Example

```yaml
docker/setup-qemu-action@v3

docker/setup-buildx-action@v3

docker/build-push-action@v6
```

---

# Issue 8

## Problem

Deployment existed but Pods never became Ready.

### Cause

Docker image was not updated.

### Solution

Verify

* DockerHub Image
* Flux ImagePolicy
* ImageAutomation
* Deployment Image Tag

---

# Final Troubleshooting Checklist

Whenever the deployment fails, verify the following sequence:

```text
GitHub Push
      │
      ▼
GitHub Actions Success
      │
      ▼
Docker Image Available
      │
      ▼
Flux Image Repository Updated
      │
      ▼
Image Policy Updated
      │
      ▼
Image Automation Commit Created
      │
      ▼
GitHub GitOps Repository Updated
      │
      ▼
Flux Reconciliation Successful
      │
      ▼
Deployment Updated
      │
      ▼
Pods Restarted
      │
      ▼
Application Running
```

If every step above succeeds, the complete GitOps pipeline is functioning correctly.

# 13. Errors Faced During Implementation and Their Solutions

While implementing Flux Multi-Tenancy, several real-world issues were encountered. Instead of simply applying the official documentation, each issue was analyzed, debugged, and resolved. This section documents every major problem, its root cause, and the final solution.

---

# Error 1 : `kustomization path not found`

## Error

```text
kustomization path not found:
stat /tmp/kustomization.../tenants/dev:
no such file or directory
```

---

## When did it occur?

After creating the `dev` and `prod` tenants, Flux failed to reconcile the Kustomization resources.

---

## Root Cause

Initially, the `tenants` folder was created outside the Flux root directory.

Example:

```text
testing-proj-gitops

├── tenants
│     ├── dev
│     └── prod

└── clusters
      └── my-cluster
```

Flux only scans the directory configured inside `gotk-sync.yaml`.

```yaml
path: ./clusters/my-cluster
```

Anything outside this directory is completely ignored.

---

## Solution

Move the entire tenants folder inside the cluster directory.

Correct structure

```text
clusters
└── my-cluster
      ├── apps
      ├── tenants
      │     ├── dev
      │     └── prod
      ├── flux-system
      └── image-automation.yaml
```

---

# Error 2 : No Resources Found in Namespace

## Error

```text
kubectl get pods -n dev

No resources found.
```

---

## Root Cause

The tenant manifests existed but were never included in the Root Kustomization.

Flux therefore never applied them.

---

## Wrong Configuration

```yaml
resources:
- ./apps/react-app
```

---

## Correct Configuration

```yaml
resources:
- ./apps/react-app
- ./image-automation.yaml
- ./tenants/dev
- ./tenants/prod
```

---

## Verification

```bash
kubectl get deployment -A
```

Expected

```text
default

react-app

dev

react-app-dev

prod

react-app-prod
```

---

# Error 3 : Flux Created Only One Deployment

## Problem

Only

```text
default namespace
```

received updates.

Development and Production never updated.

---

## Root Cause

Image Automation path

```yaml
path:
./clusters/my-cluster/apps/react-app
```

Flux was scanning only one folder.

---

## Solution

Change

```yaml
update:
  path: ./clusters/my-cluster
```

Now Flux scans every deployment file inside the cluster.

---

# Error 4 : Image Tag Never Updated

## Symptoms

Docker image was successfully pushed.

ImageRepository detected the latest tag.

ImagePolicy selected the latest image.

But deployment.yaml remained unchanged.

---

## Root Cause

Image Automation updated only

```text
apps/react-app
```

and ignored

```text
tenants/dev

tenants/prod
```

---

## Solution

1.

Add Flux setter

```yaml
# {"$imagepolicy": "flux-system:react-app"}
```

inside every deployment.

2.

Update Image Automation path.

3.

Reconcile Flux.

---

# Error 5 : Flux Still Reading GitLab Repository

## Problem

Even after changing GitHub URL,

Flux still attempted to communicate with GitLab.

---

## Root Cause

Old GitRepository Secret

```text
gitlab-auth
```

was still configured.

---

## Solution

Update GitRepository.

Replace GitLab configuration with GitHub.

Verify

```bash
kubectl get gitrepository flux-system \
-o yaml
```

Expected

```text
https://github.com/...
```

---

# Error 6 : Context Deadline Exceeded

## Error

```text
context deadline exceeded
```

---

## Root Cause

Flux couldn't authenticate with GitHub.

---

## Solution

* Remove old GitLab Secret
* Create GitHub PAT
* Configure GitRepository
* Reconcile Source

---

# Error 7 : Authentication Required

## Error

```text
authentication required

No anonymous write access
```

---

## Root Cause

Flux could pull the repository because it was public.

However,

Image Automation needs

```text
Write Permission
```

to create commits.

---

## Solution

Create

GitHub Personal Access Token

Permissions

```text
Repository

Contents

Read

Write
```

Create Kubernetes Secret

Configure GitRepository

Verify automatic commits.

---

# Error 8 : Flux Never Created Commit

## Expected

```text
ci: update react-app image [skip ci]
```

---

## Actual

No commit appeared.

---

## Root Cause

Flux lacked GitHub credentials.

---

## Solution

GitHub PAT

↓

GitHub Secret

↓

Flux Authentication

↓

Automatic Commit

---

# Error 9 : Flux Detected Image but Deployment Didn't Change

## Root Cause

Setter comment missing.

Flux replaces only fields containing

```yaml
# {"$imagepolicy": "flux-system:react-app"}
```

---

## Solution

Add setter in

```text
apps/react-app

tenants/dev

tenants/prod
```

---

# Error 10 : GitHub Actions Failed

## Error

```text
Node 20 Deprecated

QEMU Error
```

---

## Root Cause

Old GitHub Actions versions.

---

## Solution

Upgrade Actions

```yaml
docker/setup-qemu-action@v3

docker/setup-buildx-action@v3

docker/build-push-action@v6
```

---

# Error 11 : kubectl apply -k Failed

## Error

```text
flag needs an argument
```

---

## Cause

Incorrect command.

---

## Wrong

```bash
kubectl apply -k
```

---

## Correct

```bash
kubectl apply -k clusters/my-cluster
```

---

# Error 12 : Deployment Updated but Pods Didn't Restart

## Root Cause

Flux reconciliation had not yet occurred.

---

## Solution

```bash
flux reconcile source git flux-system

flux reconcile kustomization flux-system --with-source
```

---

# Error 13 : Wrong Folder Structure

Initially

```text
clusters

apps

tenants
```

were located incorrectly.

Flux could not build the manifests.

---

## Final Structure

```text
clusters
└── my-cluster
      ├── apps
      ├── tenants
      ├── flux-system
      ├── image-automation.yaml
      └── kustomization.yaml
```

---

# Final Lessons Learned

During implementation, the following key learnings were obtained:

* Flux only watches the configured root path.
* Every deployment must contain the Flux image setter.
* Image Automation requires GitHub write access.
* GitRepository must point to the correct repository.
* Multi-Tenancy is easiest to implement using namespaces.
* Root Kustomization controls the complete deployment tree.
* ImagePolicy only selects images; it does not deploy them.
* ImageUpdateAutomation performs Git commits.
* Flux Kustomization applies manifests to Kubernetes.
* GitOps repositories should always be clean and well structured.

---

# 14. Final GitOps Flow

The following diagram represents the complete GitOps workflow implemented in this project.

```text
                     Developer
                          │
                          │
                 Git Push Source Code
                          │
                          ▼
        +--------------------------------+
        | Testing_proj_CICD Repository   |
        +--------------------------------+
                          │
                          ▼
        +--------------------------------+
        | GitHub Actions Workflow         |
        +--------------------------------+
                          │
          Build Docker Image + Push Image
                          │
                          ▼
        +--------------------------------+
        | Docker Hub                     |
        +--------------------------------+
                          │
                          ▼
        +--------------------------------+
        | Flux Image Repository          |
        +--------------------------------+
                          │
                          ▼
        +--------------------------------+
        | Flux Image Policy              |
        +--------------------------------+
                          │
                          ▼
        +--------------------------------+
        | Flux Image Update Automation   |
        +--------------------------------+
                          │
          Update Deployment YAML Files
                          │
                          ▼
        +--------------------------------+
        | testing-proj-gitops Repository |
        +--------------------------------+
                          │
             Automatic Git Commit
                          │
                          ▼
        +--------------------------------+
        | Flux Git Repository            |
        +--------------------------------+
                          │
                          ▼
        +--------------------------------+
        | Flux Kustomization             |
        +--------------------------------+
                          │
                          ▼
              Kubernetes Cluster
                          │
      ┌───────────┬────────────┬────────────┐
      ▼           ▼            ▼
  default       dev          prod
      │           │            │
      ▼           ▼            ▼
 React App    React App    React App
```

---

# Complete Deployment Flow

```text
Developer

↓

Git Push

↓

GitHub Actions

↓

Docker Build

↓

Docker Push

↓

Docker Hub

↓

Flux Image Repository

↓

Flux Image Policy

↓

Image Update Automation

↓

Git Commit

↓

GitOps Repository

↓

Flux Reconcile

↓

Kubernetes Deployment

↓

Pods Restart

↓

Application Updated
```

---

# 15. Interview Questions

## Basic

### Q1. What is GitOps?

**Answer**

GitOps is a deployment methodology where Git acts as the single source of truth. Every infrastructure and application change is stored in Git, and tools like Flux automatically synchronize the Kubernetes cluster with the repository.

---

### Q2. What is Flux CD?

Flux is a GitOps operator that continuously monitors a Git repository and reconciles Kubernetes resources with the desired state stored in Git.

---

### Q3. What is Multi-Tenancy?

Multi-Tenancy is a deployment architecture where multiple isolated environments share the same Kubernetes cluster.

Example

```text
default

dev

prod
```

---

### Q4. Why use Namespaces?

Namespaces provide logical isolation between environments while sharing the same cluster.

---

### Q5. Difference between Namespace and Cluster?

| Namespace         | Cluster                         |
| ----------------- | ------------------------------- |
| Logical isolation | Physical Kubernetes environment |
| Lightweight       | Heavyweight                     |
| Shares resources  | Independent resources           |

---

### Q6. What is Flux Image Repository?

Scans DockerHub periodically and discovers available image tags.

---

### Q7. What is Flux Image Policy?

Selects which image tag should be deployed.

---

### Q8. What is Image Update Automation?

Automatically updates Kubernetes manifests with the latest Docker image and commits the changes to Git.

---

### Q9. Why use Image Setter?

The setter allows Flux to update only the Docker image tag without modifying the rest of the YAML file.

---

### Q10. Why GitHub PAT?

Flux requires write access to create automatic commits in the GitOps repository.

---

### Q11. Why Separate GitOps Repository?

It separates application source code from deployment manifests, making GitOps cleaner and easier to manage.

---

### Q12. Why Replica Count is Different?

Development

```text
1 Replica
```

Production

```text
3 Replicas
```

This simulates a production environment with higher availability.

---

### Q13. What happens after Git Push?

Complete answer

```text
Git Push

↓

GitHub Actions

↓

Docker Build

↓

Docker Hub

↓

Flux Detects Image

↓

Image Policy

↓

Image Automation

↓

Git Commit

↓

Flux Reconcile

↓

Deployment Update

↓

Pods Restart
```

---

# 16. Production Improvements

The current project demonstrates a complete GitOps pipeline suitable for learning and portfolio purposes.

To make it production-ready, the following improvements can be implemented.

---

## 1. Helm Charts

Replace raw YAML manifests with Helm charts for reusable deployments.

---

## 2. HelmRelease

Manage applications using Flux HelmRelease resources instead of plain Kustomize.

---

## 3. Ingress Controller

Instead of NodePort

Use

```text
NGINX Ingress Controller
```

Domains

```text
dev.example.com

prod.example.com
```

---

## 4. Monitoring

Integrate

* Prometheus
* Grafana

to monitor cluster health and application metrics.

---

## 5. Logging

Use

* Loki
* Promtail

or

* ELK Stack

for centralized logging.

---

## 6. Secrets Management

Replace plain Secrets with

* Sealed Secrets
* External Secrets Operator
* HashiCorp Vault

---

## 7. Horizontal Pod Autoscaler

Automatically increase replicas based on CPU or Memory utilization.

---

## 8. Resource Limits

Configure

```yaml
requests:

limits:
```

for CPU and Memory.

---

## 9. Network Policies

Restrict communication between namespaces for improved security.

---

## 10. RBAC

Create tenant-specific Roles and RoleBindings.

---

## 11. Multiple Clusters

Upgrade from

```text
Standalone Flux
```

to

```text
Hub and Spoke Architecture
```

where a management cluster controls multiple production clusters.

---

## 12. Progressive Delivery

Use

```text
Flagger

or

Argo Rollouts
```

for Canary and Blue-Green deployments.

---

# Conclusion

This project successfully demonstrates an end-to-end GitOps pipeline using **GitHub Actions, Docker Hub, Flux CD, Kubernetes, and Multi-Tenancy**. It covers automated image builds, image scanning, policy-driven updates, GitOps synchronization, and deployment into multiple namespaces (`default`, `dev`, and `prod`) within a single Minikube cluster.

During implementation, several real-world challenges—including authentication issues, path configuration errors, image automation problems, and Flux reconciliation failures—were identified and resolved. These troubleshooting experiences provide valuable practical knowledge beyond a basic tutorial.

The final solution represents a production-inspired GitOps workflow where every code push automatically propagates through the CI/CD pipeline, updates the GitOps repository, and deploys the latest application version to Kubernetes without manual intervention.
