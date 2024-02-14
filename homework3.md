
## Create an external table using the Green Taxi Trip Records Data for 2022.
Create a table in BQ using the Green Taxi Trip Records for 2022 (do not partition or cluster this table). After uploading all the parquet files  to a specific bucket **green_taxi_de_amilcar** create the database using this **SQL** directly on BigQuery:

<pre>
# external table
CREATE OR REPLACE EXTERNAL TABLE `dezoomcamp24.amilcar_de_dataset.external_green_tripdata`
OPTIONS (
  format = 'parquet',
  uris = ['gs://green_taxi_de_amilcar/*.parquet']
);

# table on bigquery, nonpartitioned

CREATE OR REPLACE TABLE `dezoomcamp24.amilcar_de_dataset.nonpartioned_green_tripdata`
AS SELECT * FROM `dezoomcamp24.amilcar_de_dataset.external_green_tripdata`;
</pre>

### Question 1: What is count of records for the 2022 Green Taxi Data??

Answer: this can be achieved with:

<pre>
SELECT COUNT(*) 
FROM `dezoomcamp24.amilcar_de_dataset.external_green_tripdata`;


Row	f0_
1	840402
</pre>


## Question 2:
Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.</br> 
What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?

Answer: this can be achieved with creating two sql queries, and select it without running. BigQuery will estimate showing the right corner the memory used.

<pre>
SELECT DISTINCT PULocationID
FROM `dezoomcamp24.amilcar_de_dataset.external_green_tripdata`;


SELECT DISTINCT PULocationID
FROM `dezoomcamp24.amilcar_de_dataset.nonpartioned_green_tripdata`;
</pre>


**0 MB for the External Table and 6.41MB for the Materialized Table**

## Question 3:
How many records have a fare_amount of 0?

<pre>
SELECT COUNT(*)
FROM `dezoomcamp24.amilcar_de_dataset.nonpartioned_green_tripdata`
WHERE fare_amount = 0;

Row     f0_
1	1622
</pre>

## Question 4:
What is the best strategy to make an optimized table in Big Query if your query will always order the results by PUlocationID and filter based on lpep_pickup_datetime? (Create a new table with this strategy)

Answer:

Partitioning and clustering strategies in BigQuery can significantly improve query performance, especially when you frequently order by certain columns and filter based on others. Based on your scenario where you always order results by PUlocationID and filter based on lpep_pickup_datetime, here's the recommended approach:

Partitioning:
Since you're filtering based on lpep_pickup_datetime, it makes sense to partition your table based on this column. Partitioning helps in pruning unnecessary partitions during query execution, thus reducing the amount of data scanned. You can partition by day, month, or year, depending on the volume of data and your querying patterns. For example, if your data spans multiple years and you frequently query data for specific months, partitioning by year and month could be beneficial.

Clustering:
Clustering is particularly useful when you often order your results by a specific column or columns. In your case, since you always order by PUlocationID, clustering your table on this column can improve query performance significantly. Clustering physically organizes data based on the clustering columns, which reduces the amount of data shuffled during query execution.

So, to summarize:

**Partitioning Key: lpep_pickup_datetime** </br>
**Clustering Key(s): PUlocationID**

Answer: **Partition by lpep_pickup_datetime Cluster on PUlocationID**

<pre>
CREATE OR REPLACE TABLE `dezoomcamp24.amilcar_de_dataset.part_clust_green_tripdata`
PARTITION BY DATE(lpep_pickup_datetime)
CLUSTER BY PUlocationID 
AS SELECT * FROM `dezoomcamp24.amilcar_de_dataset.external_green_tripdata`;
</pre>


## Question 5:
Write a query to retrieve the distinct PULocationID between lpep_pickup_datetime
06/01/2022 and 06/30/2022 (inclusive)</br>

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 4 and note the estimated bytes processed. What are these values? </br>

<pre>
SELECT DISTINCT PULocationID
FROM `dezoomcamp24.amilcar_de_dataset.nonpartioned_green_tripdata`
WHERE DATE(lpep_pickup_datetime) BETWEEN '2022-06-01' AND '2022-06-30';

SELECT DISTINCT PULocationID
FROM `dezoomcamp24.amilcar_de_dataset.part_clust_green_tripdata`
WHERE DATE(lpep_pickup_datetime) BETWEEN '2022-06-01' AND '2022-06-30';
</pre>

**12.82 MB for non-partitioned table and 1.12 MB for the partitioned table**

## Question 6: 
Where is the data stored in the External Table you created?

**GCP Bucket**

## Question 7:
It is best practice in Big Query to always cluster your data:
**False**


## (Bonus: Not worth points) Question 8:
No Points: Write a `SELECT count(*)` query FROM the materialized table you created. How many bytes does it estimate will be read? Why?

**0B(zero bytes). One answer is that BigQuery can determine the exact count of rows from metadata or from cached query results, it might not need to access the actual data. For example, if the table statistics are up-to-date and the count is available in the metadata, BigQuery can return the count without reading the data.**