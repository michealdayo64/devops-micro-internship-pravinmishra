# Assignment 5 — Deploy a Highly Available Two-Tier Application on AWS (VPC + ALB + ASG + Multi-AZ RDS)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will design and deploy a highly available two-tier web application on AWS: highly available networking across two Availability Zones, an Application Load Balancer, an Auto Scaling Group for the web tier, and a private Multi-AZ RDS database. You must prove high availability with real failure tests.

---

# Task 1 — Create HA Networking (VPC + 4 Subnets + IGW + NAT + Route Tables)

## Goal

Build a VPC (10.0.0.0/16) with two public and two private subnets across two Availability Zones, an Internet Gateway, a NAT Gateway, and the matching public/private route tables.

### Evidence

#### Screenshot 1 — VPC details showing CIDR 10.0.0.0/16

![paste file](screenshots/week-06-screenshot-34.jpg)

---

#### Screenshot 2 — Subnets list showing four subnets and their Availability Zones

![paste file](screenshots/week-06-screenshot-35.jpg)

---

#### Screenshot 3 — Public route table showing the Internet Gateway route and both public-subnet associations

![paste file](screenshots/week-06-screenshot-36.jpg)

![paste file](screenshots/week-06-screenshot-37.jpg)

---

#### Screenshot 4 — Private route table showing the NAT Gateway route and both private-subnet associations

![paste file](screenshots/week-06-screenshot-37.jpg)

![paste file](screenshots/week-06-screenshot-38.jpg)

---

#### Screenshot 5 — NAT Gateway status showing Available and the Elastic IP

![paste file](screenshots/week-06-screenshot-40.jpg)

---

# Task 2 — Create Security Groups (ALB, EC2, RDS) with Least Privilege

## Goal

Create `ha-alb-sg` (HTTP public), `ha-web-sg` (HTTP only from `ha-alb-sg`, SSH from your IP), and `ha-db-sg` (database port only from `ha-web-sg`).

### Evidence

#### Screenshot 6 — ALB Security Group inbound rules

![paste file](screenshots/week-06-screenshot-41.jpg)

---

#### Screenshot 7 — EC2 Security Group inbound rules showing the ALB Security Group reference and SSH from your IP

![paste file](screenshots/week-06-screenshot-42.jpg)

---

#### Screenshot 8 — RDS Security Group inbound rule showing the database port allowed only from the EC2 Security Group

![paste file](screenshots/week-06-screenshot-43.jpg)

---

# Task 3 — Deploy Database Tier (RDS Multi-AZ in Private Subnets)

## Goal

Launch a private, Multi-AZ RDS database (MySQL or PostgreSQL) using the private DB Subnet Group and `ha-db-sg`.

### Evidence

#### Screenshot 9 — RDS summary showing Multi-AZ = Yes and Publicly accessible = No

![paste file](screenshots/week-06-screenshot-44.jpg)

![paste file](screenshots/week-06-screenshot-45.jpg)

---

#### Screenshot 10 — RDS connectivity section showing the DB Subnet Group and Security Group

![paste file](screenshots/week-06-screenshot-46.jpg)

---

# Task 4 — Build a Launch Template (User Data Installs App + Connects to DB)

## Goal

Create a Launch Template whose user data installs the web-server runtime, deploys the application, configures the database connection, and starts the required services.

### Evidence

#### Screenshot 11 — Launch Template details showing that user data exists, including a visible snippet

![paste file](screenshots/week-06-screenshot-47.jpg)

---

#### Screenshot 12 — A running instance created from the template showing that the application responds on port 80 through a local test or browser using its public IP

![paste file](screenshots/week-06-screenshot-48.jpg)

---

# Task 5 — Create an Application Load Balancer (ALB) Across 2 Public Subnets

## Goal

Create an internet-facing ALB across both public subnets with an HTTP listener and a healthy instance target group.

### Evidence

#### Screenshot 13 — ALB details showing two public subnets in two Availability Zones

![paste file](screenshots/week-06-screenshot-49.jpg)

---

#### Screenshot 14 — Target group showing at least one healthy target

![paste file](screenshots/week-06-screenshot-50.jpg)

---

# Task 6 — Create Auto Scaling Group (ASG) in 2 Public Subnets

## Goal

Create an Auto Scaling Group from the Launch Template across both public subnets, with desired capacity 2, minimum 2, and maximum 4, registered to the ALB target group.

### Evidence

#### Screenshot 15 — Auto Scaling Group showing desired, minimum, and maximum capacity and the selected subnet Availability Zones

![paste file](screenshots/week-06-screenshot-51.jpg)

![paste file](screenshots/week-06-screenshot-52.jpg)

---

#### Screenshot 16 — EC2 instances list showing two running instances in different Availability Zones

![paste file](screenshots/week-06-screenshot-53.jpg)

---

# Task 7 — Configure App to Use RDS + Validate Read/Write

## Goal

Confirm the application communicates with the RDS database through the ALB DNS name with at least one read and one write operation.

### Evidence

#### Screenshot 17 — Browser showing the application loaded through the ALB DNS name with the URL visible

![paste file](screenshots/week-06-screenshot-54.jpg)

---

#### Screenshot 18 — Proof of a database write through a UI message or database query output

![paste file](screenshots/week-06-screenshot-55.jpg)

