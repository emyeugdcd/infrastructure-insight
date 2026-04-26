# Checklist for reviewing Infrastructure Insight
For some of the following checklists, it migth be better for me to demonstrate during the review call session. If so, I will leave it empty with no explanation of what to do For others, I have included the commands to run in the terminal to test them.

1. A project's documentation has clear objectives, scope, requirements, and procedures
- [ ] Check README.md for objectives, scope, requirements, and procedures

2. Student has prepared the application server, web servers, and load balancer for hosting containerized applications
- [ ] Ask the student to describe the configurations and the purpose of the application server, web servers, and load balancer.

3. Necessary containerization tools and dependencies are installed on each server
- [ ] Ask the student to show the installed containerization tools.
```
docker --version 
```

4. Firewall rules are configured to allow appropriate traffic between servers and from external sources
- [ ] Ask the student to show and explain the firewall rules. No unused ports must be open.
You can check for this by running the following command on the servers:
```
sudo ufw status verbose
```
Expected result:

```text
Status: active

22/tcp ALLOW
```
Everything else should be blocked.
Example secure policy:
```text
Default: deny incoming
```

5. All servers can communicate with each other as required for the application deployment
- [ ] Ask the student to ping each server from the others.

To prove that DNS / `/etc/hosts` routing works natively, we must test that the machines know each other by their string name, not just IP.
Log into `loadbalancer` by running:
```bash
ssh -i .vagrant/machines/loadbalancer/vmware_desktop/private_key devops@192.168.56.11
```

Then run this command from `loadbalancer`:
```bash
ping webserver1
```
Expected:
```text
64 bytes from 192.168.56.12
```
You can repeat the test above to check that VMs **can reach each other**. Exit `loadbalancer` by running:
```bash
exit
```

Then log into `webserver1`:
```bash
ssh -i .vagrant/machines/webserver1/vmware_desktop/private_key devops@192.168.56.12
```

Then run from `webserver1`:
```bash
ping appserver
```

Expected:

```text
64 bytes from 192.168.56.14
```
Repeat between all servers. Also try:
```bash
ping 192.168.56.12
```


If this works, our **static network and /etc/hosts are correct**.

`telnet` and `traceroute` are considered legacy/insecure tools by modern Linux standards and are not installed on Ubuntu 22.04 by default. Instead of `telnet`, Linux engineers use `nc` (Netcat) to test if a specific port is open, or `curl`.
To prove the Webserver can reach the Appserver on port 8080:
Log into webserver1, then run this command:
```
nc -zv 192.168.56.14 8080
```

It will output:
```
Connection to 192.168.56.14 8080 port [tcp/http-alt] succeeded!
```

6. Student has developed a frontend and backend application
- [ ] Review the student's code for modularity, clarity, and adherence to best practices.

7. The application displays relevant infrastructure metrics
- [ ] Ask the student to show the infrastructure details displayed by the application. It must show at least hostname, OS type, responding web server, memory usage, and CPU information.
curl <application_url>/metrics to fetch and display metrics.

To pull the raw JSON metrics, you have to query the Go Backend directly on port 8080
Run this in your terminal:
```bash
curl http://192.168.56.14:8080/metrics
```
This will successfully dump the JSON containing CPU, Uptime, Memory, etc!


8. The application is designed for containerization
- [ ] Ask the student to show, explain, and demonstrate containerization configuration.

This project used docker compose to orchestrate the containers. Check docker-compose.yml file. This is clearly explained in README.md

9. Application is designed to work with a separate backend and frontend
- [ ]Ask the student to describe the separation of backend and frontend components.
`docker images
docker inspect <image_id> to review image details.`

Check number of containers and ensure that there are 2 front-end containers and 1 back-end container. This is clearly demonstrated in README.md

10. Backend container is deployed on the application server
- [ ]Ask the student to show the running backend container.
docker ps or equivalent
docker logs <container_id> to check the logs for any issues.

I can demonstrate this during the review. However, if you want to test, you can log in to the application server and run the following command:
```bash
ssh -i .vagrant/machines/appserver/vmware_desktop/private_key devops@192.168.56.14

docker ps
```
Expected result will show the backend container running.
```text
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

11. Frontend containers are deployed on both web servers
- [ ] Ask the student to show the running frontend containers on both web servers.
docker ps or equivalent
docker exec -it <container_id> /bin/sh to access the container shell and verify the environment.

I can demonstrate this during the review. However, if you want to test, you can log in to the web servers (1 or 2) and run the following command:
```bash
ssh -i .vagrant/machines/webserver1/vmware_desktop/private_key devops@192.168.56.12

docker ps
```
Expected result will show the frontend container running.
```text
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```


12. Load balancer is configured to distribute traffic between the two web servers
- [  ] Ask the student to show and explain the load balancer configuration, including the load balancing algorithm. Refresh the application multiple times and confirm that responses come from both web servers.
cat /etc/nginx/nginx.conf or relevant configuration file.
sudo systemctl status nginx or relevant service to check the status.

I can demonstrate this during the review. However, if you want to test, you can log in to the loadbalancer and run the following command:
```bash
ssh -i .vagrant/machines/loadbalancer/vmware_desktop/private_key devops@192.168.56.11

