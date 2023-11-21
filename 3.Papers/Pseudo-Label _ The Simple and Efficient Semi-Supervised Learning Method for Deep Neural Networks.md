
- **PublishYear**:: 2013 
- **Author**:: Dong-Hyun Lee
- **Link**:: 
- **Tags**:: #paper #semi-supervised #pseudo-label
- **Cite Key**:: [@leePseudoLabelSimpleEfficient2013]

### Abstract
```
We propose the simple and efficient method of semi-supervised learning for deep neural networks. Basically, the proposed network is trained in a supervised fashion with labeled and unlabeled data simultaneously. For un-labeled data, Pseudo-Label s, just picking up the class which has the maximum predicted probability, are used as if they were true labels. This is in effect equivalent to Entropy Regularization. It favors a low-density separation between classes, a commonly assumed prior for semi-supervised learning. With De-noising Auto-Encoder and Dropout, this simple method outperforms conventional methods for semi-supervised learning with very small labeled data on the MNIST handwritten digit dataset.
```

### Notes
#### Introduction
All of the successful methods for training deep neural networks have something in common : they rely on an unsupervised learning algorithm

Most work in two main phases. 
- In a first phase, unsupervised pre-training, the weights of all layers are initialized by this layer-wise unsupervised training. 
- In a second phase, fine-tuning, the weights are trained globally with labels using backpropagation algorithm in a supervised fashion.

The proposed network is trained in a supervised fashion with labeled and unlabeled data simultaneously.

Pseudo-Label just picking up the class which has the maximum predicted probability every weights update, are used as if they were true labels.

Denoising Auto-Encoder and Dropout boost up the performance.

#### Methodology
Sigmoid units are used for output instead of softmax, we assume that the probability of each label is independent from each other.

Cross Entropy as a loss function:
$$
\begin{equation}
L\left(y_i, f_i\right)=-y_i \log f_i-\left(1-y_i\right) \log \left(1-f_i\right)
\end{equation}
$$
Denoising Auto-Encoder is unsupervised learning algorithm based on the idea of making the learned representations robust to partial corruption of the input pattern.
DAE can be stacked to initialize deep neural networks.
$$
\begin{equation}
\begin{aligned}
& h_i=s\left(\sum_{j=1}^{d_v} W_{i j} \widetilde{x}_j+b_i\right) \\
& \widehat{x}_j=s\left(\sum_{i=1}^{d_h} W_{i j} h_i+a_j\right)
\end{aligned}
\end{equation}
$$
	$\widetilde{x}$ is corrupted version of jth input value
	$\widehat{x}$ is the reconstruction of the jth input value
	=> Minimizing the reconstruction error between $x_i$ and $\widehat{x}$

Reconstruction error is Cross Entropy :
$$
\begin{equation}
L(x, \widehat{x})=\sum_{j=1}^{d_v}-x_j \log \widehat{x}_j-\left(1-x_j\right) \log \left(1-\widehat{x}_j\right)
\end{equation}
$$
Use DAE in a unsupervised pre-training phase.
Masking noise with a probability 0.5 is used for corruption.

**Optimizer**: SGD with dropout
An exponentially decaying learning rate and linearly increasing momentum
$$
\begin{equation}
\Delta W(t+1)=p(t) \Delta W(t)-(1-p(t)) \epsilon(t)<\nabla_W L>
\end{equation}
$$

where $\epsilon(t+1) = k\epsilon(t)$
	$p(t)= \begin{cases}\frac{t}{T} p_f+\left(1-\frac{t}{T}\right) p_i & t<T \\ p_f & t \geq T\end{cases}$

DONT USE WEIGHT REGULARIZATION

**Pseudo-label**:
Pick up the class which has maximum predicted probability for each unlabeled sample.
$$
\begin{equation}
y_i^{\prime}= \begin{cases}1 & \text { if } i=\operatorname{argmax}_{i^{\prime}} f_{i^{\prime}}(x) \\ 0 & \text { otherwise }\end{cases}
\end{equation}
$$
2 stages:
- Pre-trained: a supervised fashion with labeled and unlabeled data simultaneously.
- Finetune: with Unlabeled data, Pseudo-label recalculated every weights update are used for the same loss function of supervised learning task.

Loss function:
$$
L=\frac{1}{n} \sum_{m=1}^n \sum_{i=1}^C L\left(y_i^m, f_i^m\right)+\alpha(t) \frac{1}{n^{\prime}} \sum_{m=1}^{n^{\prime}} \sum_{i=1}^C L\left(y_i^{\prime m}, f_i^{\prime m}\right),
$$
	where n: number of mini-batch in labeled data
		  n': number of mini-batch in unlabeled data
		  $\alpha(t)$: balance coefficient
The proper scheduling of $\alpha(t)$ is very important:
- Too high: it disturbes training
- Too small: can't benefit from unlabeled data.
=> deterministic annealing process, by which $\alpha(t)$ is slowly increased.

$$
\begin{equation}
\alpha(t)= \begin{cases}0 & t<T_1 \\ \frac{t-T_1}{T_2-T_1} \alpha_f & T_1 \leq t<T_2 \\ \alpha_f & T_2 \leq t\end{cases}
\end{equation}
$$

#### Explaination
**Low-Density separation between classes**
The goal of semi-supervised learning is to improve generalization performance using unlabeled data.

The decision boundary should lie in low-density regions to improve generalization performance. It’s more likely that data samples in a high-density region have the same label.

The network output should be insensitive to variations in the directions of low-dimensional manifold.

**Entropy Regularization**
Benefit from unlabeled data in the framework of maximum a posteriori estimation.

Favors low density separation between classes without any modeling of the density by minimizing the conditional entropy of class probabilities for unlabeled data.

$$
\begin{equation}
H\left(y \mid x^{\prime}\right)=-\frac{1}{n^{\prime}} \sum_{m=1}^{n^{\prime}} \sum_{i=1}^C P\left(y_i^m=1 \mid x^{\prime m}\right) \log P\left(y_i^m=1 \mid x^{\prime m}\right)
\end{equation}
$$





---

