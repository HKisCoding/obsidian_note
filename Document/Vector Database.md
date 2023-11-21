---
quickshare-date: 2023-07-10 09:33:11
quickshare-url: "https://noteshare.space/note/cljw910j3147801pjaltmthjz#67O6GHelWAr4VUUQ1BNebcodYgZ8HZkGPAXNDqAgvXg"
---
# Overview
**A vector database** is a specialized DBMS that stores vector embeddings utilizing innovative techniques for storage, indexing, and query processing. They offer data management capabilities, such as CRUD and language bindings.
Vector database provide:
- **Data management:** Vector databases offer well-known and easy-to-use features for data storage, like inserting, deleting, and updating data. This makes managing and maintaining vector data easier
- **Metadata storage and filtering:** Vector databases can store metadata associated with each vector entry. Users can then query the database using additional metadata filters for finer-grained queries.
- **Scalability:** Vector databases are designed to scale with growing data volumes and user demands, providing better support for distributed and parallel processing.
- **Real-time updates:** Vector databases often support real-time data updates, allowing for dynamic changes to the data
- **Backups and collections:** Vector databases handle the routine operation of backing up all the data stored in the database.
- **Ecosystem integration**
- **Data security and access control:** Vector databases typically offer built-in data security features and access control mechanisms to protect sensitive information
![[Pasted image 20230710095405.png]]
### Query in Vector Database
A vector database uses a combination of different algorithms that all participate in Approximate Nearest Neighbor (ANN) search. These algorithms optimize the search through hashing, quantization, or graph-based search.

Common pipeline of vector database:
![[Pasted image 20230710095627.png]]
-   **Indexing**: The vector database indexes vectors using an algorithm such as PQ, LSH, or HNSW (more on these below). This step maps the vectors to a data structure that will enable faster searching.
    
-   **Querying**: The vector database compares the indexed query vector to the indexed vectors in the dataset to find the nearest neighbors (applying a similarity metric used by that index)
    
-   **Post Processing**: In some cases, the vector database retrieves the final nearest neighbors from the dataset and post-processes them to return the final results. This step can include re-ranking the nearest neighbors using a different similarity measure.

Every vector stored in the database also includes metadata. Vector database can also filter result based on a metadata query --> apply metadata filtering either before and after vector search 
![[Pasted image 20230710095907.png]]

# Ranking open source vector db
According to DB-Engines Ranking of Vector DBMS
![[Pasted image 20230707161054.png]]

## 1. Chroma
![[Pasted image 20230707143919.png]]

###  [Introduce](https://www.trychroma.com/)
Chroma is a database for building AI applications with embeddings. It comes with everything you need to get started built in, and runs on your machine.

Chroma offerings:
-   store embeddings and their metadata
-   embed documents and queries
-   search embeddings

Suport: `python` client SDK and `JavaScript` client SDK and server application
In Python, Chroma can run `in-memory` or in `client/server` (in alpha) mode.
> [!tip] Install Chroma
> ```python
> pip install chromadb
> ```

### Usage
Chroma can be save and load from local machine (from file) or use an on-disk database

Chroma manage collections of embeddings by `Creating`, `Inspecting` and `Deleting`

