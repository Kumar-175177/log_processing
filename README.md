# Scalable Data Pipeline for High-Volume Log Processing

This repository contains a modern, modular data pipeline that processes high-volume log data and real-time event streams. The pipeline leverages Apache Spark, Kafka, Azure Data Factory, and Azure Functions to achieve low-latency processing, efficient ETL transformations, robust error handling, and checkpointing. It is designed for scalability and operational efficiency, ensuring data quality and consistency across both streaming and batch layers.

## Table of Contents
- Overview
- Architecture & Data Flow
- Project Structure
- File Descriptions
- Setup & Prerequisites
- Usage
- Interview Explanation
- Learnings & Insights
- License

## Overview
This project demonstrates an end-to-end data pipeline that:

- Ingests and processes log data in real time using Spark Structured Streaming reading from Kafka.
- Transforms and parses log data into a structured format by applying a shared pre-processing module.
- Persists the processed data in Parquet format on Azure Data Lake Storage (ADLS), partitioned by event date for optimized storage and querying.
- Executes batch ETL transformations and aggregations with Azure Data Factory (ADF) to calculate key metrics (e.g., average TTI and TTAR per page URL).
- Orchestrates the ETL process via an Azure Function, which triggers the ADF job based on ADLS events, handles error notifications, and manages checkpoints using Azure Cosmos DB.

## Architecture & Data Flow
Below is an overview diagram of the data pipeline flow:

```
       +----------------+
       |  Kafka Topic   |
       | (Log Streams)  |
       +-------+--------+
               |
               ▼
  +---------------------------+
  | Apache Spark (PySpark)    |
  | Structured Streaming Job  |
  | - Reads from Kafka        |
  | - Parses JSON logs with   |
  |   a defined schema        |
  | - Applies shared          |
  |   pre-processing          |
  | - Writes Parquet to ADLS  |
  +--------------+------------+
                 |
                 ▼
  +---------------------------+
  |    Azure Data Lake        |
  | (Raw Parquet Data Storage)|
  +--------------+------------+
                 |
                 ▼
  +---------------------------+
  |  Azure Data Factory (ADF) |
  | (ETL Processing &         |
  |  Aggregation)             |
  | - Reads raw data from ADLS|
  | - Applies pre-processing  |
  | - Aggregates key metrics  |
  | - Loads data into         |
  |   Azure Synapse           |
  +--------------+------------+
                 |
                 ▼
  +------------------------------+
  |  Azure Function Orchestrator |
  | - Triggers ADF ETL Job       |
  | - Manages error handling,    |
  |   retries & checkpointing    |
  |   (using Cosmos DB)          |
  +------------------------------+
```

### Flow Summary
#### Real-Time Ingestion
Spark Structured Streaming ingests log events from Kafka, converts the incoming JSON data into a structured format using a predefined schema, and applies a shared pre-processing module for data cleaning and enrichment. The processed data is written as partitioned Parquet files to Azure Data Lake Storage (ADLS), ensuring optimized storage and fast query performance.

#### Batch ETL Processing
An Azure Data Factory (ADF) pipeline picks up the raw data from ADLS, re-applies the pre-processing logic to maintain consistency, and performs aggregations such as calculating average TTI (Time To Interactive) and TTAR (Time To Articulate Response) per page URL. The aggregated results are then loaded into Azure Synapse Analytics for business intelligence and analytics.

#### Orchestration & Reliability
An Azure Function orchestrates the overall process by dynamically determining the input path from ADLS events and triggering the ADF pipeline. It also manages incremental processing through checkpoints stored in Azure Cosmos DB and implements error notifications, ensuring a robust and fault-tolerant pipeline.

## Project Structure
```
my-data-pipeline/
├── spark_streaming_ingest.py         # Spark Structured Streaming job for real-time ingestion from Kafka and writing to ADLS
├── adf_etl_pipeline.py               # Azure Data Factory pipeline script for data transformation, aggregation, and loading into Synapse
├── azure_function_orchestrator.py    # Azure Function for orchestrating ADF ETL jobs with error handling and checkpoint management
├── src/common/preprocessing.py       # Shared pre-processing module for data cleaning and enrichment
├── README.md                         # Project documentation (this file)
├── requirements.txt                   # (Optional) Python dependencies for local testing
```

## File Descriptions
### spark_streaming_ingest.py
Uses Spark Structured Streaming to read data from a Kafka topic. The JSON log events are parsed using a predefined schema and processed with a shared pre-processing module. The resulting data is stored in Azure Data Lake Storage in Parquet format with partitioning by event date. Checkpointing is used to ensure fault tolerance and exactly-once processing.

### adf_etl_pipeline.py
An Azure Data Factory (ADF) script that reads the raw Parquet data from ADLS, applies the same pre-processing logic, and aggregates key metrics such as average TTI and TTAR per page URL. The aggregated data is then loaded into Azure Synapse Analytics for downstream analytics. The pipeline includes error handling mechanisms that notify the operations team via an auxiliary Azure Function.

### azure_function_orchestrator.py
An Azure Function that orchestrates the execution of the Azure Data Factory (ADF) pipeline. It listens to ADLS events to dynamically determine the input data path, retrieves the latest processing checkpoint from Cosmos DB, and triggers the ADF job accordingly. This function ensures robust orchestration with error handling and automated retries.

### src/common/preprocessing.py
Contains the common logic for data cleaning and enrichment, which is used by both the streaming ingestion job and the batch ETL job to ensure consistency in data transformation across the pipeline.

## Setup & Prerequisites
- Apache Spark configured with PySpark.
- A Kafka cluster with a topic for log data.
- An Azure Data Lake Storage (ADLS) account for data storage.
- Azure Data Factory (ADF) with necessary IAM roles and permissions.
- Azure Function configured with environment variables (e.g., for ADF pipeline name, checkpoint table, and default input path).
- An Azure Cosmos DB table for managing checkpoints (if using incremental processing).
- Azure Synapse Analytics for the data warehouse.

## Usage
1. **Deploy Spark Streaming Job**: Configure your Kafka parameters and ADLS details in `spark_streaming_ingest.py` and run the job to start ingesting real-time log data.
2. **Deploy Azure Data Factory Pipeline**: Upload `adf_etl_pipeline.py` to ADF, set the pipeline parameters (input path, Synapse URL, etc.), and schedule or trigger the job as needed.
3. **Configure Azure Function Orchestrator**: Deploy `azure_function_orchestrator.py` to Azure Functions, set the required environment variables, and configure it to trigger based on ADLS events for automated orchestration.

## Interview Explanation
This pipeline integrates real-time ingestion with batch processing using Azure services. Spark Structured Streaming reads data from Kafka and applies schema validation and pre-processing. The processed data is stored in ADLS. Later, ADF reads this data, applies the same pre-processing, aggregates key metrics, and loads the results into Synapse. An Azure Function orchestrates the workflow dynamically, ensuring scalability and fault tolerance.

## Learnings & Insights
- **Modular Design**: Ensures reusability of the pre-processing logic.
- **Fault Tolerance**: Checkpointing and Cosmos DB enable incremental processing.
- **Scalability**: Azure-managed services allow seamless scaling.
- **Operational Efficiency**: Automated orchestration reduces manual intervention.

