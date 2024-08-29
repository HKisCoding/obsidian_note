
# Spectral Clustering 

Spectral clustering convert data clustering problems to graph cut problems. 
### Spectral Clustering steps:
Given $X = \{x_1, x_2,...x_n\}$ and C cluster class
Steps:
- Calculate the similarity matrix $W = ( w)_{ij}, i, j ∈\{ 1, 2, ⋯, n\}$ between all pairs of $(x_i, x_j)$
- Calculate the diagonal Degree matrix $D = diag(d_i)$ with $d_i = \sum_{j=1}^{n} w_{i,j}, i = \{1,2,..,n\}$ 
- Create transition matrix (Laplacian matrix): $L = D - W$ 
- Spectral clustering uses the properties of Laplacian matrix L to select the eigenvectors corresponding to the smallest c eigenvalues to form the feature matrix $F ∈ R^{n×c}$.
- Each row of feature matrix $F$ can be seen as the representation of samples in $R^c$ space, and the final result can be obtained by clustering with k-means.
### Concept of Laplacian Matrix: 
https://mbernste.github.io/posts/laplacian_matrix/

### Graph cut with eigenvector: 
#### Eigendecomposition
The k smallest eigenvalues of the Laplacian matrix correspond to the smallest cuts in the graph. Finding these eigenvalues helps identify partitions of the graph that minimize the number of edges between different clusters, leading to more distinct groupings.

The smallest eigenvalues relate to the connectivity of the graph. If the graph has k connected components, the k smallest eigenvalues will be zero, indicating that the points can be separated into k distinct clusters.

The solution of many objective functions of graph clustering is written inform of `Rayleigh entropy` 
The minimum value, the second minimum value, ..., and the maximum value of Rayleigh entropy correspond to the minimum eigenvalue, the second minimum eigenvalue, ..., and the maximum eigenvalue of the Laplacian matrix, respectively. And the extreme value is obtained at the corresponding eigenvector. It can be seen that the eigenvector of the Laplacian matrix of the graph contains the category information of vertices.

The **eigendecomposition** of Laplacian matrix is used to map each data point to a low-dimensional representative point. Finally, the dataset is divided into two or more classes based on the new representative points.

#### Eigenvector selection:
Not every eigenvector contains useful information for clustering.

A larger eigenvalue means that the corresponding eigenvector contains more information, which means it has a stronger ability to distinguish data

https://stats.stackexchange.com/questions/459640/why-eigenvectors-reveal-the-groups-in-spectral-clustering

### Constructing similarity graph:
#### 1. Fully connected graph:
Using Gaussian similarity function
$$
s_{i j}=\exp \left(-\frac{d\left(x_i, x_j\right)^2}{2 \sigma^2}\right)
$$
#### 2. Sparse Graph

Construct the k-nearest neighbor graph:
- One-side k-nearest neighbors: Given two vertices $v_i$ and $v_j$. If any of them is one of the k-nearest neighbors of the other point, they are connected.  
- Two-side k-nearest neighbors: Given two vertices  $v_i$ and $v_j$. If vi is one of the k-nearest neighbors of  $v_i$ and $v_j$ is also one of the k-nearest neighbors of vi, they are connected.
$$
s_{i j}=\frac{d_{i, k+1}-d_{i j}}{k d_{i, k+1}-\sum_{j=1}^k d_{i j}}, j=1, \cdots, k
$$

# SpectralNet overview

#### Motivation: 
Traditional spectral clustering methods have problems of scalability and generalization of the spectral embedding
SpectralNet: learns a map that embeds input data points into the eigenspace of their associated graph Laplacian matrix and subsequently clusters them. 
The map learned by SpectralNet naturally generalizes the spectral embedding to unseen data points.

#### Components:
- AutoEncoder Net
- Siamese Net: unsupervised learning of an affinity given the input distance measure
- Spectral Net: Neuron network compute a map $F_θ : R^d → R^k$ by optimizing a spectral clustering objective while enforcing orthogonality.

#### Computing spectral map $F_\theta$
Affinity map $W = R^d \times R^d$ such that $w(x, x')$ expresses the similarity between x and x′
Points $x, x'$ similar to each other will have large $w(x, x')$ and embedded close to each others.

Embedding $y_i = F_\theta (x_i)$ with $y_i \in R^k$

Loss function: 
$$
L_{\text {SpectralNet }}(\theta)=\frac{1}{m^2} \sum_{i, j=1}^m W_{i, j}\left\|y_i-y_j\right\|^2
$$
with  $m$: samples in a minibatch from distribution $D$
With the constraint of orthonormal: $$
\mathbb{E}\left[y y^T\right]=I_{k \times k}
$$
For a minibatch: $\frac{1}{m} Y^T Y=I_{k \times k}, Y \in m \times k$

