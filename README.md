<div align="center">
  <h1>Infrastructure Insight</h1>
  <p><strong>Chapter 2 of 8 - A scrub nurse running on cloud. A foundational DevOps journey into Infrastructure as Code, Linux Administration, and Server Security.</strong></p>
</div>

## Project Overview
This chapter takes the hardened VM infrastructure from Chapter 1
and puts it to work. We containerise a real application, distribute
it across two web servers, and sit a load balancer in front of it.

This project serves as a diagnostic tool designed to test and monitor a secure, multi-node virtual infrastructure. Taking the form of a "Medical Bedside Vitals Monitor", the application aggregates live hardware metrics from underlying Linux systems and displays them via a containerized frontend-backend architecture.

The environment mimics an isolated hospital triage web-application, showcasing container orchestration, load balancing, firewall hardening, and real-time Linux `/proc` filesystem data extraction. 

## Objectives and Scope
The goal of this project is to successfully deploy a distributed application across an isolated internal network (`192.168.56.x`). The scope includes:
- **Backend Application:** A Golang service querying root Linux system files for CPU, Memory, and Uptime metrics.
- **Frontend Application:** A Node.js web server serving a responsive dashboard themed as an ICU monitor.
- **Load Balancing:** An Nginx instance distributing incoming medical staff requests smoothly across multiple web node instances.
- **Security:** UFW-enforced firewalls prioritizing inter-cluster traffic, ensuring SSH keys, and utilizing Unattended-Upgrades.

## Setup and Installation Instructions
*Note: This application is designed to be hosted on Virtual Machines deployed by Vagrant.*

1. **Provision Infrastructure:**
Ensure your 4-node cluster (`loadbalancer`, `webserver1`, `webserver2`, `appserver`) is live via Vagrant.
2. **Install Orchestrator Tooling:**
Ensure Docker engine and Nginx are installed across the respective nodes utilizing the provided Ansible playbook execution.
3. **Deploy Backend (App Server):**
   ```bash
   ssh devops@192.168.56.14
   docker build -t vitals-backend ./backend
   docker run -d -p 8080:8080 vitals-backend
   ```
4. **Deploy Frontend (Web Servers):**
   ```bash
   # On webserver1 & webserver2
   docker build -t vitals-frontend ./frontend
   docker run -d -p 3000:3000 -e BACKEND_URL=http://192.168.56.14:8080 vitals-frontend
   ```

## Usage Guide
Once deployed, users (medical triage) should interact strictly with the load balancer node mapping.
Navigate to your load balancer's designated IP:
`http://192.168.56.11`

The Nginx load balancer will seamlessly catch the HTTP request and route it downstream to either `webserver1` or `webserver2`. The resulting webpage will display a live, pulsing Medical Dashboard, showcasing exactly which underlying node processed your query via the "Web Server" badge.

## Bonus Functionality Implemented
- **Enhanced UI/UX:** The dashboard implements a completely re-skinned "Hospital Monitor" aesthetic. It utilizes dynamic CSS `@keyframes` to render a pulsing digital heartbeat over the CPU metric and a fluid-breathing wave under SpO2 Memory metric. It also features a fully adaptive, responsive CSS grid that works reliably on mobile devices.
- **Advanced Load Balancing Algorithms:** *(Requires Nginx configuration)*
- **Automated Backup Workflows:** *(Requires Backup VM cron job scheduling)*

## How to test
It's time to build it! Since you are locally positioned in /infrastructure-insight and have the 5th VM initialized in the code, run these exact commands in your terminal:

```
vagrant up
```
(Wait a few minutes for all 5 VMs to fully boot!)

```
ansible-playbook -i inventory.ini setup.yml
```
(Wait for Ansible to organically reach into all 5 VMs, install Nginx, pull down Docker, and cleanly start up your entire clinical cluster!)

After it finishes, visit http://192.168.56.11 on your Host Mac, and watch the Nginx load balancer natively hand you a live response from your web servers! Let me know if you hit any bumps!