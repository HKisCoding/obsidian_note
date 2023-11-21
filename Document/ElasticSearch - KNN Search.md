## 1. Create Index 
Create embedding properties with:
- type: 'dense_vector' / 'knn'
- index: true
- similarity metric 
```shell 
"properties":{
	"embedding":{
		"type": "dense_vector",
		"dims": dims,
		"index": True,
		"similarity": "cosine"
	}
}
```

- Can have multiple vectors properties
```shell 
properties: {
        "image-vector": {
          type: 'dense_vector',
          dims: 3,
          index: true,
          similarity: 'l2_norm'
        },
        "title-vector": {
          type: 'dense_vector',
          dims: 5,
          index: true,
          similarity: 'l2_norm'
        }
}
```

`index_options`: use to adjust the parameters when construct the vector space. Default: HNSW algorithm
## 2. Searching 
- Using `knn` for K-NN searching:
similarity is the threshold for return vector value
```python 
knn = {
            "field": "embedding",
            "query_vector": query_vector,
            "k": k,
            "num_candidates": num_candidates,
            "similarity": max_distance
        }
```
- Multi vector searching 
Using boost as weighting score 
```shell 
knn: [
      {
        field: 'image-vector',
        query_vector: [
          54,
          10,
          -2
        ],
        k: 5,
        num_candidates: 50,
        boost: 0.1
      },
      {
        field: 'title-vector',
        query_vector: [
          1,
          20,
          -52,
          23,
          10
        ],
        k: 10,
        num_candidates: 10,
        boost: 0.5
      }
    ]
```
=> **Score** = 0.1 * image_vector_score + 0.5 * title_vector_score
- Combine fields matching with knn search
Combine `query` with `knn` 
`knn` can be use with `aggregations` function -> calculated in combine set of `knn` and `query` match.
```shell
query: {
      match: {
        title: {
          query: 'mountain lake',
          boost: 0.9
        }
      }
    },
knn: {
      field: 'image-vector',
      query_vector: [
        54,
        10,
        -2
      ],
      k: 5,
      num_candidates: 50,
      boost: 0.1
    }
```

## 3. Tune KNN search 
https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-knn-search.html