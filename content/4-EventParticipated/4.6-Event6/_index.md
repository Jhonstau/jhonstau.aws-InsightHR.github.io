---
title: "AWS Security Specialty Workshop"
date: 2025-11-29
weight: 6
chapter: false
pre: " <b> 4.6. </b> "
---

### Event Objectives

-   Develop a thorough understanding of the **Security Pillar** within the AWS Well-Architected Framework.
-   Gain mastery over the five security domains: **Identity**, **Detection**, **Infrastructure Protection**, **Data Protection**, and **Incident Response**.
-   Receive an update on the most prevalent security threats facing the cloud market locally.
-   Participate in practical exercises focusing on auditing access permissions (**IAM**) and creating structured **Incident Response (IR) Playbooks**.

### Speakers

-   **AWS Security Experts Team**

### Key Highlights

#### Foundation & Identity (Pillar 1)
-   **Core Security Principles**: Reaffirmed the critical importance of applying **Least Privilege**, **Zero Trust**, and **Defense in Depth** methodologies.
-   **Modern Identity Management**: Emphasized the shift from using long-term credentials (**IAM Users**) to utilizing **IAM Roles** and centralizing identity via **AWS Identity Center (SSO)**.
-   **Governing Permissions**: Learned to use **Service Control Policies (SCPs)** and **Permission Boundaries** to define and restrict maximum permission scopes in complex, multi-account environments.
-   **Practical Validation**: Conducted a brief exercise on **Validating IAM Policies** to proactively identify and resolve potential security flaws.

#### Detection & Infrastructure (Pillar 2 & 3)
-   **Continuous Threat Detection**: Focused on enabling key services like **CloudTrail** (for organizational-level auditing), **GuardDuty** (for intelligent threat detection), and **Security Hub** (for consolidated compliance monitoring).
-   **Comprehensive Logging**: Discussed best practices for capturing logs at every layer, including **VPC Flow Logs** (network traffic), **ALB logs** (application load balancing), and **S3 access logs** (data storage).
-   **Network Perimeter Defense**: Strategies for **VPC segmentation** combined with **Security Groups** and **NACLs**. Integrated edge protection utilizing **WAF, Shield, and Network Firewall** was also covered.

#### Data Protection & Incident Response (Pillar 4 & 5)
-   **Encryption Strategy**: Detailed the necessary steps for encrypting data both **in-transit and at-rest** across core services (S3, EBS, RDS) using **KMS** (Key Management Service).
-   **Credential Hygiene**: Stressed the need to eliminate hard-coded credentials by leveraging **Secrets Manager** and **Parameter Store**, including automated rotation mechanisms.
-   **Automated IR**: The process of building structured **IR Playbooks** for common threats (e.g., compromised credentials, malware) and automating the response and isolation steps using services like Lambda and Step Functions.

### Key Takeaways

#### The Zero Trust Model
-   **Identity as the Perimeter**: In a cloud-native architecture, the security boundary shifts from the network edge to **Identity**, making it the most critical defense layer.
-   The principle is clear: never trust any user or resource by default; always verify all access requests.

#### Automation for Scale
-   Manual security practices are unsustainable at cloud scale and speed. Implementing **Detection-as-Code** and **Auto-remediation** is mandatory to reduce the window of exposure to threats.

### Applying to Work

-   **IAM Policy Audit**: Initiate a full audit of all **IAM Users** and their permissions, deprecating old keys and moving toward application-based **IAM Roles**.
-   **Enable Threat Intelligence**: Activate **GuardDuty** globally across all organizational accounts/regions to benefit from continuous threat and anomaly detection.
-   **Secrets Management Adoption**: Refactor applications to retrieve database credentials from **Secrets Manager** instead of storing them in configuration files.
-   **IR Readiness**: Begin developing a formal **Incident Response Playbook** specifically for a "Compromised IAM Key" scenario, including a plan for rehearsing the automated response.

### Event Experience

This workshop was a deep, technical dive into cloud security, providing the comprehensive knowledge required of a modern Cloud Engineer.

#### Structured Learning
-   Organizing the curriculum around the **5 Pillars** successfully transformed scattered security concepts into a single, cohesive, and standard framework.
-   The insight provided on **Top threats** specific to the local market context was immediately practical for risk mitigation.

#### Practical Skill Development
-   Demos, particularly those using the **IAM Policy Validator** and **Access Analyzer**, provided immediately useful skills for daily policy debugging and security hardening.
-   The focus on **Incident Response automation** underscored the shift from mere monitoring to proactive, automated system defense.

#### Some event photos

![AWS DevOps & Modern Operations](/images/4-Event/blog_6_0.jpg)
![AWS DevOps & Modern Operations](/images/4-Event/blog_6_1.jpg)
![AWS DevOps & Modern Operations](/images/4-Event/blog_6_2.jpg)