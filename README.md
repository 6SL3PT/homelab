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
│   ├── data # Reconciles changes for `databases` and `infrastructure/**/data`
│   └── production # Reconciles changes for `apps` and `infrastructure/**/production`
└── infrastructure # Infrastructure for `apps` and `databases`
    ├── configs # Depends on controllers
    │   ├── base # Base configuration
    │   ├── data # Config for `databases` infrastructure
    │   └── production # Config for `apps` infrastructure
    └── controllers
        ├── base # Base controller manifest
        ├── data # Controller for `databases` infrastructure
        └── production # Controller for `apps` infrastructure
```
