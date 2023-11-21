
### Scenario: 
- Data type: 
 > Conversation: Question and Answer 
 > SmartPay knowledge document

- Evaluate the conversation:
	- Question(Users chat): 
		- Sentiment Analysis
		- Clarity -> Document matching (?)
	- Answer:
		- Trustworthy (correctness)
		- Human resemblance

### Methodology: 
**Sentiment Analysis**
Analyse User sentiment Using pretrained Model: Phobert-based Sentiment for Vietnamese 

**Evaluation LLM Template design**
````python 
initialize model
initialize datasets
initialize evaluation_metrics

load_task_data:
    for each task in tasks:
        load data for task
        preprocess data if necessary (e.g., combine review summary and text)
        store data in datasets

embed_task_data:
    for each task in tasks:
        for each example in datasets[task]:
            obtain prompt from example
            obtain prompt_embedding using an embedding function
            store prompt_embedding in example

evaluate_model_on_task:
    for each task in tasks:
        for each example in datasets[task]:
            obtain prompt_embedding from example
            generate_answer_embedding = model.generate(prompt_embedding)

            calculate_metric = evaluation_metrics(example, generate_answer_embedding)
            store_metric_results_for_task(task)

aggregate_and_report_metrics:
    for each task in tasks:
        for each metric in evaluation_metrics:
            calculate average, median, or other aggregate metric values
            report metric value for task

main:
    load_task_data
    embed_task_data
    evaluate_model_on_task
    aggregate_and_report_metrics
````

**Metrics for offline validation:**
- [GPT Score](https://arxiv.org/pdf/2302.04166.pdf)
	![[Pasted image 20230528162153.png]]
	Evaluation aspect: 
		- **Text Evaluation**: aims to assess the quality of hypothesis text in terms of certain aspect ${y = f(h, a , S)}$
			- $h$ : Hypothesis/Generative text 
			- $a$: Eval aspect 
			- $s$: Source documents
		- **Meta Evaluation**: Meta evaluation aims to evaluate the reliability of automated metrics by calculating how well automated scores ${y_{auto}}$ correlate with human judgment ${y_{human}}$using correlation functions $g$ ${F^{data}_{f_{auto}, f_{human}}= g([f_{auto}(h_{1,1}),...,f_{auto}(h_{n,j})], [f_{human}(h_{1,1}), ...,f_{human}(h_{n,j})])}$
	GPT Score: The core idea of GPTSCORE is that a generative pre-training model will assign a higher probability of high-quality generated text following a given instruction and context.
	$$
\operatorname{GPTScore}(\boldsymbol{h} \mid d, a, \mathcal{S})=\sum_{t=1}^m w_t \log p\left(h_t \mid \boldsymbol{h}_{<t}, T(d, a, \mathcal{S}), \theta\right)
$$
	$d$: Task description
	$w_t$: weight of token at position t
	$T(.)$: promt template 
	
- [BertScore-F1](https://arxiv.org/pdf/1904.09675.pdf):
	$x=\left\langle x_1, \ldots, x_k\right\rangle$: tokenized reference sentence -> Embedding to $\left\langle\mathbf{x}_1, \ldots, \mathbf{x}_k\right\rangle$
	$\hat{x}=\left\langle\hat{x}_1, \ldots, \hat{x}_m\right\rangle$: tokenized candidates -> mapped to $\left\langle\hat{\mathbf{x}}_1, \ldots, \hat{\mathbf{x}}_l\right\rangle$
	**BertScore**: 
$$
R_{\mathrm{BERT}}=\frac{1}{|x|} \sum_{x_i \in x} \max _{\hat{x}_j \in \hat{x}} \mathbf{x}_i^{\top} \hat{\mathbf{x}}_j, \quad P_{\mathrm{BERT}}=\frac{1}{|\hat{x}|} \sum_{\hat{x}_j \in \hat{x}} \max _{x_i \in x} \mathbf{x}_i^{\top} \hat{\mathbf{x}}_j, \quad F_{\mathrm{BERT}}=2 \frac{P_{\mathrm{BERT}} \cdot R_{\mathrm{BERT}}}{P_{\mathrm{BERT}}+R_{\mathrm{BERT}}} .
$$
- [DiscoScore](https://arxiv.org/pdf/2201.11176.pdf): 

**Dialogue quality score:**
	- Deep-AM-FM
	- [MME-CRS](https://arxiv.org/pdf/2206.09403.pdf):
		- Fluency
		- Relevance
		- Topic Coherence Metric
		- Engagement
		- Specificity
**Metric comparision with human-feedback correlation** 
![[Pasted image 20230530090740.png]]
**Answer reponse and CrowdSourcing correlation.**
	- Pearson correlation 
	- Spearman correlation


# Mornitoring step for GPT Chatbot

1. Intent Matching 
Document vector database: {Product: {Intent, source_document}}
User conversation: {input : gpt-answer}
=> Find the Product -> Matching user intention 
**Pairwise text Score: Distance Metric**

2. GPT-Response Semantic Evaluation:
