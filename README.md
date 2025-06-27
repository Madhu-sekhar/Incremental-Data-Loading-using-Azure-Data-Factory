# Incremental-Data-Loading-using-Azure-Data-Factory

📝 Overview
This project demonstrates a two-stage incremental data loading process using Azure Data Factory (ADF) and Azure Data Lake Storage (ADLS). 
It is designed to ingest and transform a large-scale synthetic medical claims dataset in hourly intervals, simulating real-time data flow. 
The pipeline ensures efficient data processing, structured storage, and lifecycle management of ingested data.

🎯 **Project Goals**
- [x] Ingest medical claim data from CSV files into ADLS in hourly intervals  
- [x] Convert raw CSV files into Parquet format using Azure Data Factory  
- [x] Load and append unique records into dimension and fact tables with surrogate keys  
- [x] Implement incremental data loading to avoid reprocessing previously ingested data  
- [x] Archive processed files after 1 day and purge them after 2 days to manage storage  


Resources
![image](https://github.com/user-attachments/assets/5208a15f-59f9-47e0-b824-c345bea7c85f)

This project is built around a modular and automated data pipeline architecture using Azure Data Factory (ADF) and Azure Data Lake Storage (ADLS). 
It simulates incremental data loading by processing medical claim data in hourly intervals.

📂 **Folder Structure**
```
stg/                  # Raw CSV subsets  
preprocess/           # Parquet files  
sink/                 # Final fact/dimension tables  
stg/archive/          # Archived raw data  
preprocess/archive/   # Archived parquet data  
sink/archive/         # Archived final tables  
```


![image](https://github.com/user-attachments/assets/1bb31990-1bed-41e0-b199-184150f2278b)

->>subset files are uploded in the stg folder
![image](https://github.com/user-attachments/assets/6bdea213-7e2f-42b0-81e8-52da932cd974)


📑 ADF Pipelines
Pipelines and Purpose
1️⃣ CSV to Parquet Ingestion	Converts raw CSV files into Parquet format and stores them in ADLS
2️⃣ Parquet to Fact/Dim Tables	Loads Parquet data into structured fact and dimension tables


🔄 Pipeline 1: CSV to Parquet Ingestion
Purpose: Converts raw medical claim data from CSV format into Parquet format for optimized storage and processing.

📌 **Activities**
1️⃣ **Source Dataset**: Connects to the CSV file located in the `stg/` folder in ADLS  
2️⃣ **Copy Data Activity**: Reads the CSV and writes it to the `preprocess/` folder in Parquet format  
3️⃣ **Trigger**: Automatically starts when a new CSV file is dropped into the `stg/` folder  
4️⃣ **Output**: A `.parquet` file with the same name as the input CSV, stored in `preprocess/`  


![image](https://github.com/user-attachments/assets/f54d3646-a513-4da9-915d-9b7a09a572cb)

🔄 Pipeline 2: Parquet to Fact/Dimension Tables
Purpose: Processes structured Parquet files and loads data into final fact and dimension tables, ensuring uniqueness and continuity through surrogate key generation.


📌 **Activities**
1️⃣ **Source Dataset**: Connects to the Parquet file located in the `preprocess/` folder in ADLS  
2️⃣ **Lookup Activity**: Retrieves the last maximum surrogate key value from each dimension table to maintain continuity  
3️⃣ **Derived Column Activity**: Generates new surrogate keys starting from the last known value  
4️⃣ **Conditional Split** (optional): Ensures only new or valid records are processed  
5️⃣ **Copy Data Activity**: Appends new records to the appropriate tables:
   - `dim_beneficiary.csv`  
   - `dim_provider.csv`  
   - `fact_claim.csv` in the `sink/` folder  
6️⃣ **Trigger**: Automatically starts when a new Parquet file is dropped into the `preprocess/` folder  
7️⃣ **Output**: Updated dimension and fact tables stored in the `sink/` folder  


Dataflow 
![image](https://github.com/user-attachments/assets/d70c42be-01ef-4506-bb86-99162b0328d6)

Pipeline-2
![image](https://github.com/user-attachments/assets/734d7fdc-c91d-4f04-92a8-376c9a2a36c7)

🔄 Pipeline 3: Data Archiving 
Purpose: This pipeline ensures proper data lifecycle management by archiving files older than 1 day and cleaning up the original folders. 

📌 **Activities**
- The pipeline starts by retrieving metadata from the `stg/`, `preprocess/`, and `sink/` folders  
- It filters files that are **older than 1 day**  
- These files are then copied to their respective archive folders:  
  - `stg/archive/`  
  - `preprocess/archive/`  
  - `sink/archive/`  
- After successful copying, the same files are deleted from the original folders to free up space and maintain folder hygiene  
- This pipeline is scheduled to run periodically (e.g., daily or hourly)  

📈 **Data Flow Summary**
- **Source Folders**: `stg/`, `preprocess/`, `sink/`  
- **Archive Folders**: `stg/archive/`, `preprocess/archive/`, `sink/archive/`  
- **Archiving Rule**: Copy files older than 1 day to archive, then delete from original folders  


![image](https://github.com/user-attachments/assets/3976e0ad-3725-488e-b0e0-3b6d4e0cd2cc)

->>>Purging of archived files is automated using Azure Storage Lifecycle Management, which deletes files from archive folders after 2 days based on configured retention rules.
![image](https://github.com/user-attachments/assets/9018a217-0958-4e89-a5c7-8d5a36202227)

![image](https://github.com/user-attachments/assets/ea021042-5158-4519-b876-065236c6f19a)



Datasets

![image](https://github.com/user-attachments/assets/d187fed2-e056-424b-a70e-b83415df7e6c)

Linkedservices

![image](https://github.com/user-attachments/assets/36ea1f2d-07f3-43c5-acce-54ea576e2c86)



Pipelines

![image](https://github.com/user-attachments/assets/6c2e85fb-2a3b-4bb8-883a-735d330be539)


triggers

![image](https://github.com/user-attachments/assets/6aba5e13-79cc-481c-a943-a338b11405bf)


📘 Project Summary
This project demonstrates the design and implementation of a scalable, automated data ingestion and transformation pipeline using a synthetic medical claims dataset of 1 million records. The objective was to simulate a real-world incremental data load scenario and build a robust ETL system using PySpark and Azure Data Lake Storage (ADLS).

🔧 What I Built
Data Preparation

Downloaded and explored the Medical Claims Synthetic Dataset
Split the dataset into 6 equal subsets to simulate hourly data arrival
Added an md5_hash column to each record for uniqueness tracking
Pipeline 1: CSV to Parquet Conversion

Triggered when a new CSV file is uploaded to the stg/ folder in ADLS
Converted CSV to Parquet format using PySpark
Stored the output in the preprocess/ folder
Pipeline 2: Parquet to Fact & Dimension Tables

Triggered when a new Parquet file appears in the preprocess/ folder
Read the Parquet file and generated surrogate keys for dimension modeling
Transformed and appended data to:
dim_beneficiary.csv
dim_provider.csv
fact_claim.csv
Stored the final tables in the sink/ folder
Pipeline 3: Data Lifecycle Management

Implemented Storage Lifecycle Management policies in ADLS
Automatically moved processed files to their respective archive/ folders after 1 day
Purged archived files after 2 days to manage storage efficiently
Automation & Scheduling

All pipelines were scheduled to run hourly, simulating real-time data ingestion
Dependencies were managed to ensure sequential execution (Pipeline 2 waits for Pipeline 1)
🧰 Tools & Technologies Used
PySpark – Distributed data processing
Pandas – Initial data manipulation and hashing
Azure Data Lake Storage (ADLS) – Scalable cloud storage
Surrogate Key Logic – For dimension table modeling
Storage Lifecycle Management – For automated archiving and purging
Automation & Scheduling – Simulated using time-based triggers