cat /etc/nginx/sites-available/default
```
Then you will see the `upstream vitals` block:
   ```nginx
   upstream vitals {
       least_conn;
       server 192.168.56.12:3000;
       server 192.168.56.13:3000;
   }
   ```
**Explanation:** Standard Round-Robin sends traffic 50/50 blindly. I implemented `least_conn`. This algorithm intelligently checks which web server has the fewest active connections and routes new patients to that server, preventing traffic jams if one server gets bogged down.


13. All components can communicate effectively
- [ ]Ask the student to show the frontend retrieving data from the backend.
curl <load_balancer_ip> to test the application.
curl -I <load_balancer_ip> to check HTTP headers and server responses.

Again, log into the loadbalancer and run this command:
```bash
curl http://192.168.56.11/config
```
Expected result:
```json
{
  "webServer": "webserver1",
  "cpu": "65%",
  "memory": "2GB",
  "disk": "5GB",
  "uptime": "1 day"
}
```

14. Accessing the application through the load balancer displays the correct information
- [ ]Ask the student to access the application through the load balancer and show the output.
```bash
curl http://192.168.56.11
```
Expected result:
```text
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Infrastructure Insight</title>
    <script src="http://192.168.56.14:8080/metrics"></script>
</head>
<body>
    <h1>Infrastructure Insight</h1>
    <div class="card">
        <h2>CPU Usage</h2>
        <p id="cpu">65%</p>
    </div>
    <div class="card">
        <h2>Memory Usage</h2>
        <p id="memory">2GB</p>
    </div>
    <div class="card">
        <h2>Disk Usage</h2>
        <p id="disk">5GB</p>
    </div>
    <div class="card">
        <h2>Uptime</h2>
        <p id="uptime">1 day</p>
    </div>
</body>
</html>
```

Extra
15. Backup VM is set up and configured
- [ ] Verify the existence and configuration of the backup VM

16. Weekly full backups are automated for application data, /home, and /etc
- [ ] Review the backup scripts and schedule. Confirm that backups are being created as expected.
crontab -l to check the cron jobs.
cat /etc/cron.d/backup to review the backup schedule.
ls /path/to/backup to verify the presence of backup files.

For number 15 and 16, I can demonstrate this during the review. However, if you want to test, you can log into the backup server and run the following command:
```bash
ssh -i .vagrant/machines/backup/vmware_desktop/private_key devops@192.168.56.15
```
Prove the cron job exists: `crontab -l`
*(This will print the exact tar command scheduled to run weekly).*
Show the directory: `ls -la /backups`

17. Data can be restored from backups
- [ ]Ask the student to demonstrate the restoration process for each type of backed-up data.

I can demonstrate how to extract the `.tar.gz` archive for backup files if needed.

Again log into the `backup` server, manually run the backup command so you have a file to test with:
```bash
   sudo tar -czf /backups/system_backup_test.tar.gz /etc /home 2>/dev/null
   ```
Then, to extract it into a temporary folder:
```bash
   mkdir /tmp/restore_test
   sudo tar -xzf /backups/system_backup_test.tar.gz -C /tmp/restore_test
   ls /tmp/restore_test/home
   ```
  You have successfully extracted the archived files. `devops` and `vagrant` are the exact two user profiles stored inside the `/home` directory on that server. You have successfully restored the entire user-data state of the machine

18. Student has improved the user interface and experience of the diagnostic application
- [ ] Application should be responsive, easy, and pleasant to use.

Go to the URL: http://192.168.56.11
You will see the application running. It is responsive and easy to use. One thing to note is that you will feel like the "Refresh" Button seems to not working. There is absolutely nothing broken here! It's because of an advanced networking feature called HTTP Keep-Alive!

When you type http://192.168.56.11 into Chrome or Safari and hit refresh, your browser doesn't actually create a new connection to Nginx. To save time and CPU, modern browsers keep the "TCP socket" open. Because Nginx is using least_conn (Least Connections), it sees that your browser already has a dedicated, active connection to webserver1, so it keeps sending you there instead of switching to webserver2. This is normal and expected behavior.
 
So how do you know if the load balancer is working: Since browsers "cheat" by holding the connection open, you have to use a tool that creates a brand new connection every time. Run this command a few times in a row on your Mac:
```
curl http://192.168.56.11/config
```
Because curl drops the connection immediately after reading the data, Nginx is forced to re-evaluate the algorithm every single time. You will instantly see the JSON output flip-flop between "webServer":"webserver1" and "webServer":"webserver2"! Alternatively, if you want to show it in the UI, just open a normal Chrome window, and then open a private Incognito Window side-by-side. The Incognito window will force a new connection and you will see the other web server!

19. Student has implemented and compared different load balancing algorithms
- [ ] Ask the student to show the configuration and explain the use case of the implemented algorithm.
Check above in number 18.

20. Student has implemented additional technologies, security enhancements and/or features beyond the core requirements

- [ ] Ask the student to show and explain the additional technologies, security enhancements and/or features implemented beyond the core requirements.
I have implemented technologies and security enhancements from previous project (server-sorcery-101) that is currently protecting this project including UFW, Fail2ban, WireGuard, and Unattended Upgrades.

1. **Uncomplicated Firewall (UFW):** Strict default-deny rules.
2. **Fail2Ban:** Active IPS brute-force protection.
3. **WireGuard:** Kernel-level secure VPN tunneling ready for remote admin access.
4. **Unattended Upgrades:** Automated silent security patching.

