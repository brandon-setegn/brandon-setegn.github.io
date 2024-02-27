---
title:  "Data Transformations with BigQuery and DBT"
date:   2024-02-26 20:00:00 -0500
categories: [data warehouse, bigquery, dbt]
mermaid: true
tags: [dbt, bigquery, GCP]
image:
  path: /assets/img/bigquery-dbt.png
---

  Cloud tools have reconstructed the modern data stack.  Data warehouses such as [Google BigQuery](https://cloud.google.com/bigquery) and [Snowflake](https://www.snowflake.com/en/) provide scalable solutions for storing, analyzing, and managing large volumes of data, making them essential tools for data-driven processes.  Previously, suites of tools existed to transform your data before loading it into your warehouse.  It was either too difficult or too costly to load raw into data warehouse.  Now, we can easily load data into the warehouse first, tranforming inside the warehouse with [DBT (Data Build Tool)](https://www.getdbt.com/product/what-is-dbt).

  > 💡**Old: Bring the data to the compute.  New: we bring the compute to the data.**

Included below is an example of how to load data into BigQuery and transform it with DBT.  [Historical Loan Performance data from Fannie Mae](https://capitalmarkets.fanniemae.com/credit-risk-transfer/single-family-credit-risk-transfer/fannie-mae-single-family-loan-performance-data) will be used.  It will be loaded into BigQuery in its raw form and transformed with SQL generated from DBT to make it usable.

## Google BigQuery
### Prerequisites
If you would like to try this example yourself, you must first setup BigQuery in Google Cloud Platform (GCP).
> Follow this guide to get started with BigQuery: [Load and query data with the Google Cloud console](https://cloud.google.com/bigquery/docs/quickstarts/load-data-console)

### Downloading Example Data

The data used in this example comes from Fannie Mae.  It will use the historical loan performance data available through Fannie Mae's Data Dynamics web portal.  This will require you to create an account with Fannie Mae.  Once you've created an account and logged in, you should see 

#### Steps to Download Data
![Fannie Mae Download](/assets/img/2024-02-26-FannieMae-HistoricalLoanCreditData.png){: width="624" height="256" }
_Data Dynamics_
1. Go to [Fannie Mae - Data Dynamics](https://datadynamics.fanniemae.com/data-dynamics/#/downloadLoanData/Single-Family)
2. Scroll down to the latest year (2023 at the time of writing this)
3. Download all files for that year

#### Column Headers
Column headers are not included with the data file.  They can be found here: [Single-Family Loan Performance Dataset and Credit Risk Transfer - Glossary and File Layout](https://capitalmarkets.fanniemae.com/media/6931/display).  This data isn't very usuable in its current form, so we will use Excel to extract the data from PDF.  To extract the data from Excel follow these steps:
1. Download the [header PDF file](https://capitalmarkets.fanniemae.com/media/6931/display)
2. Open up a new worksheet in Excel
3. Go to the `Data` tab
4. Find the `Get Data` button on the left
5. Navigate the `Get Data` menu to go to `From File` -> `From PDF`
6. Choose the PDF file to import from
7. Several options will be available on how to import the data, choose the one that works best for you
8. Double check that all columns from the file have been imported to Excel as some may have been cut off



> An example showing the `tip` type prompt.
{: .prompt-tip }