# Module 5 Homework

In this homework we'll put what we learned about Spark in practice.

For this homework we will be using the Yellow 2024-10 data from the official website: 

```bash
wget https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-10.parquet
```


## Question 1: Install Spark and PySpark

- Install Spark
- Run PySpark
- Create a local spark session
- Execute spark.version.

What's the output?
```
3.5.5
```

> [!NOTE]
> To install PySpark follow this [guide](https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/05-batch/setup/pyspark.md)


## Question 2: Yellow October 2024

Read the October 2024 Yellow into a Spark Dataframe.

Repartition the Dataframe to 4 partitions and save it to parquet.

What is the average size of the Parquet (ending with .parquet extension) Files that were created (in MB)? Select the answer which most closely matches.

- 6MB
- **25MB**
- 75MB
- 100MB

#### Answer : 25MB

Average size of the .parquet files: 23.04MB


## Question 3: Count records 

How many taxi trips were there on the 15th of October?

Consider only trips that started on the 15th of October.

- 85,567
- 105,567
- **125,567**
- 145,567

#### Answer : 125,567

```
df_yellow_tripcount = spark.sql("""
SELECT 
    COUNT(*) AS number_records
FROM
    yellow
WHERE
    tpep_pickup_datetime >= '2024-10-15 00:00:00' 
    AND tpep_pickup_datetime <= '2024-10-15 23:59:59'
""").show()
```
The output was 128893.


## Question 4: Longest trip

What is the length of the longest trip in the dataset in hours?

- 122
- 142
- **162**
- 182

#### Answer : 162

```
from pyspark.sql.functions import col, unix_timestamp

df_with_duration = df.withColumn(
    "duration_seconds",
    (unix_timestamp("tpep_dropoff_datetime") - unix_timestamp("tpep_pickup_datetime"))
)

df_with_duration = df_with_duration.withColumn("duration_hours", col("duration_seconds") / 3600)

longest_trip_in_hours = df_with_duration.orderBy(col("duration_hours").desc()).first()

print("Longest trip details (in hours):")
print(longest_trip_in_hours)
```


## Question 5: User Interface

Spark’s User Interface which shows the application's dashboard runs on which local port?

- 80
- 443
- **4040**
- 8080

#### Answer : 4040

Spark User Interface is available at localhost:4040.


## Question 6: Least frequent pickup location zone

Load the zone lookup data into a temp view in Spark:

```bash
wget https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv
```

Using the zone lookup data and the Yellow October 2024 data, what is the name of the LEAST frequent pickup location Zone?

- **Governor's Island/Ellis Island/Liberty Island**
- Arden Heights
- Rikers Island
- Jamaica Bay

#### Answer : Governor's Island/Ellis Island/Liberty Island

```
zone = spark.read \
    .option("header", "true") \
    .csv('taxi_zone_lookup.csv')

from pyspark.sql.functions import col
zone = zone.withColumn("LocationID", col("LocationID").cast("int"))

zone.registerTempTable('zones')

spark.sql("""
select t2.Zone, count(*) as num_trips from trips_data t1 inner join zones t2 on t1.PULocationID = t2.LocationID
group by t2.Zone
order by num_trips
LIMIT 1
""").show(truncate=False)
```

