# Chatbot from custom Knowledge Base
```mermaid
flowchart TD
	document(datafile) --> documentSplit[Splitter]
	documentSplit --> |Chunks|Embedding[Embedding Generator]
	Embedding --> |embeddings| vectordb1[(Vector DB)]
	Input[Query] --> Embedding
	vectordb1 --> |relevant docs| queryBlock[Query + Context]
	Input --> queryBlock
	queryBlock --> model[GPT3, GPT4]
	model --> output[Answer]

```

# Report Structure
1. Introduction
	1. Image retrieval 
	2. Related method
2. Methodology 
	1. Diffusion method for image retrieval
	2. Dimension reduction
3. Experiment and result
	1. Dataset 
	2. Evaluation
4. Conclusion
5. Citation



