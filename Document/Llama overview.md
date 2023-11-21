# Llama 1: 
### Abstract
- The best performances are not achieved by the largest models, but by smaller models trained on more data.
- The focus of this work is to train a series of language models that achieve the best possible performance at various inference budgets, by training on more tokens than what is typically used.
- Only use publicly available data.
### Methodology 
1. Pretraining
- Training dataset is a mixture of several sources
- Entire training dataset contains roughly 1.4T tokens after tokenization
- For most of our training data, each token is used only once during training, with the exception of the Wikipedia and Books domains, over which we perform approximately two epochs.
2. Architecture
- Pre-normalization [GPT3]: 
	RMSNorm normalizing function to normalize the input of each transformer sub-layer
- SwiGLU activation function [PaLM].
- Rotary Embeddings [GPTNeo]:
	Remove the absolute positional embeddings, and instead, add rotary positional embeddings (RoPE)
3. Training implementation
- Use an efficient implementation of the causal multi-head attention to reduce memory usage and runtime
- Not storing the attention weights and not computing the key/query scores that are masked due to the causal nature of the language modeling task.
- Reduced the amount of activations that are recomputed during the backward pass with checkpointing. More precisely, we save the activations that are expensive to compute, such as the outputs of linear layers. --> manually implementing the backward function.
### Conclusion 
- LLaMA-13B outperforms GPT-3 while being more than 10× smaller.
- LLaMA has several tests on Bias, Toxicity

# Llama2: RLHF for chat finetuning
### Abstract
- Although the training methodology is simple, high computational requirements have limited the development of LLMs to a few players
- Closed product LLMs are heavily fine-tuned to align with human preferences, which greatly enhances their usability and safety
- Measures to increase the safety of these models, using safety-specific data annotation and tuning, as well as conducting red-teaming and employing iterative evaluations.
- **Llama 2**, an updated version of Llama 1, trained on a new mix of publicly available data. Increased the size of the pretraining corpus by 40%, doubled the context length of the model, and adopted grouped-query attention.
![[Pasted image 20230731133131.png]]
### Improvement with Llama1:
- An optimized auto-regressive transformer, but made several changes to improve performance.
- More robust data cleaning, updated our data mixes, trained on 40% more total tokens, doubled the context length.
- Used grouped-query attention (GQA): Replace traditional Multihead-attetion only have one key-value projection with 8 key-value projection to optimize training time, increase batchsize and context length.
### Finetuning Llama2 - Chat:
**Dataset:** 
- A limited set of clean instruction-tuning data can be sufficient to reach a high level of quality
- Total of 27,540 annotations
- For the fine-tuning process, each sample consists of a prompt and an answer.
**Finetuning:**
- Utilize an autoregressive objective and zero-out the loss on tokens from the user prompt, so as a result, we backpropagate only on answer tokens
- Finetune for 2 epochs
- For supervised fine-tuning, we use a cosine learning rate schedule with an initial learning rate of 2 × 10−5, a weight decay of 0.1, a batch size of 64, and a sequence length of 4096 tokens.
### RLHF
**Data collection:**
- Binary comparison protocol --> maximize the diversity of collected prompts
- Annotators to first write a prompt, then choose between two sampled model responses, based on provided criteria. In order to maximize the diversity, the two responses to a given prompt are sampled from two different model variants, and varying the temperature hyper-parameter. In addition to giving participants a forced choice, we also ask annotators to label the degree to which they prefer their chosen response over the alternative: either their choice is *significantly better*, *better*, *slightly better*, or *negligibly better/ unsure.
- Focus on helpfulness and safety.
- Collected a large dataset of over 1 million binary comparisons based on humans applying our specified guidelines, which we refer to as Meta reward modeling data
- Human annotations were collected in batches on a weekly basis. As we collected more preference data, our reward models improved, and we were able to train progressively better versions for Llama 2-Chat. Llama 2-Chat improvement also shifted the model’s data distribution. Since reward model accuracy can quickly degrade if not exposed to this new sample distribution, it is important before a new Llama 2-Chat tuning iteration to gather new preference data using the latest Llama 2-Chat iterations. This step helps keep the reward model on-distribution and maintain an accurate reward for the latest model.
**Reward Modeling:**
- The reward model takes a model response and its corresponding prompt (including contexts from previous turns) as inputs and outputs a scalar score to indicate the quality
- Train two separate reward models, one optimized for helpfulness (referred to as Helpfulness RM) and another for safety (Safety RM).
- The model architecture and hyper-parameters are **identical** to those of the pretrained language models, except that the classification head for next-token prediction is replaced with a regression head for outputting a scalar reward.
- To train the reward model, Convert collected pairwise human preference data into a binary ranking label format (i.e., chosen & rejected) and enforce the chosen response to have a higher score than its counterpart.
$$
\mathcal{L}_{\text {ranking }}=-\log \left(\sigma\left(r_\theta\left(x, y_c\right)-r_\theta\left(x, y_r\right)\right)\right)
$$
- Given that preference ratings is decomposed as a scale of four points. It can be useful to leverage this information to explicitly teach the reward model to assign more discrepant scores to the generations that have more differences. --> add a margin component in the loss:
$$
\mathcal{L}_{\text {ranking }}=-\log \left(\sigma\left(r_\theta\left(x, y_c\right)-r_\theta\left(x, y_r\right) - r\left(m\right)\right)\right)
$$
- The reward models are up to date with the Chat-llama to ensure they have the same data distribution 
- **Training Details**: We train for one epoch over the training data. In earlier experiments, we found that training longer can lead to over-fitting. We use the same optimizer parameters as for the base model. The maximum learning rate is 5 × 10−6 for the 70B parameter Llama 2-Chat and 1 × 10−5 for the rest. The learning rate is decreased on a cosine learning rate schedule, down to 10% of the maximum learning rate. We use a warm-up of 3% of the total number of steps, with a minimum of 5. The effective batch size is kept fixed at 512 pairs, or 1024 rows per batch.
**Finetuning:**
- RLHF fine-tuning with two main algorithms:
	- Proximal Policy Optimization (PPO)
	- Rejection Sampling fine-tuning.


