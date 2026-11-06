# Capstone Assignment — Deploy the Book Review App Using Terraform and Claude Code Agentic AI

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Student Details

**Full Name:** Micheal Omotosho
**Cloud Platform:** AWS
**GitHub Repository URL:** https://github.com/michealdayo64/devops-micro-internship-pravinmishra 
**Public Application URL / Load-Balancer DNS:** Add the public URL or DNS here

---

## Purpose

Deploy the Book Review App using Terraform on AWS or Azure in a secure, highly available, production-style three-tier architecture. Use Claude Code, specialized subagents, Terraform MCP, and validation hooks to support the engineering workflow while keeping all infrastructure-changing operations under human control.

---

# Task 0 — Prepare the Project and Agentic AI Environment

## Goal

Prepare the Book Review App project and configure the provided Claude Code Agentic AI starter kit with project context, specialized subagents, Terraform MCP, validation hooks, and safety guardrails.

## Evidence

### Screenshot 1 — Project `CLAUDE.md`

Add a screenshot of the project `CLAUDE.md` showing the three-tier architecture, security boundaries, Terraform requirements, and human-approval rules.

![paste file](screenshots/week-08-screenshot-75.png)

---

### Screenshot 2 — Terraform Engineer Subagent

Add a screenshot showing the Terraform Engineer subagent configuration.

![paste file](screenshots/week-08-screenshot-76.png)

---

### Screenshot 3 — Architecture and Security Reviewer Subagent

Add a screenshot showing the Architecture and Security Reviewer subagent configuration.

![paste file](screenshots/week-08-screenshot-77.png)

---

### Screenshot 4 — Terraform MCP Connection

Add a screenshot showing Terraform MCP connected and available.

![paste file](screenshots/week-08-screenshot-78.png)

---

### Screenshot 5 — Validation Hooks

Add a screenshot showing the configured Claude Code validation hooks.

![paste file](screenshots/week-08-screenshot-79.png)

---

# Task 1 — Design the Three-Tier Architecture

## Goal

Design the required secure, highly available three-tier architecture and create an architecture diagram before building the infrastructure.

The diagram must show:

- VPC or VNet
- Availability Zones or equivalent availability locations
- Six subnets
- Internet connectivity
- NAT or outbound design
- Public load balancer
- Web Tier
- Internal load balancer
- Application Tier
- Managed MySQL
- Read replica
- Main traffic flow

## Architecture Diagram

![paste file](screenshots/3tier-design.png)

---

# Task 2 — Build the Terraform Networking and Security Layers

## Goal

Create the modular Terraform project and implement the network and security layers across the required public and private subnets.

## Evidence

### Screenshot 6 — Modular Terraform Project Structure

Add a screenshot showing the modular Terraform project structure.

![paste file](screenshots/week-08-screenshot-81.png)

---

### Screenshot 7 — Six-Subnet Architecture

Add a screenshot showing the six-subnet architecture across two availability locations.

![paste file](screenshots/week-08-screenshot-80.png)

---

### Screenshot 8 — Public and Private Tier Separation

Add a screenshot showing the public and private tier separation, including routing and security boundaries.

![paste file](screenshots/week-08-screenshot-83.png)

---

# Task 3 — Build the Load-Balancing and Compute Layers

## Goal

Deploy the public and internal load balancers and the Web and Application compute resources required by the Book Review App.

## Evidence

### Screenshot 9 — Web and Application Compute

Add a screenshot showing the Web and Application compute resources in their required subnets.

![paste file](screenshots/week-08-screenshot-82.png)

---

### Screenshot 10 — Public Load Balancer

Add a screenshot showing the internet-facing public load balancer.

![paste file](screenshots/week-08-screenshot-85.png)

---

### Screenshot 11 — Internal Load Balancer

Add a screenshot showing the private internal load balancer.

![paste file](screenshots/week-08-screenshot-86.png)

---

### Screenshot 12 — Healthy Targets

Add a screenshot showing healthy target groups or backend pools.

![paste file](screenshots/week-08-screenshot-87.png)

---

# Task 4 — Build the Managed MySQL Database Layer

## Goal

Deploy a private, highly available managed MySQL database with a read replica and restrict database connectivity to the Application Tier.

## Evidence

### Screenshot 13 — Managed MySQL Database

Add a screenshot showing the managed MySQL database deployment.

![paste file](screenshots/week-08-screenshot-88.png)

---

### Screenshot 14 — High Availability

Add a screenshot showing the Multi-AZ or high-availability configuration.

![paste file](screenshots/week-08-screenshot-89.png)

---

### Screenshot 15 — Read Replica

Add a screenshot showing the read replica configuration.

![paste file](screenshots/week-06-screenshot-68.jpg)

---

### Screenshot 16 — Private Database Access