---

# Task 8 — High Availability Tests (Must Do Both)

## Goal

Test A: terminate one web instance and confirm the Auto Scaling Group replaces it automatically without interrupting the ALB.

Test B: simulate an Availability Zone impact (stop, detach, or reduce desired capacity in one AZ) and confirm the application stays available.

### Evidence

#### Screenshot 19 — EC2 showing the terminated instance and the newly launched instance; timestamps are helpful

![paste file](screenshots/week-06-screenshot-56.jpg)

---

#### Screenshot 20 — Target group showing healthy targets after replacement

![paste file](screenshots/week-06-screenshot-57.jpg)

---

#### Screenshot 21 — Evidence that an instance was removed, detached, placed in Standby, or stopped in one Availability Zone

![paste file](screenshots/week-06-screenshot-58.jpg)

---

#### Screenshot 22 — Browser showing that the ALB DNS endpoint still works during the change

![paste file](screenshots/week-06-screenshot-59.jpg)

---

# Task 9 — Architecture and Test-Results Summary

## Goal

Summarize the VPC/subnet layout, the ALB and Auto Scaling Group setup, the private Multi-AZ RDS setup, and the results of both high-availability tests.

### Evidence

#### Screenshot 23 — A simple architecture diagram, which may be hand-drawn, or an AWS console overview showing the components

![paste file](screenshots/week-06-screenshot-60.jpg)

---

### Notes

Summarize the VPC and subnets across the two Availability Zones.

A Virtual Private Cloud (VPC) was created with the 10.0.0.0/16 CIDR block to provide a secure and isolated network for communication between the application's compute, storage, databse, and memory resources.

Within the VPC, subnets were created to divide the network into smaller segments. These subnets can be either public (allowing communication with the internet) or private (restricting communication to resources within the VPC).

The infrastructure was distributed across two Availability Zones to improve high availability and fault tolerance. This design ensures that if one Availability Zone experiences a failure, such as a power outage or natural disaster, the application can continue running in the other Availability Zone.

Summarize the ALB and Auto Scaling Group setup.

An Application Load Balancer (ALB) was configured to receive incoming traffic from users and distribute requests across healthy EC2 instances running the application. This improves traffic management, ensures even load distribution, and increases the application's availability.

An Auto Scaling Group (ASG) was also implemented to automatically monitor the workload on the EC2 instances. When demand increases, the Auto Scaling Group launches additional instances to maintain application performance. When demand decreases, it can terminate unnecessary instances, helping to optimize resource utilization and reduce costs.

Summarize the private Multi-AZ RDS setup.

A private Multi-AZ Amazon RDS database was configured to provide secure, highly available, and reliable data storage for the application. The database was deployed in private subnets, preventing direct access from the public internet and allowing communication only with authorized resources within the VPC.

The Multi-AZ deployment replicated the database across two Availability Zones to improve fault tolerance and ensure business continuity. If the primary database becomes unavailable because of a hardware failure, power outage, or other unexpected event, Amazon RDS automatically fails over to the standby database, minimizing downtime and maintaining application availability.

Summarize the results of both high-availability tests.

The first test verified that the Application Load Balancer and Auto Scaling Group could automatically redirect traffic to healthy EC2 instances when an instance became unavailable. This ensured that users could continue accessing the application without service interruption.

The second test validated the Multi-AZ RDS deployment by confirming that the database could automatically fail over to the standby instance in another Availability Zone when the primary database became unavailable. This reduced downtime and ensured that the application remained connected to the database.

Overall, both tests demonstrated that the infrastructure was resilient, fault-tolerant, and highly available, enabling the application to remain operational during unexpected failures.

---

# LinkedIn Post (Required)

## Goal

Publish a LinkedIn post about the high-availability build, including the ALB URL (or a redacted screenshot), three to five lines on what you built and how you tested high availability, and one proof screenshot.

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://www.linkedin.com/posts/micheal-omotosho-577230199_devops-aws-cloudcomputing-share-7495280081540571136-WfvX/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAC58XisBJdoafJCMJEdvAEQtCZ209939LWg

---

#### Screenshot of LinkedIn post

![paste file](screenshots/week-06-screenshot-61.jpg)

---

# Submission Instructions

- Add all required screenshots in your submission
- Do not expose passwords, connection strings, private keys, or account IDs

---

# Completion Checklist

- [ ] Task 1: VPC, four subnets, IGW, NAT Gateway, and route tables created (Screenshots 1–5)
- [ ] Task 2: Least-privilege ALB, EC2, and RDS security groups created (Screenshots 6–8)
- [ ] Task 3: Private Multi-AZ RDS created (Screenshots 9–10)
- [ ] Task 4: Self-configuring Launch Template created and tested (Screenshots 11–12)
- [ ] Task 5: ALB created across both public subnets (Screenshots 13–14)
- [ ] Task 6: Auto Scaling Group running two instances across two AZs (Screenshots 15–16)
- [ ] Task 7: Application verified through the ALB with a database read and write (Screenshots 17–18)
- [ ] Task 8: Both high-availability tests completed (Screenshots 19–22)
- [ ] Task 9: Architecture and test-results summary completed (Screenshot 23 & Notes)
- [ ] LinkedIn post published and URL submitted
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*