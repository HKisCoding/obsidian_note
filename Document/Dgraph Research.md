**Dgraph** is the native graph database with native GraphQL support 

## 1. Architecture 
### 1.1. Overview
Dgraph runs as a cluster of server nodes which communicate to form a single data store.
2 main types of processess nodes in Dgraph: **Zeros** and **Alphas**
- Dgraph Zeros: hold metadata, coordinate distributed transaction, re-balance data among server group.
- Dgraph Alpha: store graph data and indices. Dgraph alphas store and index “predicates” which represent the relations among data elements. This unique indexing approach allows Dgraph to perform a database query with depth N in only N network hops
![[Pasted image 20230414163309.png]]

### 1.2. Dgraph Zeros
- Controls the Dgraph cluster and stores informations about it
- Automatically moves data between different Dgraph Alphas based on the size of the data served by each instances. Zero monitors the space occupied by predicates in each group and moves predicates between groups as-needed to rebalance the cluster.
- For high-availability, Dgraph recommended to runs with 3 zeros and 3 alphas
- **Endpoints:** Zero exposes HTTP on port 6080 (default). To get the informations about the nodes that are part of the cluster,includes information about the size of predicates and which groups they belong to => Add `/state` endpoints in GET request.  
Source: https://dgraph.io/docs/deploy/dgraph-zero/

### 1.3. Nodes and edges
- `Nodes`: concepts or entities 
- `Edges`: relationship between 2 graphs. 
- `Predicates`: Associated properties among nodes. Edges also defined as a predicate
**In the clashflow projects:**
Create the graph for smartpay transaction: P2P, S2P, CASHIN, CASHOUT, Smartpos card transaction.
- Nodes: Smartpay user or banker and device 
- Predicates:
	- Id: prefix + phone number(smp)/customerCode(unknown banker)
	- Name:
		- Smartpay user: INDIVIDUAL/MERCHANT
		- Cashin/cashout bank: bank code 
		- Smartpos card: bank code 
	- Label: smp/bank/smpcard
- Edges: Transaction among nodes. Attributes:
	- txn_date: year-month level
	- txn_amount: total txn amount in a month

### 1.4. UID
Each nodes in graph is created with a unique ID called UID (an index encoded in hexa). 
UID can be used to query and mutate the graph.
Dgraph automatically assign UID to every nodes but allow users to manually set the UID in a specific range by adding endpoints in dgraph zeros http:
`/assign?what=uids&num=100`: allocates a range of UIDs specified by the `num` argument, and returns a JSON map containing the `startId` and `endId` that defines the range of UIDs (inclusive). This UID range can be safely assigned externally to new nodes during data ingestion.

## 2. Loading graph 
### 2.1. Bulk loader 
- Can only be used to load data into a new cluster 
- Recommended to perform initial import of large datasets into Draph.
- CLI: 
```shell 
$dgraph bulk --help
$dgraph bulk -f <path-to-gzipped-RDF-or-JSON-file> ... 
$dgraph bulk -f <./path-to-gzipped-RDF-or-JSON-files> ... 
$dgraph bulk -f <file1.rdf, file2.rdf> ... -s schema.schema --reduce_shards=2  --http host:8000 --zero host:port
```

- `--reduce_shards` : The number of Alpha groups
- `--map_shards`: higher number helps the bulk loader evenly distribute predicates between the reduce shards.
- ![[Pasted image 20230414171355.png]]

- Bulk loader utilises the map/shuffle/reduce paradigm.
![[Pasted image 20230414172718.png]]

**For more detail description:** https://dgraph.io/blog/post/bulkloader/

### 2.2. Live loader 
Bulk loader can only used when Dgraph Alpha not starting -> using live loader to batch loading data when Dgraph Alpha is running
> $ `dgraph live -a IP:9080 -z zeros_IP:5080 -f file.rdf -s schema.schema`

**Loading data instruction:** https://discuss.dgraph.io/t/how-should-i-loading-data/10006

### 2.3. RDF format 
Basic formant:
```RDF
<_:00x0989050365> <id> "00x0989050365" .
<_:00x0989050365> <name> "INDIVIDUAL" .
<_:00x0989050365> <label> "SMP" .

<_:01x0989050365> <id> "01x0989050365" .
<_:01x0989050365> <name> "bank" .
<_:01x0989050365> <label> "VPB" .

<_:00x0989050365> <CASHOUT> <_:01x0989050365> (txn_date = "202301", total_amount = 100000) .
```
`_:` : define that Dgraph will assign UID automatically.

### 2.4. Schema 
Dgraph can automatically define the schema base on RDF/JSON file loader to assign data type for predicates.
Dgraph maintains a list of all predicates names and types in the Dgraph schema.
**Data type**: 
![[Pasted image 20230416152412.png]]

Users can create own defined schema in DQL base on GraphQL API format 
Similarly, GraphQL mutations are implemented on top of DQL in the sense that a GraphQL query is converted internally into a DQL query, which is then executed.
```go
type Product {
   productID: ID!
   name: String @search(by: [term])
   reviews: [Review] @hasInverse(field: about)
}

type Customer {
   username: String! @id @search(by: [hash, regexp])
   reviews: [Review] @hasInverse(field: by)
}

type Review {
   id: ID!
   about: Product!
   by: Customer!
   comment: String @search(by: [fulltext])
   rating: Int @search
}
```

When deploying a GraphQL Schema, Dgraph will generates DQL predicates and types for the graph backend.

![[Pasted image 20230416154441.png]]

