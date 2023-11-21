
- **PublishYear**:: 2020 
- **Author**:: Aviad Aberdam, Ron Litman, Shahar Tsiper, Oron Anschel, Ron Slossberg, Shai Mazor, R. Manmatha, Pietro Perona
- **Link**:: http://arxiv.org/abs/2012.10873
- **Tags**:: #paper #self-supervised #Sequence2Sequence #OCR #ContrastiveLearning
- **Cite Key**:: [@aberdamSequencetoSequenceContrastiveLearning2020]

### Abstract
```
We propose a framework for sequence-to-sequence contrastive learning (SeqCLR) of visual representations, which we apply to text recognition. To account for the sequence-to-sequence structure, each feature map is divided into different instances over which the contrastive loss is computed. This operation enables us to contrast in a sub-word level, where from each image we extract several positive pairs and multiple negative examples. To yield effective visual representations for text recognition, we further suggest novel augmentation heuristics, different encoder architectures and custom projection heads. Experiments on handwritten text and on scene text show that when a text decoder is trained on the learned representations, our method outperforms non-sequential contrastive methods. In addition, when the amount of supervision is reduced, SeqCLR significantly improves performance compared with supervised training, and when fine-tuned with 100% of the labels, our method achieves state-of-the-art results on standard handwritten text recognition benchmarks.
```

### Notes

The key idea is to apply contrastive learning to the individual elements of the sequence, while maintaining information about their order.

Introduce an instance-mapping function that yields an instance from every few consecutive frames in a sequence feature map. The instance is the atomic element that will be used in contrastive learning.

The key contributions of our work are: 
- A contrastive learning approach for visual sequenceto-sequence recognition. 
- Viewing each feature map as a sequence of individual instances, leading to contrastive learning in a sub-word level, such that each image yields several positive pairs and multiple negative examples.
- Defining sequence preserving augmentation procedures, and custom projection heads. 
- Extensive experimental validation showing state-ofthe-art performance on handwritten text.

![[Pasted image 20221212112220.png]]

![[Pasted image 20221212112339.png]]

**Contrastive loss**:
$$
\begin{equation}
\begin{aligned}
\mathcal{L}\left(\mathcal{Z}^a, \mathcal{Z}^b\right) & =\sum_{r \in\left|\mathcal{Z}^a\right|} \ell_{\mathrm{NCE}}\left(\mathbf{z}_r^a, \mathbf{z}_r^b ; \mathcal{Z}^a \cup \mathcal{Z}^b\right) \\
& +\sum_{r \in\left|\mathcal{Z}^b\right|} \ell_{\mathrm{NCE}}\left(\mathbf{z}_r^b, \mathbf{z}_r^a ; \mathcal{Z}^a \cup \mathcal{Z}^b\right),
\end{aligned}
\end{equation}
$$
$$
\begin{equation}
\ell_{\mathrm{NCE}}\left(\mathbf{u}^a, \mathbf{u}^b ; \mathcal{U}\right)=-\log \frac{\exp \left(\operatorname{sim}\left(\mathbf{u}^a, \mathbf{u}^b\right) / \tau\right)}{\sum_{\mathbf{u} \in \mathcal{U} \backslash \mathbf{u}^a} \exp \left(\operatorname{sim}\left(\mathbf{u}^a, \mathbf{u}\right) / \tau\right)}
\end{equation}
$$
**Base Encoder**: Consider two candidates as the sequential representations $\mathbf{R}_i \in \mathbb{R}^{F \times T_i}$:
	- The visual features: $\mathbf{R}_i = \mathbf{V}_i$
	- The contextual feature map: $\mathbf{R}_i = \mathbf{H}_i$ 
**Projection head**: Small auxiliary neural network discarded after pretrain stage: MLP projection and BiLSTM projection.
**Instance-mapping**: 
	- All to instance: All frames avraged to single instance $m(\mathbf(P)) = AVG(\mathbf(P))$ -> reduce the number of negative in each batches -> deteriorate the equality of the learned representation.
	- Window to Instance: Adaptive average pooling to $T'$ instances. -> represent the trade-off between mis-alignment robustness and sample efficiency.
	- Frame to Instance: Each frame define as seperate instance.

**Augmentation**: the augmentation pipeline consists of 
	- linear contrasting, 
	- blurring, sharpening, 
	- horizontal cropping 
	- light affine transformations

**Result** 
![[Pasted image 20221212134318.png]]


---

