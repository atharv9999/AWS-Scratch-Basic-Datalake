Cloud-Native Data Lake on AWS
A Medallion Architecture Project using S3, EC2 Spot Instances, Spark, Docker, and Delta Lake
1. Project Aim & Overview
The primary objective of this Cloud Computing project was to design, build, and deploy a scalable, cost-effective, and robust data lake on Amazon Web Services (AWS). The project implements the modern Medallion Architecture (Bronze, Silver, Gold layers) to process a large dataset of 10 million transactional records, transforming them from raw CSV files into aggregated business insights ready for visualization.

A key focus was to leverage cloud-native features for cost optimization, security, durability, and automation, demonstrating a practical understanding of building data systems in a real-world cloud environment. This document outlines the entire journey, from initial infrastructure setup and troubleshooting to the final ETL pipeline and analysis.

2. Core Technologies Used
Cloud Provider: Amazon Web Services (AWS)

Why? AWS offers a mature, comprehensive suite of services for data processing, storage, and networking, making it an ideal platform for this project.

Storage: Amazon S3 (Simple Storage Service)

Why? S3 provides virtually infinite, durable, and cost-effective object storage, which is the standard foundation for modern data lakes.

Compute: Amazon EC2 (Elastic Compute Cloud) - Spot Instance

Why? EC2 provides flexible virtual server capacity. Using a Spot Instance allowed us to access high-memory and high-CPU resources at a ~80% discount.

Processing Engine: Apache Spark

Why? Spark is the leading open-source engine for large-scale distributed data processing, capable of handling the performance demands of our 10 million row dataset.

Table Format: Delta Lake

Why? Delta Lake adds reliability to our S3 data lake by providing ACID transactions, schema enforcement, and time travel capabilities, preventing data corruption.

Containerization: Docker

Why? Docker allowed us to create a lightweight, isolated, and reproducible environment for Spark and Jupyter, ensuring our setup works consistently anywhere.

Development Environment: Jupyter Notebook

Why? Jupyter provides an interactive environment perfect for developing, testing, and documenting our PySpark ETL scripts step-by-step.

Orchestration: Python (PySpark) & Boto3

Why? PySpark is the Python API for Spark. Boto3, the AWS SDK for Python, was used to programmatically manage S3 objects.

3. Final Architecture
The project follows a classic data lake architecture, where data flows through progressively more refined layers.

Landing Zone (S3): Raw, unaltered CSV files for sales, customers, products, and locations are uploaded here. This zone acts as the permanent, immutable source of truth.

EC2 Spot Instance: A cost-effective t3.xlarge instance serves as the compute node. It runs a Docker container where all Spark processing occurs.

Bronze Layer (S3): Data from the landing zone is read by Spark, minimally processed (e.g., adding an ingestion timestamp), and stored in the efficient Delta Lake format. The structure mirrors the source.

Silver Layer (S3): The Bronze tables are cleaned, validated, and transformed. Here, we generate numerical surrogate keys, correct data types, and join dimension tables with the sales table to create a trusted, enriched set of master tables.

Gold Layer (S3): The clean Silver tables are aggregated into business-level "marts". These are smaller, denormalized tables optimized for specific reporting and analytics queries.

Analytics & Visualization: The final Gold marts are read into a Jupyter Notebook using Pandas for analysis and visualization with Matplotlib/Seaborn.

4. Step-by-Step Project Execution
The project was executed in a series of logical phases, from infrastructure setup to final analysis.

Phase 1: AWS Infrastructure Setup
S3 Buckets: Four distinct S3 buckets were created to logically separate the data layers: your-landing-bucket, your-bronze-bucket, your-silver-bucket, and your-gold-bucket.

IAM Role: An IAM Role (ec2-fa1-datalake-role) with AmazonS3FullAccess was created and attached to the EC2 instance. This granted Spark secure, temporary access to S3 without needing to store any secret keys in the code.

EC2 Instance Selection:

Initial Failed Approach: The project was initially started on a t3.medium On-Demand instance. However, this proved to be underpowered for processing 10 million rows. The Spark session failed to start due to JAVA_GATEWAY_EXITED errors, which were traced back to memory exhaustion.

Successful Approach: We pivoted to a t3.xlarge EC2 Spot Instance. This provided 4x the memory (16 GB) at a fraction of the cost, successfully resolving the memory issues and demonstrating a key cloud cost-optimization strategy.

