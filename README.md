# homelab-documents
This repository contains all of my documentation for setting up my homelab from start to finish and serves as a place to track my progress, share wins, and store reference information.

## Table of Contents
- [Lab Goals](#lab-goals)
- [Hardware Used](#hardware-used)
- [Operating System Installation](#operating-system-installation)

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

When ready, I plugged the flash drive into the target machine and opened the boot menu. I selected the flash drive as the primary boot device, rebooted, and followed the OS installer's prompts. The Arch Linux setup is a similar process for the Ubuntu, I took a slightly different route by downloading Proxmox onto my Mini Pc:
- https://www.proxmox.com/en/downloads

Proxmox will allow me to practice building Arch Linux from the ground up.
