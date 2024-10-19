
- **PublishYear**:: 2024 
- **Author**:: Jianyong Zhu, Jiaying Zheng, Zhenchen Zhou, Qiong Ding, Feiping Nie
- **Link**:: https://link.springer.com/10.1007/s10462-024-10868-2
- **Tags**:: #paper
- **Cite Key**:: [@zhuSelfadjustedGraphBased2024]

### Abstract
```
Graph-based semi-supervised feature selection has aroused continuous attention in processing high-dimensional data with most unlabeled and fewer data samples. Many graphbased models perform on a pre-defined graph, which is separated from the procedure of feature selection, making the model hard to select the discriminative features. To address this issue, we exploit a self-adjusted graph for semi-supervised embedded feature selection method (SAGFS), which learns an optimal sparse similarity graph to replace the predefined graph to alleviate the effect of data noise. SAGFS allows the learned graph itself to be adjusted according to the local geometric structure of the data and the procedure of selecting features to select the most representative features. Besides that, we introduce l2,p -norm to constrain the projection matrix for efficient feature selection. An efficient alternating optimization algorithm is presented, together with analyses on its convergence. Systematical experiments on several publicly datasets are performed to analyze the proposed model from several aspects, and demonstrate that our approaches outperform other comparison methods.
```

### Notes

- **Keys**: 
	- Manifold embedding
	- Manifold Regularization 
	- Lagrange Multiplier function

## Motivation 
- Graph-based model perform on pre-defined graph -> not robust to noise and hard to select discriminative features
- The graph-based models are sensitive to the initialized graph, which could lead to a low quality of feature selection if an initial affinity graph is of low quality.
## Method proposes:
- Self-adjusted graph for semi-supervised embedded feature selection method (SAGFS) that can adjusted based on local geometric structure of the data
- Introduce $l_{2,p}$ -norm to constrain the projection matrix for efficient feature selection.

## Methodology 

The graph-based semi-supervised learning models construct a graph by using all training data -> Construct the similarity matrix  with Gaussian function 

#### Semi-supervised feature selectrion 

Ranks features by calculating a score $s_j$ for the j−th feature
$$
	s_j=\lambda \frac{\sum_{i, h=l+1}^{l+u}\left(g_i-g_h\right)^2 \times a_{i h}}{2 \sum_{i, h=l+1}^{l+u} g_i^2 \times \mathbf{d}_i}+(1-\lambda)\left(1-\operatorname{NMI}\left(\hat{\mathbf{g}}, \mathbf{Y}_L\right)\right)
$$

#### Initialize similarity graph


---