Add a screenshot showing that the database is private and accepts MySQL traffic only from the Application Tier.

![paste file](screenshots/week-08-screenshot-90.png)

---

# Task 5 — Validate, Review, and Apply the Terraform Configuration

## Goal

Validate the Terraform configuration, review the execution plan using both Agentic AI and human judgment, and apply the infrastructure changes only after all required checks pass.

## Evidence

### Screenshot 17 — Terraform Validation

Add a screenshot showing successful `terraform validate` output.

![paste file](screenshots/week-08-screenshot-91.png)

---

### Screenshot 18 — Terraform Plan

Add a screenshot showing the Terraform plan output.

![paste file](screenshots/week-08-screenshot-92.png)

---

### Screenshot 19 — Terraform Apply

Add a screenshot showing successful `terraform apply` completion.

![paste file](screenshots/week-08-screenshot-59.png)

---

# Task 6 — Deploy and Configure the Book Review Application

## Goal

Deploy and configure the Book Review App across the Web, Application, and Database tiers and verify the complete application functionality.

## Evidence

### Screenshot 20 — Homepage

Add a screenshot showing the Book Review App homepage through the public endpoint.

![paste file](screenshots/week-08-screenshot-93.png)

---

### Screenshot 21 — Login or Authentication

Add a screenshot showing successful login or authentication.

![paste file](screenshots/week-08-screenshot-94.png)

---

### Screenshot 22 — Book Data

Add a screenshot showing the book listing or book details.

![paste file](screenshots/week-08-screenshot-95.png)

---

### Screenshot 23 — Review Functionality

Add a screenshot showing the review functionality working successfully.

![paste file](screenshots/week-08-screenshot-96.png)

---

### Screenshot 24 — Backend or API Evidence

Add a screenshot showing that the backend or API is working successfully.

![paste file](screenshots/week-08-screenshot-97.png)

---

### Screenshot 25 — Database Reads and Writes

Add a screenshot showing successful database reads and writes.

![paste file](screenshots/week-08-screenshot-95.png)

## Public Application URL

**Public Application URL / DNS:** http://book-review-public-alb-1737082241.us-east-1.elb.amazonaws.com

---

# Task 7 — Demonstrate the Agentic AI Workflow

## Goal

Demonstrate how Claude Code assisted with Terraform generation, architecture and security review, and evidence-based troubleshooting while infrastructure-changing decisions remained under human control.

You do not need to submit your complete Claude Code conversation history. Include only focused evidence.

## Evidence

### Screenshot 26 — AI-Assisted Terraform Generation

Add a screenshot showing one useful example of AI-assisted Terraform generation or improvement.

![paste file](screenshots/week-08-screenshot-98.png)

---

### Screenshot 27 — Architecture or Security Review

Add a screenshot showing one structured architecture or security review result.

![paste file](screenshots/week-08-screenshot-99.png)

---

### Screenshot 28 — AI-Assisted Troubleshooting

Add a screenshot showing one AI-assisted troubleshooting interaction based on collected evidence.

![paste file](screenshots/week-08-screenshot-100.png)

---

# Task 8 — Complete the Final Architecture Review

## Goal

Review the completed infrastructure against the original capstone requirements and resolve significant architecture, security, reliability, and cost issues.

Confirm that the final review covers:

- Tier separation
- Availability
- Public exposure
- Routing
- Security rules
- Load balancing
- Database privacy
- Secrets
- Terraform quality
- Module structure
- Reliability
- Obvious cost risks

Use Screenshot 27 as the focused evidence for the structured architecture or security review.

---

# Task 9 — Answer the Reflection Questions

## Goal

Reflect on the architecture, Terraform implementation, and Agentic AI workflow. Answer each question briefly in your own words.

## Architecture

### 1. Why did you separate the Web, Application, and Database tiers?

I separated the Web, Application, and Database tiers to implement defense-in-depth and enforce the principle of least privilege.
The Web tier handles presentation and incoming user requests, the Application tier processes business logic, and the Database tier manages persistent data.
Each tier operates within its own security boundary, with security group rules restricting communication to only the necessary ports and sources. For example, the database accepts connections only from the application tier, never directly from the public internet.
This architecture reduces the attack surface, limits the blast radius if one component is compromised, improves scalability, and makes the infrastructure easier to maintain independently."

### 2. Why is the Application Tier private?

The Application Tier is private because it hosts the backend services that handle sensitive business logic and communicate directly with the database. By placing it in private subnets without public IP addresses, we prevent direct internet access and reduce the attack surface.
Access to the Application Tier is controlled through the internal Application Load Balancer, which receives requests from the Web Tier. Security Groups further restrict inbound traffic to only authorized sources and ports.
This ensures that the backend services are accessible only through approved internal communication paths, following the principle of least privilege.

### 3. Why is MySQL private?