#### Implement: 
Spectral Net is a multi layers perceptron (MLP) with a **last layer enforces the orthogonality constraint**
The last layer is a Linear layer that get inputs with (b, k) shape and return (b, k) shape
$Y'$: input of the layer 
$Y$: output of the layer
Linear map that orthogonalizes the columns of $Y'$ is computed through its `QR decomposition`.
Based on the Cholesky decomposition $A^T A = LL^T$, => $Q= A(L^{-1})^T$

Final Output: $Y = Y' \times \sqrt{m}(L^{-1})^T$  with $L$ is obtained from the Cholesky decomposition of  $Y$

With the output is approximate to a true eigenvector, the loss function is rewritten as: 
$$
L_{\text {SpectralNet }}(\theta)=\frac{2}{m^2} \operatorname{trace}\left(Y^T(D-W) Y\right)
$$
for general $k$, under the constraint, the minimum is attained when the column space of $Y$ is the subspace of the $k$ eigenvectors corresponding to the smallest $k$ eigenvalues of $D − W$ .

#### Building affinity matrix:

- Gaussian kernel: For a set of nearest neighbor pairs: $$
W_{i, j}= \begin{cases}\exp \left(-\frac{\left\|x_i-x_j\right\|^2}{2 \sigma^2}\right), & x_j \text { is among the nearest neighbors of } x_i \\ 0, & \text { otherwise, }\end{cases}
$$
- Siamese network: Neuron network trained on a collection of similar (positive) and dissimilar (negative) pairs of data points. By labeling $(x_i, x_j)$ is positive if $|| x_i - x_j||$ is small and negative otherwise 
	=>  Siamese network, therefore, is trained to learn an adaptive nearest neighbor metric.
	Siamese network, therefore, is trained to learn an adaptive nearest neighbor metric.
	$$
L_{\text {siamese }}\left(\theta_{\text {siamese }} ; x_i, x_j\right)= \begin{cases}\left\|z_i-z_j\right\|^2, & \left(x_i, x_j\right) \text { is a positive pair } \\ \max \left(c-\left\|z_i-z_j\right\|, 0\right)^2, & \left(x_i, x_j\right) \text { is a negative pair }\end{cases}
$$
Objective is to minimize contrastive loss
After training, the Siamese net is used to define a batch affinity matrix for Spectral Net

# Meta learning overview
### Adaptive Meta learning for tuning hyperparameters

Propose Adaptive Learning of hyperparameters for Fast Adaptation that enables training to be more effective with task-conditioned inner-loop updates from any given initialization.
#### Motivation:
Fast adaptation when test task is different from train task 
Introduce a small meta-network that can adaptively generate per-step hyper-parameters: learning rate and weight decay coefficients.
#### Methodology:
With a $l2$ regularization added to the loss function, the inner loop update: 
$$
\begin{aligned}
\boldsymbol{\theta}_{i, j+1} & =\boldsymbol{\theta}_{i, j}-\alpha\left(\nabla_{\boldsymbol{\theta}} \mathcal{L}_{\mathcal{T}_i}^{\mathcal{D}_i}\left(f_{\boldsymbol{\theta}_{i, j}}\right)+\lambda \boldsymbol{\theta}_{i, j}\right) \\
& =\beta \boldsymbol{\theta}_{i, j}-\alpha \nabla_{\boldsymbol{\theta}} \mathcal{L}_{\mathcal{T}_i}^{\mathcal{D}_i}\left(f_{\boldsymbol{\theta}_{i, j}}\right)
\end{aligned}
$$
the adaptation process via the hyperparameters in the inner-loop update equation, which are scalar constants of **learning rate $\alpha$** and regularization hyperparameter $β = 1 - \alpha\lambda$ 

For task $T_i$ at time step j, The learning state can be defined as $\boldsymbol{\tau}_{i, j}=\left[\nabla_{\boldsymbol{\theta}} \mathcal{L}_{\mathcal{T}_i}^{\mathcal{D}_i}\left(f_{\boldsymbol{\theta}_{i, j}}\right), \boldsymbol{\theta}_{i, j}\right]$
The proposed meta-learner $g_φ$ generates the adaptive hyperparameters $α_{i,j}$ and $β_{i,j}$ using the current parameters $θ_{i,j}$ and its gradients $∇_θ L^{D_i}_{T_i}$. $$
\left(\boldsymbol{\alpha}_{i, j}, \boldsymbol{\beta}_{i, j}\right)=g_{\boldsymbol{\phi}}\left(\boldsymbol{\tau}_{i, j}\right) .
$$
For every inner-loop update step, the generator produce the learning rate and regularization hyper-parameters, which they are used to control the direction and magnitude of the weight update.

To train the network $g_\phi$, the outer-loop optimization using new examples $D'_i$ and task-adapted weights $\theta'_i$ is performed as in:

$$\phi \leftarrow \phi - \eta \nabla_\phi \sum_{T_i} L_{D'_i}(f_{\theta'_i})$$
#### Implementing:
Generator network $g_φ$ is a 3-layer MLP with ReLU activation between the layers.
The task specific learning state is reduce into layer-wise mean of gradients and weights thus resulting in 2 state values per layer.

