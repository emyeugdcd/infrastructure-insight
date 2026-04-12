# INFRASTRUCTURE INSIGHT

## THE SITUATION
Congratulations on setting up your secure virtual infrastructure! 🎉
The CTO is impressed with your work and now wants to take the next step. It's time to prepare the servers for deployment and create a simple application to test the setup. This application will serve as a diagnostic tool, displaying key infrastructure information. Your mission is to develop, containerize, and deploy this application across your virtual machines, ensuring everything works seamlessly together.

## FUNCTIONAL REQUIREMENTS
### SERVER SETUP FOR DEPLOYMENT
With your secure infrastructure in place, it's time to prepare the servers for hosting your application. The CTO wants you to set up the environment for containerization and ensure proper communication between all components.
Prepare the application server, web servers, and load balancer for hosting containerized applications.
Install necessary containerization tools and dependencies on each server as needed.
Configure firewall rules to allow appropriate traffic between servers and from external sources.
Ensure all servers can communicate with each other as required for the application deployment.

### APPLICATION DEVELOPMENT
To test your infrastructure, you need to create a simple yet informative application. This will serve as a diagnostic tool for your DevOps work.
Develop a frontend and backend application in your preferred programming languages.
The application's /metrics endpoint should display various infrastructure details such as hostname, OS type, responding web server, memory usage, CPU information, and any other relevant metrics you deem important.
Ensure the application is modular and can be easily containerized.
Design the application to work with a separate backend (on the application server) and frontend (on the web servers).

### DEPLOYING THE APPLICATION
With your application ready, it's time to deploy it across your infrastructure and ensure everything works together smoothly.
Containerize your application, creating separate containers for the frontend and backend components.
Deploy the backend container on the application server.
Deploy the frontend containers on both web servers.
Configure the load balancer to distribute traffic between the two web servers.
Ensure all components can communicate effectively, with the frontend able to retrieve data from the backend.
Verify that accessing the application through the load balancer displays the correct information and alternates between the two web servers.

### EXTRA REQUIREMENTS
#### BACKUP 
Set up an additional VM dedicated to backups. Configure it to perform the following automated backup tasks:
Full weekly backup of application related data, devops /home & /etc directories.
Restoring the same data from backup.

#### ENHANCE UI/UX
Improve the user interface and experience of the diagnostic application. Some ideas for enhancements:
Responsive Design: Ensure the application works well on various screen sizes and devices.
Animations: Add subtle animations to improve user engagement and provide visual feedback.
Visual Consistency: Enhance visual consistency across different components of the application.

#### IMPLEMENT DIFFERENT LOAD BALANCING ALGORITHMS
Enhance the load balancing strategy by implementing and comparing one of the following algorithms.
Weighted Round-Robin: Implement a weighted round-robin algorithm to account for potential differences in server capabilities.
Least Connection: Add a least connection algorithm to distribute requests based on the number of active connections on each server.
Resource-Based (Adaptive): Implement a resource-based load balancing algorithm that makes decisions based on real-time server metrics like CPU usage and memory.

### WHAT YOU WILL LEARN
Setting up and managing containerized applications
Configuring and managing load balancers
Basic application development and deployment
Verifying and demonstrating a working infrastructure

### DELIVERABLES AND REVIEW REQUIREMENTS
All source code and configuration files

A README file with:
  Project overview
  Setup and installation instructions
  Usage guide
  Any additional features or bonus functionality implemented

During the review, be prepared to:
  Demonstrate your application's functionality
  Explain your code and design choices
  Discuss any challenges you faced and how you overcame them