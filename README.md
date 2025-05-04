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
In this data pipeline, we integrate real-time ingestion with batch processing to efficiently manage high-volume log data. We use Spark Structured Streaming to read data from Kafka, where a strict schema ensures data consistency. The incoming JSON logs are parsed and enriched using a shared pre-processing module, which is used across both the streaming and batch layers. Processed data is stored as partitioned Parquet files in Azure Data Lake Storage (ADLS) for optimized storage. Later, an Azure Data Factory (ADF) pipeline reads this raw data, re-applies the pre-processing logic for consistency, and aggregates key performance metrics like average TTI and TTAR per page URL. The aggregated results are loaded into Azure Synapse Analytics for analytics. An Azure Function orchestrates the entire process by dynamically determining the input path from ADLS events, retrieving checkpoints from Azure Cosmos DB for incremental processing, and triggering the ADF pipeline. This design demonstrates a robust, scalable, and fault-tolerant approach to data processing and highlights best practices in modern data engineering.

 Project Title: Scalable Data Pipeline for High-Volume Log Processing in Azure

🔹 Business Context:
At Sony, we dealt with camera and PlayStation sales data, including customer interactions, website logs, and purchase details. The goal was to process and analyze high-volume log data from various sources to gain insights into sales trends and customer behavior.

🔹 Key Challenges:
Handling both real-time and batch data efficiently.
Ensuring schema consistency for log data.
Managing large-scale data processing in a cost-efficient manner.
Providing aggregated business metrics for sales and performance analysis.

🔹 Solution Approach:
1️⃣ Real-Time Ingestion with Kafka & Spark Structured Streaming
Kafka captures log data from multiple sources (e.g., website events, transactions).
Spark Structured Streaming reads these logs in real-time, ensuring schema consistency.

2️⃣ Pre-processing & Storage in Azure Data Lake Storage (ADLS)
Logs are parsed, validated, and enriched in a shared module (used for both streaming & batch).
Processed data is stored in partitioned Parquet format for efficiency.

3️⃣ Batch Processing with Azure Data Factory (ADF)
ADF pipelines read the raw Parquet data, apply aggregations (e.g., avg TTI, TTAR per page URL).
The final processed data is loaded into Azure Synapse Analytics for reporting & analytics.

4️⃣ Orchestration with Azure Functions & Cosmos DB
An Azure Function listens for new files in ADLS, triggering incremental processing.
Checkpoints stored in Cosmos DB ensure no data duplication or loss.

Business Context
At Sony, we handled large-scale camera and PlayStation sales data, including:

Website interaction logs (clickstreams)

Customer purchase records

Regional sales trends

Objective: Efficiently process and analyze high-volume log data to derive business insights on:

Website performance (e.g., response times)

User behavior

Product performance across regions

🚧 Key Challenges
Handling dual-mode data ingestion (real-time + batch)

Ensuring schema consistency for semi-structured JSON logs

Managing large-scale data in a cost-efficient and fault-tolerant way

Generating business-friendly aggregated metrics (e.g., avg TTI/TTAR per page)

🏗️ High-Level Architecture
css
Copy
Edit
[Log Sources] → [Kafka] → [Spark Structured Streaming] → [ADLS - Bronze]
                                                ↓
                                      [Azure Function Trigger]
                                                ↓
                                  [ADF Batch Job (Spark Pool)]
                                                ↓
                           [Aggregations & Enrichment (Silver & Gold)]
                                                ↓
                         [Azure Synapse Analytics] & [Power BI Dashboards]
🧪 Data Schema
1. JSON Log Structure
json
Copy
Edit
{
  "session_id": "abc123",
  "page_url": "/product/playstation5",
  "timestamp": "2024-04-20T14:35:21Z",
  "TTI": 1500,
  "TTAR": 800,
  "user_actions": [
    {"action": "click", "target": "buy_now", "time": 1650},
    {"action": "scroll", "target": "reviews", "time": 1400}
  ]
}
2. Structured Data (Sales CSV)
ProductID	ProductName	Region	Revenue	UnitsSold
P123	PS5 Console	US	100000	500

🔄 Data Flow & Transformations
1️⃣ Real-Time Ingestion (Bronze Layer)
Tooling: Kafka + Spark Structured Streaming

Schema enforcement: Defined via Spark StructType, e.g.:

python
Copy
Edit
StructType([
    StructField("session_id", StringType()),
    StructField("page_url", StringType(), nullable=False),
    StructField("TTI", IntegerType()),
    StructField("TTAR", IntegerType()),
    StructField("user_actions", ArrayType(
        StructType([
            StructField("action", StringType()),
            StructField("target", StringType()),
            StructField("time", IntegerType())
        ])
    ))
])
Pre-Processing (Shared Module):

Validate schema

Flatten user_actions into separate rows

Set default TTI = 0 if missing

Append metadata: ingest_timestamp, source_type

Storage Format: Partitioned Parquet files

Partitioned by: year, month, day, hour

Stored in: Azure Data Lake Storage Gen2 (Bronze)

Data Volume:

~20-30 GB/day

~10K events/sec during peak traffic

2️⃣ Batch Processing & Enrichment (Silver Layer)
Trigger: Azure Function on ADLS file arrival

Pre-Processing Logic: Reused same module to ensure consistency with streaming

Aggregations:

TTI (Time to Interactive): Average time for a page to become interactive

TTAR (Time to Action Response): Time taken for the UI to respond to user interaction

Example Spark SQL Query:

sql
Copy
Edit
SELECT page_url, 
       AVG(TTI) AS avg_tti, 
       AVG(TTAR) AS avg_ttar
FROM cleaned_clickstream
GROUP BY page_url
Storage:

Cleaned & aggregated output saved to Silver Layer (Delta Lake in ADLS)

Partitioned by page_url, date

3️⃣ Final Aggregations (Gold Layer)
Join Operations:

Join aggregated metrics (avg_tti, avg_ttar) with structured sales data and product master data

Outputs:

Azure Synapse Analytics for BI & reporting

Power BI dashboards (plugged into Synapse views)

Azure Cognitive Search for indexing metrics like:

page_url, avg_tti, avg_ttar, region, product_category

⚙️ Orchestration & Automation
Azure Function:

Triggered by ADLS blob creation events

Identifies new input path dynamically

Stores & retrieves checkpoint info from Azure Cosmos DB

Triggers ADF pipeline for batch job

Azure Data Factory:

Spark Notebook activity handles transformation

Parameterized pipeline supports reusability across multiple product categories or regions

🧠 Cluster Configuration
Synapse Spark Pools:

Node Type: Memory-Optimized

Cluster Size: 8 vCores, 64 GB RAM (auto-scale enabled)

Driver Memory: 16 GB

Executors: 4–8 based on load

Runtime Version: Spark 3.2, Delta Lake enabled

📊 Monitoring & Reliability
Azure Monitor:

Tracks Spark job metrics, ADF run status, Function logs

Retry Logic:

Exponential backoff via Azure Functions: Retry every 1s → 2s → 4s → Max 5 attempts

Alerting:

Email/Teams alert on SLA violations (e.g., delay > 15 mins or Function failure)

Logic Apps:

Trigger manual approval or fallback mechanism in critical failure scenarios



## Learnings & Insights
- **Modular Design**: Ensures reusability of the pre-processing logic.
- **Fault Tolerance**: Checkpointing and Cosmos DB enable incremental processing.
- **Scalability**: Azure-managed services allow seamless scaling.
- **Operational Efficiency**: Automated orchestration reduces manual intervention.

