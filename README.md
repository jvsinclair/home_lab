# Home Lab - Turing Pi 2 Cluster Setup with K3s, Kubernetes Dashboard and exo (for running LLMs)

This repository contains Ansible playbooks to configure a Turing Pi 2 cluster with three compute modules running Ubuntu Server 24.04. The setup deploys SSH keys, updates the nodes, sets static IPs, installs K3s (a lightweight Kubernetes distribution), and deploys the Kubernetes Dashboard for cluster management.

## Prerequisites

- **Control Machine**: An Ubuntu computer on the same network (e.g., 192.168.10.0/24) as the Turing Pi 2 cluster.
- **Nodes**:
  - Node 1: Turing RK1 (16GB) - Initial DHCP IP: 10.42.0.x
  - Node 2: Turing RK1 (16GB) - Initial DHCP IP: 10.42.0.y
  - Node 3: Raspberry Pi CM410800 - Initial DHCP IP: 10.42.0.z
  - All nodes run Ubuntu Server 24.04 with login `ubuntu` and password prompted. (only used for inital setup)
- **SSH Keys**: Generate an SSH key pair on the control machine (`ssh-keygen -t rsa -b 4096`) if not already present before starting setup. 

## Setup Instructions

1. **Clone the Repository**
   ```bash
   git clone https://github.com/jvsinclair/home_lab.git
   cd home_lab

2. **Install Dependencies on the Control Machine (I call mine Proxy since its a jump server for the cluster and has 2 nics)**
   ```bash
   sudo apt update
   sudo apt install git ansible sshpass curl 
   ansible-galaxy collection install kubernetes.core
   ```

3. **Run Initial Setup**
   This playbook configures the nodes with SSH keys, updates packages, and sets static IPs. When running it, you will be prompted to enter the password for the nodes:
   ```bash
   ansible-playbook -i inventory/dhcp.ini playbooks/initial_setup.yml
   ```
   NOTE: if you already gave your devices statsu ips then just use those for `ansible_host=` in the `dchp.ini` inventory.

4. **Run K3s and Dashboard Setup**
   This playbook installs K3s, configures the cluster (Node 1 as master, Nodes 2 and 3 as workers), installs Helm, deploys the Kubernetes Dashboard, and provides access details:
   ```bash
   ansible-playbook -i inventory/static.ini playbooks/k3s_setup.yml
   ```

5. **Access the Kubernetes Dashboard**
   - After the second playbook completes, it outputs a URL (e.g., https://192.168.10.11:30000) and a token.
   - Open the URL in a browser, accept the self-signed certificate warning, and log in with the token.

