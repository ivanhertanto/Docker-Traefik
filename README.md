# Traefik 2.0 + Docker — a Simple Step by Step Guide

## Introduction

In this tutorial we will go trough the following things:
1. Setup and configure Traefik in a Docker container
2. Let’s Encrypt setup for automatic HTTPS certificates
3. Deploy a simple service (Portainer) and expose it to the internet

## Prerequisites

In order to follow along, you need these things:

- Docker
- Docker Compose
- A domain
- Ports 80 and 443 forwarded to your Docker host

## 1. Setup and configure Traefik with Let’s Encrypt

Let’s get started by setting up Traefik.

First, create a directory for our containers:

```
mkdir -p /opt/containers/{traefik,portainer}
```

Create the data folder and config files for Traefik:

```
touch data/acme.json
chmod 600 data/acme.json
```

The acme.json file is the storage file for the HTTPS certificates.

Now we create the basic Traefik configuration file:
