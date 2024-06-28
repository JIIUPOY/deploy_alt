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
```

### Configure Docker and Docker Compose

```bash
sudo apt-get update
sudo apt-get install -y docker-engine docker-compose-v2
sudo systemctl enable --now docker
```

### Build and Deploy ONLYOFFICE Document Server
Before starting to work with docker, after installing the deploy package in /etc/ansible/hosts, you need to write the following lines, where all:vars is the user executing the commands, local is the IP address that will be managed.
```bash
[all:vars]
ansible_user=root
[local]
10.0.2.15
```
**Build the Docker Image**:
Navigate to the repository directory and build the Docker image:
```bash
docker-compose build
```

**Deploy ONLYOFFICE Document Server**:
Start the services using Docker Compose:
```bash
docker compose up -d
```

**Verify the Deployment**:
Check the running containers:
```bash
docker ps
```
Ensure ONLYOFFICE Document Server is accessible at the configured ports.

## Configuration Files
### Dockerfile

The Dockerfile sets up the ONLYOFFICE Document Server with all required dependencies and configurations. Key sections include:
- Installation of necessary packages and fonts.
- Configuration of PostgreSQL, Redis, and RabbitMQ.
- Download and installation of ONLYOFFICE Document Server.
- Exposing necessary ports for HTTP and HTTPS.


### docker-compose.yml

The docker-compose.yml file defines the multi-container setup for ONLYOFFICE Document Server, including services for PostgreSQL and RabbitMQ. Key configurations include:
- Environment variables for database connection.
- Volume mappings for persistent data storage.
- Port mappings for external access.

### Ansible Playbooks

Two Ansible playbooks (deploy_onlyoffice_1.yml and deploy_onlyoffice_2.yml) automate the setup and deployment process:

**deploy_onlyoffice_1.yml**

This playbook installs necessary packages, clones the OnlyOffice repository, configures Docker, and sets up JWT and network settings.

```bash
- name: Deploy Onlyoffice. Pulling configurations
  hosts: local
  remote_user: root
  tasks:
    - name: Install Git
      apt_rpm:
        name: git
        state: present
    - name: Clone official Onlyoffice repository
      git:
        repo: 'https://github.com/ONLYOFFICE/Docker-DocumentServer.git'
        dest: /etc/Docker-DocumentServer
        version: 'v7.3.3.60'
    - name: Install Docker Compose
      apt_rpm:
        name: docker-compose-v2
        state: present
    - name: Install Docker Engine
      apt_rpm:
        name: docker-engine
        state: present
    - name: Enable Docker
      systemd:
        name: docker
        enabled: yes
    - name: Set Docker version
      lineinfile:
        path: /etc/Docker-DocumentServer/Dockerfile
        regexp: '^ARG PACKAGE_VERSION='
        line: 'ARG PACKAGE_VERSION=7.3.3'
        state: present
    - name: Add nameservers
      lineinfile:
        path: /etc/resolv.conf
        line: "{{ item }}"
        state: present
      with_items:
        - 'nameserver 8.8.8.8'
        - 'nameserver 8.8.8.4'
    - name: Uncomment JWT_ENABLED line
      replace:
        path: /etc/Docker-DocumentServer/docker-compose.yml
        regexp: '#- JWT_ENABLED=true'
        replace: '- JWT_ENABLED=true'
    - name: Uncomment JWT_SECRET line
      replace:
        path: /etc/Docker-DocumentServer/docker-compose.yml
        regexp: '#- JWT_SECRET=secret'
        replace: '- JWT_SECRET=secret'
    - name: Change ports for Onlyoffice
      replace:
        path: /etc/Docker-DocumentServer/docker-compose.yml
        regexp: '80:80'
        replace: '8081:80'
    - replace:
        path: /etc/Docker-DocumentServer/docker-compose.yml
        regexp: '443:443'
        replace: '4443:443'
```

**deploy_onlyoffice_2.yml**

This playbook pulls the ONLYOFFICE Document Server container and starts the service.

```bash
- name: Deploy Onlyoffice. Pulling container
  hosts: local
  become: yes
  remote_user: root
  tasks:
    - name: Pulling container
      command: docker compose up -d
      args:
        chdir: /etc/Docker-DocumentServer
    - name: Getting container id
      command: docker ps -q -n 1
      register: docker_cont_id
    - name: Execute supervisorctl to start service
      command: docker exec {{ docker_cont_id.stdout }} sudo supervisorctl start ds:example
    - name: Restarting docker
      command: docker compose restart
      args:
        chdir: /etc/Docker-DocumentServer
    - name: Executing container again
      command: docker exec {{ docker_cont_id.stdout }} supervisorctl start ds:example
```
## Ports

The ONLYOFFICE Document Server is configured to use the following ports:
- 80: HTTP.
- 443: HTTPS.

If an error occurred during the execution of the second playbook, then run the following command manually, where docker_id - id of the running container.
```bash
docker exec <docker_id> supervisorctl start ds:example
```

By following these instructions, you can successfully automate the deployment of ONLYOFFICE Document Server, ensuring a consistent and reproducible setup.
