# Assignment 6 — Build an AI-Assisted Linux Health Check (AI-Assisted Linux Incident Triage)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will build a read-only Bash triage script that checks the health of your Ubuntu server and Nginx application, connect it to Claude Code as a reusable `/linux-triage` skill, simulate a controlled Nginx incident, use the skill to gather and analyze evidence, recover the service manually, and verify recovery. The workflow follows the Agentic Loop: Gather → Analyze → Human Act → Verify.

---

# Task 1 — Confirm the Healthy Baseline and Create the Workspace

## Goal

Confirm that Nginx and the React application are healthy before building the automation.

### Evidence

#### Screenshot 1 — Output of `systemctl is-active nginx`, `ss -ltn | grep ':80'`, and `curl -I http://localhost`

![paste file](screenshots/week-03-assignment-06-screenshot-1.png)

![paste file](screenshots/week-03-assignment-06-screenshot-2.png)

![paste file](screenshots/week-03-assignment-06-screenshot-3.png)

---

#### Screenshot 2 — Output of `pwd` and `find . -maxdepth 4 -type d | sort` showing the workspace folder structure

![paste file](screenshots/week-03-assignment-06-screenshot-4.png)

---

### Notes

Answer the following in your own words:

**1. What proves that Nginx is running?**

Nginx is confirmed to be running by checking its service status with sudo systemctl status nginx. If the output shows Active: active (running), it proves that the Nginx service is running successfully.

---

**2. What proves that the server is listening for HTTP traffic?**

The server is proven to be listening for HTTP traffic by checking the listening ports using the ss -tuln (or ss -tulen) command. If the output shows that port 80 is in the LISTEN state and is associated with the Nginx process, it confirms that the server is listening for incoming HTTP traffic.

---

**3. Why must you capture a healthy baseline before simulating an incident?**

A healthy baseline is captured to record the system's normal state before simulating an incident. This provides a reference point for identifying changes, diagnosing problems, and confirming that the system has returned to normal after recovery.

---

# Task 2 — Create Project Context and Safety Rules in CLAUDE.md

## Goal

Tell Claude exactly what this project does and what it is not allowed to do.

### Evidence

#### Screenshot 3 — CLAUDE.md open in VS Code showing all four sections (Project Overview, Incident Workflow, Safety Rules, Output Rules)

![paste file](screenshots/week-03-assignment-06-screenshot-5.PNG)

---

### Notes

Answer the following in your own words:

**1. Why should Claude receive project-specific operational rules?**

Claude should receive project-specific operational rules so it understands the project's purpose, follows the correct procedures, and avoids actions that could cause problems. This ensures its responses align with the project's incident workflow and remain accurate, consistent, and safe.

---

**2. Why is the human required to execute the recovery command?**

The human is required to execute the recovery command because they are responsible for verifying that it is safe and appropriate. Claude can suggest recovery steps, but only a human should make changes to the server to prevent unintended actions or mistakes.

---

**3. Which rule prevents Claude from making an unsupported diagnosis?**

The rule, “Do not claim a root cause unless the report contains supporting evidence,” prevents Claude from making an unsupported diagnosis. It ensures that conclusions are based only on verified evidence from the report, not assumptions or guesses.

---

# Task 3 — Use Agentic AI to Plan Before Writing the Script

## Goal

Use Claude Code to inspect the environment and produce a read-only plan before creating any Bash code.

### Evidence

#### Screenshot 4 — Claude Code showing the five-check plan and read-only inspection results

![paste file](screenshots/week-03-assignment-06-screenshot-6.PNG)

![paste file](screenshots/week-03-assignment-06-screenshot-7.PNG)

![paste file](screenshots/week-03-assignment-06-screenshot-8.PNG)

---

### Notes

Answer the following in your own words:

**1. Which part of this task represents the Gather phase?**

The Gather phase is the read-only inspection of the Ubuntu server. During this phase, Claude collects information about Nginx, port 80, the HTTP response, disk usage, and available memory using read-only commands without making any changes to the system.

---

**2. Did Claude follow the instruction not to create files? How did you verify this?**

Yes, Claude followed the instruction not to create files. I verified this by checking the workspace and confirming that no new files, such as Bash scripts or other artifacts, were created. Claude only performed read-only inspection commands.

---

**3. Why is planning before coding useful in DevOps automation?**

Planning before coding is useful in DevOps automation because it defines the script's objectives, identifies the checks to perform, and ensures safe, efficient execution. It also helps detect potential issues early, reducing errors and unnecessary changes during implementation.

---

# Task 4 — Build the Linux Triage Bash Script

## Goal

Create one Bash script that gathers consistent Linux and Nginx health evidence.

