---
title:  "Data Transformations with Databricks and DBT"
date:   2024-02-26 20:00:00 -0500
categories: [data warehouse, databricks, dbt]
mermaid: true
tags: [dbt, bigquery, GCP]
image:
  path: /assets/img/databricks-dbt.png
---

In my previous post, [Bigquery with DBT](/posts/BigQuery-with-DBT/), we saw the power of using DBT with Google BigQuery to perform our data transformations.  But BigQuery is just one of many big data solutions currently available.  Databricks is a another highly popular big data solution that offers a powerful data warehousing toolset. It provides a unified analytics platform that combines data engineering, data science, and business analytics, making it a versatile choice for data-driven organizations.

Databricks SQL allows us to interact with our data with ordinary SQL (ANSI SQL) meaning we can use DBT to perform our transformations.  Databricks has many different tools for building your data pipeline.  While DBT may not be your first choice in the Databricks ecosystem it is still viable option.  This will also help example us compare Databricks with BigQuery which is important when deciding which cloud data warehouse to choose.

## Setup

I will only briefly go over some of the steps required to setup the sample data in Databricks for this example.  I used the same data as the BigQuery with DBT post and simply moved it over to Azure Blob Storage.  To obtain the CSV used in this example, follow the instructions in the previous post: [Steps to Download Data](/posts/BigQuery-with-DBT/#downloading-example-data).

### Cloud Costs
> Databricks is very expensive!  If you do spin up your own Databricks watch your costs very closely.  Make sure to set your clusters to `auto stop` after 30 minutes.  Also, set budget alerts to make sure you're not spending more than you think.
{: .prompt-danger }

I used Azure Databricks, taking advantage of the free trial for both both Azure and Databricks separately.  I was able to keep costs well within the $200 trial amount for Azure.  The Databricks trial is only 14 days and they give you very little insight into how much you are spending.  Make sure to cancel the trial before you are charged.


### Creating our Bronze Table
Once the data is uploaded to Azure Blob Storage we can access it from Databricks.  There are many different ways to load your data into Databricks.  I chose to connect directly to Azure from Databricks Spark as this was the simplest at the time.  It took several minutes to load the data and is most likely not the most performant option.  

> The following `Python` snippets were run in a Databricks notebook.
{: .prompt-info }

#### Setup Databricks Spark to Connect to Azure Blob Storage
The `wasb(s)` driver is being retired, please read more about the proper way to connect here: [Connect to Azure Data Lake Storage](https://learn.microsoft.com/en-us/azure/databricks/connect/storage/azure-storage).  The storage account keys can be found on the Storage Account page in Azure.  They are specific to the storage account you placed the CSV files in and not tied to a user.

![Storage Account](/assets/img/databricks-dbt-storage-account-keys.png){: width="264" height="632" }
_Storage Account_

The key can be put into Databricks secrets or copied directly into your notebook (which is dangerous).
```python
# These settings are needed to connect to Databricks
storage_account_name = "{your storage account name}"
container_name = "{your container name}"

base_path = f"wasbs://{container_name}@{storage_account_name}.blob.core.windows.net/cas/cas_csvs/"

# I stored my Azure Blob Storage Key in Databricks Secrets
# You can set this directly to your key (just a string), but that is not secure
storage_account_access_key = dbutils.secrets.get(scope="{your-scope}", key="{your-secret-key}")
file_type = "csv"
```

Those settings are then used to update our spark config.
```python
spark.conf.set(
  "fs.azure.account.key." + storage_account_name + ".blob.core.windows.net",
  storage_account_access_key)
```

Now we can test if we are connected to our storage account.  This should show the files in our blob storage if the setting we had were correct.
If it fails, try double checking your Azure settings and key.
```python
dbutils.fs.ls(base_path)
```

Next, we create our schema similiar to how we did when loading BigQuery.  However, this time we will use PySpark.
To view the whole schema, checkout the file in my Github Repo: [example-schema.py](https://github.com/brandon-setegn/loan-performance-dbt/blob/master/loanperf_databricks/example-schema.py).
```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, DecimalType

# Define schema
schema = StructType([
    StructField("reference_pool_id", StringType()),
    StructField("loan_identifier", StringType()),
    StructField("monthly_reporting_period", StringType()),
    ...
```

Next, we use our schema and Azure path to load the CSVs into a dataframe.  This may take several minutes.
```python
df = spark.read.format(file_type) \
  .schema(schema) \
  .option("header", "false") \
  .option("delimiter", "|") \
  .load(base_path)
```

Now, we can save our table to the Unity Catalog of our choice.  This will allow us to query it later without running this notebook using only a SQL Warehouse in Databricks.
```python
df.write.saveAsTable(f"{catalog}.{schema}.cas_deals")
```