MySQL is deployed in isolated private subnets because it stores sensitive application data, including user records and business information. Exposing the database directly to the internet would significantly increase its attack surface.
To enforce security, the database Security Group allows inbound traffic only on TCP port 3306 from the Application Tier's Security Group. This ensures that only authorized backend services can establish database connections.

### 4. Why are multiple Availability Zones used?

Multiple Availability Zones are used to achieve high availability and fault tolerance. By distributing application resources across multiple Availability Zones within the same AWS Region, we reduce the risk of a single point of failure.

### 5. What is the difference between Multi-AZ/high availability and a read replica?

- Multi-AZ (High Availability)

Amazon RDS Multi-AZ is designed to provide high availability and fault tolerance. It maintains a standby database in a different Availability Zone and synchronously replicates database changes to that standby.

- Read Replica (Read Scalability)

A Read Replica uses asynchronous replication to maintain a copy of the primary database.
Unlike a Multi-AZ standby, a Read Replica can serve read-only application requests. This allows you to distribute read-heavy workloads across multiple database instances, reducing the query load on the primary database.

## Terraform

### 6. How did you divide your Terraform into modules?

I structured my Terraform configuration into reusable modules based on functional responsibilities and infrastructure layers. This follows the principle of separation of concerns, making the infrastructure easier to maintain, test, troubleshoot, and scale.

I divided the infrastructure into five main modules:
- Network
- Security
- Load-balancer
- Compute
- Database

### 7. How do the modules communicate through variables and outputs?

The root module orchestrates communication: lower-level modules (like Network) expose resource attributes as outputs (e.g., subnet_ids, security_group_ids), and the root main.tf passes those outputs into dependent modules as input variables.

### 8. What did you specifically check in `terraform plan`?

I verified the proposed action summary (+ to create, ~ to modify, - to destroy), checked that security groups had precise CIDRs without open ingress on internal ports, confirmed resource count, and ensured private subnets did not accidentally map public IPs.

## Agentic AI

### 9. What was the purpose of `CLAUDE.md`?

CLAUDE.md serves as a persistent project instruction and context file for Claude Code. It provides the AI agent with essential project information, architectural decisions, coding standards, and operational constraints before it starts working on the codebase.

### 10. What work did the Terraform Engineer subagent perform?

The Terraform Engineer subagent was responsible for infrastructure code development and refinement.
I delegated Terraform-specific tasks to it so that it could focus on generating, reviewing, and improving Terraform configurations while following the architectural guidelines defined in CLAUDE.md.

### 11. What did the Architecture and Security Reviewer identify?

The Architecture and Security Reviewer subagent was responsible for evaluating the proposed Terraform configurations against the project's architectural requirements and AWS security best practices.
It focused on identifying potential security vulnerabilities, misconfigurations, and architectural weaknesses before infrastructure deployment.

### 12. Why did you use Terraform MCP instead of relying only on Claude's existing Terraform knowledge?

I integrated Terraform MCP with Claude Code to give the AI agent access to up-to-date Terraform provider schemas, resource documentation, and configuration references.
Although Claude has existing Terraform knowledge from its training data, that knowledge may not always reflect the latest AWS provider versions, resource attributes, or deprecated arguments.
Terraform MCP provides access to current technical references, helping the agent generate configurations that align with the installed provider version and supported resource schemas.

### 13. What was the purpose of your validation hooks?

I implemented validation hooks in Claude Code to enforce automated quality checks and prevent invalid or non-compliant Terraform configurations from progressing through the development workflow.
The hooks acted as automated guardrails by executing predefined checks at specific points during Claude Code's interaction with project files.

### 14. Describe one real issue Claude helped you troubleshoot.

During deployment, I encountered an issue where the internal Application Load Balancer (ALB) target group continuously reported my Node.js backend instances as unhealthy.
I used Claude Code to investigate the issue by reviewing the Terraform target group configuration, health check settings, and the Node.js application configuration.
The investigation identified a mismatch between the health check endpoint configured in the ALB target group and the actual health endpoint exposed by the Node.js application.
The application exposed /health, but the ALB target group was configured to check a different path.

### 15. Describe one recommendation you reviewed, modified, or rejected instead of accepting blindly.

Claude suggested placing the NAT Gateway in a private subnet to "keep it secure." I rejected this recommendation because it misunderstood the purpose and routing requirements of a NAT Gateway.
A NAT Gateway must be deployed in a public subnet with a route to the Internet Gateway. Private subnet resources then route their outbound internet traffic through the NAT Gateway.
I corrected the Terraform configuration by placing the NAT Gateway in a public subnet, associating it with an Elastic IP, and updating the private subnet route tables to direct internet-bound traffic through the NAT Gateway.
This allowed private EC2 instances to access external services for software updates and package installations without exposing them directly to inbound internet traffic.

