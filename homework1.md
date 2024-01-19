## Module 1 Homework

## Docker & SQL

In this homework we'll prepare the environment 
and practice with Docker and SQL


## Question 1. Knowing docker tags

Run the command to get information on Docker 

```docker --help```

Now run the command to get help on the "docker build" command:

```docker build --help```

Do the same for "docker run".

Which tag has the following text? - *Automatically remove the container when it exits* 

- `--delete`
- `--rc`
- `--rmc`
- `--rm`

answer: 

<pre>--rm    Automatically remove the container when it exits</pre>


## Question 2. Understanding docker first run 

Run docker with the python:3.9 image in an interactive mode and the entrypoint of bash.
Now check the python modules that are installed ( use ```pip list``` ). 

What is version of the package *wheel* ?

- 0.42.0
- 1.0.0
- 23.0.1
- 58.1.0

answer: 

<pre>
root@b00291a3690c:/# pip list
Package    Version
---------- -------
pip        23.0.1
setuptools 58.1.0
wheel      0.42.0
</pre>

# Prepare Postgres

Run Postgres and load data as shown in the videos
We'll use the green taxi trips from September 2019:

```wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz```

You will also need the dataset with zones:

```wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv```

Download this data and put it into Postgres (with jupyter notebooks or with a pipeline)


## Question 3. Count records 

How many taxi trips were totally made on September 18th 2019?

Tip: started and finished on 2019-09-18. 

Remember that `lpep_pickup_datetime` and `lpep_dropoff_datetime` columns are in the format timestamp (date and hour+min+sec) and not in date.

- 15767
- 15612
- 15859
- 89009

answer: 
<pre>
# using Pandas:

In[1]: len(df[(df['lpep_pickup_datetime'].dt.strftime('%Y-%m-%d') == '2019-09-18') & (df['lpep_dropoff_datetime'].dt.strftime('%Y-%m-%d') == '2019-09-18') ])

Out[1]: 15612

# using SQL

SELECT COUNT(*)
FROM green_taxi_data
WHERE lpep_pickup_datetime >= '2019-09-18 00:00:00'
AND lpep_dropoff_datetime <= '2019-09-18 23:59:59'

count bigint 15612
</pre>

## Question 4. Largest trip for each day

Which was the pick up day with the largest trip distance
Use the pick up time for your calculations.

- 2019-09-18
- 2019-09-16
- 2019-09-26 &larr;
- 2019-09-21

answer:
<pre>
# using Pandas:

In[1]: df.sort_values(by='trip_distance',ascending=False)[:1]['lpep_pickup_datetime'].item().strftime('%Y-%m-%d')

Out[1]: 2019-09-26

# using SQL:

SELECT to_char(lpep_pickup_datetime,'YYYY-MM-DD')
FROM green_taxi_data
ORDER BY trip_distance DESC
LIMIT 1

to_char text
2019-09-26
</pre>

## Question 5. The number of passengers

Consider lpep_pickup_datetime in '2019-09-18' and ignoring Borough has Unknown

Which were the 3 pick up Boroughs that had a sum of total_amount superior to 50000?
 
- "Brooklyn" "Manhattan" "Queens" &larr;
- "Bronx" "Brooklyn" "Manhattan"
- "Bronx" "Manhattan" "Queens" 
- "Brooklyn" "Queens" "Staten Island"

answer:
<pre>
# using Pandas:

# first merged the zones with the green dataframe

merged_zones = pd.merge(df_zones[['LocationID','Borough']],df[['PULocationID','total_amount','lpep_pickup_datetime']],left_on='LocationID',right_on='PULocationID')

# groupby and aggregate by total_amount filtered by date 2019-09-18

In[1]: merged_zones[(merged_zones['lpep_pickup_datetime'].dt.strftime('%Y-%m-%d') == '2019-09-18')][['Borough','total_amount']].groupby(by='Borough',as_index=False).agg({"total_amount":"sum"}).sort_values(by='total_amount',ascending=False)

Out[1:]
+----+---------------+----------------+
|    | Borough       |   total_amount |
|----+---------------+----------------|
|  1 | Brooklyn      |       96333.2  |
|  2 | Manhattan     |       92271.3  |
|  3 | Queens        |       78671.7  |
|  0 | Bronx         |       32830.1  |
|  5 | Unknown       |         728.75 |
|  4 | Staten Island |         342.59 |
+----+---------------+----------------+

# using SQL:

