## Module 3 Homework

ATTENTION: At the end of the submission form, you will be required to include a link to your GitHub repository or other public code-hosting site.
This repository should contain your code for solving the homework. If your solution includes code that is not in file format (such as SQL queries or
shell commands), please include these directly in the README file of your repository.

<b><u>Important Note:</b></u> <p> For this homework we will be using the Yellow Taxi Trip Records for **January 2024 - June 2024 NOT the entire year of data**
Parquet Files from the New York
City Taxi Data found here: </br> https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page </br>
If you are using orchestration such as Kestra, Mage, Airflow or Prefect etc. do not load the data into Big Query using the orchestrator.</br>
Stop with loading the files into a bucket. </br></br>

**Load Script:** You can manually download the parquet files and upload them to your GCS Bucket or you can use the linked script [here](./load_yellow_taxi_data.py):<br>
You will simply need to generate a Service Account with GCS Admin Priveleges or be authenticated with the Google SDK and update the bucket name in the script to the name of your bucket<br>
Nothing is fool proof so make sure that all 6 files show in your GCS Bucket before begining.</br><br>

<u>NOTE:</u> You will need to use the PARQUET option files when creating an External Table</br>

<b>BIG QUERY SETUP:</b></br>
Create an external table using the Yellow Taxi Trip Records. </br>
Create a (regular/materialized) table in BQ using the Yellow Taxi Trip Records (do not partition or cluster this table). </br>

</p>

## Question 1:

Question 1: What is count of records for the 2024 Yellow Taxi Data?

- 65,623
- 840,402
- 20,332,093
- 85,431,289

Answer :

```sql
-- Count of records for the 2024 Yellow Taxi Data
SELECT COUNT(*) AS total_records
FROM `de-zoomcamp-447504.ny_taxi.native_yellow_tripdata_2024`;
-- 20332093
```

## Question 2:

Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.</br>
What is the **estimated amount** of data that will be read when this query is executed on the External Table and the Table?

- 18.82 MB for the External Table and 47.60 MB for the Materialized Table
- 0 MB for the External Table and 155.12 MB for the Materialized Table
- 2.14 GB for the External Table and 0MB for the Materialized Table
- 0 MB for the External Table and 0MB for the Materialized Table

Answer :

```sql
SELECT COUNT(DISTINCT PULocationID)
FROM `de-zoomcamp-447504.ny_taxi.external_yellow_tripdata_2024`;
-- external estimated amount 0MB

SELECT COUNT(DISTINCT PULocationID)
FROM `de-zoomcamp-447504.ny_taxi.native_yellow_tripdata_2024`;
-- native estimated amount 155.12MB
```

## Question 3:

Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table. Why are the estimated number of Bytes different?

- BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires
  reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.
- BigQuery duplicates data across multiple storage partitions, so selecting two columns instead of one requires scanning the table twice,
  doubling the estimated bytes processed.
- BigQuery automatically caches the first queried column, so adding a second column increases processing time but does not affect the estimated bytes scanned.
- When selecting multiple columns, BigQuery performs an implicit join operation between them, increasing the estimated bytes processed

Answer :

```sql
SELECT PULocationID
FROM `de-zoomcamp-447504.ny_taxi.native_yellow_tripdata_2024`;

SELECT PULocationID, DOLocationID
FROM `de-zoomcamp-447504.ny_taxi.native_yellow_tripdata_2024`;
```

## Question 4:

How many records have a fare_amount of 0?

- 128,210
- 546,578
- 20,188,016
- 8,333

Answer :

```sql
-- records have a fare_amount of 0
SELECT COUNT(1)
FROM `de-zoomcamp-447504.ny_taxi.regular_yellow_taxi`
WHERE fare_amount = 0;
-- 8333
```

## Question 5:

What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)

- Partition by tpep_dropoff_datetime and Cluster on VendorID
- Cluster on by tpep_dropoff_datetime and Cluster on VendorID
- Cluster on tpep_dropoff_datetime Partition by VendorID
- Partition by tpep_dropoff_datetime and Partition by VendorID

Answer :

```sql
-- filter based on tpep_dropoff_datetime and order the results by VendorID
CREATE OR REPLACE TABLE `de-zoomcamp-447504.ny_taxi.yellow_tripdata_2024_partitioned`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM `de-zoomcamp-447504.ny_taxi.external_yellow_tripdata_2024`;
```

- Partitioning by tpep_dropoff_datetime
  Since the query will always filter based on tpep_dropoff_datetime, partitioning by this column ensures that queries only scan the necessary partitions instead of the entire table, improving performance and reducing costs.
- Clustering on VendorID
  Since the results are ordered by VendorID, clustering on this column will help BigQuery efficiently sort and retrieve data with minimal sorting overhead.
  Clustering also helps optimize storage and retrieval by physically organizing data within each partition.

## Question 6:

Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime
03/01/2024 and 03/15/2024 (inclusive)</br>

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 4 and note the estimated bytes processed. What are these values? </br>

Choose the answer which most closely matches.</br>

- 12.47 MB for non-partitioned table and 326.42 MB for the partitioned table
- 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table
- 5.87 MB for non-partitioned table and 0 MB for the partitioned table
- 310.31 MB for non-partitioned table and 285.64 MB for the partitioned table

Answer :

```sql
-- distinct VendorIDs between tpep_dropoff_datetime 2024-03-01 and 2024-03-15 (inclusive)

SELECT DISTINCT VendorID
FROM `de-zoomcamp-447504.ny_taxi.native_yellow_tripdata_2024`
WHERE tpep_dropoff_datetime >= '2024-03-01'
AND tpep_dropoff_datetime <=  '2024-03-15';
-- 310.24MB for native table

SELECT DISTINCT VendorID
FROM `de-zoomcamp-447504.ny_taxi.yellow_tripdata_2024_partitioned`
WHERE tpep_dropoff_datetime >= '2024-03-01'
AND tpep_dropoff_datetime <=  '2024-03-15';
-- 26.84MB for partitioned table
```

## Question 7:

Where is the data stored in the External Table you created?

- Big Query
- Container Registry
- GCP Bucket
- Big Table

Answer :

- GCP Bucket

  When you create an external table in BigQuery, the actual data remains in its original location (e.g., Google Cloud Storage (GCS), Google Drive, or external databases), and BigQuery only reads the data when queried.

  An external table allows you to query data without loading it into BigQuery, which can save storage costs and provide more flexibility.

  Key Features:

  - Data Stays in the Source (e.g., GCS, Google Drive, Bigtable, or Cloud SQL)
  - No Ingestion Costs (since data is not stored in BigQuery)
  - Queries May Be Slower than native BigQuery tables
  - Supports Various Formats (CSV, JSON, Parquet, Avro, ORC)

## Question 8:

It is best practice in Big Query to always cluster your data:

- True
- False

Answer :

- False

## (Bonus: Not worth points) Question 8:

No Points: Write a `SELECT count(*)` query FROM the materialized table you created. How many bytes does it estimate will be read? Why?

Answer :

```sql
SELECT COUNT(*)
FROM `de-zoomcamp-447504.ny_taxi.native_yellow_tripdata_2024`;
-- 0B
```

## Submitting the solutions

Form for submitting: https://courses.datatalks.club/de-zoomcamp-2025/homework/hw3