**Code generation task:**
![[Pasted image 20230801004718.png]]

### RLHF: Algorithmns
#### 1.Proximal Policy Optimization (PPO)
Start with inittial policy that fine-tuned on the desired dataset
Repeat iteratively: 
- **Collect samples from existing policies and send comparisions to humans:** for each data records, sample from several sources include current policy, initial policy, original reference policy. Send the batches of data to human evaluate -> select the best policy
- **Learn reward model from human:** Train the reward model to predict the log odds that the better one judges by labelers.
- **Opmize Policy against the reward model:** Treat the logit output of the reward model as a reward that optimize using reinforment learning

In Llama2, uses the reward model as an estimate for the true reward function (human preference) and the pretrained language model as the policy to optimize.

Objective function: 
$$
\arg \max _\pi \mathbb{E}_{p \sim \mathcal{D}, g \sim \pi}[R(g \mid p)]
$$
Reward function used: 
$$
R(g \mid p)=\tilde{R}_c(g \mid p)-\beta D_{K L}\left(\pi_\theta(g \mid p) \| \pi_0(g \mid p)\right)
$$
Prioritize the scores from the safety model by choose the threshold of 0.15 for filtering unsafe responses, corresponding to a precision of 0.89 and a recall of 0.55 evaluated on the Meta Safety test set.

![[Pasted image 20230814091449.png]]

AdamW optimizer (Loshchilov and Hutter, 2017), with β1 = 0.9, β2 = 0.95, eps = 10−5. We use a weight decay of 0.1, gradient clipping of 1.0, and a constant learning rate of 10−6. For each PPO iteration we use a batch size of 512, a PPO clip threshold of 0.2, a mini-batch size of 64, and take one gradient step per mini-batch. For the 7B and 13B models, we set β = 0.01 (KL penalty), and for the 34B and 70B models, we set β = 0.005.

