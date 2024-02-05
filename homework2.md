## Week 2 Homework

### Assignment

The goal will be to construct an ETL pipeline that loads the data, performs some transformations, and writes the data to a database (and Google Cloud!).

- Create a new pipeline, call it `green_taxi_etl`
- Add a data loader block and use Pandas to read data for the final quarter of 2020 (months `10`, `11`, `12`).
  - You can use the same datatypes and date parsing methods shown in the course.
  - `BONUS`: load the final three months using a for loop and `pd.concat`
- Add a transformer block and perform the following:
  - Remove rows where the passenger count is equal to 0 _or_ the trip distance is equal to zero.
  - Create a new column `lpep_pickup_date` by converting `lpep_pickup_datetime` to a date.
  - Rename columns in Camel Case to Snake Case, e.g. `VendorID` to `vendor_id`.
  - Add three assertions:
    - `vendor_id` is one of the existing values in the column (currently)
    - `passenger_count` is greater than 0
    - `trip_distance` is greater than 0
- Using a Postgres data exporter (SQL or Python), write the dataset to a table called `green_taxi` in a schema `mage`. Replace the table if it already exists.
- Write your data as Parquet files to a bucket in GCP, partioned by `lpep_pickup_date`. Use the `pyarrow` library!
- Schedule your pipeline to run daily at 5AM UTC.

### Questions

## Question 1. Data Loading

Once the dataset is loaded, what's the shape of the data?

* 266,855 rows x 20 columns &larr;
* 544,898 rows x 18 columns
* 544,898 rows x 20 columns
* 133,744 rows x 20 columns

answer:
<pre>
# Create a Python DATA LOADER block on MAGE
# using a for to iterate through 10 to 12
# and returning a concatenated dataframe

import io
import pandas as pd
import requests
if 'data_loader' not in globals():
    from mage_ai.data_preparation.decorators import data_loader
if 'test' not in globals():
    from mage_ai.data_preparation.decorators import test


@data_loader
def load_data_from_api(*args, **kwargs):

    urls = []
    for month in range(10,13):
        print('fetching url https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2020-'+str(month)+'.csv.gz')
        urls.append('https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2020-'+str(month)+'.csv.gz')

    taxi_dtypes = {
        'VendorID': pd.Int64Dtype(),
        'passenger_count':pd.Int64Dtype(),
        'trip_distance':float,
        'RatecodeID':pd.Int64Dtype(),
        'store_and_fwd_flag':str,
        'PULocationID':pd.Int64Dtype(),
        'DOLocationID':pd.Int64Dtype(),
        'payment_type':pd.Int64Dtype(),
        'fare_amount':float,
        'extra':float,
        'mta_tax':float,
        'tip_amount':float,
        'tolls_amount':float,
        'improvement_surcharge':float,
        'total_amount':float,
        'congestion_surcharge': float
    }
    parse_dates = ['lpep_pickup_datetime', 'lpep_dropoff_datetime']

    pds = []
    for url in urls:
        pds.append(pd.read_csv(url, sep=",", compression="gzip", dtype=taxi_dtypes, parse_dates=parse_dates))

    return pd.concat(pds)

@test
def test_output(output, *args) -> None:
    """
    Template code for testing the output of the block.
    """
    assert output is not None, 'The output is undefined'
</pre>
<pre>
266855 rows x 20 columns
</pre>

## Question 2. Data Transformation

Upon filtering the dataset where the passenger count is greater than 0 _and_ the trip distance is greater than zero, how many rows are left?

* 544,897 rows
* 266,855 rows
* 139,370 rows &larr;
* 266,856 rows

answer:
<pre>
'''
Add a transformer block and perform the following:
    Remove rows where the passenger count is equal to 0 or the trip distance is equal to zero.
    Create a new column lpep_pickup_date by converting lpep_pickup_datetime to a date.
    Rename columns in Camel Case to Snake Case, e.g. VendorID to vendor_id.
Add three assertions:
    vendor_id is one of the existing values in the column (currently)
    passenger_count is greater than 0
    trip_distance is greater than 0
''' 

if 'transformer' not in globals():
    from mage_ai.data_preparation.decorators import transformer
if 'test' not in globals():
    from mage_ai.data_preparation.decorators import test


@transformer
def transform(data, *args, **kwargs):
    print(f"Preprocessing: rows with zero passengers: {data['passenger_count'].isin([0]).sum()}")
    data = data[(data['passenger_count'] > 0) & (data['trip_distance'] > 0)]
    print(f"Creating new column lpep_pickup_date")
    data['lpep_pickup_date'] = data['lpep_pickup_datetime'].dt.date
    print(f"Converting Camel Case to Snake Case")
    data.rename(
        columns={
            'VendorID': 'vendor_id',
            'RatecodeID': 'ratecode_id',
            'PULocationID': 'pu_location_id',
            'DOLocationID':'do_location_id',
            },
        inplace=True
        )

    return data


@test
def test_output(output, *args) -> None:
    assert 'vendor_id' in output.columns.to_list(), 'Vendor_id is not a column name.'
    assert output['passenger_count'].isin([0]).sum() == 0, 'There are rides with zero passengers.'
    assert output['trip_distance'].isin([0]).sum() == 0, 'There are rides with zero distance.'
</pre>
<pre>
139370 rows x 21 columns
</pre>
## Question 3. Data Transformation

Which of the following creates a new column `lpep_pickup_date` by converting `lpep_pickup_datetime` to a date?

* `data = data['lpep_pickup_datetime'].date`
* `data('lpep_pickup_date') = data['lpep_pickup_datetime'].date`
* `data['lpep_pickup_date'] = data['lpep_pickup_datetime'].dt.date` &larr;
* `data['lpep_pickup_date'] = data['lpep_pickup_datetime'].dt().date()`

answer:
The correct function to return date from datetime is dt.date

## Question 4. Data Transformation

What are the existing values of `VendorID` in the dataset?

* 1, 2, or 3
* 1 or 2 &larr;
* 1, 2, 3, 4
* 1
  
answer:
<pre>
# using a SQL DATA LOADER block 

-- Docs: https://docs.mage.ai/guides/sql-blocks
SELECT DISTINCT vendor_id FROM mage.green_taxi;
</pre>
<pre>
+----+---------------+
|    | vendor_id     |
|----+---------------+
|  0 | 1             |
|  1 | 2             |       
+----+---------------+
</pre>
## Question 5. Data Transformation

How many columns need to be renamed to snake case?

* 3
* 6
* 2
* 4  &larr;

answer:
There are 4 columns:

'VendorID': 'vendor_id'
'RatecodeID': 'ratecode_id'
'PULocationID': 'pu_location_id'
'DOLocationID':'do_location_id'

## Question 6. Data Exporting

Once exported, how many partitions (folders) are present in Google Cloud?

* 96  &larr;
* 56
* 67
* 108

answer:
Using GCP and my folder got 95 partitions, so I'm selecting the closest one:

