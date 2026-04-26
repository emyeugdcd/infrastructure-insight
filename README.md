<div align="center">
  <h1>Infrastructure Insight</h1>
  <p><strong>Chapter 2 of 8 - A scrub nurse running on cloud. A foundational DevOps journey into Infrastructure as Code, Linux Administration, and Server Security.</strong></p>
</div>

## Project Overview
This chapter takes the hardened VM infrastructure from Chapter 1 and puts it to work. We containerise a real application, distribute it across two web servers, and sit a load balancer in front of it.

This project serves as a diagnostic tool designed to test and monitor a secure, multi-node virtual infrastructure. Taking the form of a "Medical Bedside Vitals Monitor", the application aggregates live hardware metrics from underlying Linux systems and displays them via a containerized frontend-backend architecture.

The application mimics a hospital monitor used in surgical wards to track patient vitals and readiness for surgery (which actually is the metrics of the VM's servers). The app has two parts. The frontend is a web application that displays the metrics and the backend is a Golang service that collects the metrics from the servers.

## Objectives and Scope
The goal of this project is to successfully deploy a distributed an application across an isolated internal network (`192.168.56.x`). The scope includes:
- **Backend Application:** A Golang service querying root Linux system files for CPU, Memory, and Uptime metrics.
- **Frontend Application:** A Node.js web server serving a responsive dashboard themed as an ICU/surgical room monitor.
- **Load Balancing:** An Nginx instance distributing incoming requests smoothly across multiple web node instances.
- **Security:** UFW-enforced firewalls prioritizing inter-cluster traffic, ensuring SSH keys.

## Setup and Installation Instructions
*Note: This application is designed to be hosted on Virtual Machines deployed by Vagrant.*

1. **Provision Infrastructure:**
Ensure your 5-node cluster (`loadbalancer`, `webserver1`, `webserver2`, `appserver`, `backup`) is live via Vagrant with the command in the project directory:
```
vagrant up
```   
2. **Install Orchestrator Tooling:**
Ensure Docker engine and Nginx are installed across the respective nodes utilizing the provided Ansible playbook execution using `inventory.ini` file by running the following commands from your host machine to check connectivity first:
```
ansible -i inventory.ini servers -m ping
```
If that returns green, run the playbook to install Docker and Nginx and configure the network settings:
```
ansible-playbook -i inventory.ini setup.yml
```

3. **Deploy Backend (App Server):**
   ```
   docker build -t vitals-backend ./backend
   docker run -d -p 8080:8080 vitals-backend
   ```
4. **Deploy Frontend (Web Servers):**
   ```
   docker build -t vitals-frontend ./frontend
   docker run -d -p 3000:3000 -e BACKEND_URL=http://192.168.56.14:8080 vitals-frontend
   ```

## Usage Guide
Once deployed, you can access the application by navigating to the load balancer's designated IP address in your web browser:
```
http://192.168.56.11
```
The Nginx load balancer will seamlessly catch the HTTP request and route it downstream to either `webserver1` or `webserver2`. The resulting webpage will display a live, pulsing Medical Dashboard, showcasing exactly which underlying node processed your query via the "Web Server" badge.

## Bonus Functionality Implemented
- **Enhanced UI/UX:** The dashboard implements a completely re-skinned "Hospital Monitor" aesthetic. It utilizes dynamic CSS `@keyframes` to render a pulsing digital heartbeat over the CPU metric and a fluid-breathing wave under SpO2 Memory metric. It also features a fully adaptive, responsive CSS grid that works reliably on mobile devices.
- **Load Balancing Algorithms:** The Nginx load balancer uses `least_conn` (least connections) algorithm to distribute incoming requests. This algorithm intelligently checks which web server has the fewest active connections and routes new patients to that server, preventing traffic jams if one server gets bogged down.
- **Backup Workflows:** The backup server uses cron job scheduling to automatically back up the system at regular intervals. The backup is stored in the `/backups` directory on the backup server. 

## How to test
You can go to the how-to-test.md file for instructions on how to check for testing requirements for kood/sisu.