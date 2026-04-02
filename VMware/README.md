# 🚀 Ansible Execution Environment (VMware Automation)

## 📌 What is an Ansible Execution Environment (EE)?

An Ansible Execution Environment (EE) is a container image that includes:

- **Ansible Core** – The automation engine
- **Ansible Runner** – Execution framework
- **Required Python libraries** – Dependencies for modules
- **Ansible collections** (e.g., VMware modules) – Pre-built automation tasks
- **System dependencies** – OS-level packages

> 👉 It provides a **consistent, isolated runtime** for running Ansible automation.

---

## 🎯 Why do we need Execution Environments?

### Traditional Ansible Challenges ❌
- Dependencies are installed on control nodes manually
- Version conflicts happen frequently
- **Problem:** When automating multiple platforms (Cisco, VMware, Kubernetes), each requires its own dependencies
- Different environments = different behavior
- Difficult to reproduce issues

### Execution Environment Solutions ✅
- Everything is containerized
- Fully portable across environments
- Fully reproducible builds
- Works seamlessly with **AWX** and **Red Hat Ansible Automation Platform (AAP)**

**Example:** For VMware automation, you create an execution environment (a container) that runs your playbook, then exits. The same approach applies to Cisco, Kubernetes, and other platforms.

---

## 🏗️ Value for Platform Engineering

Execution Environments enable:

### ✅ Standardization
Same environment across all stages:
- Development (Dev)
- User Acceptance Testing (UAT)
- Production

### ✅ Self-Service Automation
Teams can run automation safely without breaking dependencies.

### ✅ Scalability (Kubernetes / AWX)
AWX dynamically spins pods using EE images:

```
AWX → Kubernetes Pod → Pull EE Image → Run Playbook
```

### ✅ Version Control
Each EE version = immutable automation runtime

---

## 📂 Project Structure

```
VMware/
├── execution-environment.yml      # Main EE blueprint
├── requirements.txt               # Python dependencies
├── required-collections.yml       # Ansible collections
├── bindep.txt                    # System-level packages
└── context/                      # Auto-generated during build
```

---

## 📄 File Breakdown & Explanation

### 1️⃣ execution-environment.yml

#### 📌 Purpose
This is the **main blueprint** that defines how the EE image is built.

#### File Content
```yaml
version: 3

dependencies:
  ansible_core:
    package_pip: ansible-core
  ansible_runner:
    package_pip: ansible-runner
  python: requirements.txt
  galaxy: required-collections.yml
  system: bindep.txt

images:
  base_image:
    name: quay.io/centos/centos:stream9
```

#### 🔑 Explanation of Parameters

**`version: 3`**
- Uses the modern Ansible Builder v3 schema
- Required for structured dependency handling

**`images.base_image.name: quay.io/centos/centos:stream9`**
- Base Operating System for your container
- Clean, minimal OS
- You control everything installed on top

**`dependencies.ansible_core` → `package_pip: ansible-core`**
- Installs Ansible itself
- **Why:** Required to run playbooks
- Builder ensures it's properly installed in the container

**`dependencies.ansible_runner` → `package_pip: ansible-runner`**
- Used by AWX to execute playbooks
- **Why:** AWX does NOT call Ansible directly; it uses `ansible-runner` internally

**`dependencies.python` → `python: requirements.txt`**
- Installs Python libraries specified in `requirements.txt`
- Example: pyvmomi (VMware SDK), requests (HTTP library)

**`dependencies.galaxy` → `galaxy: required-collections.yml`**
- Installs Ansible collections from Galaxy
- Example: community.vmware (VMware modules), ansible.posix (Linux modules)

**`dependencies.system` → `system: bindep.txt`**
- Installs OS-level packages via dnf/yum
- Example: gcc, git, sshpass, OpenSSL libraries

---

### 2️⃣ requirements.txt

#### File Content
```
pyvmomi
requests
```

#### 📌 Purpose
Python dependencies required by your automation.

