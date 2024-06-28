# ONLYOFFICE Document Server Automation Setup

This repository provides the necessary configuration files and instructions to automate the setup and deployment of ONLYOFFICE Document Server using Docker and Docker Compose. The automation process is enhanced with Ansible playbooks for a seamless deployment.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
  - [Clone the Repository](#clone-the-repository)
  - [Configure Docker and Docker Compose](#configure-docker-and-docker-compose)
  - [Build and Deploy ONLYOFFICE Document Server](#build-and-deploy-onlyoffice-document-server)
- [Configuration Files](#configuration-files)
  - [Dockerfile](#dockerfile)
  - [docker-compose.yml](#docker-composeyml)
  - [Ansible Playbooks](#ansible-playbooks)
- [Volumes](#volumes)
- [Ports](#ports)

## Overview

The ONLYOFFICE Document Server provides powerful online document editing capabilities. This setup leverages Docker and Docker Compose for containerized deployment, ensuring a consistent and easily reproducible environment. Additionally, Ansible playbooks are provided to further automate the setup and deployment process.

## Prerequisites

- Alt Linux installed
- Docker Engine
- Docker Compose
- Ansible

## Setup Instructions

### Clone the Repository

Clone the official ONLYOFFICE Document Server repository:

```bash
git clone https://github.com/ONLYOFFICE/Docker-DocumentServer.git /etc/Docker-DocumentServer
cd /etc/Docker-DocumentServer
git checkout v7.3.3.60
