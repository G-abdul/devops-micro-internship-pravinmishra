# Assignment 3 — Production Maintenance Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will treat your already deployed React application (on Ubuntu VM with Nginx) as a live production system. You will perform structured operational checks covering network validation, service health, log analysis, resource monitoring, configuration verification, and incident simulation with recovery — mirroring real on-call DevOps responsibilities.

---

# Task 1 — Server Access & Networking Validation

## Goal

Verify that the deployed React application is reachable from the browser and confirm basic network connectivity of the Ubuntu VM.

### Evidence

#### Screenshot 1 — Browser showing the React app with your Full Name visible on the UI

![Screenshot-1](./screenshots/Deployed-React-application.png)

---

#### Screenshot 2 — Output of `ip a`

![Screenshot-2](./screenshots/Output-of-ip-a.png)

---

#### Screenshot 3 — Output of `sudo ss -tulpen`

![Screenshot-3](./screenshots/sudo-ss-tulpen.png) 

---

#### Screenshot 4 — Output of `sudo ufw status`

![Screenshot-4](./screenshots/sudo-ufw-status.png)

---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

The sudo ss -tulpen output showed Nginx listening on 0.0.0.0:80. The 0.0.0.0 address means Nginx is accepting connections on all network interfaces, while port 80 is the default HTTP port. This confirms that the web server is available to users accessing the server through its public IP address.
---

**2. What proves SSH is active on port 22?**

The sudo ss -tulpen output showed the SSH daemon (sshd) listening on 0.0.0.0:22 and [::]:22. This confirms that SSH is running and accepting remote connections on port 22 over both IPv4 and IPv6 networks.

---

**3. Did you find any unexpected open ports? Explain briefly.**

No, I did not find any unexpected open ports. Besides SSH on port 22 and Nginx on port 80, I noticed ports used by standard Ubuntu services such as systemd-resolved (DNS on port 53), systemd-networkd (DHCP on port 68), and chronyd (time synchronization on port 323). These are normal system services and not something I intentionally configured, but they are expected on a typical Ubuntu server.

---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

![Screenshot-1](./screenshots/systemctl-status-nginx--no-pager.png)

---

#### Screenshot 2 — Output of `sudo nginx -t`

![Screenshot-2](./screenshots/sudo-nginx-t.png)

---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

![Screenshot-3](./screenshots/sudo-sport.png)

---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

If Nginx fails to restart in production, users may be unable to access the website or application being served by Nginx. Any new configuration changes would not take effect, and the service could become unavailable if Nginx stops completely. This is why it's important to test configurations before restarting and check error logs when issues occur.

---

**2. What's your basic rollback plan?**

My rollback plan is to restore the last known working Nginx configuration or application build, test the configuration using sudo nginx -t, restart Nginx, and verify that the service is running correctly. If the issue was caused by a deployment, I would redeploy the previous working version before investigating the root cause.

---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

![Screenshot-1](./screenshots/sudo-access-log.png) 

---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

![Screenshot-2](./screenshots/sudo-error-log.png) 

---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

![Screenshot-3](./screenshots/sudo-journalctl.png) 

---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

I did not find any critical errors in the Nginx error log during my check. The only recent entry was:

2026/07/16 15:58:37 [notice] 27079#27079: using inherited sockets from "5;6;"

This is a notice rather than an error. It indicates that Nginx reused existing network sockets during a reload or restart, allowing it to continue serving traffic without interruption. Since there were no actual error messages, it suggests that Nginx was running normally at the time of my inspection.

---

**2. If there were no errors, what does that indicate about the system?**

If the error log shows no recent errors, it generally indicates that Nginx is operating correctly and handling requests without major problems. It suggests that the web server configuration is valid, the service is stable, and there are no obvious issues preventing users from accessing the application. However, it does not guarantee that everything is perfect, so access logs and application behavior should also be checked.

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

Yes, my curl request was visible in the access log. I found the following entry:

