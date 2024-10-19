## Library:
[Imbalance-Learn](https://imbalanced-learn.org/stable/index.html)
[SOS: Score-based Oversampeling for Tabular data](https://github.com/JayoungKim408/SOS?utm_source=catalyzex.com)
[SEMO: Structure-Preserving Oversampling for Imbalanced Multivariate Time Series Data](https://github.com/jiedali/oversampling_algorithm_multivariate_time_series)


# Over-Sampling for Tabular data 
## SMOTE
Algorithms creates artificial data based on the feature space similarities between existing minority class by introducing non-replicated minority class. 

The new minority instances are extrapolated and created out of existing minority class imbalances using `k-NN algorithm`. 
#### Summary:
- Take difference between a sample and its nearest neighbor 
- Multiply the difference by a random number between 0 and 1
- Add the diff to sample to generate a new synthetic example in the feature space 
- Continue on next nearest neighbor up to K number
#### Drawback:
- SMOTE is not able to manage the bias in the majority class for the classifier where the data is high dimensional.
- Over generalization of the minority class space.
- If there are observations in the minority class which are outlying and appears in the majority class, it causes a problem for SMOTE, by creating a line bridge with the majority class.

``` python 
from collections import Counter
from sklearn.datasets import make_classification
from imblearn.over_sampling import SMOTE
X, y = make_classification(n_classes=2, class_sep=2,
							weights=[0.1, 0.9], n_informative=3, n_redundant=1, flip_y=0,
							n_features=20, n_clusters_per_class=1, n_samples=1000, random_state=10)
print('Original dataset shape %s' % Counter(y))
>> Original dataset shape Counter({1: 900, 0: 100})
sm = SMOTE(random_state=42)
X_res, y_res = sm.fit_resample(X, y)
print('Resampled dataset shape %s' % Counter(y_res))
>> Resampled dataset shape Counter({0: 900, 1: 900})
```


## Borderline-SMOTE
Presents two new minority over-sampling methods, borderline-SMOTE1 and borderline-SMOTE2
Only the minority examples near the borderline are over-sampled.
#### Motivation
The examples on the borderline and the ones nearby (borderline examples) are more apt to be misclassified than the ones far from the borderline, and thus more important for classification.
#### Summary:
- Split the Minority class set $P$ and Majority class set $N$ 
- For every $p_i$ in $P$ -> Calculate $m$ nearest neighbor from the whole training set $T$ -> $m'$ samples of Majority Set ($0<=m'<=m$)
- Consider $m'$:
	- If $m' = m$ : $p_i$ is noise
	- If $0<=m'<=m/2$ : $p_i$ is safe 
	- If $m/2<=m'<=m$  -> Put into DANGER Set (Borderline data of the Minority class $P$)
	- 
- For each sample in DANGER -> Calculate $k$ nearest neighbor from $P$ using SMOTE algorithm

```python 

from collections import Counter
from sklearn.datasets import make_classification
from imblearn.over_sampling import BorderlineSMOTE
X, y = make_classification(n_classes=2, class_sep=2,
							weights=[0.1, 0.9], n_informative=3, n_redundant=1, flip_y=0,
							n_features=20, n_clusters_per_class=1, n_samples=1000, random_state=10)
print('Original dataset shape %s' % Counter(y))
>> Original dataset shape Counter({1: 900, 0: 100})
sm = BorderlineSMOTE(random_state=42)
X_res, y_res = sm.fit_resample(X, y)
print('Resampled dataset shape %s' % Counter(y_res))
>> Resampled dataset shape Counter({0: 900, 1: 900})
```


## K-means SMOTE 
Over-sample applying a clustering before to oversample using SMOTE.
#### Summary: 
- Do K-mean Cluster on the Data
- Select cluster with $N$ proportion of Minority class samples
- Apply conventional SMOTE to these selected clusters.
#### DrawBack:
- Must find the right $K$

``` python
import numpy as np
from imblearn.over_sampling import KMeansSMOTE
from sklearn.datasets import make_blobs
blobs = [100, 800, 100]
X, y  = make_blobs(blobs, centers=[(-10, 0), (0,0), (10, 0)])
# Add a single 0 sample in the middle blob
X = np.concatenate([X, [[0, 0]]])
y = np.append(y, 0)
# Make this a binary classification problem
y = y == 1
sm = KMeansSMOTE(
    kmeans_estimator=MiniBatchKMeans(n_init=1, random_state=0), random_state=42
)
X_res, y_res = sm.fit_resample(X, y)
# Find the number of new samples in the middle blob
n_res_in_middle = ((X_res[:, 0] > -5) & (X_res[:, 0] < 5)).sum()
print("Samples in the middle blob: %s" % n_res_in_middle)
>> Samples in the middle blob: 801
print("Middle blob unchanged: %s" % (n_res_in_middle == blobs[1] + 1))
>> Middle blob unchanged: True
print("More 0 samples: %s" % ((y_res == 0).sum() > (y == 0).sum()))
>> More 0 samples: True
```

## SMOTE-NC
Over-Sampling with both Numerical and Categorical Data
#### Summary:
- For each corresponding categorical variable:
	- If Both are different: 
		- Add $M$: median of the std of the numerical features of Minority set
	- Else:
		- Add 0
	$$
	\displaylines{
	Vector1: [n1_1,n1_2,A,B]\\ 
	Vector2: [n2_1,n2_2,A,C]\\
	Distance(Vec1, Vec2) = \sqrt{(n1_1-n2_1)^2 + (n1_2-n2_2)^2 + 0 + M^2}
	}
	$$
- To get a categorical feature, assign the value occurring in the majority of the K-Nearest Neighbors (all belonging to the minority class).

## Comparing Over-Sampling performance
**SOURCE:**[A PERFORMANCE COMPARISON OF OVERSAMPLING METHODS FOR DATA GENERATION IN IMBALANCED LEARNING TASKS](https://run.unl.pt/bitstream/10362/31307/1/TEGI0396.pdf)
![[Pasted image 20240410153127.png]]

=> Borderline-SMOTE method to be most efficient for dealing with the class imbalance problem.

## SOS: Score-based Oversampling for Tabular Data


# Over-Sampling for Time Series data

## SEMO: Structure-Preserving Expectation-Maximization Oversampling
