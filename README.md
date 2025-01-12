# Airline Data Ingestion Project | AWS 
## Introduction 
Airline-Data-Ingestion-Project is a data pipeline that collects, processes, and stores airline data in a central data warehouse. It uses AWS cloud services to automate tasks like detecting new files, running data transformations, and storing the results.

This project ensures that data is handled efficiently and is ready for analysis, making it easy to generate insights and reports. It’s designed to be reliable, scalable, and simple to use.


## Architecture 
![Project Architecture](Architecture.png)

## Tech Stack 
- AWS S3
- Cloudtrail Notification 
- Event Bridge Pattern Rule 
- Glue Crawler 
- Glue Visual ETL 
- AWS SNS 
- AWS Redshift 
- AWS Step Function

## Dataset Description

The dataset contains daily flight information for airlines in the United States. It tracks details such as delays, departure and arrival locations, and carrier information. The data is sourced from daily CSV uploads to an Amazon S3 bucket and ingested into Redshift through an ETL pipeline.

Business Problems Addressed by the ETL Job
- Delay Analysis: Enables identifying problematic routes and airports by analyzing flight delays across locations, helping airlines improve efficiency.
- Carrier Performance: Provides insights into carrier delays on specific routes, aiding decisions on resource allocation and policy adjustments.

### Table Details

1. #### Fact Table: `daily_flights_fact`
- Purpose:
  Stores enriched flight data, combining flight metrics with details from the dimension tables for analysis.

- Schema:

| Column Name    | Data Type      | Description                                         |
|----------------|----------------|-----------------------------------------------------|
| `carrier`      | VARCHAR(10)    | Airline identifier . |
| `dep_airport`  | VARCHAR(200)   | Departure airport name.                            |
| `arr_airport`  | VARCHAR(200)   | Arrival airport name.                              |
| `dep_city`     | VARCHAR(100)   | City of the departure airport.                     |
| `arr_city`     | VARCHAR(100)   | City of the arrival airport.                       |
| `dep_state`    | VARCHAR(100)   | State of the departure airport.                    |
| `arr_state`    | VARCHAR(100)   | State of the arrival airport.                      |
| `dep_delay`    | BIGINT         | Departure delay in minutes.                        |
| `arr_delay`    | BIGINT         | Arrival delay in minutes.                          |

---

2. #### Dimension Table: `airports_dim`
- Purpose: Provides descriptive details about airports for enrichment.

- Schema: 

| Column Name    | Data Type      | Description                                         |
|----------------|----------------|-----------------------------------------------------|
| `airport_id`   | VARCHAR(50)    | Unique identifier for the airport.                 |
| `city`         | VARCHAR(100)   | City where the airport is located.                 |
| `state`        | VARCHAR(100)   | State where the airport is located.                |
| `name`         | VARCHAR(200)   | Full name of the airport.                          |

---

3. #### Sample CSV Data (`flights.csv`)

| Carrier | Origin Airport ID | Dest Airport ID | Dep Delay (mins) | Arr Delay (mins) |
|---------|--------------------|------------------|------------------|------------------|
| AA      | ORD                | LAX              | 15               | 10               |
| DL      | JFK                | ATL              | -5               | -3               |
| UA      | DEN                | SFO              | 20               | 25               |

---

## ETL Pipeline 
1. ### EventBridge Trigger
    - Daily flight data is uploaded to an S3 bucket in a partitioned manner, where the folder name corresponds to the date. The data is stored in a flights.csv file. An EventBridge rule detects file uploads with the suffix /flights.csv in the S3 bucket and triggers a Step Function State Machine.
    - ![Event Bridge Rule](Event_bridge_1.png)

  
2. ###  Stepfunctions StateMachine
   - The Step Function State Machine orchestrates the workflow of ETL pipeline,ensuring each step in the pipeline is executed in the correct order with error handling and notifications.

   - ![Stepfunction Statemachine Workflow](Stepfunction_Statemachine.png)
3. ### Glue ETL job 
       - 3.1 Running Glue Crawler to detect the data in the new partition and read the file from S3.
     - ![Glue_crawler_to detect new partitions in S3 bucket](Glue_crawle_S3_daily_data.png)
     - 
     - ![Code_to_read_S3_data_from_glue_catalog ](Code_to_read_S3_data_from_glue_catalog.png)
   

  
       - Extract the airports_dim table data stored in Redshift and transforming and enriching the daily flights data in the incoming csv file.
       - Storing the cleaned and enriched data in a daily_flights_fact Redshift table.
