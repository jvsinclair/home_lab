# Home Lab -Turing Pi 2 Cluster Setup with K3s and Kubernetes Dashboard

This repository contains Ansible playbooks to configure a Turing Pi 2 cluster with three compute modules running Ubuntu Server 24.04. The setup deploys SSH keys, updates the nodes, sets static IPs, installs K3s (a lightweight Kubernetes distribution), and deploys the Kubernetes Dashboard for cluster management.

## Prerequisites

- **Control Machine**: An Ubuntu computer on the same network (e.g., 10.42.0.0/24) as the Turing Pi 2 cluster.
- **Nodes**:
  - Node 1: Turing RK1 (16GB) - Initial DHCP IP: 10.42.0.145
  - Node 2: Turing RK1 (16GB) - Initial DHCP IP: 10.42.0.138
  - Node 3: Raspberry Pi CM410800 - Initial DHCP IP: 10.42.0.60
  - All nodes run Ubuntu Server 24.04 with login `ubuntu` and password `ubuntu`.
- **SSH Keys**: Generate an SSH key pair on the control machine (`ssh-keygen -t rsa -b 4096`) if not already present.

## Setup Instructions

1. **Clone the Repository**
   ```bash
   git clone <repository-url>
   cd turing-pi-cluster

2. **Install Dependencies on the Control Machine**
   ```bash
   sudo apt update
   sudo apt install git ansible
   ansible-galaxy collection install kubernetes.core
   ```

3. **Run Initial Setup**
   This playbook deploys SSH keys, updates packages, sets static IPs (10.42.0.11–10.42.0.13), and reboots the nodes:
   ```bash
   ansible-playbook -i inventory/dhcp.ini playbooks/initial_setup.yml
   ```

4. **Run K3s and Dashboard Setup**
   This playbook installs K3s, configures the cluster (Node 1 as master, Nodes 2 and 3 as workers), installs Helm, deploys the Kubernetes Dashboard, and provides access details:
   ```bash
   ansible-playbook -i inventory/static.ini playbooks/k3s_setup.yml
   ```

5. **Access the Kubernetes Dashboard**
   - After the second playbook completes, it outputs a URL (e.g., https://10.42.0.10:30000) and a token.
   - Open the URL in a browser, accept the self-signed certificate warning, and log in with the token.

