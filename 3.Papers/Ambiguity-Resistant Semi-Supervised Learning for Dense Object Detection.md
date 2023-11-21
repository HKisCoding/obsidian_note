
- **PublishYear**:: 2023 
- **Author**:: Chang Liu, Weiming Zhang, Xiangru Lin, Wei Zhang, Xiao Tan, Junyu Han, Xiaomao Li, Errui Ding, Jingdong Wang
- **Link**:: https://ieeexplore.ieee.org/document/10204537/
- **Tags**:: #paper #semi-supervised  #ObjectDetection 
- **Cite Key**:: [@liuAmbiguityResistantSemiSupervisedLearning2023]

### Abstract
```
With basic Semi-**Supervised** Object Detection (SSOD) techniques, one-stage detectors generally obtain limited promotions compared with two-stage clusters. We experimentally find that the root lies in two kinds of ambiguities: (1) Selection ambiguity that selected pseudo labels are less accurate, since classification scores cannot properly represent the localization quality. (2) Assignment ambiguity that samples are matched with improper labels in pseudolabel assignment, as the strategy is misguided by missed objects and inaccurate pseudo boxes. To tackle these problems, we propose a Ambiguity-Resistant Semi-supervised Learning (ARSL) for one-stage detectors. Specifically, to alleviate the selection ambiguity, Joint-Confidence Estimation (JCE) is proposed to jointly quantifies the classification and localization quality of pseudo labels. As for the assignment ambiguity, Task-Separation Assignment (TSA) is introduced to assign labels based on pixel-level predictions rather than unreliable pseudo boxes. It employs a ’divideand-conquer’ strategy and separately exploits positives for the classification and localization task, which is more robust to the assignment ambiguity. Comprehensive experiments demonstrate that ARSL effectively mitigates the ambiguities and achieves state-of-the-art SSOD performance on MS COCO and PASCAL VOC. Codes can be found at https://github.com/PaddlePaddle/PaddleDetection.
```


### Keywords:
- One stage detector 
- 2 stage detector 
- FCOS 
- RPN 
- ROI POOLING
- IoU-estimation methods
- Sub-optimal state 
### Problems: 
- *Selection ambiguity*: 
	- caused by the inconsistency between the classification scores and localization quality.
	- 
	- the selected pseudo labels for unlabeled images are less accurate. It is caused by the mismatch between classification scores and localization quality.
- *Asignment ambiguity*: ![[Pasted image 20231112204824.png]]
### Contribution: 
- Analyze the semi-supervised learning on one-stage detector -> limitation lies on selection and assignment ambiguities of pseudo labels
- Propose JCE and TSA:
	- **Joint-Confidence Estimation (JCE)** is proposed to jointly quantifies the classification and localization quality of pseudo labels.
	- **Task-Separation Assignment (TSA)** is introduced to assign labels based on pixel-level predictions rather than unreliable pseudo boxes.
- Achieves state-of-the-art performance on MS COCO and PASCAL VOC.

### Methodology 
![[Pasted image 20231112201541.png]]

Using FCOS as base model:
	1. Applied basic SSOD framework to FCOS as baseline
	2. Applied ARSS framework to FCOS and compare with baseline

#### 1. Baseline framework 
- 2 stages: 
	- burn-in stage: FCOS is pre-trained on the labeled data and duplicated into a student and teacher model.
	- self-training stage: the teacher generates pseudo labels for unlabeled images and guides the student. 

$$
L=L_{\text {sup }}+\beta L_{\text {unsup }},
$$
- Pseudo-labels predicted in weakly-aug and filtered by confidence score = classification * centerness score
- Converted pseudo-label into pixel-level targets for assignment strategy 
- Student is trained on labeled img and strong-aug unlabeled imgs 
- Teacher model updated based on the EMA of the student model 

#### 2. Proposed framework 
2.1. JCE 
![[Pasted image 20231112205734.png]]

**Join confidence score**  = Classification score * predicted IoU score
$$
\hat{S}=\hat{S}_{c l s} * \hat{S}_{i o u}
$$
The united classification $L_{cls} = FL(\hat{S}, S)$ (Focal Loss)

The learning target S are set according to the teacher’s predictions: 
$$
S= \begin{cases}\{0, \cdots, \operatorname{IoU}, \cdots, 0\}, & \text { Labeled } \\ \left\{0, \cdots, \operatorname{Max}\left(\hat{S}_t\right), \cdots, 0\right\}, & \text { Unlabeled }\end{cases}
$$
$\operatorname{Max}(\hat{S}_t)$ is the largest score of the teacher’s responses among all categories.

To make the auxiliary branch focus on IoU estimation, an additional IoU loss: 
$$
L_{iou} = BCE(\hat{S}_{iou}, IoU)
$$

**Task-Separation Assignment (TSA)** uses negative and positive thresholds $\{τ_{neg}, τ_{pos}\}$ to divide samples into negatives, ambiguous candidates, and positives
$$
x_i=\left\{\begin{array}{ll}
\text { Negative, } & \operatorname{Max}\left(\hat{S}_i\right)<\tau_{\text {neg }} \\
\text { Candidate, } & \tau_{\text {neg }} \leq \operatorname{Max}\left(\hat{S}_i\right) \leq \tau_{\text {pos }} \\
\text { Positive, } & \operatorname{Max}\left(\hat{S}_i\right)>\tau_{p o s}
\end{array},\right.
$$
$τ_{neg}$ is fixed to 0.1, and $τ_{pos}$ is dynamically calculated based on the mean and standard deviation of candidates and positives:
$$
\tau_{\text {pos }}=\operatorname{Max}(\hat{S})_{\text {mean }}+\operatorname{Max}(\hat{S})_{s t d}
$$

**Exploit the candidate samples:**
- Classification Mining: All the candidate samples participate in the consistency learning to mimic the classification response of the teacher 
- Localization Mining: select potential positives according to their similarity with positives, and set the matching positives as localization targets. Similarity include:
	- Classification similarity
	- Localization similarity 
	- Position similarity 
Given a potential positive sample, its localization target B is calculated:
$$
B=\frac{\sum_{i=1}^N \operatorname{Max}\left(\hat{S}_i\right) * \hat{B}_i}{\sum_{i=1}^N \operatorname{Max}\left(\hat{S}_i\right)}
$$
N represents the number of matched positives, $\hat{S}_i$ and $\hat{B}_i$ are the joint confidence and bounding box of i-th positives predicted by the teacher.

**Loss function:**
$$
\begin{aligned}
L_{\text {unsup }} & =\frac{1}{N_{c l s}} \sum_{i=1}^{N_{c l s}} L_{c l s}\left(s_i, \hat{s}_i\right)+\frac{1}{N_{l o c}} \sum_{i=1}^{N_{\text {loc }}} L_{l o c}\left(b_i, \hat{b}_i\right) \\
& +\frac{\lambda}{N_{l o c}} \sum_{i=1}^{N_{\text {loc }}} L_{i o u}\left(p_i, \hat{p}_i\right)
\end{aligned}
$$
$L_{l o c}$ denotes the GIoU loss, and the weighting terms λ is set to 0.5.

### Conclusion 
The verification experiments demonstrate that our methods can effectively alleviate the ambiguities. Compared with the baseline, ARSL obtains a remarkable improvement and achieves state-of-the-art performance on MS COCO and PASCAL VOC



---