#### 🔑 Explanation

**`pyvmomi`**
- VMware API SDK for vSphere automation
- **Used for:** VM creation, snapshots, power operations, inventory management

**`requests`**
- HTTP communication library
- **Used by:** Many Ansible modules for API calls

---

### 3️⃣ required-collections.yml

#### File Content
```yaml
collections:
  - name: community.vmware
  - name: ansible.posix
```

#### 📌 Purpose
Defines Ansible collections to install from Ansible Galaxy.

#### 🔑 Explanation

**`community.vmware`**
- Provides VMware-specific Ansible modules
- **Includes:** VM creation, snapshot management, power operations, network configuration

**`ansible.posix`**
- Provides Linux/Unix system modules
- **Includes:** File operations, permissions, system tasks, service management

---

### 4️⃣ bindep.txt

#### File Content
```
gcc [platform:rpm]
python3-devel [platform:rpm]
openssl-devel [platform:rpm]
libffi-devel [platform:rpm]
git [platform:rpm]
sshpass [platform:rpm]
```

#### 📌 Purpose
Installs system-level packages needed during build and runtime.

#### 🔑 Explanation

**`gcc`**
- GNU Compiler Collection
- **Why:** Required to compile Python packages from source

**`python3-devel`**
- Python development headers and libraries
- **Why:** Required for building Python extension modules

**`openssl-devel`**
- OpenSSL development libraries
- **Why:** Required for SSL/TLS connections (used by VMware API communications)

**`libffi-devel`**
- Foreign Function Interface library
- **Why:** Needed by cryptography libraries for secure communications

**`git`**
- Version control system
- **Why:** Required if pulling roles or collections directly from Git repositories

**`sshpass`**
- SSH password authentication tool
- **Why:** Used for password-based SSH automation

---

### 5️⃣ context/ (Auto-generated)

#### 📌 Purpose

Generated automatically by:
```bash
ansible-builder build
```

#### Contains:
- `Containerfile` (Dockerfile equivalent for container build)
- Build scripts and configurations
- Processed dependencies

> ⚠️ **Important:** You do NOT edit this folder manually. It's regenerated on every build.

---

## ⚙️ Build Process

### How to Build the Execution Environment

```bash
ansible-builder build -t ee-vmware:v1
```

### What Happens Internally:
1. Create build context from configuration files
2. Pull base image (CentOS Stream 9)
3. Install system packages from `bindep.txt`
4. Install Python dependencies from `requirements.txt`
5. Install Ansible Core and Runner
6. Install Ansible collections from `required-collections.yml`
7. Build final container image and tag it as `ee-vmware:v1`

---

## 🧠 How AWX Uses This

### Execution Flow:

1. AWX job is triggered
2. AWX creates a Kubernetes Pod
3. Pod pulls the EE image from registry
4. `ansible-runner` executes your playbook inside the container
5. Playbook runs in isolated environment
6. Results returned to AWX
7. Pod terminates

**Benefit:** Each job gets a fresh, clean environment with no state or dependency issues.

---

## 🚀 Benefits Summary

| Feature | Benefit |
|---------|---------|
| 🐳 **Containerized EE** | No dependency conflicts between projects |
| 📦 **Versioned Images** | Reproducible automation across teams |
| 🔗 **AWX Integration** | Scalable execution with Kubernetes |
| 🔒 **Isolated Runtime** | Secure execution without side effects |
| 🌍 **Portable** | Works on any system with container runtime |
| 📋 **Version Control** | Track automation runtime versions like code |

---

## 🏁 Conclusion

This Execution Environment provides:

✨ **A standardized automation runtime** – Same experience everywhere  
✨ **VMware-ready automation stack** – Pre-configured for vSphere automation  
✨ **Full compatibility with AWX / AAP** – Enterprise-grade execution  
✨ **A foundation for platform engineering automation** – Scalable, reproducible, secure  

**Start building your VMware automation on a solid, containerized foundation!**