
- **PublishYear**:: 2022 
- **Author**:: Hung-Chieh Fang, Kuo-Han Hung, Chao-Wei Huang, Yun-Nung Chen
- **Link**:: http://arxiv.org/abs/2211.09401
- **Tags**:: #paper #Open-Domain-QA #ConversationQA #Retrieval 
- **Cite Key**:: [@fangOpenDomainConversationalQuestion2022]

### Abstract
```
Open-domain conversational question answering can be viewed as two tasks: passage retrieval and conversational question answering, where the former relies on selecting candidate passages from a large corpus and the latter requires better understanding of a question with contexts to predict the answers. This paper proposes ConvADR-QA that leverages historical answers to boost retrieval performance and further achieves better answering performance. In our proposed framework, the retrievers use a teacher-student framework to reduce noises from previous turns. Our experiments on the benchmark dataset, OR-QuAC, demonstrate that our model outperforms existing baselines in both extractive and generative reader settings, well justifying the effectiveness of historical answers for open-domain conversational question answering.
```

### Notes
Comment: AACL-IJCNLP 2022
# Annotations  
(6/1/2023, 5:20:39 PM)

[Go to annotation](zotero://open-pdf/library/items/ZPMTMCCB?page=1&annotation=EE6HXADX)“Open-domain conversational question answering can be viewed as two tasks: passage retrieval and conversational question answering” ([Fang et al., 2022, p. 1](zotero://select/library/items/AP3GQ6JU))

[Go to annotation](zotero://open-pdf/library/items/ZPMTMCCB?page=1&annotation=58YZ8KJC)“proposes ConvADR-QA that leverages historical answers to boost retrieval performance and further achieves better answering performance.” ([Fang et al., 2022, p. 1](zotero://select/library/items/AP3GQ6JU))

[Go to annotation](zotero://open-pdf/library/items/ZPMTMCCB?page=1&annotation=IZMQQYD6)“A unique challenge to CQA is that the questions are context-dependent.” ([Fang et al., 2022, p. 1](zotero://select/library/items/AP3GQ6JU))

[Go to annotation](zotero://open-pdf/library/items/ZPMTMCCB?page=2&annotation=7KQUVIFL)“Open-Domain QA Without a given target passage, most work for this task was built upon the dense retrieval framework for retrieving relevant passages for QA.” ([Fang et al., 2022, p. 2](zotero://select/library/items/AP3GQ6JU))

[Go to annotation](zotero://open-pdf/library/items/ZPMTMCCB?page=2&annotation=WZE8IJFA)“focus on enhancing model’s ability on handling multiturn conversational questions.” ([Fang et al., 2022, p. 2](zotero://select/library/items/AP3GQ6JU))

[Go to annotation](zotero://open-pdf/library/items/ZPMTMCCB?page=2&annotation=LZMZM8SC)“Our work does not introduce extra parameters and complex modeling” ([Fang et al., 2022, p. 2](zotero://select/library/items/AP3GQ6JU))

[Go to annotation](zotero://open-pdf/library/items/ZPMTMCCB?page=2&annotation=45RYUG98)“The difficulty of open-domain CQA is that the current question usually requires context information from previous turns, which makes it harder for the system to capture the latent information compared with the open-domain QA task.” ([Fang et al., 2022, p. 2](zotero://select/library/items/AP3GQ6JU))

[Go to annotation](zotero://open-pdf/library/items/ZPMTMCCB?page=2&annotation=7TEIHHRI)“Previous work on open-domain conversational search addressed the problem by concatenating the current and historical questions without answers” ([Fang et al., 2022, p. 2](zotero://select/library/items/AP3GQ6JU))

[Go to annotation](zotero://open-pdf/library/items/ZPMTMCCB?page=2&annotation=LCTBIGCY)“Our motivation is that historical answers can also provide the important signal for the current question” ([Fang et al., 2022, p. 2](zotero://select/library/items/AP3GQ6JU))

[Go to annotation](zotero://open-pdf/library/items/ZPMTMCCB?page=2&annotation=PQIFGRNC)“propose ConvADR-QA which includes a retriever for obtaining relevant passages from a large collection and a reader for CQA.” ([Fang et al., 2022, p. 2](zotero://select/library/items/AP3GQ6JU))

[Go to annotation](zotero://open-pdf/library/items/ZPMTMCCB?page=5&annotation=3FGSIE8J)“The experiments on a benchmark dataset demonstrate that our proposed method outperforms all baselines for both retrieval and answering performance.” ([Fang et al., 2022, p. 5](zotero://select/library/items/AP3GQ6JU))

# Methodology
 ![[Pasted image 20230601171311.png]]

### Task description: 
$C$: passage collections
$N$: passages $\{p_i\}^N_{i=1}$ with $p_i$ is a sequence of tokens $p^1_i,...,p^l_i$ 
Given the $t$-th question $q_t$ and all historical questions $\{p_i\}^{t-1}_{i=1}$ in a conversation => the task is to predict $a_t$ from $C$ 

### Detail 
1. Retriever 
- The model uses a dual-encoder architecture to map passages and questions to the same embedding space.
- The input for question encoder is the concatenation of historical questions with current answer
$$
p=E_p(p),q{'}_k = E_Q({q_i, a_i}^{k-1}_i=1; q_k)
$$
- Retrieval score is the dot prodcut of the passage embedding and question embedding
$$
S_{rt}(q_t, p) = p.q'_{k}
$$
**Training**: 
- Each question contains one gold passage $p+$ and a set of negative passages $P-$
	=> Optimize using negative log likelihood loss: 
	$$
\mathcal{L}_{\mathrm{NLL}}=-\log \frac{e^{S_{\mathrm{rt}}\left(q_k, p^{+}\right)}}{e^{S_{\mathrm{rt}}\left(q_k, p^{+}\right)}+\sum_{p^{-} \in P^{-}} e^{S_{\mathrm{rt}}\left(q_k, p^{-}\right)}}
$$
- Knowledge Distillation: the input of teacher model is a manually rewritten context-independent query $q^*_k$ 
  Knowledge distillation loss (KDLoss): 
$$
\begin{gathered}
\boldsymbol{p}=E_P^{\prime}(p), \boldsymbol{q}_{\boldsymbol{k}}^{\star}=E_Q^{\prime}\left(q_k^{\star}\right), \\
\mathcal{L}_{\mathrm{KD}}=\operatorname{MSE}\left(\boldsymbol{q}_{\boldsymbol{k}}^{\star}, \boldsymbol{q}_{\boldsymbol{k}}^{\prime}\right) .
\end{gathered}
$$
=> Retrieval loss: $\mathcal{L}_{NLL} + \mathcal{L}_{KD}$
**Reader:**
- Task: extract a span from passages as the final answer. -> Using BERT model for machine comprehension task 
- $t$-th question $q_t$ and top-K candidate passages $\{p_i\}^K_{i=1}$ -> extracts span for each passages by choosing the highest score of **start and end tokens**
- Score for $m$-th token is defined as: 
$$
\begin{gathered}
S_{\text {start }}^{[m]}\left(q_t ; p\right)=W_{\text {start }} \operatorname{BERT}\left(\left\{q_i\right\}_{i=1}^t ; p\right)[m], \\
S_{\text {end }}^{[m]}\left(q_t ; p\right)=W_{\text {end }} \operatorname{BERT}\left(\left\{q_i\right\}_{i=1}^t ; p\right)[m], \\
S_{\mathrm{rd}}\left(q_t ; p\right)=\max _{m 1, m 2}\left[S_{\text {start }}^{[m 1]}\left(q_t ; p\right)+S_{\text {end }}^{[m 2]}\left(q_t ; p\right)\right]
\end{gathered}
$$
Choose the final answer by multiplying the retriever score $S_{rt}$ and the sum of start/end token score as the reader score $S_{rd}$:
$$S(q_t, p)=S_{rt}(q_t, p).S_{rd}(q_t, p)$$
