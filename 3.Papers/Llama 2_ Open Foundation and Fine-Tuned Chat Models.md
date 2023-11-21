
- **PublishYear**:: 2023 
- **Author**:: Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, Dan Bikel, Lukas Blecher, Cristian Canton Ferrer, Moya Chen, Guillem Cucurull, David Esiobu, Jude Fernandes, Jeremy Fu, Wenyin Fu, Brian Fuller, Cynthia Gao, Vedanuj Goswami, Naman Goyal, Anthony Hartshorn, Saghar Hosseini, Rui Hou, Hakan Inan, Marcin Kardas, Viktor Kerkez, Madian Khabsa, Isabel Kloumann, Artem Korenev, Punit Singh Koura, Marie-Anne Lachaux, Thibaut Lavril, Jenya Lee, Diana Liskovich, Yinghai Lu, Yuning Mao, Xavier Martinet, Todor Mihaylov, Pushkar Mishra, Igor Molybog, Yixin Nie, Andrew Poulton, Jeremy Reizenstein, Rashi Rungta, Kalyan Saladi, Alan Schelten, Ruan Silva, Eric Michael Smith, Ranjan Subramanian, Xiaoqing Ellen Tan, Binh Tang, Ross Taylor, Adina Williams, Jian Xiang Kuan, Puxin Xu, Zheng Yan, Iliyan Zarov, Yuchen Zhang, Angela Fan, Melanie Kambadur, Sharan Narang, Aurelien Rodriguez, Robert Stojnic, Sergey Edunov, Thomas Scialom
- **Link**:: http://arxiv.org/abs/2307.09288
- **Tags**:: #paper
- **Cite Key**:: [@touvronLlamaOpenFoundation2023]

### Abstract
```
In this work, we develop and release Llama 2, a collection of pretrained and fine-tuned large language models (LLMs) ranging in scale from 7 billion to 70 billion parameters. Our fine-tuned LLMs, called Llama 2-Chat, are optimized for dialogue use cases. Our models outperform open-source chat models on most benchmarks we tested, and based on our human evaluations for helpfulness and safety, may be a suitable substitute for closed-source models. We provide a detailed description of our approach to fine-tuning and safety improvements of Llama 2-Chat in order to enable the community to build on our work and contribute to the responsible development of LLMs.
```

### Notes
- Although the training methodology is simple, high computational requirements have limited the development of LLMs to a few players
- Closed product LLMs are heavily fine-tuned to align with human preferences, which greatly enhances their usability and safety
- Measures to increase the safety of these models, using safety-specific data annotation and tuning, as well as conducting red-teaming and employing iterative evaluations.
- **Llama 2**, an updated version of Llama 1, trained on a new mix of publicly available data. We also increased the size of the pretraining corpus by 40%, doubled the context length of the model, and adopted grouped-query attention (Ainslie et al., 2023)
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
- Human annotations were collected in batches on a weekly basis. As we collected more preference data, our reward models improved, and we were able to train progressively better versions for Llama 2-Chat (see the results in Section 5, Figure 20). Llama 2-Chat improvement also shifted the model’s data distribution. Since reward model accuracy can quickly degrade if not exposed to this new sample distribution, i.e., from hyper-specialization (Scialom et al., 2020b), it is important before a new Llama 2-Chat tuning iteration to gather new preference data using the latest Llama 2-Chat iterations. This step helps keep the reward model on-distribution and maintain an accurate reward for the latest model.
**Reward Modeling:**
- The reward model takes a model response and its corresponding prompt (including contexts from previous turns) as inputs and outputs a scalar score to indicate the quality
- Train two separate reward models, one optimized for helpfulness (referred to as Helpfulness RM) and another for safety (Safety RM).
- The model architecture and hyper-parameters are **identical** to those of the pretrained language models, except that the classification head for next-token prediction is replaced with a regression head for outputting a scalar reward.
- To train the reward model, we convert our collected pairwise human preference data into a binary ranking label format (i.e., chosen & rejected) and enforce the chosen response to have a higher score than its counterpart.
$$
\mathcal{L}_{\text {ranking }}=-\log \left(\sigma\left(r_\theta\left(x, y_c\right)-r_\theta\left(x, y_r\right)\right)\right)
$$
- Given that our preference ratings is decomposed as a scale of four points. It can be useful to leverage this information to explicitly teach the reward model to assign more discrepant scores to the generations that have more differences. --> add a margin component in the loss:
$$
\mathcal{L}_{\text {ranking }}=-\log \left(\sigma\left(r_\theta\left(x, y_c\right)-r_\theta\left(x, y_r\right) - r\left(m\right)\right)\right)
$$
- The reward models are up to date with the Chat-llama to ensure they have the same data distribution 
- **Training Details**: We train for one epoch over the training data. In earlier experiments, we found that training longer can lead to over-fitting. We use the same optimizer parameters as for the base model. The maximum learning rate is 5 × 10−6 for the 70B parameter Llama 2-Chat and 1 × 10−5 for the rest. The learning rate is decreased on a cosine learning rate schedule, down to 10% of the maximum learning rate. We use a warm-up of 3% of the total number of steps, with a minimum of 5. The effective batch size is kept fixed at 512 pairs, or 1024 rows per batch
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

#### 2.Rejection Sampling fine-tuning. 
- Sample K outputs from the model and select the best candidate with our reward
- Perform rejection sampling only with our largest 70B Llama 2-Chat. All smaller models are fine-tuned on rejection sampled data from the larger model, thus distilling the large-model capabilities into the smaller ones.
- At each iterative stage, we sample K answers for each prompt from the most recent model. We score each sample given the best reward model accessible at the time of the experiment, and then select the best answer for a given prompt.
=> Adjust the optimal temperature. For Llama 2-Chat-RLHF, the optimal temperature when sampling between 10 and 100 outputs is T ∈ [1.2, 1.3].


---

