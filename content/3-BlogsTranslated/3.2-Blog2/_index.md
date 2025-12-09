---
title: "Blog 2"
date: ""
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Tailor Amazon SageMaker Unified Studio project environments to your needs using custom blueprints

by Aditya Challa and Khushbu Agarwal on 17 SEP 2025 in Amazon SageMaker Unified Studio, Announcements, Intermediate (200), Technical How-to |Permalink | Comments | Share

Amazon SageMaker Unified Studio is a single data and AI development environment that brings together data preparation, analytics, machine learning (ML), and generative AI development in one place. By unifying these workflows, it saves teams from managing multiple tools and makes it straightforward for data scientists, analysts, and developers to build, train, and deploy ML models and AI applications while collaborating seamlessly.

In SageMaker Unified Studio, a project is a boundary where you can collaborate with other users to work on a business use case. A blueprint defines what AWS tools and services members of a project can use as they work with their data. Blueprints are defined by an administrator and are powered by AWS CloudFormation. Instead of manually piecing together project structures or workflow configurations, teams can rapidly spin up secure, compliant, and consistent analytics and AI environments. This streamlined approach significantly reduces setup time and provides standardized workspaces across the organization. Out of the box, SageMaker Unified Studio comes with several default blueprints.

We recently launched the custom blueprints feature in SageMaker Unified Studio. Organizations can now incorporate their specific dependencies, security controls using their own managed AWS Identity and Access Management (IAM) policies, and best practices, making it straightforward for them to align with internal standards. Because they’re defined through infrastructure as code (IaC), blueprints are straightforward to version control, share across teams, and evolve over time. This speeds up onboarding and keeps projects consistent and governed, no matter how big or distributed your data organization becomes.

For enterprises, this means more time focusing on insights, models, and innovation. The custom blueprints feature is designed to help teams move faster and stay consistent while maintaining their organization’s security controls and best practices. In this post, we show how to get started with custom blueprints in SageMaker Unified Studio.

## Solution overview

We provide a CloudFormation template to implement a custom blueprint in SageMaker Unified Studio. The template deploys the following resources in the project environment:

AWS Glue database
Amazon Redshift Serverless namespace and workgroup
AWS Lake Formation permissions for the newly created project to access the AWS Glue database
Custom managed policies for AWS Glue and Amazon Redshift

## Prerequisites

The post assumes you have a preexisting SageMaker Unified Studio domain. If you don’t have one, refer to Create a Amazon SageMaker Unified Studio domain – quick setup for instructions to create one.

## Define reserved environment parameters

The CloudFormation template uses parameters that are reserved to your SageMaker environment, such as datazoneEnvironmentEnvironmentId, datazoneEnvironmentProjectId, s3BucketArn, and privateSubnets. These parameters are automatically populated by SageMaker when creating the project. The parameters also help in retrieving other environment variables, such as SecurityGroupIds, as shown in the following snippets.

The following code illustrates defining reserved environment parameters:
```bash

    "Parameters": {
        "datazoneEnvironmentEnvironmentId": {
            "Type": "String",
            "Description": "EnvironmentId for which the resource will be created for."
        },
        "datazoneEnvironmentProjectId": {
            "Type": "String",
            "Description": "DZ projectId for which project the resource will be created for."
        },
        "s3BucketArn": {
            "Type": "String",
            "Description": "Project S3 Bucket ARN"
        },
        "privateSubnets": {
            "Type": "String",
            "Description": "Project Private Subnets"
        }
    }

The following code illustrates using reserved environment parameters to import other necessary values:

    "SecurityGroupIds": [
        {
            "Fn::ImportValue": {
                "Fn::Join": [
                    "",
                    [
                        "securityGroup-",
                        { "Ref": "datazoneEnvironmentProjectId" },
                        "-dev"
                    ]
                ]
            }
        }
    ]
```

## Attach custom IAM policies to project role

By default, SageMaker Unified Studio creates a project role and attaches several managed policies to the role. These managed policies are defined in the tooling blueprint. With custom blueprints, you can configure and attach your own IAM policies, in addition to the default policies, to the project role. To do this, include the IAM policies in your CloudFormation template and use the Export feature in the Outputs section, as shown in the following code. SageMaker Unified Studio gathers the policy information and adds it to the project role.
```bash
    "GlueAccessManagedPolicy": {
        "Description": "ARN of the created managed policy",
        "Value": { "Ref": "GlueAccessManagedPolicy" },
        "Export": {
            "Name": {
                "Fn::Sub": "datazone-managed-policy-glue-${glueDbName}-${datazoneEnvironmentEnvironmentId}"
            }
        }
    },
    "RedshiftAccessManagedPolicy": {
        "Description": "ARN of the created Redshift managed policy",
        "Value": { "Ref": "RedshiftAccessManagedPolicy" },
        "Export": {
            "Name": {
                "Fn::Sub": "datazone-managed-policy-redshift-${redshiftWorkgroupName}-${datazoneEnvironmentEnvironmentId}"
            }
        }
    }
```
## Create custom blueprint