For outputs, the learning rate $\alpha^1_{i,j}$ and the weight-decay term $\beta^1_{i,j}$ are first generated layer-wise and then repeated to the dimensions of the respective parameters $\theta_{i,j}$.
The learning rate and weight-decay terms are generated at the $j$-th step for the task $T_i$ as follows:

$$\alpha_{i,j} = \alpha^0_{i,j} \odot \alpha^1_{i,j}(\bar{\tau}_{i,j})$$
$$\beta_{i,j} = \beta^0_{i,j} \odot \beta^1_{i,j}(\bar{\tau}_{i,j})$$

where:
- $\alpha^0_{i,j}, \beta^0_{i,j}$ are meta-learnable post-multipliers
- $\alpha^1_{i,j}(\tau_{i,j}), \beta^1_{i,j}(\tau_{i,j})$ are generated layer-wise multiplier values

All of these terms are repeated to the dimension of $\theta_{i,j}$.

# Meta learning for adapting Gaussian scale 

Based on the above methodology, a meta learner is added to the training state.
- Input: Randomly sample batch from original data distribution 
- Output: Gaussian scale to create affinity matrix
Architecture: 3 linear layer with LeakyRelu() activation. The last layer output return 1 scalar for each image feature. 

$$
	scale = \frac{1}{m}(\sum_{i = 0}^m{F_{meta}(x_i)})
$$
## Experiment

Dataset:
- Image: MRSC, Caltech-101, MNIST
- Tabular: prokaryotic
Metric: 
- **ACC** - accuracy score 

- **NMI**: NMI scores range from 0 to 1:
	- 0 indicates no mutual information (completely random clustering)
	- 1 indicates perfect correlation with the true labels
	- NMI(U, V) = $\frac{2 * I(U, V)}{H(U) + H(V)}$
	Where:
	- U and V are two different clusterings (e.g., predicted clustering and true labels)
	- I(U, V) is the mutual information between U and V: $I(U, V) = \sum_{i=1}^{|U|} \sum_{j=1}^{|V|} \frac{|U_i \cap V_j|}{N} \log \frac{N|U_i \cap V_j|}{|U_i||V_j|}$
	- H(U) and H(V) are the entropies of U and V respectively: $-\sum_{i=1}^{|U|} (\frac{|U_i|}{N} \log \frac{|U_i|}{N})$
	
- **PURITY**:  measures the extent to which each cluster contains data points from primarily one class.

$$Purity = (1/N) * Σ(k) max(n_k^i)$$
	Where:
	- N is the total number of data points
	- k is the number of clusters
	- n_k^i is the number of data points of class i in cluster k
	To calculate Purity:
	1. For each cluster, count the number of data points from each class.
	2. Take the maximum count for each cluster.
	3. Sum these maximum counts.
	4. Divide by the total number of data points.

## Result

| Method                                                                              | Dataset     | ACC   | NMI   | PURITY |
| ----------------------------------------------------------------------------------- | ----------- | ----- | ----- | ------ |
| Using Resnet18 as Feature Extractor<br>\+ Siamese Net                               | MRSC        | 0.868 | 0.882 | 0.901  |
|                                                                                     | MNIST       |       |       |        |
|                                                                                     | Caltech-101 | 0.332 | 0.601 | 0.401  |
|                                                                                     | prokaryotic | 0.35  | 0.312 | 0.579  |
| Using Resnet18 as Feature Extractor<br>\+ Using Gaussian scale: 20 nearest neighbor | MRSC        | 0.669 | 0.806 | 0.899  |
|                                                                                     | MNIST       |       |       |        |
|                                                                                     | Caltech-101 | 0.364 | 0.581 | 0.396  |
|                                                                                     | prokaryotic | 0.339 | 0.18  | 0.526  |
| Using Resnet18 as Feature Extractor<br>\+ Using Gaussian scale: all dataset         | MRSC        | 0.794 | 0.856 | 0.895  |
|                                                                                     | MNIST       |       |       |        |
|                                                                                     | Caltech-101 | 0.33  | 0.593 | 0.374  |
|                                                                                     | prokaryotic |       |       |        |
| Using Resnet18 as Feature Extractor<br>\+ Meta learning to find Gaussian scale      | MRSC        | 0.493 | 0.495 | 0.535  |
|                                                                                     | MNIST       |       |       |        |
|                                                                                     | Caltech-101 | 0.121 | 0.272 | 0.464  |
|                                                                                     | prokaryotic | 0.468 | 0.181 | 0.711  |

# To do:
- Research multi-view clustering approach
- Optimize meta learning cluster with better adaptive loss: 
Tuning K for clustering: https://www.sciencedirect.com/science/article/abs/pii/S0950705120301209
Meta learning for cluster: https://arxiv.org/pdf/1910.14134