Chroma collections are created with a name and an optional embedding function.
```python
collection = client.create_collection(name="my_collection", embedding_function=emb_fn)
collection = client.get_collection(name="my_collection", embedding_function=emb_fn)
```
By using collection, Chroma let users:
- add data to collection
- query collection
- choosing data to return
- filter by metadata.
- filter by document content
Ref: [usage-guide](https://docs.trychroma.com/usage-guide)

### Similarity metrics:
Chroma provide [HNSW](https://github.com/nmslib/hnswlib) - fast approximate nearest neighbor search with 3 metrics:
- L2: "l2"
- Inner product: "ip"
- Consine: "cosine"
Setting the metric by customize `hnsw:space` in `metadata` argument 
```python
 collection = client.create_collection(
        name="collection_name",
        metadata={"hnsw:space": "cosine"} # l2 is the default
    )
```

### Conclusion:
- Pros:
	- Easy to use: [cheatsheet](https://docs.trychroma.com/api-reference)
	- Support deployment in many environments
	- Intergrate with Langchains, LlamaIndex (GPT-index)
	- Light: Chroma and its underlying database need at least 2gb of RAM
- Cons:
	- Not benchmark yet 
	- Chroma Server is currently in Alpha

## 2. Milvus

### [Introduce](https://milvus.io/docs/overview.md)
Milvus was created in 2019 with a singular goal: store, index, and manage massive [embedding vectors](https://milvus.io/docs/overview.md#Embedding-vectors) generated by deep neural networks and other machine learning (ML) models.

As a database specifically designed to handle queries over input vectors, it is capable of indexing vectors on a trillion scale. Unlike existing relational databases which mainly deal with structured data following a pre-defined pattern, Milvus is designed from the bottom-up to handle embedding vectors converted from [unstructured data](https://milvus.io/docs/overview.md#Unstructured-data).

Milvus also supports data sharding, data persistence, streaming data ingestion, hybrid search between vector and scalar data, time travel, and many other advanced functions. The platform offers performance on demand and can be optimized to suit any embedding retrieval scenario.

Milvus adopts a shared-storage architecture featuring storage and computing disaggregation and horizontal scalability for its computing nodes.

Milvus comprises [four layers](https://milvus.io/docs/four_layers.md): access layer, coordinator service, worker node, and storage. These layers are mutually independent when it comes to scaling or disaster recovery.

![[Pasted image 20230707162704.png]]

### Installment
Milvus offers installment Standalone and Cluster with Docker Compose or with Kubernetes
For offline installment, Milvus support Helm charts in offline environment
Support: `python` `java` `go` `NodeJs` SDK
> [!tip] Install Milvus
> ```python
> python -m pip install pymilvus==2.2.x
> ```

### Storage
Mivus stores snapshot files of logs, index files for scalar and vector data, and intermediate query results on Object Storage such as MinIO. 

To improve its performance and lower the costs, Milvus plans to implement cold-hot data separation on a memory- or SSD-based cache pool.

Mivus uses LogBroker for streaming data persistence, execution of reliable asynchronous queries, event notification, read-write disaggregation and return of query results. Supported platforms: Pulsar, RockDB, Kafka
> [!note] LogBroker
> The log broker is a publish-subscribe system that supports playback. It is responsible for streaming data persistence, execution of reliable asynchronous queries, event notification, and return of query results. It also ensures integrity of the incremental data when the worker nodes recover from system breakdown.

### Usage
Milvus provide management on: Connection, Database, Collection, Partitions, Data, Indexes.

Collection in Milvus consits of one or more partitions. User can define Data Schema when create Collection.

Milvus allows users to load a collection as multiple replicas to utilize the CPU and memory resources of extra query nodes. This feature boosts the overall QPS and throughput without extra hardware. Before loading a collection, ensure that collection have already been indexed.

### Similarity metrics:
Milvus provide L2 and Inner Product similarity .

Index help improve query performence, Most of the vector index types supported by Milvus use approximate nearest neighbors search (ANNS) algorithms.

Milvus index support `In-memory` and `On-disk`

![[Pasted image 20230707172320.png]]

### Conclusion:
- Pros:
	- Designed for similarity search on dense vector datasets containing millions, billions, or even trillions of vectors.
	- Detailed document
	- Offer multiple indexings.
	- Feature storage with capable of indexing vectors on a trillion scale
	- Support deployment on multi env
	- Intergrate with multi platform: OpenAI, Cohere, HuggingFace, Pytorch, LlamaIndex, LangChain
	- Provide Mornitoring and visualize dashboard
-  Cons: None

## 3. Qdrant
[Document](https://qdrant.tech/documentation/quick-start/)
Almost alike Milvus except some:
- Qdrant currently only uses HNSW as a vector index.
- Schema free
- Qdrant store and run query on RAM and on disk with Memap with RockDB
	- **In-memory storage** - Stores all vectors in RAM, has the highest speed since disk access is required only for persistence.

	- **Memmap storage** - Creates a virtual address space associated with the file on disk. [Wiki](https://en.wikipedia.org/wiki/Memory-mapped_file). Mmapped files are not directly loaded into RAM. Instead, they use page cache to access the contents of the file. This scheme allows flexible use of available memory. With sufficient RAM, it is almost as fast as in-memory storage.
- Qdrant provides Quantizationto reduce the memory footprint and accelerate the search process in high-dimensional vector spaces. 
> [!note]
> Quantization is an optional feature in Qdrant that enables efficient storage and search of high-dimensional vectors. By transforming original vectors into a new representations, quantization compresses data while preserving close to original relative distances between vectors. Different quantization methods have different mechanics and tradeoffs.

### Compare with Milvus:
All the test and benchmark are in this [link](https://qdrant.tech/benchmarks/?gad=1&gclid=Cj0KCQjw756lBhDMARIsAEI0AgmtwrvAJ4o-GAErhty2Ok-I1M9_Gwy5gIYkB1eYvGpNy3zORnYr0CAaAuT6EALw_wcB)


