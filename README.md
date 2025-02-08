# Azure-enterprise-data-platform
## Overview
This project focuses on building a comprehensive data engineering pipeline using **Azure Data Factory, Databricks, and PySpark**. It leverages modern cloud technologies and best practices to ensure efficient data processing and governance.
  
![image](https://github.com/user-attachments/assets/d45accfa-4a30-48e5-9348-5bd3cad26566)
  

### Key Components:
- **Azure Data Factory:** Orchestrates data workflows.
- **Databricks & PySpark:** Enables scalable data transformations.
- **Unity Catalog:** Implements data governance and access control.
- **Delta Lake & Delta Tables:** Provides efficient and reliable data storage.
- **Medallion Architecture:** Structures data pipelines into Bronze, Silver, and Gold layers.
- **Dimensional Data Modeling in Azure Databricks:** Designs and optimizes data warehouses.
- **Slowly Changing Dimensions (SCD):** Implements historical data tracking in Databricks.

## Project Architecture
This project follows the **Medallion Architecture**:
1. **Bronze Layer:** Raw data ingestion from multiple sources.
2. **Silver Layer:** Cleansing and transformation of data.
3. **Gold Layer:** Business-ready, analytics-optimized data.

## Prerequisites
Before starting, ensure you have the following:
- An **Azure Subscription** with access to Azure Data Factory and Databricks.
- Basic understanding of **PySpark, SQL, and Data Engineering concepts**.
- Familiarity with **Azure services** and **data pipeline orchestration**.

## Acknowledgments  
This project was inspired by and developed following Ansh Lamba's excellent tutorial [Azure End-To-End Data Engineering Project (Job Ready) | Azure Data Engineering Bootcamp](https://www.youtube.com/watch?v=6_hXeNg9TJ0&t=20753s). His comprehensive guide on building job-ready data engineering pipelines with Azure services was instrumental in creating this project. I highly recommend checking out his YouTube channel for more in-depth data engineering content.

## Setup Instructions
To get started with the project, follow the setup guide available at:
[Setup Guide](setup.md) 

---

