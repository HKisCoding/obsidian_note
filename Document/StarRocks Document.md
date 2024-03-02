
## Introduction
**StarRocks** 
- massively parallel processing (MPP) database
- uses a columnar storage format
- Special features:
	- Indexing like OLTP
	- Fully vectorized engine
	- newly designed cost-based optimizer (CBO)
	- Fast query especially for multi-table joins.
	- Create tables that use various schemas, such as flat, star, and snowflake schemas.

## Architecture
Storage-compute separation architecture
- Data storage is separated and stored in remote object storage or HDFS
- CN nodes are used for caching hot data to accelerate queries
![[Pasted image 20240105154543.png]]

Data Management 
- column-oriented database system
- uses the partitioning and bucketing mechanism to manage data
- Data in a table is defined into multiple partitions then multiple tablets -> schedule one SQL statement to all the tablets for parallel processing
- Data will be automatically migrated if new BE added or removed

## Feature 
### MPP framework 
- One query request is split into multiple physical computing units that can be executed in parallel on multiple machines
- Distributes the load across the entire cluster.
- For complex queries, such as Group By on high-cardinality fields and large table joins, StarRocks' MPP framework has noticeable performance advantages over the Scatter-Gather framework.
- ![[Pasted image 20240108005503.png]]

### Fully vectorized execution engine 
### Cost-based optimizer 
StarRocks designs a brand-new [CBO](https://docs.starrocks.io/docs/using_starrocks/Cost_based_optimizer/) from scratch.
The CBO enables StarRocks to deliver better multi-table join query performance than competitors, especially in complex multi-table join queries.

**CBO:**
The CBO rewrites and transforms the logical plan into multiple physical execution plans. Then estimates the execution cost of each operator in the plan (such as CPU, memory, network, and I/O) and chooses a query path with the lowest cost as the final physical plan.

### Real time, updatable columnar storage engine 

### Intelligent materialized view 
StarRocks' materialized views automatically update data according to the data changes in the base table without requiring additional maintenance operations.
StarRocks' MV can replace the traditional ETL data modeling process: Instead of transforming data in the upstream applications, transform data with MV within StarRocks, simplifying the data processing pipeline.
When executing a query on base tables with materialized views built on, the system automatically judges whether the pre-computed results in the materialized view can be reused for the query.

Usage: 
#### Synchronous materialized view:
- All changes in the base table are simultaneously updated to the corresponding synchronous materialized views
- The refresh of a synchronous materialized view is triggered automatically.
- Can be created only on a single base table from [the default catalog](https://docs.starrocks.io/docs/data_source/catalog/default_catalog/)
#### Asynchronous materialized view:
An asynchronous materialized view is a special physical table that holds pre-computed query results from one or more base tables.

- Support multi-table join and more aggregate functions.
- The refresh of asynchronous materialized views can be triggered manually or by scheduled tasks.
- When perform complex queries on the base table, StarRocks returns the pre-computed results from the relevant materialized views to process these queries.
- 

**Current situation:** Running batch job to run query and create View at a certain time -> Require a mechanism to auto calculate accumulate on day, month, ... dimension

### Data lake analytics 
StarRocks can work as the compute engine to analyze data stored in [data lakes](https://docs.starrocks.io/docs/data_source/catalog/catalog_overview/) such as Apache Hive, Apache Iceberg, Apache Hudi, and Delta Lake.
Capability to query external data sources seamlessly, eliminating the need for data migration

## OLAP DESIGN
### 1. Table types
4 table types: 
#### 1.1. Duplicate Key table: 
Load log data or time-series data. New data is written in append-only mode, and existing data is not updated.
#### 1.2. Aggregate table: 
- Define sort key columns and metric columns and can specify an aggregate function for the metric columns
- If the records to be loaded have the same sort key, the metric columns are aggregated.
- Starting from data ingestion to data querying, data with the same sort key in a table that uses the Aggregate table is aggregated multiple times as follows:

	1. In the data ingestion phase, when data is loaded as batches into the table, each batch comprises a data version. After a data version is generated, StarRocks aggregates the data that has the same sort key in the data version.
	2. In the background compaction phase, when the files of multiple data versions that are generated at data ingestion are periodically compacted into a large file, StarRocks aggregates the data that has the same sort key in the large file.
	3. In the data query phase, StarRocks aggregates the data that has the same sort key among all data versions before it returns the query result.
	
#### 1.3.  Unique Key table
Queries return the most recent record among a group of records that have the same primary key -> better support real-time and frequent data updates.

#### 1.4. Primary Key table
Queries return the most recent record among a group of records that have the same primary key
Primary Key table does not require aggregate operations during queries and supports the pushdown of predicates and indexes.
**Use case:**
- Stream data in real time from transaction processing systems into StarRocks
- Join multiple streams by performing update operations on individual columns:
	The Primary Key table is well suited in these scenarios, because it supports updates to individual columns. Each app or system can update only the columns that hold the data within its own service scope while benefiting from real-time data additions, deletions, and updates at high query performance.

### 2. Sort keys
When data is loaded into a table created by using a certain table type, data is sorted and stored according to one or more columns defined as the sort key when the table is created.
- Duplicate Key table: `DUPLICATE KEY` + not assigned a UNIQUE constraint
- Aggregate table: `AGGREGATE KEY` + assigned a UNIQUE constraint
- Unique key table: `UNIQUE KEY` + assigned a UNIQUE constraint
- Primary key table: `PRIMARY KEY` + assigned a UNIQUE and NOT NULL constraint

