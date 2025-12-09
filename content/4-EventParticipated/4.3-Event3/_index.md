---
title: "Workshop: Data Science on AWS"
date: 2025-10-16
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

### Event Objectives

-   Provide a thorough overview of the lifecycle of a modern Data Science solution, from conceptualization to deployment.
-   Detail the structure of the **AWS Data Science Pipeline**, covering data persistence, transformation, and model deployment stages.
-   Facilitate practical, hands-on learning using a real dataset (IMDb) to construct an application (Sentiment Analysis).
-   Compare the operational and financial advantages and disadvantages of **Cloud-based versus On-premise** infrastructures.

### Speakers

-   **Mr. Van Hoang Kha** – Cloud Solutions Architect, AWS Community Builder
-   **Mr. Bach Doan Vuong** – Cloud DevOps Engineer, AWS Community Builder

### Key Highlights

#### 1. Cloud in Data Science & Pipeline Overview
-   **Cloud Necessity**: Explored the reasons modern data science necessitates the cloud: required agility, scalability, and seamless integration, which static on-premise setups cannot offer.
-   **Core Pipeline Components**:
    -   **Data Lake**: Emphasized **Amazon S3** as the essential, scalable storage layer.
    -   **Serverless ETL**: Introduced **AWS Glue** for automated, serverless data transformation and cleaning.
    -   **ML Environment**: Focused on **Amazon SageMaker** as the unified platform for the entire machine learning model lifecycle. The broader **AWS AI/ML stack**, including specialized AI services, was also reviewed.

#### 2. Practical Demos
-   **Demo 1: Data Preparation**: Illustrated the process of cleaning and transforming the **IMDb dataset** using **AWS Glue**. Techniques for feature engineering were shown, contrasting low-code options like **SageMaker Canvas** with traditional, code-driven methods using libraries like **Numpy/Pandas**.
-   **Demo 2: Model Deployment**: Demonstrated the complete ML workflow for a **Sentiment Analysis model**. This included the "Train, Tune, Deploy" sequence in **SageMaker Studio**. The presenters also covered the flexibility of deploying custom models via the **Bring Your Own Model (BYOM)** approach, compatible with popular frameworks like TensorFlow and PyTorch.

#### 3. Strategic Discussions
-   **Platform Comparison**: A focused discussion on optimizing for both performance and budget. It was stressed that the **elasticity of the cloud** makes it superior for experimenting with computationally heavy workloads compared to the prohibitive capital investment of fixed on-premise hardware.
-   **Follow-up Project**: A suggested mini-project was introduced to help attendees solidify the hands-on knowledge acquired during the workshop.

### Key Takeaways

#### Technical Workflow
-   **Technical Integration**: A functional ML workflow demands **seamless, automated integration** across core services: data lake storage (S3), processing (Glue), and the modeling environment (SageMaker).
-   **Strategic Tooling**: Competence involves knowing when to utilize high-level managed AI services (e.g., Comprehend, Textract) and when to develop custom models within SageMaker to maximize efficiency and ROI.

#### Industry Application
-   **Operational Focus**: The main challenge in moving from classroom concepts to enterprise reality is achieving **automation and scalable operations**.
-   **Cost-Benefit Balance**: Successful data projects require a constant balance between achieving high model precision and managing the associated operational costs.

### Applying to Work

-   **Modernize ETL**: Transition existing local data transformation and ETL scripts to leverage **AWS Glue** for its serverless automation capabilities, especially for processing large-scale data.
-   **Standardize Modeling**: Adopt **SageMaker Studio** as the primary, professional environment for iterating on and deploying experimental models, thereby standardizing the ML lifecycle.
-   **Reinforce Learning**: Commit to executing the suggested follow-up project to solidify the end-to-end knowledge gained from processing and analyzing real-world datasets.

### Event Experience

The **“Data Science on AWS”** workshop effectively served as the vital connection point between theoretical academic study and the practical realities of enterprise deployment.
-   **Real-World Application**: I gained direct exposure to the specific cloud technologies utilized by leading global tech organizations.
-   **Workflow Clarity**: The live demonstration of cleaning the **IMDb data** and deploying a **Sentiment Analysis model** clarified the practical implementation steps for building cloud-based AI systems.
-   **Strategic Insight**: My interaction with the AWS Community Builders provided valuable strategic context for the economic and operational arguments supporting **cloud migration** over the maintenance of traditional on-premise solutions.

#### Some event photos

![Workshop: Data Science on AWS](/images/4-Event/blog_3_0.jpg)
![Workshop: Data Science on AWS](/images/4-Event/blog_3_1.jpg)
![Workshop: Data Science on AWS](/images/4-Event/blog_3_2.jpg)