### Evidence

#### Screenshot 5 — Top section of `linux-triage.sh` showing variables, thresholds, and the checks array

![paste file](screenshots/week-03-assignment-06-screenshot-9.PNG)

---

#### Screenshot 6 — Middle section showing check functions and conditionals

![paste file](screenshots/week-03-assignment-06-screenshot-10.PNG)

![paste file](screenshots/week-03-assignment-06-screenshot-11.PNG)

![paste file](screenshots/week-03-assignment-06-screenshot-12.PNG)

---

#### Screenshot 7 — Bottom section showing the loop, summary function, and exit behavior

![paste file](screenshots/week-03-assignment-06-screenshot-13.PNG)

---

#### Screenshot 8 — Output of `bash -n scripts/linux-triage.sh` (no syntax errors) and `ls -l scripts/linux-triage.sh` showing executable permission

![paste file](screenshots/week-03-assignment-06-screenshot-14.PNG)

![paste file](screenshots/week-03-assignment-06-screenshot-15.PNG)

---

### Notes

Answer the following in your own words:

**1. What is stored in the checks array?**

The checks array stores the names of the five diagnostic functions used to verify the system's health. These functions check the Nginx service, port 80, the HTTP response, disk usage, and available memory.

---

**2. How does the `for` loop use that array?**

The for loop iterates through each function name stored in the checks array and executes the corresponding function one by one. This ensures that all five health checks are performed automatically in the specified order.

---

**3. Why are the health checks separated into functions?**

The health checks are separated into functions so that each function performs one specific task. This makes the script easier to read, maintain, test, and troubleshoot, since changes to one check do not affect the others.

---

**4. What is the purpose of `$(...)` in this script?**

$(...) is used for command substitution. It executes a command and stores its output so it can be assigned to a variable or used within another command. In this script, it captures values such as the timestamp, hostname, HTTP status code, disk usage, available memory, and recent Nginx logs.

---

**5. Why does the script use different exit codes for HEALTHY, WARN, and FAIL?**

The script uses different exit codes to indicate the overall health status of the Ubuntu server after completing all five health checks. This allows users and automation tools to determine the result without reading the full report: exit code 0 means all checks passed (HEALTHY), 1 indicates a warning (WARN), and 2 indicates that one or more checks failed (FAIL). This makes it easier to automate monitoring and respond appropriately based on the severity of the issue.

---

# Task 5 — Run and Understand the Healthy-State Report

## Goal

Run the Bash script against the healthy server and verify that it creates a report.

### Evidence

#### Screenshot 9 — Output of `./scripts/linux-triage.sh` showing your Full Name and all five check results

![paste file](screenshots/week-03-assignment-06-screenshot-16.PNG)

---

#### Screenshot 10 — Output showing the captured exit code and final summary

![paste file](screenshots/week-03-assignment-06-screenshot-17.PNG)

---

### Notes

Answer the following in your own words:

**1. What is the overall status of your healthy baseline?**

The overall status of my healthy baseline is HEALTHY. All health checks passed successfully, with no failed checks detected, indicating that the system is operating normally and is ready for the incident simulation.

---

**2. Which exact Linux evidence proves the application is serving traffic?**

The Linux evidence proving the application is serving traffic is:
[PASS] Port 80 is listening — This confirms the server is accepting HTTP connections on port 80.
[PASS] Local HTTP check returned status 200 — This confirms the application is responding successfully to HTTP requests.

---

**3. Did your script return exit code 0 or 1? Explain why.**

My script returned exit code 0 because all five health checks passed successfully. Nginx was running, port 80 was listening, the application returned an HTTP 200 response, and both disk usage and available memory were within the defined healthy thresholds. This indicates the system was in a HEALTHY state.

---

**4. What is the difference between a warning and a failure in this script?**

A warning indicates that the server and application are still functioning, but a resource threshold requires attention. In this script, warnings occur when root disk usage is between 80% and 89% or available memory falls below 100 MB.

A failure indicates that a critical health check has failed and may affect the application's availability. Failures occur when Nginx is inactive, port 80 is not listening, the application does not return an HTTP 200 response, or root disk usage reaches 90% or higher.

---

# Task 6 — Create and Run the /linux-triage Skill

## Goal

Turn the Bash script into a reusable, manually invoked Agentic AI workflow.

### Evidence

#### Screenshot 11 — `SKILL.md` showing the frontmatter, allowed tool restrictions, and safety rules

Add your screenshot here.

---

#### Screenshot 12 — `/linux-triage` output for the healthy server

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. Why does this skill have Bash, Read, and Grep, but not Write?**

Add your answer here.