**Predicates Indexing:**
Filtering on a predicate by applying a [function](https://dgraph.io/docs/query-language/functions/) requires an index.
![[Pasted image 20230416160208.png]]
Specify index using `@index(tokenizer)` 

## 3. Query design 

### 3.1. Posting list 

Posting lists are a form of inverted index, a posting list is a list of all triples that share a `<subject>+<predicate>` pair.
A posting list in Dgraph for a node is an unorder set of `object` for each `predicate` 

Examples:
person1UID+friend->[person2UID, person4UID]
person2UID+friend->[person1UID]
person3UID+friend->[person2UID, person4UID]
person4UID+friend->[person1UID, person2UID, person3UID]

Posting lists are the unit of data access and caching in Dgraph. The underlying key-value store stores and retrieves posting lists as a unit. Queries that access larger posting lists will use more cache and may incur more disk access for un-cached posting lists.

### 3.2. Query processing 

```json
{
    me(func: uid(0x1)) {
      rel_A
      rel_B {
        rel_B1
        rel_B2
      }
      rel_C {
        rel_C1
        rel_C2 {
          rel_C2_1
      }
      }
  }
}
```

Examples query above specify the uid list to start fisrt (uid: 0x1) and retrieve the posting list for `rel_a`, `rel_b`, `rel_c` 
From uid result of each relation, recursively retrieve the posting lists as deep as the query requires.

### 3.3. Query function 

**A DQL query has**

-   an optional parameterization, ie a name and a list of parameters
-   an opening curly bracket
-   at least one [query block](https://dgraph.io/docs/dql/dql-syntax/dql-query/#query-block), but can contain many blocks
-   optional var blocks
-   a closing curly bracket
![[Pasted image 20230416165849.png]]

**A query block**

-   must have name
-   must have a node criteria defined by the keyword `func:`
-   may have ordering and pagination information
-   may have a combination of filters (to apply to the root nodes)
-   must provide the list of attributes and relationships to fetch for each node matching the root nodes.

**Function**
Root criteria and filters are using [functions](https://dgraph.io/docs/query-language/functions/) applied to nodes attributes or variables.

Dgraph offers functions for

-   testing string attributes
    -   term matching : [allofterms](https://dgraph.io/docs/query-language/functions/#allofterms), [anyofterms](https://dgraph.io/docs/query-language/functions/#anyofterms)
    -   regular Expression : [regexp](https://dgraph.io/docs/query-language/functions/#regular-expressions)
    -   fuzzy match : [match](https://dgraph.io/docs/query-language/functions/#fuzzy-matching)
    -   full-text search : [alloftext](https://dgraph.io/docs/query-language/functions/#full-text-search)
-   testing attribute value
    -   equality : [eq](https://dgraph.io/docs/query-language/functions/#equal-to)
    -   inequalities : [le,lt,ge,gt](https://dgraph.io/docs/query-language/functions/#less-than-less-than-or-equal-to-greater-than-and-greater-than-or-equal-to)
    -   range : [between](https://dgraph.io/docs/query-language/functions/#between)
-   testing if a node
    -   has a particular predicate (an attribute or a relation) : [has](https://dgraph.io/docs/query-language/functions/#has)
    -   has a given UID : [uid](https://dgraph.io/docs/query-language/functions/#uid)
    -   has a relationship to a given node : [uid_in](https://dgraph.io/docs/query-language/functions/#uid_in)
    -   is of a given type : type()
-   testing the number of node relationships
    -   equality : [eq](https://dgraph.io/docs/query-language/functions/#equal-to)
    -   inequalities : [le,lt,ge,gt](https://dgraph.io/docs/query-language/functions/#less-than-less-than-or-equal-to-greater-than-and-greater-than-or-equal-to)
-   testing geolocation attributes
    -   if geo location is within distance : [near](https://dgraph.io/docs/query-language/functions/#near)
    -   if geo location lies within a given area : [within](https://dgraph.io/docs/query-language/functions/#within)
    -   if geo area contains a given location : [contains](https://dgraph.io/docs/query-language/functions/#contains)
    -   if geo area intersects a given are : [intersects](https://dgraph.io/docs/query-language/functions/#intersects)

**variable(var) blocks** 
- Variable blocks (`var` blocks) start with the keyword `var` instead of a block name.
- Used to compute [query-variables](https://dgraph.io/docs/query-language/query-variables/) which are lists of node UIDs, or [value-variables](https://dgraph.io/docs/query-language/value-variables/) which are maps from node UIDs to the corresponding scalar values.

**Summarizing functions**

- When dealing with array attributes or with relationships to many node, the query may use summary functions [count](https://dgraph.io/docs/query-language/count/) , [min](https://dgraph.io/docs/query-language/aggregation/#min), [max](https://dgraph.io/docs/query-language/aggregation/#max), [avg](https://dgraph.io/docs/query-language/aggregation/#sum-and-avg) or [sum](https://dgraph.io/docs/query-language/aggregation/#sum-and-avg)

- The query may also contain [mathematical functions](https://dgraph.io/docs/query-language/math-on-value-variables/) on value variables.

- Summary functions can be used in conjunction with [@grouby](https://dgraph.io/docs/query-language/groupby/) directive to create aggregated value variables.

**Graph traversal**
- When you specify nested blocks and filters you basically describe a way to traverse the graph.

[@recurse](https://dgraph.io/docs/query-language/recurse-query/) and [@ignorereflex](https://dgraph.io/docs/query-language/ignorereflex-directive/) are directives used to optionally configure the graph traversal.

## 4. Visualize Tool 

**Ratel** is a tool for data visualization and cluster management that’s designed from the ground-up to work with Dgraph and [DQL](https://dgraph.io/docs/query-language/). 

Ratel is built on D3.js framework for rendering graph from Dgraph database 

### D3 Graph 

The Graph visualize on the Ratel UI is based on Force Directed Layout with velocity Verlet integration.



