> A [proprietary](https://www.databricks.com/product/delta-live-tables) feature by Databricks. It enables a more unified control over your pipelines. DLT is simply implemented with some declarative syntax (on top of some steps to transform data). It serves to basically link pieces of a pipeline together, and thus the linkage creates a pipeline that is more reliable, maintainable, and testable. GREAT! Highly [similar to dbt](https://www.reddit.com/r/dataengineering/comments/vsb309/comparing_dbt_with_delta_live_tables_for_doing/) but slightly more vendor-locked, albeit [each tool has their place](https://community.databricks.com/t5/data-engineering/what-are-the-advantages-of-using-delta-live-tables-dlt-over-data/td-p/4210).

### DLT datasets
The resulting data following a transformation using DLT must be defined as either of three; a *streaming table*, *materialized view* or a *view*. Each type defines how the records are processed through the associated transformation.[\*](https://learn.microsoft.com/en-us/azure/databricks/delta-live-tables/#--what-are-delta-live-tables-datasets)

|Dataset type|How are records processed through defined queries?|
|---|---|
|Streaming table|Each record is processed exactly once. This assumes an append-only source.|
|Materialized views|Records are processed as required to return accurate results for the current data state. Materialized views should be used for data sources with updates, deletions, or aggregations, and for change data capture processing (CDC).|
|Views|Records are processed each time the view is queried. Use views for intermediate transformations and data quality checks that should not be published to public datasets.|
#### Streaming tables
A streaming table is meant for incremental data processing where the source is append-only.[\*](https://learn.microsoft.com/en-us/azure/databricks/delta-live-tables/#streaming-table) This means that a streaming table only builds up over time.

>While streaming tables are always defined against streaming sources, you can also use streaming sources with `APPLY CHANGES INTO` to apply updates from CDC feeds. See [Change data capture with Delta Live Tables](https://learn.microsoft.com/en-us/azure/databricks/delta-live-tables/cdc).

A streaming table is ideal for ingested-to datasets (like a bronze-table) where the source grows over time. A good example would be a data lake where new files arrive continuously (this further promotes use of the [[Auto Loader]] feature)

Consider using a streaming table when:
- A query is defined against a data source that is continuously or incrementally growing.
- Query results should be computed incrementally.
- High throughput and low latency is desired for the pipeline.

#### Materialized views
A materialized view (aka *live table*) is a view where results have been precomputed.[\*](https://learn.microsoft.com/en-us/azure/databricks/delta-live-tables/#materialized-view) 

#### when to use the different datasets
https://learn.microsoft.com/en-us/azure/databricks/delta-live-tables/transform#tables-vs-views