Complete the following steps to create a custom blueprint using the CloudFormation template:

1. On the Amazon SageMaker console, open the domain where you want to create a custom blueprint.
2. On the Blueprints tab, choose Create.
3. Under Name and description, enter a name and optional description.
4. Under Upload CloudFormation template, select Upload a template file and upload the provided template.
5. Choose Next.
SageMaker will automatically detect the reserved parameters defined in the template, as shown in the following screenshot.
![Blog](/images/3-Blog/blog_2_0.png)
6. For Editable parameters, edit the Value column if necessary, and specify whether the values can be editable at the time of project creation.
7. Choose Next.
As shown in the following screenshot, the reserved parameters described earlier are not shown on this page.
![Blog](/images/3-Blog/blog_2_1.png)
8. Select Enable blueprint.
9. Choose the provisioning role to be used by SageMaker to provision the environment resources.
10. Choose the domain units authorized to use the blueprint.
11. Choose Next.
![Blog](/images/3-Blog/blog_2_2.png)
12. Review the blueprint information and choose Create blueprint.

## Create project profile
Complete the following steps to create a custom project profile that includes the custom blueprint created in the previous section:

1. On the SageMaker console, open your domain.
2. On the Project profiles tab, choose Create.
![Blog](/images/3-Blog/blog_2_3.png)
3. Enter the project profile name and optional description.
4. Select Custom create.
5. Choose the blueprints to be included in the project profile, including the custom blueprint you created in the previous section.
![Blog](/images/3-Blog/blog_2_4.png)
6. Choose the account and AWS Region to be used.
7. Choose the authorized users.
8. Select Enable project profile on creation.
9. Choose Create project profile.
## Create project
Complete the following steps to create a new project that is based on the custom project profile and custom blueprint created in the previous sections:

1. In the SageMaker Unified Studio environment, choose Create project.
2. Enter a project name and optional description.
3. For Project profile, choose the profile created in the previous section.
4. Choose Continue.
![Blog](/images/3-Blog/blog_2_5.png)
5. On the Customize blueprint parameters page, review the parameters, modify as necessary, and choose Continue.
![Blog](/images/3-Blog/blog_2_6.png)
6. Review your selections and choose Create project.
![Blog](/images/3-Blog/blog_2_7.png)
SageMaker Unified Studio will create the project environments with the resources defined in your custom blueprint.
![Blog](/images/3-Blog/blog_2_8.png)
It will also attach the custom IAM policies defined and add them to the project role, as shown in the following screenshot.
![Blog](/images/3-Blog/blog_2_9.png)

## Clean up

To avoid incurring additional costs, complete the following steps:

1. Delete the project you created in SageMaker Unified Studio.
2. Delete the custom project profile and custom blueprint you created.
3. Delete the CloudFormation template.

## Conclusion

In this post, we discussed custom blueprints, a new option during administrator setup in SageMaker Unified Studio. We showed how to create new custom blueprints and create custom project profiles that include the newly created custom blueprints. We also demonstrated how to create projects that implement custom blueprints.

Custom blueprints in SageMaker Unified Studio are intended to streamline and standardize data, analytics and AI workflows. By helping organizations create templated environments with preconfigured resources, security controls, and best practices, custom blueprints can reduce setup time while providing consistency and compliance across projects.

Organizations can now enforce their specific security standards and access controls at the project level using the ability to incorporate custom IAM policies directly into these blueprints. This granular control over permissions helps organizations create projects that adhere to corporate security policies right from inception. Custom blueprints can help you scale analytics and AI/ML operations securely, by including tooling designed to version control these templates, share them across teams, and automatically apply custom IAM policies.

To learn more about custom blueprints in SageMaker Unified Studio, refer to Custom blueprints.

## About the Authors

### Aditya Challa
![Blog](/images/3-Blog/blog_2_10.jpeg)
Aditya is a Senior Solutions Architect at Amazon Web Services with over a decade of experience architecting and implementing cloud-based solutions. Specializing in data, analytics, and machine learning, he has helped numerous enterprises transform their data infrastructure and build scalable AI/ML solutions on AWS. As a trusted advisor to clients across industries, Aditya is passionate about helping organizations navigate their cloud transformation journeys and unlock business value through data-driven innovation. Beyond his technical pursuits, Aditya is an avid traveler and history enthusiast who finds inspiration in engineering marvels across cultures and eras. He maintains a growth mindset and believes in continuous learning—a philosophy that drives both his professional development and his approach to helping customers achieve their technology goals through AWS.

### Khushbu Agarwal
![Blog](/images/3-Blog/blog_2_11.jpeg)
Khushbu is a Senior Product Manager at AWS. She is focused on improving the customer onboarding and platform capabilities within Amazon SageMaker Unified Studio and making it the best-in-class choice for AWS analytics, generative AI, and ML services.
