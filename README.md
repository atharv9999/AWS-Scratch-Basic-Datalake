# Cloud-Native Data Lake on AWS: A Medallion Architecture Project

---

## 1. Project Aim & Executive Summary
The primary objective of this Cloud Computing project was to design, build, and deploy a scalable, cost-effective, and robust data lake on Amazon Web Services (AWS). The project implements the modern Medallion Architecture to process a dataset of **1 million transactional records**, transforming them from raw CSV files into aggregated, query-ready business insights.

This document serves as a comprehensive log of the entire project lifecycle, from the initial architectural design and infrastructure setup to the iterative process of troubleshooting, development, and final analysis. A key focus was to move beyond basic implementation and leverage cloud-native features for:
- Cost optimisation (EC2 Spot Instances)
- Security (IAM Roles, Security Groups)
- Durability (S3 Versioning)
- Automation  

---

## 2. Core Technologies & Rationale

| Technology       | Implementation                | Rationale for Selection |
|------------------|-------------------------------|--------------------------|
| **Cloud Provider** | Amazon Web Services (AWS)     | Mature, comprehensive, and reliable suite of services for data processing, storage, and networking. |
| **Storage**       | Amazon S3                     | Virtually infinite, highly durable, and cost-effective object storage. |
| **Compute**       | EC2 Spot Instance (t3.xlarge) | High-memory (16 GB) and CPU (4 vCPUs) at ~80% discount vs On-Demand. |
| **Processing Engine** | Apache Spark 3.5.2         | Leading distributed data processing engine; in-memory computation suited for 1M records. |
| **Table Format**  | Delta Lake                    | Provides ACID transactions, schema enforcement, time travel, and reliability. |
| **Containerisation** | Docker                     | Lightweight, reproducible environments for Spark and Jupyter. |
| **Development**   | Jupyter Notebook              | Interactive environment for developing, testing, and documenting PySpark ETL scripts. |
| **Orchestration** | Python (PySpark & Boto3)      | PySpark for ETL, Boto3 for managing AWS S3 programmatically. |

---

## 3. Final Data Lake Architecture
The project follows the three-layered **Medallion Architecture**.

- **Landing Zone (S3):** Raw CSV files for sales, customers, products, and locations. Immutable source of truth.  
- **EC2 Spot Instance:** Cost-effective t3.xlarge instance running Spark inside Docker.  
- **Bronze Layer (S3):** Raw data converted into Delta format with metadata (e.g., `ingestion_timestamp`).  
- **Silver Layer (S3):** Cleaned, validated, enriched tables with surrogate keys, correct datatypes, null handling, and joins.  
- **Gold Layer (S3):** Aggregated business-level marts (e.g., `monthly_sales_summary`).  
- **Analytics & Visualisation:** Gold marts analysed in Jupyter with Pandas and visualised with Matplotlib/Seaborn.  

---

## 4. Step-by-Step Project Execution & Learning Journey

### Phase 1: AWS Infrastructure Setup
- **S3 Buckets:** Landing, Bronze, Silver, and Gold.  
- **IAM Role:** `ec2-fa1-datalake-role` with `AmazonS3FullAccess`.  
- **EC2 Selection:**  
  - Initial attempt: `t3.medium` On-Demand → Failed due to memory issues.  
  - Successful approach: `t3.xlarge` Spot Instance (16 GB RAM, cost-efficient).  

### Phase 2: Environment Setup & Troubleshooting
- **Docker Container:** `jupyter/pyspark-notebook/spark:3.5.2` image.  
- **Jupyter Installation Journey:**  
  - Virtual environment attempt failed (dependency conflicts).  
  - Stable solution: Install Jupyter & dependencies directly into container's root environment.  

### Phase 3: The ETL Pipeline
- **Bronze Layer:** Raw CSV → Delta tables (`sales_transactions`, `dim_customers`, etc.).  
- **Silver Layer:** Data cleaning, surrogate keys (`monotonically_increasing_id()`), joins (resolved `AMBIGUOUS_REFERENCE` error with explicit column references).  
- **Gold Layer:** Aggregated marts like `monthly_sales_summary`.  

### Phase 4: Visualisation
- Gold marts → Pandas DataFrame → Charts with Matplotlib/Seaborn.  

---

## 5. Cloud Computing Features Showcase

### Amazon S3 Features
- **Tiered Storage Structure:** Landing, Bronze, Silver, Gold buckets.  
- **S3 Versioning:** Enabled on all buckets.  
- **S3 Lifecycle Policies:** Older data moved to **S3 Standard-IA (30 days)** → **S3 Glacier (90 days)**.  

### Amazon EC2 Features
- **Spot Instances:** ~80% cost reduction.  
- **IAM Roles for EC2:** Secure temporary credentials, avoiding static keys.  

### Networking & Security Features
- **Security Groups:** Restricted inbound traffic (SSH only).  
- **IP Change Hack:** Manual update for changing local IP.  
- **SSH Tunnelling:** Local port forwarding (8888 → 8888) to securely access Jupyter.  

---

## 7. Conclusion
This project demonstrates the **power and flexibility of AWS** for modern data platforms. By combining:
- Scalability of S3  
- Cost-efficiency of EC2 Spot Instances  
- Reliability of Delta Lake  
- Reproducibility of Docker  

We built a **secure, robust, and economically viable end-to-end data lake** that successfully handled 1 million transactional records in a production-ready Medallion Architecture.  
