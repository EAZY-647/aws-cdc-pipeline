# Real-Time Change Data Capture (CDC) Pipeline: PostgreSQL to Databricks

## Project Overview
This project implements a scalable Change Data Capture (CDC) pipeline designed to stream database changes from a local PostgreSQL instance to a Delta Lake table within Databricks. The architecture leverages AWS S3 as an intermediate landing zone and utilizes Databricks Auto Loader for efficient ingestion.

This pipeline demonstrates a modern "Lakehouse" architecture, transforming raw transaction logs into structured, queryable tables with ACID transaction guarantees.

## Architecture
**PostgreSQL (Source)** -> **Python Producer** -> **AWS S3 (Landing Zone)** -> **Databricks Auto Loader** -> **Delta Table (Silver Layer)**

### Key Features
* **Real-Time Ingestion:** A custom Python script captures and streams database changes (`INSERT`, `UPDATE`) immediately as they occur.
* **Cloud Storage Integration:** Raw transaction logs are offloaded to AWS S3 as JSON files, decoupling the source database from the analytical processing layer.
* **Efficient Processing:** Databricks Auto Loader is used to detect and ingest new files from S3 statefully, avoiding the need to list the entire bucket for every batch.
* **ACID Compliance:** The pipeline writes to Delta Lake, ensuring data consistency, atomicity, and schema enforcement.
* **Data Parsing & Transformation:** Unstructured log strings are parsed using Regular Expressions to extract typed columns (`id`, `city`, `temperature`).

## Repository Structure
* `local_scripts/`: Contains the Python producer scripts responsible for extracting data from PostgreSQL and uploading to S3.
* `databricks_notebooks/`: Contains the Databricks notebook with the Spark Structured Streaming logic (Extract -> Transform -> Load).
* `data/`: Contains sample datasets for testing purposes.

## Prerequisites
* **Python 3.8+**: Required for running the local data producer.
* **AWS Account**: Access to an S3 bucket and IAM credentials with write permissions.
* **Databricks Workspace**: A cluster running Databricks Runtime 10.4 LTS or higher (supports Auto Loader and Delta Lake).
* **PostgreSQL**: A local or remote instance of PostgreSQL.

## Usage Guide

### 1. Local Setup (The Producer)
1.  Install the required Python dependencies:
    `pip install boto3 psycopg2-binary`
2.  Configure your AWS credentials and PostgreSQL connection strings in `local_scripts/sync_worker.py`.
3.  Execute the script to begin generating and streaming data:
    `python local_scripts/sync_worker.py`

### 2. Cloud Setup (The Consumer)
1.  Import the `databricks_notebooks/cdc_ingestion.ipynb` notebook into your Databricks workspace.
2.  Attach the notebook to a compute cluster.
3.  Execute the cells to initialize the streaming query.
4.  Monitor the `temperature_updates` Delta table to verify real-time data arrival.

## Future Enhancements
* **Orchestration:** Implement Databricks Workflows to schedule and manage the ingestion job.
* **Visualization:** Connect the final Delta Table to a Databricks SQL Dashboard to visualize temperature trends.
* **Schema Evolution:** Enable schema inference to automatically handle new columns added to the source database.