---

# Task 10 — Publish the Mandatory LinkedIn Post

## Goal

Publish a LinkedIn post describing the capstone, the technical work completed, the Agentic AI workflow, and the lessons learned.

Write the post in your own words, include at least one project image or other proof, and ensure that it can be viewed by the submission reviewer.

## LinkedIn Post URL

**LinkedIn Post URL:** Add your LinkedIn post URL here

---

# Submission Instructions

- Complete Tasks 0–10 in sequence.
- Include all Screenshots 1–28 exactly as specified.
- Ensure that your full name is visible in the required screenshots.
- Include the selected cloud platform.
- Include the completed architecture diagram.
- Include the modular Terraform project structure.
- Include the working public application URL or public load-balancer DNS.
- Include all required Agentic AI workflow evidence.
- Answer all 15 reflection questions briefly in your own words.
- Include the published LinkedIn post URL.
- Do not expose cloud credentials, database passwords, SSH private keys, JWT secrets, access tokens, account IDs, Terraform state containing sensitive values, or other confidential information.
- Review all screenshots and project files carefully before submitting through GitHub.

---

# Completion Checklist

- [ ] Selected AWS or Azure
- [ ] Added and reviewed the Agentic AI starter files
- [ ] Configured `CLAUDE.md`
- [ ] Configured the Terraform Engineer subagent
- [ ] Configured the Architecture and Security Reviewer subagent
- [ ] Connected Terraform MCP
- [ ] Configured validation hooks and safety guardrails
- [ ] Created the architecture diagram
- [ ] Created the six-subnet design
- [ ] Configured public Web Tier routing
- [ ] Kept the Application Tier private
- [ ] Kept the Database Tier private
- [ ] Configured tier-specific Security Groups or NSGs
- [ ] Restricted backend port `3001`
- [ ] Restricted MySQL port `3306` to the Application Tier
- [ ] Created the public load balancer
- [ ] Created the internal load balancer
- [ ] Configured listeners and health checks
- [ ] Deployed the Web Tier compute resources
- [ ] Deployed the private Application Tier compute resources
- [ ] Provisioned private managed MySQL
- [ ] Configured Multi-AZ or high availability
- [ ] Configured a read replica
- [ ] Created the modular Terraform project
- [ ] Used variables, outputs, and module dependencies
- [ ] Used current Terraform documentation through MCP
- [ ] Used hooks for deterministic validation
- [ ] Completed `terraform fmt`
- [ ] Completed `terraform validate`
- [ ] Reviewed `terraform plan`
- [ ] Completed the Terraform Engineer review
- [ ] Completed the Architecture and Security review
- [ ] Applied the infrastructure only after human approval
- [ ] Deployed and configured the backend
- [ ] Deployed and configured the frontend
- [ ] Configured Nginx where required
- [ ] Configured the internal backend endpoint
- [ ] Configured the public frontend endpoint
- [ ] Verified the homepage
- [ ] Verified login or authentication
- [ ] Verified book data
- [ ] Verified review functionality
- [ ] Verified the backend API
- [ ] Verified database reads and writes
- [ ] Verified healthy load-balancer targets
- [ ] Included AI-assisted Terraform generation evidence
- [ ] Included one architecture or security review
- [ ] Included one AI-assisted troubleshooting example
- [ ] Completed the final architecture review
- [ ] Answered all 15 reflection questions
- [ ] Published the mandatory LinkedIn post
- [ ] Added the LinkedIn post URL
- [ ] Captured all 28 required screenshots
- [ ] Confirmed that my full name is visible in the required screenshots
- [ ] Checked that no secrets or sensitive information are exposed

---

## About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory), focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations through hands-on experience.

---

## Resources

- Book Review App Repository: [https://github.com/pravinmishraaws/book-review-app](https://github.com/pravinmishraaws/book-review-app)
- DMI Official Website: [https://dmi.pravinmishra.com](https://dmi.pravinmishra.com)
- University: [https://university.pravinmishra.com](https://university.pravinmishra.com)
- Discord Community: [https://discord.pravinmishra.com](https://discord.pravinmishra.com)
- Blog: [https://dmi.pravinmishra.com/blog](https://dmi.pravinmishra.com/blog)
- YouTube Playlist: [https://www.youtube.com/playlist?list=PLFeSNDtI4Cho](https://www.youtube.com/playlist?list=PLFeSNDtI4Cho)
- Pravin Mishra on LinkedIn: [https://www.linkedin.com/in/pravin-mishra-aws-trainer/](https://www.linkedin.com/in/pravin-mishra-aws-trainer/)
- CloudAdvisory on LinkedIn: [https://www.linkedin.com/company/thecloudadvisory/](https://www.linkedin.com/company/thecloudadvisory/)

---

*This submission is part of the DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
