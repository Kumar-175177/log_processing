# log_processing
Azure-Based Scalable Data Pipeline for High-Volume Log Processing
This repository contains a modern, modular data pipeline that processes high-volume log data and real-time event streams. The pipeline leverages Azure Databricks (Apache Spark), Azure Event Hubs, Azure Data Lake Storage, Azure Data Factory, and Azure Functions to achieve low-latency processing, efficient ETL transformations, robust error handling, and checkpointing. It is designed for scalability and operational efficiency, ensuring data quality and consistency across both streaming and batch layers.

Table of Contents
Overview
Architecture & Data Flow
Project Structure
File Descriptions
Setup & Prerequisites
Usage
Interview Explanation
Learnings & Insights
License
Overview
This project demonstrates an end-to-end data pipeline that:

✅ Ingests and processes log data in real-time using Azure Databricks Structured Streaming reading from Azure Event Hubs.
✅ Transforms and parses log data into a structured format by applying a shared pre-processing module.
✅ Persists the processed data in Parquet format on Azure Data Lake Storage, partitioned by event date for optimized storage and querying.
✅ Executes batch ETL transformations and aggregations with Azure Data Factory, calculating key metrics (e.g., average TTI (Time To Interactive) and TTAR (Time To Articulate Response) per page URL).
✅ Orchestrates the ETL process via Azure Functions, which trigger the Azure Data Factory pipeline based on storage events, handle error notifications, and manage checkpoints using Azure Cosmos DB.

Architecture & Data Flow
Below is an overview diagram of the Azure-based data pipeline flow:

pgsql
Copy
Edit
        +---------------------+
        |  Azure Event Hubs   |  
        | (Log Streams)       |  
        +---------+-----------+  
                  │  
                  ▼  
    +----------------------------+  
    | Azure Databricks (PySpark) |  
    | Structured Streaming Job   |  
    | - Reads from Event Hubs    |  
    | - Parses JSON logs with    |  
    |   a defined schema         |  
    | - Applies shared           |  
    |   pre-processing           |  
    | - Writes Parquet to ADLS   |  
    +-------------+--------------+  
                  │  
                  ▼  
    +----------------------------+  
    |  Azure Data Lake Storage   |  
    | (Raw Parquet Data Storage) |  
    +-------------+--------------+  
                  │  
                  ▼  
    +----------------------------+  
    |  Azure Data Factory (ADF)  |  
    | (ETL Processing &          |  
    | Aggregation)               |  
    | - Reads raw data from ADLS |  
    | - Applies pre-processing   |  
    | - Aggregates key metrics   |  
    | - Loads data into          |  
    |   Azure Synapse Analytics  |  
    +-------------+--------------+  
                  │  
                  ▼  
    +---------------------------------+  
    | Azure Function Orchestrator     |  
    | - Triggers ADF Pipeline         |  
    | - Manages error handling,       |  
    |   retries & checkpointing       |  
    |   (using Cosmos DB)             |  
    +---------------------------------+  
Flow Summary
Real-Time Ingestion
Azure Databricks (Spark Structured Streaming) ingests log events from Azure Event Hubs, converts the incoming JSON data into a structured format using a predefined schema, and applies a shared pre-processing module for data cleaning and enrichment.
The processed data is written as partitioned Parquet files to Azure Data Lake Storage (ADLS), ensuring optimized storage and fast query performance.
Batch ETL Processing
An Azure Data Factory (ADF) pipeline picks up the raw data from Azure Data Lake Storage, re-applies the pre-processing logic, and performs aggregations such as calculating average TTI and TTAR per page URL.
The aggregated results are then loaded into Azure Synapse Analytics (formerly SQL Data Warehouse) for business intelligence and analytics.
Orchestration & Reliability
An Azure Function orchestrates the entire process by dynamically determining the input path from ADLS storage events and triggering the Azure Data Factory pipeline.
It also manages incremental processing through checkpoints stored in Azure Cosmos DB and implements error notifications, ensuring a robust and fault-tolerant pipeline.
Project Structure
pgsql
Copy
Edit
my-azure-data-pipeline/  
│── databricks_streaming_ingest.py   # Databricks Structured Streaming job for real-time ingestion from Event Hubs and writing to ADLS  
│── adf_etl_pipeline.json            # Azure Data Factory pipeline for ETL transformation and loading into Synapse  
│── azure_function_orchestrator.py   # Azure Function for orchestrating ADF pipelines with error handling and checkpoint management  
│── src/common/preprocessing.py      # Shared pre-processing module for data cleaning and enrichment  
│── README.md                        # Project documentation (this file)  
│── requirements.txt                  # (Optional) Python dependencies for local testing  
File Descriptions
1️⃣ databricks_streaming_ingest.py
Uses Azure Databricks (Spark Structured Streaming) to read data from Azure Event Hubs.
The JSON log events are parsed using a predefined schema and processed with a shared pre-processing module.
The resulting data is stored in Azure Data Lake Storage in Parquet format, partitioned by event date.
Checkpointing is used to ensure fault tolerance and exactly-once processing.
2️⃣ adf_etl_pipeline.json
An Azure Data Factory (ADF) pipeline that reads the raw Parquet data from ADLS, applies the same pre-processing logic, and aggregates key metrics such as average TTI and TTAR per page URL.
The aggregated data is then loaded into Azure Synapse Analytics for downstream analytics.
The job includes error handling mechanisms that notify the operations team via an Azure Function.
3️⃣ azure_function_orchestrator.py
An Azure Function that orchestrates the execution of the Azure Data Factory ETL pipeline.
Listens to Azure Storage events to dynamically determine the input data path.
Retrieves the latest processing checkpoint from Cosmos DB and triggers the ADF pipeline accordingly.
Ensures robust orchestration with error handling and automated retries.
4️⃣ src/common/preprocessing.py
Contains the common logic for data cleaning and enrichment, which is used by both the streaming ingestion job and the batch ETL job to ensure consistency in data transformation.
Setup & Prerequisites
Azure Databricks configured with PySpark.
Azure Event Hubs for real-time log streaming.
Azure Data Lake Storage (ADLS) for data storage.
Azure Data Factory with necessary IAM roles and permissions.
Azure Function configured with environment variables.
Azure Cosmos DB for managing checkpoints.
Azure Synapse Analytics for the data warehouse.
Interview Explanation
This pipeline integrates real-time ingestion with batch processing to efficiently manage high-volume log data in an Azure-based architecture.

✅ Azure Databricks (Structured Streaming) reads data from Azure Event Hubs.
✅ Azure Data Lake Storage ensures optimized storage and fast queries.
✅ Azure Data Factory transforms and aggregates data.
✅ Azure Synapse Analytics is used for reporting & analytics.
✅ Azure Functions automate and orchestrate the process.