#### 2.Rejection Sampling fine-tuning. 
- Sample K outputs from the model and select the best candidate with our reward
- Perform rejection sampling only with our largest 70B Llama 2-Chat. All smaller models are fine-tuned on rejection sampled data from the larger model, thus distilling the large-model capabilities into the smaller ones.
- At each iterative stage, we sample K answers for each prompt from the most recent model. We score each sample given the best reward model accessible at the time of the experiment, and then select the best answer for a given prompt.
=> Adjust the optimal temperature. For Llama 2-Chat-RLHF, the optimal temperature when sampling between 10 and 100 outputs is T ∈ [1.2, 1.3].

### Improve multi-turn consistency 
Initial RLHF models tended to forget the initial instruction after a few turns of dialogue.
-> Propose Ghost Attention (GAtt): hacks the fine-tuning data to help the attention focus in a multi-stage process.
1. Define an instruction, inst, that should be respected throughout the dialogue and synthetically concatenate this instruction to all the user messages of the conversation.
2. For the training instructions, we created a few synthetic constraints to sample from: Hobbies (“You enjoy e.g. Tennis”), Language (“Speak in e.g. French”), or Public Figure (“Act as e.g. Napoleon”). To obtain the lists of hobbies and public figures, we asked Llama 2-Chat to generate it, avoiding a mismatch between the instruction and model knowledge (e.g., asking the model to act as someone it had not encountered during training). To make the instructions more complex and diverse, we construct the final instruction by randomly combining the above constraints.
3. Modify the original instruction half of the time to be less verbose

### Result 
**Evaluate the scale of model and data size on Helpful Test set:**
![[Pasted image 20230814092650.png]]
-> 
Larger models obtain higher performance for a similar volume of data
The scaling performance has not yet plateaued given the existing volume of data annotation used for training, a signal that there is room for more improvement with more annotations.
**Human evaluate:**
![[Pasted image 20230814063133.png]]
**Comparing models:**
- Collect a diverse set of over 4000 single and multi turn prompts. We manually collected single turn prompts spanning the following categories: factual questions, writing and content creation, language assistance, recommendations, and dialogue. For multi-turn prompts, annotators interacted with another model to generate a set of multi-turn prompts. To help ensure f airness, we asked annotators to collect multi-turn prompts by using four different interaction methods: (a) ChatGPT as the interaction model, (b) Llama 2-Chat as the interaction model, (c) best response between ChatGPT and Llama 2-Chat at every turn as selected by the annotators, (d) alternating between ChatGPT and Llama 2-Chat at every turn.
![[Pasted image 20230814065513.png]]
- While collecting generations, we append a system prompt prior to the prompt for evaluation. The system prompt for each model is shown in Table 31. Since ChatGPT, PaLM, and Falcon do not provide a system prompt, we use the same system prompt as Llama 2-Chat model.
- Limitations of human evaluations:  
	- By academic and research standards, we have a large prompt set of 4k prompts. However, it does not cover real-world usage of these models, which will likely cover a significantly larger number of use cases. 
	- Diversity of the prompts could be another factor in our results. For example, our prompt set does not include any coding- or reasoning-related prompts. 
	- We only evaluate the final generation of a multi-turn conversation. A more interesting evaluation could be to ask the models to complete a task and rate the overall experience with the model over multiple turns. 
	- Human evaluation for generative models is inherently subjective and noisy. As a result, evaluation on a different set of prompts or with different instructions could result in different results.

### Evaluation Methodology
For evaluations, the human annotators are presented with a prompt and generations from two models side-by-side. They are asked to answer the following question: Considering both model responses, which is better (helpful while also being safe and honest), Model A or Model B? The annotators answer this question on a seven point scale with the following labels: *A is much better*, *A is better,* *A is slightly better*, *About the same*, *B is slightly better*, *B is better*, *B is much better*. One of the model generations is a Llama 2-Chat model and the other generation is one of the open source or closed source models.
