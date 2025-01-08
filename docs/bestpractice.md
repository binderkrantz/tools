# Databricks

## Delta Live Tables (DLT)

DLT is a <u>declarative</u> framework for building <u>reliable</u> pipelines.\
<u>Reliability</u> in pipelines can mean many different things. With DLT, you use SQL (Databricks flaired) to declare certain elements , and DLT aids by dealing with many of them. DLT is <u>declarative</u> in the sense that all the reliability is declared through SQL. DLT offers the following benefits[*](https://techcommunity.microsoft.com/t5/analytics-on-azure-blog/easier-data-model-management-for-power-bi-using-delta-live/ba-p/3500698):

* Declarative APIs to easily build your transformations and aggregations using SQL or Python
* Support for streaming to provide fresh and up-to-date data by only processing new data
* Have confidence in your data with built-in data quality testing and monitoring
* Automatically generated lineage graph between all tables
* Automated management and auto-scaling of the clusters to run the pipeline

### Ingestion (raw to bronze)

Use [streaming tables](https://docs.databricks.com/en/sql/language-manual/sql-ref-syntax-ddl-create-streaming-table.html)

* Ingestion is usually an append-only process anyway
* You often want to clean the data
* You don't want to go back in time and read it
* Raw data as text files shouldn't be accessed anyway (think GDPR)

## Transformation

Most often, use [materialized views](https://docs.databricks.com/en/sql/user/materialized-views.html)
* Pre-computes queries