---

**2. Why is `disable-model-invocation: true` useful for this skill?**

Add your answer here.

---

**3. What part is performed by Bash, and what part is performed by Claude?**

Add your answer here.

---

**4. Why is this better than asking Claude "Is my server healthy?" without giving it evidence?**

Add your answer here.

---

# Task 7 — Simulate an Nginx Incident and Let the Skill Diagnose It

## Goal

Create a controlled service failure, gather evidence through Bash, and let Claude analyze the evidence without taking recovery action.

### Evidence

#### Screenshot 13 — Output showing Nginx is inactive and the HTTP request fails

Add your screenshot here.

---

#### Screenshot 14 — `/linux-triage` output showing failed evidence, most likely cause, and a suggested recovery command

Add your screenshot here.

---

#### Screenshot 15 — `incident-failure-report.txt` showing the failed checks and your Full Name

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. Which three checks failed?**

Add your answer here.

---

**2. What evidence supports the conclusion that Nginx is unavailable?**

Add your answer here.

---

**3. Did Claude execute the recovery command? Why is that important?**

Add your answer here.

---

**4. Which phase of the Agentic Loop is represented by the Bash report?**

Add your answer here.

---

**5. Which phase is represented by Claude's explanation?**

Add your answer here.

---

# Task 8 — Recover Manually, Verify Again, and Write the Incident Summary

## Goal

Recover the service as the human operator and prove that the system is healthy again.

### Evidence

#### Screenshot 16 — Output showing Nginx is active and `curl -I http://localhost` returns 200 OK

Add your screenshot here.

---

#### Screenshot 17 — Second `/linux-triage` output showing successful recovery with no FAIL results

Add your screenshot here.

---

#### Screenshot 18 — Output of `ls -lah reports` showing both `incident-failure-report.txt` and `recovery-report.txt`

Add your screenshot here.

---

#### Screenshot 19 — `incident-summary.md` showing all required sections and your Full Name

Add your screenshot here.

---

### Notes

Answer the following in your own words:

**1. What action did you execute manually?**

Add your answer here.

---

**2. What evidence proves that the service recovered?**

Add your answer here.

---

**3. Why is the second triage run necessary?**

Add your answer here.

---

**4. What could go wrong if an AI agent automatically restarted every failed service?**

Add your answer here.

---

**5. In one sentence, explain the difference between using AI as a chatbot and using AI in this agentic workflow.**

Add your answer here.

---

# Incident Summary

Fill in all seven sections below in your own words.

**Full Name:** Add your full name here

**Date:** DD/MM/YYYY

---

**1. Reported Symptom**

Add your answer here.

---

**2. Evidence Collected**

Add your answer here.

---

**3. Most Likely Cause**

Add your answer here.

---

**4. Human-Approved Recovery Action**

Add your answer here.

---

**5. Verification**

Add your answer here.

---

**6. Safety Decision**

Add your answer here.

---

**7. Agentic Loop Mapping**

Add your answer here.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`Add your URL here`

---

#### Screenshot — Published LinkedIn post

Add your screenshot here.

---

# GitHub Repository URL

Paste the URL of your GitHub folder or repository containing the assignment files here:

`Add your URL here`

---

# Submission Instructions

- Add all required screenshots in your submission
- Full Name must be visible in required screenshots and the Bash report
- All written answers must be in your own words
- Do not expose sensitive information (keys, passwords, AWS account IDs, tokens)
- GitHub URL must be included in this document

---

# Completion Checklist

- [ ] Task 1: Healthy baseline confirmed, workspace created (Screenshots 1–2, Notes answered)
- [ ] Task 2: CLAUDE.md created with all four sections (Screenshot 3, Notes answered)
- [ ] Task 3: Five-check plan produced by Claude using read-only tools (Screenshot 4, Notes answered)
- [ ] Task 4: `linux-triage.sh` created, syntax validated, executable permission set (Screenshots 5–8, Notes answered)
- [ ] Task 5: Healthy-state report generated with no FAIL result (Screenshots 9–10, Notes answered)
- [ ] Task 6: `/linux-triage` skill created and run successfully on healthy server (Screenshots 11–12, Notes answered)
- [ ] Task 7: Nginx incident simulated, failed evidence captured, Claude did not execute recovery (Screenshots 13–15, Notes answered)
- [ ] Task 8: Nginx recovered manually, recovery verified, reports saved, incident summary complete (Screenshots 16–19, Notes answered)
- [ ] Incident summary contains all seven required sections
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots and the Bash report
- [ ] Skill does not have Write permission
- [ ] Skill did not execute any recovery commands
- [ ] No sensitive data exposed

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