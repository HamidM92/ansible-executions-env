# Ansible Execution Environment for VMware Automation

## Table of Contents
1. [Introduction](#introduction)
2. [Configuration Files](#configuration-files)
   - [ansible.cfg](#ansiblecfg)
   - [requirements.yml](#requirementsyml)
   - [Dockerfile](#dockerfile)
3. [Build Process](#build-process)
4. [AWX Integration](#awx-integration)
5. [Benefits](#benefits)

## Introduction
The Ansible Execution Environment (EE) for VMware is a specialized environment designed to automate processes in VMware infrastructure using Ansible. This environment allows for the seamless execution of playbooks and roles to manage virtual machines, networks, and storage resources effectively.

## Configuration Files
Configuration files define the behavior and capabilities of your Ansible Execution Environment. Here are some of the key files you will encounter:

### ansible.cfg
```ini
[defaults]
inventory = ./inventory
remote_user = ansible
```
- **Purpose**: Sets default configurations such as inventory location and remote user for Ansible operations.

### requirements.yml
```yaml
---
roles:
  - name: geerlingguy.vmware
    src: geerlingguy/vmware
    version: latest
```
- **Purpose**: Lists the Ansible roles required for executing specific tasks related to VMware environments.

### Dockerfile
```Dockerfile
FROM ansible/ansible-runner:latest
COPY . /app
WORKDIR /app
RUN ansible-galaxy install -r requirements.yml
```
- **Purpose**: Defines the base Docker image and the build process to ensure the environment has all necessary dependencies installed.

## Build Process
1. Clone the repository: `git clone https://github.com/HamidM92/ansible-executions-env.git`
2. Navigate to the directory: `cd ansible-executions-env`
3. Build the Docker image: `docker build -t ansible-vmware-ee .`
4. Run a container from the image: `docker run --rm -it ansible-vmware-ee`

## AWX Integration
To integrate with AWX (the open-source version of Ansible Tower):
1. Ensure AWX is properly installed and configured.
2. In AWX, create a new project pointing to your repository.
3. Set up a new template that utilizes the Ansible Execution Environment.
4. Execute tasks and view results directly from the AWX dashboard.

## Benefits
- **Efficiency**: Automates repetitive tasks, increasing productivity.
- **Consistency**: Ensures all deployments are consistent and follow set configurations.
- **Scalability**: Easily manage multiple virtual machines across various environments.
- **Integration**: Seamlessly integrates with existing tools like AWX for streamlined operations.

---
This README outlines the primary components of the Ansible Execution Environment for VMware automation. For further details, please refer to the official [Ansible Documentation](https://docs.ansible.com).
