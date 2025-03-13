# Home Lab - Turing Pi 2 Cluster Setup with K3s, Kubernetes Dashboard and exo (for running LLMs)

This repository contains Ansible playbooks to configure a Turing Pi 2 cluster with three compute modules running Ubuntu Server 24.04. The setup deploys SSH keys, updates the nodes, sets static IPs, installs K3s (a lightweight Kubernetes distribution), and deploys the Kubernetes Dashboard for cluster management.

## Prerequisites

- **Control Machine**: An Ubuntu computer on the same network (e.g., 192.168.10.0/24) as the Turing Pi 2 cluster.

   ***NOTE***: Make sure your network is not starting with 10.42.x.x otherwise you will run into issues becuase that is the default network for Docker / Kubernetes

- **Nodes**:
  - Node 1: Turing RK1 (16GB) - Initial DHCP IP: 192.168.10.x
  - Node 2: Turing RK1 (16GB) - Initial DHCP IP: 192.168.10.y
  - Node 3: Raspberry Pi CM410800 - Initial DHCP IP: 192.168.10.z
  - All nodes run Ubuntu Server 24.04 with login `ubuntu` and password prompted. (only used for inital setup) 
  - - Flashing instructing for Turing RK1: https://docs.turingpi.com/docs/turing-rk1-flashing-os 
  - - Flashing instructing for Raspberry PI CM4: https://docs.turingpi.com/docs/raspberry-pi-cm4-flashing-os
- **SSH Keys**: Generate an SSH key pair on the control machine (`ssh-keygen -t rsa -b 4096`) if not already present before starting setup. 

## Setup Instructions

1. **Clone the Repository**
   ```bash
   git clone https://github.com/jvsinclair/home_lab.git
   cd home_lab

2. **Install Dependencies on the Control Machine (I call mine Proxy since its a jump server for the cluster and has 2 nics)**
   ```bash
   sudo apt update
   sudo apt install git ansible sshpass curl python3-pip python3-kubernetes
   ansible-galaxy collection install kubernetes.core
   ```
   ***NOTE:*** this assumes the Control Machine is Ubuntu. you will need to adjust this for your host if its not ubuntu

3. **Run Initial Setup**
   This playbook configures the nodes and the BMC in the Turing pi with SSH keys, updates packages, and sets static IPs. When running it, you will be prompted to enter the password for the nodes:
   ```bash
   ansible-playbook -i inventory/dhcp.ini playbooks/initial_setup.yml
   ```
   ***NOTE***: If you already gave your devices static IPs then just use those for `ansible_host=` in the `dchp.ini` inventory. Be sure to set your gateway and name servers as well otherwise your devices will be unreachable and you will need to use the Turing PI BMC to access UART on eac node to fix this. Example: 
   ```bash
   ssh root@turingpi.local
   picocom /dev/ttyS2 -b 115200 # To access the first note. for other nodes look here: https://docs.turingpi.com/docs/tpi-uart#using-picocom
   sudo nano /etc/netplan/01-netcfg.yaml # Make sure it matches the templates/netplan.j2. if nano doesnt look right, exit, do a clear and then try again. 
   reboot #You can use other commands to reload the network config but the Rockchip based RK1 works better with reboot 
   #To exit picocom press ctrl-a then ctrl-x
   ```

4. **Run K3s and Dashboard Setup**
   This playbook installs K3s, configures the cluster (Node 1 as master, Nodes 2 and 3 as workers), installs Helm, deploys the Kubernetes Dashboard, and provides access details:
   ```bash
   # DEPRICATED ansible-playbook -i inventory/static.ini playbooks/k3s_setup.yml
   #This sets up the K3s cluster and generates kubeconfig. it needs to run first.
   ansible-playbook -i inventory/static.ini playbooks/k3s_cluster_setup.yml

   #This installs Helm and kubectl on your control machine. Skip if already installed.
   ansible-playbook -i inventory/static.ini playbooks/helm_kubectl_setup.yml

   #This runs the Dashboard setup. at the end it will generate a url and a token to access the dashboard. save this token in a safe place.
   ansible-playbook -i inventory/static.ini playbooks/dashboard_setup.yml


   ```

5. **Access the Kubernetes Dashboard**
   - After the daskboard playbook completes, it outputs a URL (e.g., https://192.168.10.11:30000) and a token.
   - Open the URL in a browser, accept the self-signed certificate warning, and log in with the token.

6. **Exo Setup**
NOTE: I was able to get it to run on the CM4 but running into issues with the Turing RK1. [Tinygrad](https://docs.tinygrad.org/) (which [Exo](https://github.com/exo-explore/exo) is built on) is not recodnising the hardware and setting it to unknown instead of arm. Will update once I figure it out.
UPDATE March 13th 2025: there is a bug in exo / tinygrad that is causing exo to not run on RK1 and its crasing alot on CM4. for testing on set to localhos and run on an 13th gen Intel NUC and it was slow but worked. I looked into the specific issue and it looks liek Clang doenst like these embeded systems. not going to try Exo any more for now