WITH t AS 
(
	SELECT z."LocationID", z."Borough", g."PULocationID", g."total_amount", g."lpep_pickup_datetime"
	FROM public.green_taxi_data g
	INNER JOIN public.zones z
	ON z."LocationID" = g."PULocationID"
	AND DATE(g."lpep_pickup_datetime") = '2019-09-18'
)
SELECT t."Borough", SUM(t."total_amount")
FROM t
GROUP BY t."Borough"
ORDER BY SUM(t."total_amount") DESC


"Brooklyn"	96333.23999999906
"Manhattan"	92271.29999999865
"Queens"	78671.70999999886
"Bronx"	32830.089999999895
"Unknown"	728.75
"Staten Island"	342.59000000000003
</pre>

## Question 6. Largest tip

For the passengers picked up in September 2019 in the zone name Astoria which was the drop off zone that had the largest tip?
We want the name of the zone, not the id.

Note: it's not a typo, it's `tip` , not `trip`

- Central Park
- Jamaica
- JFK Airport &larr;
- Long Island City/Queens Plaza

answer:
<pre>
# Using PANDAS:

# two functions to ease the access to zone names by id and vice-versa
# this avoids another merge to access PUZone and DOZone respectively

def return_zonename_by_id(id):
    try:
        return df_zones[df_zones.LocationID == id]['Zone'].values[0]
    except:
        print('No Zone found for id', id)

def return_zoneid_by_name(name):
    try:
        return df_zones[df_zones.Zone == name]['LocationID'].values[0]
    except:
        print('No ID found for zone', zone)

In[1]: return_zoneid_by_name('Astoria')
Out[1]: 7

# filter by Astoria, september 2019, sort by tip amount 
# get the DOLocationID and return the zonename

In[2]: return_zonename_by_id(df[(df.PULocationID == 7) & (merged_zones['lpep_pickup_datetime'].dt.strftime('%Y-%m') == '2019-09')].sort_values(by='tip_amount', ascending=False)[:1]['DOLocationID'].values[0])

Out[2]: 'JFK Airport'
</pre>

## Terraform

In this section homework we'll prepare the environment by creating resources in GCP with Terraform.

In your VM on GCP/Laptop/GitHub Codespace install Terraform. 
Copy the files from the course repo
[here](https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/main/week_1_basics_n_setup/1_terraform_gcp/terraform) to your VM/Laptop/GitHub Codespace.

Modify the files as necessary to create a GCP Bucket and Big Query Dataset.


## Question 7. Creating Resources

After updating the main.tf and variable.tf files run:

```
terraform apply
```

Paste the output of this command into the homework submission form.

```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_bigquery_dataset.amilcar_de_dataset will be created
  + resource "google_bigquery_dataset" "amilcar_de_dataset" {
      + creation_time              = (known after apply)
      + dataset_id                 = "amilcar_de_dataset"
      + default_collation          = (known after apply)
      + delete_contents_on_destroy = false
      + effective_labels           = (known after apply)
      + etag                       = (known after apply)
      + id                         = (known after apply)
      + is_case_insensitive        = (known after apply)
      + last_modified_time         = (known after apply)
      + location                   = "US"
      + max_time_travel_hours      = (known after apply)
      + project                    = "dezoomcamp24"
      + self_link                  = (known after apply)
      + storage_billing_model      = (known after apply)
      + terraform_labels           = (known after apply)
    }

  # google_storage_bucket.amilcar_de_bucket will be created
  + resource "google_storage_bucket" "amilcar_de_bucket" {
      + effective_labels            = (known after apply)
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "US"
      + name                        = "amilcar_de_bucket"
      + project                     = (known after apply)
      + public_access_prevention    = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + terraform_labels            = (known after apply)
      + uniform_bucket_level_access = (known after apply)
      + url                         = (known after apply)

      + lifecycle_rule {
          + action {
              + type = "Delete"
            }
          + condition {
              + age                   = 1
              + matches_prefix        = []
              + matches_storage_class = []
              + matches_suffix        = []
              + with_state            = (known after apply)
            }
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_bigquery_dataset.amilcar_de_dataset: Creating...
google_storage_bucket.amilcar_de_bucket: Creating...
google_bigquery_dataset.amilcar_de_dataset: Creation complete after 2s [id=projects/dezoomcamp24/datasets/amilcar_de_dataset]
google_storage_bucket.amilcar_de_bucket: Creation complete after 3s [id=amilcar_de_bucket]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

```