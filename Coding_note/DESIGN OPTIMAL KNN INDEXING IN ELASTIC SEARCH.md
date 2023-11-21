
# Basic terms 
**Elasticsearch index** is a logical namespace that holds a collection of documents, where each document is a collection of fields — which, in turn, are key-value pairs that contain  data.

**Elasticsearch cluster** as a database that can contain many indices you can consider as a table, and within each index, you have many documents.

**A shard** is a physical partition of an index. Each shard is stored on a single node in the cluster.

-   RDBMS => Databases => Tables => Columns/Rows
-   Elasticsearch => Clusters => Indices => Shards => Documents with key-value pairs

# Storage
Elasticsearch is a distributed document store.
	- Text fields are stored in **inverted indices**
	- numeric and geo fields are stored in **BKD trees.**
	- Vecotor are stored in **HNSW** 
Shards have a primary and a replica shard, replicas provide redundant copies of data

# KNN ES plugin 

### Index Settings: 
`knn.algo_param.ef_search (int)`: The size of the dynamic list used during KNN searches. Higher values lead to more accurate, but slower searches.

`knn.algo_param.ef_construction (int)`: The size of the dynamic list used during KNN graph creation. Higher values lead to a more accurate graph, but slower indexing speed.

`knn.algo_param.m (int)`: The number of bidirectional links that the plugin creates for each new element. Increasing and decreasing this value can have a large impact on memory consumption. **Keep this value between 2-100.**

`knn.space_type`: The vector space used to calculate the distance between vectors. Currently, the KNN plugin supports the `l2` space (Euclidean distance) and `cosinesimil` space (cosine similarity).

```python
from elasticsearch import Elasticsearch

es = Elasticsearch()

mapping = {

	'settings' : {
	
		'index' : {
		
			'knn': True,
			
			'knn.algo_param' : {
			
				'ef_search' : 256,
				
				'ef_construction' : 128,
				
				'm' : 48
			
			},
			
			'refresh_interval': -1,
			
			'translog.flush_threshold_size': '10gb',
			
			'number_of_replicas': 0
		
		},
	
	},

	'mappings': {
	
		'properties': {
		
			'fvec': {
			
				'type': 'knn_vector',
				
				'dimension': dim
				
			}
			
		}
		
	}

}

res = es.indices.create(index=idx_name, body=mapping, ignore=400)
```

## Cluster Settings:
`knn.algo_param.index_thread_qty (int)`: The number of threads used for graph creation. Keeping this value low reduces the CPU impact of the KNN plugin, but also reduces indexing performance.

`knn.memory.circuit_breaker.enabled` : enable the KNN memory circuit breaker.

`knn.memory.circuit_breaker.limit`: The native memory limit for graphs. At the default value, if a machine has 100 GB of memory and the JVM uses 32 GB, the k-NN plugin uses 50% of the remaining 68 GB (34 GB). If memory usage exceeds this value, KNN removes the least recently used graphs.

`knn.plugin.enabled`: Enables or disables the KNN plugin.

```python 
res = es.cluster.put_settings({'persistent': {'knn.algo_param.index_thread_qty': 2}})
```


