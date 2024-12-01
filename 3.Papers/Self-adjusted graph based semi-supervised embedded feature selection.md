
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

## Motivation 
- Graph-based model perform on pre-defined graph: graph Laplacian to reflect the structure of high-dimensional training data -> not robust to noise and hard to select discriminative features
- The graph-based models are sensitive to the initialized graph, which could lead to a low quality of feature selection if an initial affinity graph is of low quality.
## Method proposes:
A novel graph-based semi-supervised embedded feature selection model that learns a similarity graph and combines the excellent properties from **manifold learning**.

- Self-adjusted graph for semi-supervised embedded feature selection method (SAGFS) that can adjusted based on local geometric structure of the data:
	- Approximate primitive affinity matrix by learning a sparse similarity graph 
	- Adjust the graph according to the local geometric structure by incorporates the graph regularization, such that the geometrical structure could be embedded into the manifold learning.
- Introduce $l_{2,p}$ -norm to constrain the projection matrix (row sparse projection matrix) for efficient feature selection

## Based methods:
- Manifold regularization 
- Spectral graph theory 
- Flexible manifold embedding

## Methodology 

The graph-based semi-supervised learning models construct a graph by using all training data -> Construct the similarity matrix  with Gaussian function 
$$a_{i j}=\left\{\begin{array}{cl}
e^{\frac{-\left\|\mathbf{x}_i-x_j\right\|^2}{2 a^2}} & \text { if } \mathbf{x}_i \in N_k\left(\mathbf{x}_j\right) \text { or } \mathbf{x}_j \in N_k\left(\mathbf{x}_i\right) \\
0 & \text { otherwise }
\end{array}\right.$$

$F =[F_l;F_u]∈ ℝ^{n×c}$ is a predicted label matrix consisting of $F_l$ and $F_u$, in which $F_l$ are consistent with the known labels $Y_l$ and $F_u$ is unknown variable.

$F$ can be computed through solving the following function:
$$
	\begin{array}{ll}
	\min _{\mathbf{F}} \sum_{i, j} \left\|\mathbf{f}_i-\mathbf{f}_j\right\|^2 a_{i j}=\min_{\mathbf{F}} \operatorname{Tr} \left(\mathbf{F}^T \mathbf{L F}\right) \\
	\text { s.t. } \quad \mathbf{F}_l=\mathbf{Y}_l
	\end{array}
$$

$$
	\begin{aligned}
	\boldsymbol{\theta} _{i, j+1} & =\boldsymbol{\theta} _{i, j}-\alpha \left(\nabla _{\boldsymbol{\theta}} \mathcal{L} _{\mathcal{T}_i}^{\mathcal{D}_i} \left(f _{\boldsymbol{\theta} _{i, j}}\right) + \lambda \boldsymbol{\theta} _{i, j}\right)\\
	& =\beta \boldsymbol{\theta} _{i, j} - \alpha \nabla _{\boldsymbol{\theta}} \mathcal{L} _{\mathcal{T}_i}^{\mathcal{D}_i} \left(f _{\boldsymbol{\theta} _{i, j}}\right)
	\end{aligned}
$$
#### Semi-supervised feature selectrion 

Ranks features by calculating a score $s_j$ for the j−th feature
$$s_j=\lambda \frac{\sum_{i, h=l+1}^{l+u}\left(g_i-g_h\right)^2 \times a_{i h}}{2 \sum_{i, h=l+1}^{l+u} g_i^2 \times \mathbf{d}_i}+(1-\lambda)\left(1-\text{NMI}\left(\hat{\mathbf{g}}, \mathbf{Y}_L\right)\right)$$
the similarity matrix $A$ is also calculated using Gaussian function
#### Initialize similarity graph

Suppose that close data points, measured as $‖xi − xj‖$, have high affinity values of A and constrain each row of A with l2-norm as the regularization to obtain the affinity values of A

inital similarity graph is learned by optimizing: 
$$
	\begin{gathered}
	\min _{\mathbf{A}} \sum _{j=1}^n\left\|\mathbf{x} _i-\mathbf{x} _j\right\| _2^2 a _{i j}+\theta \sum _{j=1}^n a _{i j}^2 \\
	\text { s.t. } \mathbf{a} _i^T \mathbf{1}=1, \mathbf{a} _i \geq \mathbf{0}
	\end{gathered}
$$
#### Optimize function:

- Manifold regularization: W and b estimated by 
$$\begin{gathered}
\min _{\mathbf{W}, \mathbf{b}} \frac{1}{n} \sum_{i=1}^n\left\|\mathbf{W}^T \mathbf{x}_i+\mathbf{b}-\mathbf{y}_i^T\right\|^2+\lambda_A\|\mathbf{W}\|^2 \\
+\lambda_I \operatorname{Tr}\left(\mathbf{W}^T \mathbf{X} \mathbf{L} \mathbf{X}^T \mathbf{W}\right)
\end{gathered}$$
Or:
$$\begin{aligned}
& \min _{\mathbf{W}, \mathbf{F}, \mathbf{F}} \alpha \operatorname{Tr}\left(\mathbf{F}^T \mathbf{L F}\right)+\beta \| \mathbf{X}^T \mathbf{W}+\mathbf{1 \mathbf { b } ^ { T } - \mathbf { F } \| _ { F } ^ { 2 } + \gamma \| \mathbf { W } \| _ { 2 , p } ^ { p }} \\
& \text { s.t. } \quad \mathbf{F}_l=\mathbf{Y}_l
\end{aligned}$$

To learn the sparse matric S to approximate the pre-defined A SAGFS framework with self-adjusted graph is formulated as:

$$\begin{aligned}
& g(\mathbf{S}, \mathbf{W}, \mathbf{F}, \mathbf{b})=\min _{\mathbf{s}, \mathbf{W}, \mathbf{b}, \mathbf{F}}\|\mathbf{S}-\mathbf{A}\|_F^2+\alpha \operatorname{Tr}\left(\mathbf{F}^T \mathbf{L}_S \mathbf{F}\right) \\
& \quad+\beta\left\|\mathbf{X}^T \mathbf{W}+\mathbf{1 b}^T-\mathbf{F}\right\|_F^2+\gamma\|\mathbf{W}\|_{2, p}^p \\
& \text { s.t. } \quad \mathbf{F}_l=\mathbf{Y}_l, \mathbf{S} \geq 0, \mathbf{S} \mathbf{1}=\mathbf{1}
\end{aligned}$$
Assumption that two close points on the learned graph S have similar properties, such that the graph S ’s manifold smoothness could be enhanced. Through the optimal sparse graph regularization, the geometrical structure could be embedded into the manifold learning, which retains the most important information.



	

---

