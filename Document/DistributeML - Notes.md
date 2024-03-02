## Distributed ML ecosystem
- Large volumn data processing problems -> Distributed platform: **Apache Spark**, **MLlib**
- ML lib now receive support to execute on distributed machine: **Keras - TensorFlow**, **PyTorch**
- Ecosystem natively build for distributed machine learning and designed around a specific algorithmic and operational model, e.g. **Distributed Ensemble Learning, Parallel Synchronous Stochastic Gradient Descent (SGD) or Parameter Servers**
![[Pasted image 20230822234907.png]]
### 1. Distributed Computing framework 
- Utilizing a number of commodity servers, each of them with a relatively small storage capacity and computing power, **more effective** than one expensive large server.
- The scale-out model offers a higher aggregate I/O bandwidth compared to using a smaller number of more powerful machines since every node comes with its own I/O subsystem.
- Data ingestion is a significant part of the workload.
#### 1.1. Storage 
Based on Google File System: 
- Split data into chunk
- Replicate the chunk 
- The data on the chunk servers can then be accessed by a user through contacting the master, which serves as a name node and provides the locations for every chunk of a file.
- Open source framework: Hadoop  -> HDFS layer

#### 1.2. Compute Framework 
- **Map Reduce**:
	- *Map phase*: data is split into tuples (called key-value pairs) -> executed fully parallel since there are no data dependencies between mapping a function to two different values in the set.
	- *Shuffle phase*: tuples are exchanged between nodes and passed on. -> all tuples belonging to the same key are processed by the same node for correctness.
	- *Reduce phase*: the aggregation is performed on the tuples to generate a single output value per key.

	> Tasks of the same phase have not data dependencies and can therefore be executed entirely in parallel.	

	MapReduce architecture is similar to the **Bulk-Synchronous processing (BSP)** paradigm,

- **Apache Spark:** 
	- Executing a directed acyclic graph of transformations (like mappings) and actions (like reductions) fully in memory.
	- Data structure which Spark was originally designed around is called a **Resilient Distributed Dataset (RDD).**
	- Datasets are read-only, and new instances can only be created from data stored on the disk or by transforming existing RDDs
	- Each RDD is given a lineage graph that shows what transformations have been executed on it. -> if some data is lost, Spark can trace the path the RDD has followed from the lineage graph and recalculate any lost data.
	- Spark allows for checkpointing of data to prevent extensive re-computation.

### 2. Distributed ML system implementation 
#### 2.1. Distributed Ensemble Learning 
- **Main idea:** 
	- Training separate models on subsets of data -> training individual models on independent machines in parallel.
	- Predict outputs in each instances the combined through ensemble model aggregation 
- **Pros and cons:**
	- Pros: 
		- Not depend on library
		- Any ML framework can be used 
	- Cons: 
		- Models can output biases due to biases in training subsets -> need proper subdivision
#### 2.2. Parallel sync SGD 
**Most approaches rely on the AllReduce operation**
Takes an array of input elements on each process and returns an array of output elements to the root process. The output elements contain the reduced result.
![[Pasted image 20230826002836.png]]
   
There are several algorithms to implement the operation. For example, a straightforward one is to select one process as a master, gather all arrays into the master, perform reduction operations locally in the master, and then distribute the resulting array to the rest of the processes.
![[Pasted image 20230826003001.png]]

**Pipelinne:**
The entire training data is divided into N parts, where N is the number or worker nodes in the cluster, and each part is sent to only one worker node.
A single neural network model is then defined and copied to each of the nodes.
Uses a mini-batch SGD for the localized training
Once all workers are done with their training process, all nodes transmit, to the parameter server, the new model they just trained.
The result is a new neural network model (incrementally trained) that is then redistributed to the worker nodes and the process then repeats

![[Pasted image 20230826010745.png]]

**Ring Reduce**
![[Pasted image 20230826010519.png]]
![[Pasted image 20230826010535.png]]
![[Pasted image 20230826010542.png]]