16.16.100.112 - - [17/Jul/2026:22:42:05 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/8.18.0"

This proves that the request successfully reached the Nginx server, was processed, and received a 200 OK response. It also confirms that traffic is flowing correctly between the client and the web server, and that Nginx is logging incoming requests as expected.

---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`

![Screenshot-1](./screenshots/uptime.png)

---

#### Screenshot 2 — Output of `free -h`

![Screenshot-2](./screenshots/free-h.png)

---

#### Screenshot 3 — Output of `df -h`

![Screenshot-3](./screenshots/df-h.png)

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

![Screenshot-4](./screenshots/sudo-du-sh.png)

---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

Disk usage looks the most critical right now. The CPU load is very low (0.00, 0.00, 0.00), which means the server is not under processing pressure. Memory usage is also reasonable, with about 552 MB available. However, the root disk is already 64% utilized (4.2 GB used out of 6.7 GB). While this is not an immediate problem, disk space is the resource I would monitor most closely because it can gradually fill up with application files, logs, and updates over time.

---

**2. What happens if disk becomes 100% full in a production server?**

If a production server reaches 100% disk usage, applications may stop working properly because they can no longer write data to storage. Log files may stop being generated, uploads can fail, temporary files cannot be created, and databases may experience errors. In severe cases, parts of the application may become unavailable until disk space is freed. This is why monitoring disk usage and cleaning up unnecessary files is important in production environments.

---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

![Screenshot-1](./screenshots/head-n-20.png)

---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

![Screenshot-2](./screenshots/grep-R-Deployed-by.png)

---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

![Screenshot-3](./screenshots/grep-n-try_files.png)

---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

I can confirm that the correct version of the application is deployed by checking the files in the Nginx web root (/var/www/html) and verifying that the expected content exists. Using the grep command to search for a unique identifier such as "Deployed by" helps confirm that the files currently being served are the ones I deployed. I can also verify this by opening the application in a browser and checking that the expected name, date, or other personalized changes are displayed correctly.

---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

![Screenshot-1](./screenshots/broken-config.png)

---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)

![Screenshot-2](./screenshots/fixed-config.png)

---

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![Screenshot-3](./screenshots/200-OK.png)

---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

The configuration failure was caused by a syntax error in the Nginx configuration file. Specifically, the semicolon (;) at the end of the try_files directive was removed. Nginx directives must end with a semicolon, so removing it made the configuration invalid and prevented Nginx from accepting the configuration.

---

**2. How did you fix the issue?**

I fixed the issue by editing the Nginx configuration file and re-adding the missing semicolon to the try_files line. After making the correction, I ran sudo nginx -t to verify that the configuration was valid. Once the test was successful, I restarted Nginx using sudo systemctl restart nginx to apply the changes.

---

**3. How can you avoid this kind of issue in real production systems?**

To avoid this type of issue in production, I would always test configuration changes using sudo nginx -t before restarting or reloading Nginx. I would also review changes carefully, use version control to track configuration updates, and follow a change management process so mistakes can be identified and rolled back quickly if needed. This helps prevent small syntax errors from causing service disruptions.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

![Screenshot-2](./screenshots/non-200-response.png)

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![Screenshot-2](./screenshots/200-OK.png)

---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

The application broke because of an incorrect configuration change that prevented Nginx from serving the application properly. In this exercise, a syntax error was intentionally introduced into the Nginx configuration file, which caused the configuration validation to fail. Since Nginx relies on valid configuration files to route requests correctly, even a small mistake can impact application availability.
---

**2. How did you fix the issue and restore the application?**

I fixed the issue by reviewing the Nginx configuration file, identifying the incorrect change, and restoring the correct configuration. After making the correction, I ran sudo nginx -t to verify that the configuration was valid. Once the test passed successfully, I restarted Nginx and confirmed that the application was accessible again through the server's public IP address.

---

**3. What steps would you take to prevent this kind of issue in real production systems?**

To prevent this type of issue, I would always validate configuration changes using nginx -t before applying them. I would also keep configuration files under version control, create backups before making changes, and test updates in a staging environment whenever possible. These practices help catch errors early and make rollbacks easier if something goes wrong.

---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

SSH key-based authentication is more secure because it uses cryptographic keys instead of passwords that can be guessed, reused, or stolen. Private keys remain on the user's device and are not transmitted during login, making unauthorized access much more difficult. It also reduces the risk of brute-force password attacks.

---

**2. Why should only required ports be open on a production server?**

Every open port increases the server's attack surface. By keeping only the necessary ports open, such as port 22 for SSH and port 80 for web traffic, the risk of unauthorized access and potential security vulnerabilities is reduced. Limiting exposed services is a basic security best practice.

---

**3. Why is it important for Nginx to be enabled on boot?**

Enabling Nginx on boot ensures that the web server starts automatically whenever the server is restarted. This improves reliability because the application becomes available without requiring manual intervention after maintenance, updates, or unexpected reboots.
---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

Publicly sharing credentials can allow unauthorized users to access servers, cloud accounts, databases, or applications. This can lead to data breaches, service disruptions, financial losses, and compromised systems. Once a secret is exposed, it should be considered compromised and replaced immediately.

---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

Unused cloud resources can continue generating charges even when they are not actively being used. Stopping or terminating unnecessary resources helps control costs, reduces wasted infrastructure, and minimizes the security risks associated with leaving unused services running.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

https://www.linkedin.com/posts/abdulganiyu0_dmibypravinmishra-devops-aws-ugcPost-7485001060873580547-A_38/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAFamVAYBbC0P-4_t5y56JbVGUfZFmuyqJnY

<<<<<<< Updated upstream
`Add your URL here`
=======
`______________________________________________________________________________________________`
>>>>>>> Stashed changes

---

#### Screenshot — Published LinkedIn post

![Linkedin](./screenshots/Linkedin-Prod.png) ![Linkedin](./screenshots/Linkedin-Prod-2.png)

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [✅] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [✅] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [✅] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [✅] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [✅] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [✅] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [✅] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [✅] Task 8: Security & Reliability Notes answered
- [✅] LinkedIn post published and URL submitted
- [✅] Full Name visible in all required screenshots
- [✅] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*