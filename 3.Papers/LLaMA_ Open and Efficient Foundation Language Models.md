
- **PublishYear**:: 2023 
- **Author**:: Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, Aurelien Rodriguez, Armand Joulin, Edouard Grave, Guillaume Lample
- **Link**:: http://arxiv.org/abs/2302.13971
- **Tags**:: #paper
- **Cite Key**:: [@touvronLLaMAOpenEfficient2023]

### Abstract
```
We introduce LLaMA, a collection of foundation language models ranging from 7B to 65B parameters. We train our models on trillions of tokens, and show that it is possible to train state-of-the-art models using publicly available datasets exclusively, without resorting to proprietary and inaccessible datasets. In particular, LLaMA-13B outperforms GPT-3 (175B) on most benchmarks, and LLaMA-65B is competitive with the best models, Chinchilla-70B and PaLM-540B. We release all our models to the research community.
```

### Notes
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
- LLaMA has several tests on Bias, Toxicity, 
---

