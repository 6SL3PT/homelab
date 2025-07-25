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
├── infrastructure # Infrastructure for `apps` and `databases`
│   ├── configs # Depends on infrastructure controllers
│   └── controllers
└── monitoring # Monitoring tools for `apps` and `databases`
    ├── configs # Depends on monitoring controllers
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

**Why cloud secret provider instead of self-hosted?**

Ensuring that services are deployed only after the secret manager is fully initialized can be a challenge when using a GitOps approach. In my setup, I use FluxCD, which offers a `dependsOn` option. However, this option only ensures that services are deployed after their dependencies are reconciled, meaning Flux has applied the configuration to the cluster and the resource exists in the desired state. It does not guarantee that the resource is fully operational or initialized—specifically, that the secret manager is ready for use.

Even when running the secret manager in a separate cluster to allow this cluster to start up first, maintaining the availability of a self-hosted secret manager remains complex. While a failure of the secret manager itself can be managed, the risk arises when other services depend on it and cannot restart due to the lack of access to the secret provider. This creates a single point of failure that could potentially disrupt the entire cluster.

For these reasons, I decided to transition to a cloud-based secret provider. This approach decouples secret management from the cluster, simplifying maintenance and enhancing the overall reliability and resilience of the system.

## 🌐 Service Exposing

### Publicly

<div style="display: flex; gap: 10px; align-items: center">
    <img src="https://cdn.jsdelivr.net/gh/walkxcode/dashboard-icons/png/cloudflare.png" height="30"/>
    <img src="https://cdn.jsdelivr.net/gh/walkxcode/dashboard-icons/png/cloudflare-zero-trust.png" height="50"/>
</div>

I use a [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) integrated with [Cloudflare Zero Trust](https://developers.cloudflare.com/cloudflare-one/) to expose services, instead of the more traditional Ingress + VPN setup. 

**Why Cloudflare Tunnel?**

I want to keep things simple and secure.

With this approach, there's no need to deal with public IPs, firewall rules, or complex ingress configurations. Cloudflare Tunnel establishes a secure outbound connection to Cloudflare, and Cloudflare Zero Trust provides fine-grained access control, identity-based policies, and logging—ensuring only the right users can reach internal services. This setup allows me to access services safely without exposing my cluster to the internet. It's a lightweight, secure, and hassle-free way to make services accessible when and where I need them.

### Locally

<div style="display: flex; gap: 10px; align-items: center">
    <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/cilium.svg" height="50"/>
    <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/cert-manager.svg" height="50"/>
    <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/lets-encrypt.svg" height="50"/>
    <img src="https://raw.githubusercontent.com/kubernetes-sigs/external-dns/refs/heads/master/docs/img/external-dns.png" height="50"/>
</div>

I use [Cilium](https://docs.cilium.io/en/stable/network/servicemesh/ingress/), a Kubernetes-native CNI that also provides L7 load balancing to expose HTTP services within the local network. It offers efficient L7 traffic handling, making it a great fit for my workloads (mostly HTTP). For HTTPS support, I’ve integrated [cert-manager](https://cert-manager.io/docs/) with [Let’s Encrypt](https://letsencrypt.org/) to automate TLS certificate provisioning. I also use [ExternalDNS](https://kubernetes-sigs.github.io/external-dns/) to propagate domain records to local IP addresses, enabling access to services via custom domain names within the local environment.

## 💾 Backup

<div style="display: flex; gap: 10px; align-items: center">
    <img src="https://cdn.jsdelivr.net/gh/walkxcode/dashboard-icons/png/cloudflare.png" height="30"/>
    <img src="https://cloudnative-pg.io/images/hero_image.svg" height="50"/>
</div>

I use [CloudNativePG (CNPG)](https://cloudnative-pg.io/), a Kubernetes-native operator for managing PostgreSQL clusters. It includes native support for backup and restore operations via object storage. For this setup, I’ve configured backups to be stored in [Cloudflare R2](https://www.cloudflare.com/developer-platform/products/r2/).


## 🔭 Monitoring

I use the [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack), a comprehensive collection of manifests for deploying and managing Grafana and Prometheus.