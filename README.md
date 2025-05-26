# 🏡 Homelab

## Introduction

This repository contains the configuration and documentation of my homelab environment.

The primary goal of this homelab is both educational and recreational. Lately, I’ve been exploring Kubernetes, and there’s no better way to learn than by running a cluster at home. Additionally, self-hosting applications allows me to take full ownership of the deployment and maintenance process—from start to finish.

## ⚙️ Operation System

<img src="https://www.talos.dev/images/logo.svg" width="100">

I decided to use [Talos Linux](https://github.com/siderolabs/talos) to set up my machines.
Talos is a Linux distribution purpose-built for running Kubernetes.
It’s lightweight, efficient, and relatively easy to set up—especially for home lab environments—making it an ideal fit for my needs.

## 🖥️ Hardware

I use a refurbished second-hand mini pc. They're great because they are small and cheap to buy.

- DELL Optiplex 7040 i5-6500T/8GB/240GB SSD

## 📁 Project Structure

This project is organized into two primary domains:

- Applications — End-user applications.
- Databases — Database and state management resources.

Each domain has its own controllers and configurations, all organized under `infrastructure` folder.

```bash
├── apps
├── databases
├── clusters # FluxCD bootstrap manifest
└── infrastructure # Infrastructure for `apps` and `databases`
    ├── configs # Depends on controllers
    └── controllers
```

**Future Roadmap:**
The plan is to decouple the data layer from the application, 
which will involve establishing two separate clusters—one dedicated to 
data and the other to application services for improved scalability and maintainability.

The diagram below outlines the planned changes to the folder structure:

```bash
├── apps
├── databases
├── clusters
│   ├── data # Reconciles changes for `databases` and `infrastructure/**/data`
│   └── production # Reconciles changes for `apps` and `infrastructure/**/production`
└── infrastructure # Infrastructure for `apps` and `databases`
    ├── configs
    │   ├── base # Base configuration
    │   ├── data # Config for `databases` infrastructure
    │   └── production # Config for `apps` infrastructure
    └── controllers
        ├── base # Base controller manifest
        ├── data # Controller for `databases` infrastructure
        └── production # Controller for `apps` infrastructure
```

## 🔐 Secret Management

<div style="display: flex; gap: 10px">
    <img src="https://www.isjw.uk/images/azure/keyvault.png" height="50"/>
    <img src="https://external-secrets.io/latest/pictures/eso-round-logo.svg" height="50">
</div>

I decided to use [Azure Key Vault](https://azure.microsoft.com/en-us/products/key-vault) and inject them into cluster with [External Secrets Operator](https://external-secrets.io/).

**Why cloud secret provider instead of self-hosted?**

Ensuring that services are deployed only after the secret manager is fully initialized can be a challenge when using a GitOps approach. In my setup, I use FluxCD, which offers a `dependsOn` option. However, this option only ensures that services are deployed after their dependencies are reconciled, meaning Flux has applied the configuration to the cluster and the resource exists in the desired state. It does not guarantee that the resource is fully operational or initialized—specifically, that the secret manager is ready for use.

Even when running the secret manager in a separate cluster to allow this cluster to start up first, maintaining the availability of a self-hosted secret manager remains complex. While a failure of the secret manager itself can be managed, the risk arises when other services depend on it and cannot restart due to the lack of access to the secret provider. This creates a single point of failure that could potentially disrupt the entire cluster.

For these reasons, I decided to transition to a cloud-based secret provider. This approach decouples secret management from the cluster, simplifying maintenance and enhancing the overall reliability and resilience of the system.