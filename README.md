# 🏡 Homelab

## Introduction

This repository contains the configuration and documentation of my homelab environment.

The primary goal of this homelab is both educational and recreational. Lately, I’ve been exploring Kubernetes, and there’s no better way to learn than by running a cluster at home. Additionally, self-hosting applications allows me to take full ownership of the deployment and maintenance process from start to finish.

## ⚙️ Operation System

<img src="https://www.talos.dev/img/sidero-logo.svg" width="100">

I decided to use [Talos Linux](https://github.com/siderolabs/talos) to set up my machines.
Talos is a Linux distribution purpose-built for running Kubernetes.
It’s lightweight, efficient, and relatively easy to set up for home lab environments making it ideal for my needs.

## 🖥️ Hardware

I use a refurbished second-hand mini pc. They're great because they are small and cheap to buy.

- HP ProDesk 400 G4 i5-8500T/8GB/256GB M.2
- Lenovo ThinkCentre M70q i5-10500T/24GB/256GB M.2
- Lenovo ThinkCentre M70q i5-10500T/24GB/120GB SSD

## 📁 Project Structure

This project is organized into two primary domains:

- Applications — End-user applications.
- Databases — Database and state management resources.

Each domain has its own controllers and configurations, all organized under `infrastructure` folder.

```bash
├── apps
├── databases
├── clusters # FluxCD bootstrap manifest
├── infrastructure # Infrastructure for `apps` and `databases`
│   ├── configs # Depends on infrastructure controllers
│   └── controllers
└── monitoring # Monitoring tools for `apps` and `databases`
    ├── configs # Depends on monitoring controllers
    └── controllers
```

**Future Roadmap:**
The plan is to decouple the data layer from the application, 
which will involve establishing two separate clusters,
one dedicated to data and the other to application services for improved scalability.

The diagram below outlines the planned changes to the folder structure:

```bash
├── apps
├── databases
├── clusters
│   ├── data # Reconciles changes for `databases` and `infrastructure/**/data`
│   └── production # Reconciles changes for `apps` and `infrastructure/**/production`
├── infrastructure
│   ├── configs
│   │   ├── base # Base configuration
│   │   ├── data # Config for `databases` infrastructure
│   │   └── production # Config for `apps` infrastructure
│   └── controllers
│       ├── base # Base controller manifest
│       ├── data # Controller for `databases` infrastructure
│       └── production # Controller for `apps` infrastructure
└── monitoring
    ├── configs
    │   ├── base # Base configuration
    │   ├── data # Config for `databases` monitor tools
    │   └── production # Config for `apps` monitor tools
    └── controllers
        ├── base # Base controller manifest
        ├── data # Controller for `databases` monitor tools
        └── production # Controller for `apps` monitor tools
```

## 🔐 Secret Management

<div style="display: flex; gap: 10px">
    <img src="https://www.isjw.uk/images/azure/keyvault.png" height="50"/>
    <img src="https://external-secrets.io/latest/pictures/eso-round-logo.svg" height="50">
</div>

I decided to use [Azure Key Vault](https://azure.microsoft.com/en-us/products/key-vault) and inject them into cluster with [External Secrets Operator](https://external-secrets.io/).

**Why use a cloud secret provider instead of self-hosted?**
- This approach improves reliability, resilience, and simplifies maintenance
- In a GitOps setup (using FluxCD), `dependsOn` only ensures resources are applied, not that they are fully initialized or ready to use
- Secret managers may exist in the cluster but still be unavailable when dependent services start
- A self-hosted secret manager can become a single point of failure
- If the secret manager is unavailable, dependent services may fail to start or restart
- Cloud secret providers decouple secret management from the cluster

## 🌐 Service Exposing

### Publicly

<div style="display: flex; gap: 10px; align-items: center">
    <img src="https://cdn.jsdelivr.net/gh/walkxcode/dashboard-icons/png/cloudflare.png" height="30"/>
    <img src="https://cdn.jsdelivr.net/gh/walkxcode/dashboard-icons/png/cloudflare-zero-trust.png" height="50"/>
</div>

I use a [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) integrated with [Cloudflare Zero Trust](https://developers.cloudflare.com/cloudflare-one/) to expose services, instead of the more traditional Ingress + VPN setup. 

**Why Cloudflare Tunnel?**
- I want to keep things simple and secure
- No need for public IP addresses, firewall rule, or complex ingress config
- Cluster is never directly exposed to the internet
- Only authorized users can access internal services
- Lightweight solution with minimal operational overhead

### Locally

<div style="display: flex; gap: 10px; align-items: center">
    <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/cilium.svg" height="50"/>
    <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/cert-manager.svg" height="50"/>
    <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/lets-encrypt.svg" height="50"/>
    <img src="https://raw.githubusercontent.com/kubernetes-sigs/external-dns/refs/heads/master/docs/img/external-dns.png" height="50"/>
</div>

- Use [Cilium](https://docs.cilium.io/en/stable/network/servicemesh/ingress/) as the Kubernetes CNI
- Use [cert-manager](https://cert-manager.io/docs/) for TLS management
- Integrate cert-manager with [Let’s Encrypt](https://letsencrypt.org/) for automated certificate provisioning
- Use [ExternalDNS](https://kubernetes-sigs.github.io/external-dns/) to propagate domain records to local IP

## 💾 Backup

<div style="display: flex; gap: 10px; align-items: center">
    <img src="https://cdn.jsdelivr.net/gh/walkxcode/dashboard-icons/png/cloudflare.png" height="30"/>
    <img src="https://cloudnative-pg.io/images/hero_image.svg" height="50"/>
</div>

- I use [CloudNativePG (CNPG)](https://cloudnative-pg.io/), a Kubernetes-native operator for managing PostgreSQL clusters.
- It includes native support for backup and restore operations via object storage.
- Backups are stored in [Cloudflare R2](https://www.cloudflare.com/developer-platform/products/r2/).


## 🔭 Monitoring

I use the [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack), a comprehensive collection of manifests for deploying and managing Grafana and Prometheus.
