---
title:  "Data Transformations with BigQuery and DBT"
date:   2024-02-26 20:00:00 -0500
categories: [data warehouse, bigquery, dbt]
mermaid: true
tags: [dbt, bigquery, GCP]
image:
  path: /assets/img/bigquery-dbt.png
---

Cloud tools have reconstructed the data stack.  Data warehouses such as [Google BigQuery](https://cloud.google.com/bigquery) and [Snowflake](https://www.snowflake.com/en/) provide scalable solutions for storing, analyzing, and managing large volumes of data, making them essential tools for data-driven processes.  Previously, suites of tools existed to transform your data before loading it into your warehouse.  It was either too difficult or too costly to load raw into data warehouse.  Now, we can easily load data into the warehouse first, transforming inside the warehouse with [DBT (Data Build Tool)](https://www.getdbt.com/product/what-is-dbt).  Running SQL in BigQuery offers significant advantages over something like Spark, including fully managed infrastructure, automatic scaling, and the ability to run SQL queries directly on stored data without the need to move it.

> 💡**Before: Bring the data to the compute.  After: we bring the compute to the data.**

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
2. Select a deal that is at least 6 months old, a deal with 1-2 years of history will be large enough but not too large
3. Select Data Type - select only `Remittance Data`
4. Click `Generate Files` to download a zip file with the performance history

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

Now, we can use this information to help import our data into BigQuery.  This will be used later on.


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

Now our table has been created, query it to explore the data.
```sql
select * from {dataset_name}.{security_name} limit 100;
```

## DBT
[DBT (Data Build Tool)](https://www.getdbt.com/) is a powerful addition in the modern data stack, enabling analytics engineers to convert raw data in their warehouses into reliable, actionable information. It harnesses the familiar language of SQL, allowing users to define, test, and document their data transformations in a version-controlled environment, promoting collaboration and ensuring data accuracy.  With DBT, teams can concentrate more on extracting insights and less on managing data pipelines, speeding up the journey to data-driven decision making.

### Installation

1. Setup dbt environment:
Assuming python has been setup locally, create a virtual env
```shell
virtualenv dbt-project-env
source dbt-project-env/bin/activate
```
2. Install bigquery adapter for dbt, this will also install dbt-core
```shell
pip install dbt-bigquery
```
3. Initialize the dbt project and set `dbt_dv_data` as the project name and select `1`
to choose the postgres adapter
```shell
dbt init
```
4. Follow the prompts to configure DBT
5. Move files from newly created folder.  Unfortunately DBT creates everything in a subfolder which isn't usefull since its easier having it at the same level as the
```shell
mv {dbt_project_folder_created}/* .
rmdir {dbt_project_folder_created}
```
6. Test with `dbt debug`
```shell
dbt debug
```
If your connection to BigQuery is setup properly, you should see `All checks passed!`.


### Defining Our First Source
We will use the table we created before as our source.

> Work in progress...more coming soon.
{: .prompt-warning  }