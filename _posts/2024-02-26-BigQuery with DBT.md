---
title:  "Data Transformations with BigQuery and DBT"
date:   2024-02-26 20:00:00 -0500
categories: [data warehouse, bigquery, dbt]
mermaid: true
tags: [dbt, bigquery, GCP]
image:
  path: /assets/img/bigquery-dbt.png
---

Cloud tools have reconstructed the modern data stack.  Data warehouses such as [Google BigQuery](https://cloud.google.com/bigquery) and [Snowflake](https://www.snowflake.com/en/) provide scalable solutions for storing, analyzing, and managing large volumes of data, making them essential tools for data-driven processes.  Previously, suites of tools existed to transform your data before loading it into your warehouse.  It was either too difficult or too costly to load raw into data warehouse.  Now, we can easily load data into the warehouse first, transforming inside the warehouse with [DBT (Data Build Tool)](https://www.getdbt.com/product/what-is-dbt).

> 💡**Old: Bring the data to the compute.  New: we bring the compute to the data.**

Included below is an example of how to load data into BigQuery and transform it with DBT.  [Historical Loan Performance data for the CAS deals from Fannie Mae](https://capitalmarkets.fanniemae.com/credit-risk-transfer/single-family-credit-risk-transfer/connecticut-avenue-securities) will be used.  It will be loaded into BigQuery in its raw form and transformed with SQL generated from DBT to make it usable.

## Google BigQuery
### Prerequisites
If you would like to try this example yourself, you must first setup BigQuery in Google Cloud Platform (GCP).
> Follow this guide to get started with BigQuery: [Load and query data with the Google Cloud console](https://cloud.google.com/bigquery/docs/quickstarts/load-data-console)
You will also need to setup Google Cloud Storage to have a bucket to upload the raw data to.

### Downloading Example Data

The data used in this example comes from Fannie Mae.  It will use the historical loan performance data available through Fannie Mae's Data Dynamics web portal.  This will require you to create an account with Fannie Mae.  Once you've created an account and logged in, you should see 

#### Steps to Download Data
![Fannie Mae Download](/assets/img/2024-02-26-FannieMaeDataDynamics.png){: width="624" height="257" }
_Data Dynamics_
1. Go to [Fannie Mae - Data Dynamics - Create a Custom CAS Download](https://datadynamics.fanniemae.com/data-dynamics/#/slicer/CAS)
2. Select a deal that is at least 6 months old
3. Click `Generate Files` to download a zip file with the performance history

#### Column Headers
We'll need a description of the columns to understand what we are loading: [Single-Family Loan Performance Dataset and Credit Risk Transfer - Glossary and File Layout](https://capitalmarkets.fanniemae.com/media/6931/display).  This data isn't very usable in its current form, so we will use Excel to extract the data from PDF.  To extract the data from Excel follow these steps:
1. Download the [header PDF file](https://capitalmarkets.fanniemae.com/media/6931/display)
2. Open up a new, empty worksheet in Excel
3. Go to the `Data` tab
4. Find the `Get Data` button on the left of the ribbon
5. Navigate the `Get Data` menu to go to `From File` -> `From PDF`
6. Choose the PDF file to import from
7. Several options will be available on how to import the data, choose the one that works best for you
8. Double check that all columns from the file have been imported to Excel as some may have been cut off

Now, we can use this information to help import our data into BigQuery.


### Uploading Data to Google Cloud Storage
The data we've retrieved from Fannie Mae needs to be sent to Google Cloud Storage (GCS).  Once the 
1. Extract the zip file downloaded from Data Dynamics
2. Navigate to that folder in your console
3. Run the following command to upload all files:
```bash
 gsutil cp * gs://{your_bucket_name}/{cas_deal_name}
```
4. Now verify you can see your files in the [GCP Console](https://console.cloud.google.com/)

### Creating a BigQuery Table from CSVs
The next step is to create a single table for the multiple CSV files we just uploaded.  The difficult part is that we will need to specify a schema for the table.  We have to make sure the schema fits the data perfectly as there are no column headers in the CSV files.

The following SQL can be run in the BigQuery Console to load a single table from all the csv files.
```sql
LOAD DATA OVERWRITE {dataset_name}.{security_name}
(reference_pool_id STRING,
loan_identifier STRING,
...
)
FROM FILES (
  format = 'CSV',
  field_delimiter = '|',
  uris = ['gs://{your-bucket}/{folder}/*.csv']);
```
> To find an example with all the columns, checkout this [sql script](https://github.com/brandon-setegn/loan-performance-dbt/blob/master/sql/create_table_example.sql).
{: .prompt-tip }