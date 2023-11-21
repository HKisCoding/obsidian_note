## Definition
- **Features:** The measurable data that will be input for ML models 
- **Feature Engineering:** Transformation the source data into features
**Without feature store:**
![[Pasted image 20230213143417.png]]

=> **Feature Store:** 
- Central location where the features are stored and organized for the explicit purpose of being used to either train models or make inference. 
- Allow to create or update groups of features from multiple data sources or create and update new datasets from feaure group for training models 
- Use in application that do not want to compute features but only retrieve them when making predictions.
**Overall: System that store, intergrate and manage features**
![[Pasted image 20230213143633.png]]

## Use of feature store:
- Feature reuse
- Feature sharing
- Feature consistency: central database for training and serving
- Feature mornitoring: tracking the drift of data or features's quality
- Data leakage: quaranty no leakage data in trainset

## Feature store services
### Open sources:
---
[**FEAST:**][https://docs.feast.dev/]
Customizable operational data system that re-uses existing infrastructure to manage and serve machine learning features to realtime models.
![[Pasted image 20230213104422.png]]
#### Overview

>**Pros** 
>	- Make feaures consistently available for training and serving 
>	- Avoid data leakage
>	- Decouple ML from data infrastructure
>	- Manage data stored in other systems
>	- Highly customizable and modular: Feast blocks are loosely connected and can be used independently
>**Cons** 
>	- Rely primarily on structured data
>	- Not very low latency feature retrieval
>	- **Reproducible model training / model backtesting / experiment management**
>		Feast captures feature and model metadata, but does not version-control datasets / labels or manage train / test splits.
>	- **Batch + streaming feature engineering:** (Tecton)
>		Feast primarily processes already transformed feature values (though it offers experimental light-weight transformations). Users usually integrate Feast with upstream systems (e.g. existing ETL/ELT pipelines)
>	- **Native streaming feature integration:** (Tecton)
>		Feast enables users to push streaming features, but does not pull from streaming sources or manage streaming pipelines.
>	-**Data quality / drift detection:**
>		Feast has experimental integrations with [Great Expectations](https://greatexpectations.io/), but is not purpose built to solve data drift / data quality issues.
>	- **Feature sharing:** (tecton) Feast have some limitation
> **Feast is not:**
> - ETL/ELT system
> - Data orchestration tool 
> - Data warehouse
> - Database

#### Project structure:
![[Pasted image 20230213113540.png]]
- [Data Source](https://docs.feast.dev/getting-started/concepts/data-ingestion)
- [Entity](https://docs.feast.dev/getting-started/concepts/entity)
- [Feature view](https://docs.feast.dev/getting-started/concepts/feature-view)
- [Feature retrieval](https://docs.feast.dev/getting-started/concepts/feature-retrieval) 
#### Data ingestion:
> **Offline:**
> 	- No need to ingest data 
> 	- Query existing data directly
> 	- Pushing streaming feature to batch source
> **Online:**
> 	- Connect with Online Storage: Redis
> 	- Ingesting features from batch sources and pushing streaming features

#### Architecture:
![[Pasted image 20230213134606.png]]

> **Registry:** central catalog of all the feature definitions and their related metadata. Only support file-based registries.
> **Offline store:** Interface fore working with historical feature values that stored in data sources. Include: Files, Snowflake, Spark, PostgreSQL.
> 	- Building training dataset 
> 	- Materializing features into online srote at low-latency in production.
> **Online store:** For each each Entity key, only the lastest features are stored. Include: Redis, Firestore.

#### Running Feast on production
> 1. Automatically deploy changes to feature definitions
> 	- Setting up feature repo
> 	- Setting up database-backed registry
> 	- Setting up CI/CD to automatically update the registry: Auto run **feast plan** and **feast apply**
> 	- Setting up multiple ENV
> 2. Load data to online store (materialization) and keep it up to date
> 	- Scalable Materialization:
> 		- Feast's materialization process uses an in-process materialization engine which loads all the data into memory from offline store and writes into online store
> 		- Recommend to use: [Bytewax Materialization Engine](https://docs.feast.dev/reference/batch-materialization/bytewax), or the [Snowflake Materialization Engine](https://docs.feast.dev/reference/batch-materialization/snowflake).
> 		- Materialization engine can run on an Kubernetes Cluster and configure in **feature_store.yaml**
> 	- Scheduled materialization with Airflow: https://github.com/feast-dev/feast-workshop/blob/main/module_1/README.md#step-7-scaling-up-and-scheduling-materialization
> 	- Scheduled batch transformations with Airflow + dbt: https://github.com/feast-dev/feast-workshop/blob/main/module_3/
> 3. Feast for model training
> 	3.1. Generating training data
> 	- Create **FeatureStore** object with path to the registry.
> 	- Generate entity Dataframe:
> 		- Create manually
> 		- Use SQL query to dynamically generate list of entities and timestamps to pass into Feast
> 	- Start training: retrieve data with **get_historical_features()**\
> 	3.2. Versioning features that power ML models
> 	- Feature service name and model versions should have some established convention.
> 4. Retrieving online features for prediction
> 	4.1. Use **Python SDK**:
> 		- Connect directly to online store
> 		- Pull the feature data
> 		- Run transformations locally
> 	4.2. Deploy Feast feature server on Kubernetes
> 5. Using environment variables in yaml configuration
>![[Pasted image 20230214105528.png]]



