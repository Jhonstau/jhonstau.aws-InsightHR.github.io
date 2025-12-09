---
title: "Translated Blogs"
date: ""
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

###  [Blog 1 - Implementing granular failover in multi-Region Amazon EKS](3.1-Blog1/)
This blog introduces how to implement granular, active-passive failover for applications deployed on Amazon EKS (Elastic Kubernetes Service) across multiple AWS Regions. You will learn why multi-Region disaster recovery is crucial for achieving high availability and business continuity for mission-critical containerized workloads, how granular failover allows for selective traffic shifting at the service level (instead of failing over the entire cluster), and the role of Amazon Route 53 and AWS Gateway Load Balancer (GWLB) in this high-resilience architecture. The article also guides you through the steps to set up the EKS clusters in two regions, configure Route 53 health checks and failover routing, and implement the necessary network infrastructure to ensure seamless traffic management and resilience.

###  [Blog 2 - Tailor Amazon SageMaker Unified Studio project environments to your needs using custom blueprints](3.2-Blog2/)
This blog introduces how to tailor Amazon SageMaker Studio project environments using custom project templates, or blueprints, for machine learning (ML) workflows. You will learn why custom templates are essential for standardizing environments, integrating specific resources (like custom Amazon ECR images or specialized infrastructure), and ensuring compliance across MLOps projects, how these blueprints are built using AWS Service Catalog, and the role of Amazon CodeCommit and AWS CodePipeline in managing the lifecycle of these custom templates. The article also guides you through the steps to create and share your own custom project blueprints, manage resource provisioning, and provide data scientists with a pre-configured, consistent environment tailored to their exact needs.

###  [Blog 3 - Enhance TLS inspection with SNI session holding in AWS Network Firewall](3.3-Blog3/)
This blog introduces how to enhance TLS inspection and security posture using the new Server Name Indication (SNI) session holding feature in AWS Network Firewall. You will learn why Network Firewall is a critical layer for deep packet inspection and network segmentation in your VPCs, how the SNI session holding feature prevents security policy bypass by ensuring that subsequent connections from the same client reuse the initial TLS session, and the importance of this feature for security compliance and robust threat prevention. The article also guides you through the steps to configure and deploy Network Firewall with SNI session holding, demonstrate how it improves the accuracy of TLS inspection, and ensure consistent application of security rules.