Phase 2: Environment Setup & Troubleshooting
Docker Container: A Docker container using the bitnami/spark:3.5.2 image was launched on the EC2 instance.

Jupyter Installation Journey: Setting up the interactive environment involved significant troubleshooting, which is a realistic part of cloud development:

Initial Plan (Virtual Environment): The first attempt involved creating a Python virtual environment inside the container. This led to a series of dependency and path conflicts (ModuleNotFoundError: No module named 'pyspark').

The Stable Solution (Root Install): The most stable approach was adopted:

Start a fresh Docker container.

Enter the container as the root user to gain system-wide permissions.

Install Jupyter and its dependencies (py4j, boto3) directly into the container's main Python environment using pip3.

This resolved all path issues and allowed Jupyter to seamlessly discover the pre-installed PySpark.

Phase 3: The ETL Pipeline
The core logic was executed in a series of Jupyter Notebooks, one for each layer, to maintain organization.

Bronze Layer Creation: Raw CSVs were read from the landing zone and converted into partitioned Delta tables (sales_transactions, dim_customers, etc.).

Silver Layer Creation: The Bronze tables were cleaned, and surrogate keys were generated for the dimension tables using monotonically_increasing_id(). The sales table was then enriched by joining with the dimensions to create a trusted master sales table. An AMBIGUOUS_REFERENCE error during the join was resolved by explicitly referencing columns from their source DataFrame.

Gold Layer Creation: The clean Silver tables were used to create a final monthly_sales_summary mart.

Phase 4: Visualization
The final Gold mart was loaded into a Pandas DataFrame (.toPandas()), and matplotlib/seaborn were used to create bar and line charts.

5. Cloud Computing Features Showcase
This project heavily utilized the unique features of AWS to build an efficient and secure system.

Amazon S3 Features
Tiered Storage Structure: The use of Landing, Bronze, Silver, and Gold buckets provides clear data lineage, auditability, and separation of concerns.

S3 Versioning: Enabled on all buckets, this provided a critical safety net against accidental data deletion or corruption. It ensures that any object can be restored, guaranteeing data durability.

S3 Lifecycle Policies: A lifecycle rule was configured to automatically transition older, less-frequently-accessed data in the Landing and Bronze buckets to cheaper storage tiers (S3 Standard-IA and S3 Glacier), demonstrating a powerful cost-optimization strategy.

Amazon EC2 Features
Spot Instances: Using a t3.xlarge Spot Instance was a cornerstone of the project's cost-efficiency. It provided the necessary compute power at an ~80% cost reduction compared to On-Demand pricing, a best practice for fault-tolerant workloads like Spark jobs.

IAM Roles for EC2: This was a key security feature. By attaching an IAM role, we granted the instance temporary, automatically-rotated credentials, eliminating the significant security risk of managing or embedding static access keys in our code.

Networking & Security Features
Security Groups: The EC2 instance was protected by a Security Group acting as a virtual firewall. Inbound rules were configured to only allow SSH (port 22) traffic, strictly limiting access and adhering to the principle of least privilege.

The "IP Change Hack": For development, SSH access was restricted to a specific IP address. We documented the need to manually update the Security Group's inbound rule whenever our local public IP changed, demonstrating hands-on security management.

SSH Tunneling: This was a critical networking feature. Instead of exposing the Jupyter port (8889) to the public internet, we created an encrypted private channel via SSH. This forwarded traffic from a local port (9999) to the Jupyter server inside the Docker container, ensuring the entire development session was secure.

6. How to Add Screenshots to this File
This document is a Markdown (.md) file. You can embed images using the following syntax:

![Alt text for the image](URL_to_the_image)

Recommended Workflow for this Project:

Create a Public S3 Bucket: Create a new, separate S3 bucket (e.g., fa1-project-screenshots). In the permissions, disable "Block all public access" and add a bucket policy that allows public read access.

Upload Screenshots: Take your screenshots and upload them to this bucket.

Get the Object URL: Click on an uploaded image and copy its "Object URL".

Embed in Markdown: Paste the URL into the syntax above. For example:
![My Architecture Diagram](https://fa1-project-screenshots.s3.ap-south-1.amazonaws.com/architecture.png)

7. Conclusion
This project successfully demonstrates the power and flexibility of AWS for building modern data platforms. By combining the infinite scalability of S3, the cost-efficiency of EC2 Spot Instances, the reliability of Delta Lake, and the reproducibility of Docker, we were able to build a complete, end-to-end data lake that is secure, robust, and economically viable.
