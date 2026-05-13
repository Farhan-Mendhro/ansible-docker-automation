# 🐳 Ansible Playbook — Docker Install on EC2

> This playbook automates Docker installation, ensures the service is persistent across reboots, and configures user permissions for non-root Docker management.

![Ansible](https://img.shields.io/badge/Ansible-2.x-EE0000?style=flat-square&logo=ansible&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-docker.io-2496ED?style=flat-square&logo=docker&logoColor=white)
![AWS EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=flat-square&logo=amazon-aws&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![WSL2](https://img.shields.io/badge/WSL2-Control_Node-0078D6?style=flat-square&logo=windows&logoColor=white)

---

## 📋 Table of Contents

- [📁 Project Files](#-project-files)
- [🗺️ Architecture](#️-architecture)
- [📜 The Playbook](#-the-playbook)
- [▶️ How to Run](#️-how-to-run)
- [✅ Verify Docker Works](#-verify-docker-works)
- [💡 Concepts Covered](#-concepts-covered)

---

## 📁 Project Files

```
ansible-playbook-docker-install/
├── playbook.yaml   # Ansible playbook — installs and configures Docker
├── inventory.ini         # EC2 managed nodes
└── README.md
```

---

## 🗺️ Architecture

```
┌──────────────────────────────────────────┐
│             CONTROL NODE                  │
│          WSL2 / Ubuntu (Local PC)         │
│        ansible-playbook command           │
└──────────────────┬───────────────────────┘
                   │  SSH — Passwordless Auth
         ┌─────────┴──────────┐
         ▼                    ▼
┌─────────────────┐  ┌─────────────────┐
│   EC2 Server 1  │  │   EC2 Server 2  │
│  Ubuntu 24.04   │  │  Ubuntu 24.04   │
│    t3.micro     │  │    t3.micro     │
│   Docker ✓      │  │   Docker ✓      │
│   Running ✓     │  │   Running ✓     │
│   ubuntu@docker │  │   ubuntu@docker │
└─────────────────┘  └─────────────────┘
```

---

## 📜 The Playbook

```yaml
---

- name: install docker on ec2 instances
  hosts: all
  become: true

  tasks:

    - name: Install Docker
      ansible.builtin.apt:
        name: docker.io
        state: present
        update_cache: yes

    - name: Start Docker
      ansible.builtin.service:
        name: docker
        state: started
        enabled: yes

    - name: Add user to Docker group
      ansible.builtin.user:
        name: ubuntu
        groups: docker
        append: yes
```

### What each task does

| Task | Module | Purpose |
|---|---|---|
| Install Docker | `ansible.builtin.apt` | Installs `docker.io` from Ubuntu repos with a fresh package cache |
| Start Docker | `ansible.builtin.service` | Starts the Docker daemon and enables it to auto-start on reboot |
| Add user to Docker group | `ansible.builtin.user` | Lets the `ubuntu` user run Docker without `sudo` |

---

## ▶️ How to Run

**Step 1 — Check connectivity**
```bash
ansible all -i inventory.ini -m ping
```

Expected:
```
server1 | SUCCESS => { "ping": "pong" }
server2 | SUCCESS => { "ping": "pong" }
```

**Step 2 — Dry run (recommended)**
```bash
ansible-playbook -i inventory.ini docker-install.yaml --check
```

**Step 3 — Run the playbook**
```bash
ansible-playbook -i inventory.ini docker-install.yaml
```

Expected output:
```
PLAY [install docker on ec2 instances] **********************************

TASK [Gathering Facts] **************************************************
ok: [server1]
ok: [server2]

TASK [Install Docker] ***************************************************
changed: [server1]
changed: [server2]

TASK [Start Docker] *****************************************************
changed: [server1]
changed: [server2]

TASK [Add user to Docker group] *****************************************
changed: [server1]
changed: [server2]

PLAY RECAP **************************************************************
server1   : ok=4   changed=3   unreachable=0   failed=0
server2   : ok=4   changed=3   unreachable=0   failed=0
```

---

## ✅ Verify Docker Works

SSH into any EC2 instance and run:

```bash
# Check Docker version
docker --version

# Check Docker service status
sudo systemctl status docker

# Run without sudo (only works after re-login or newgrp docker)
docker run hello-world
```

**Idempotency check** — run the playbook again, nothing should change:
```bash
ansible-playbook -i inventory.ini docker-install.yaml
# All tasks should show "ok" not "changed"
```

---

## 💡 Concepts Covered

| Concept | Description |
|---|---|
| **ansible.builtin.apt** | Fully-qualified module to install packages on Debian/Ubuntu systems |
| **ansible.builtin.service** | Manages system services — start, stop, enable on boot |
| **ansible.builtin.user** | Manages users and group memberships on remote hosts |
| **become: true** | Runs all tasks as `sudo` — required for system-level operations |
| **hosts: all** | Targets every host in the inventory in a single play |
| **enabled: yes** | Makes Docker persist across reboots — service auto-starts |
| **append: yes** | Adds the user to the docker group without removing existing groups |
| **Idempotency** | Running the playbook multiple times always produces the same safe result |
| **Passwordless SSH** | RSA key auth lets Ansible connect without a password prompt |
| **WSL2** | Windows Subsystem for Linux used as the Ansible control node |

---

<div align="center">
  <sub>Built for DevOps learning · Ansible · Docker · AWS EC2 · WSL2</sub>
</div>
