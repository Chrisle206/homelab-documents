# homelab-documents
This repository contains all of my documentation for setting up my homelab from start to finish and serves as a place to track my progress, share wins, and store reference information.

## Table of Contents
- [Lab Goals](#lab-goals)
- [Hardware Used](#hardware-used)
- [Operating System Installation](#operating-system-installation)
- [Network Topology & IP Scheme](#network-topology--ip-scheme)
- [Software Stack & Services](#software-stack--services)
- [Ansible / Automation Setup](#ansible--automation-setup)
- [Maintenance & Backup Plan](#maintenance--backup-plan)
- [Security Considerations](#security-considerations)
- [Learning Roadmap / Milestones](#learning-roadmap--milestones)
- [Troubleshooting / FAQ](#troubleshooting--faq)

## Lab goals:
I want my homelab to be a space where I can simulate real-world scenarios similar to what I'd encounter in a DevOps/SRE role. A homelab lets me freely make mistakes and learn from them without jeopardizing production networks or data. It's a hands-on way to gain experience at home. With this setup, I can practice break-fix troubleshooting, build practical skills, and document my journey.

## Hardware Used:
Before building a physical lab, I started with Oracle VirtualBox, which lets you create virtual machines(VMs) inside an existing operating system (the host). VirtualBox simulates hardware so you can install and run multiple operating systems isolated from the host, while allocating CPU, memory, and disk resources. 

I later moved away from VirtualBox because it consumed too many resources on my personal PC. Instead, I looked for old desktops, laptops, and mini PCs. I searched for anything that was available on Facebook Marketplace at a good price. I eventually found HP 8000 slim desktops, Ethernet cables, and a power strip, and began assembling the lab.

When starting out with a homelab you can use any older equipment around the house that has decent specifications (example image below):

<img width="379" height="229" alt="vm-req" src="https://github.com/user-attachments/assets/5b31e4f6-c8b0-4593-91a3-47f4db855007" />

Next, I connected all of the machines to an Ethernet switch, which provides additional network ports and routes them to my HP slim desktops for a reliable wired connection.

## Operating System Installation
I followed these installation guides for Ubuntu Server LTS and Arch Linux:

- https://wiki.archlinux.org/title/Installation_guide
- https://ubuntu.com/download/server

After downloading the ISO files, I used Rufus, a tool that writes an operating system to a USB flash drive. This process wipes the USB partition and stores the ISO image for booting:
- https://rufus.ie/en/

When ready, I plugged the flash drive into the target machine and opened the boot menu. I selected the flash drive as the primary boot device, rebooted, and followed the OS installer's prompts. The Arch Linux setup is similar to Ubuntu's, but I chose to run it inside Proxmox for easier VM management and snapshots:
- https://www.proxmox.com/en/downloads

Proxmox will allow me to practice building Arch Linux from the ground up.

## Network Topology & IP Scheme
This diagram shows my homelab setup:

<img width="803" height="644" alt="image" src="https://github.com/user-attachments/assets/673dcb1a-9aac-4ed0-8993-14528f51b768" />

Typically when you have devices connected to your home network they are automatically assigned IP addresses. These can change over time possibly due to the router restarting, device reconnects after a while, or if the DHCP lease expires and gets renewed with a different address. I learned this when I attempted to reconnect to my node after a power outage and the IP associated to the machine had changed. This is due to an internet protocol called DHCP (Dynamic Host Configuration Protocol). To set a static IP for a device you can do it in a file within /etc/netplan/. There should be a pre-configured .yml file where you can set your desired IP for the machine.

<img width="368" height="267" alt="image" src="https://github.com/user-attachments/assets/70b94c17-1663-44f2-8228-c6c73d74c75b" />

## Software Stack & Services
List of key tools to run and learn with:
- Proxmox (hypervisor & VM management)
- Kubernetes / Docker / containerd
- CI/CD tools (Jenkins, GitHub Actions runners, etc.)
- Monitoring (Prometheus, Grafana, Loki)
- Automation (Ansible, Terraform)

## Anisble / Automation Setup
Have multiple nodes in your homelab? Trying to find a way to update/install many of the same packages, but don't want to do them for each node manually? Ansible is the tool that I used to solve this issue. Ansible allows me to update or manage multiple nodes at once and is Agentless which means no need to install anything on the remote nodes. It just requires SSH to connect and execute tasks. What makes it even better is that it is Idempotent so it only applies changes if needed so re-running the same playbook won't break things. For your homelab setup you would just need to add to the inventory file found in the same folder as the Ansible package. This is what it should look like:

<img width="384" height="131" alt="image" src="https://github.com/user-attachments/assets/4e955549-e050-4d54-a5a8-d0681cb1282a" />

"homelab" is the group name, ansible host is the IP of the node, and user is the login name used for that node. These are important because it will let Ansible know which group you would like to apply the packages to. Another essential thing that must be configured is the SSH connection between the main node and the side nodes. This will defeat the hassle of having to enter the password for each node each time the playbook is ran. The procedure to get this done is as follows: 
Generate SSH keys → Create inventory → Test ping → Write playbook → Run.

ssh-keygen -t ed25519 # Create SSH to connect the nodes.
ssh-copy-id youruser@hp-node1
ssh-copy-id youruser@hp-node2
ssh-copy-id youruser@hp-node3

ansible -i inventory homelab -m ping # Check to see if the connection was successful.
hp-node1 | SUCCESS => {"ping": "pong"}

hosts: homelab # Playbook example
  become: yes
  tasks:
    - name: Update apt cache and install common packages
      apt:
        name:
          - vim
          - curl
          - docker.io
        state: present
        update_cache: yes

## Maintenance & Backup Plan

System Updates:
Use apt upgrade or unattended-upgrades for security patches on Ubuntu nodes.

For Arch Linux, run pacman -Syu or schedule a weekly Ansible job.

Snapshots & Backups:
Proxmox: Create scheduled VM snapshots before major changes.

Store VM configs and important volumes on an external drive or NAS.

Use rsync or restic to push critical configs to another node or cloud bucket.

Configuration Backup:
Keep /etc/ configs, Ansible playbooks, and Kubernetes manifests in Git.

Regularly back up your Ansible inventory and playbooks to GitHub.

## Security Considerations
User Accounts & Privileges
Create non-root admin users and give them passwordless sudo only where necessary.

Use SSH keys instead of passwords; disable root SSH login once key auth is verified.

Network Isolation / VLANs
Separate management traffic from guest workloads with VLANs if your switch supports it.

Optionally put Proxmox management on a dedicated subnet.

Secrets Management
Use .env files or Ansible Vault to store API keys, database passwords, etc.

Never commit secrets to Git. Add .gitignore rules for .env and private keys.

## Learning Roadmap / Milestones
Phase 1 – Base Infrastructure

Install Ubuntu Server LTS and Arch Linux VMs.

Configure static IPs, SSH keys, and Proxmox storage.

Phase 2 – Automation & Config Management

Write Ansible playbooks to manage updates and install common packages.

Experiment with Terraform to provision VMs in Proxmox.

Phase 3 – Cluster & Orchestration

Stand up a multi-node Kubernetes cluster (K3s or kubeadm).

Deploy sample workloads, practice scaling and rolling updates.

Phase 4 – CI/CD & Monitoring

Set up Jenkins or GitHub Actions self-hosted runners.

Deploy Prometheus + Grafana for metrics; integrate alerting (Alertmanager, Slack/Email).

Add centralized logging (ELK or Loki).

Phase 5 – Advanced Networking & Security (optional stretch)

Implement VLANs, reverse proxy with Traefik or NGINX.

Experiment with service meshes or zero-trust networking.


## Troubleshooting / FAQ
