
- **PublishYear**:: 2024 
- **Author**:: Ling Ding, Chao Li, Di Jin, Shifei Ding
- **Link**:: 
- **Tags**:: #paper
- **Cite Key**:: [@dingSurveySpectralClustering2024]

### Abstract
```
Spectral clustering converts the data clustering problem to the graph cut problem. It is based on graph theory. Due to the reliable theoretical basis and good clustering performance, spectral clustering has been successfully applied in many fields. Although spectral clustering has many advantages, it faces the challenges of high time and space complexity when dealing with large scale complex data. Firstly, this paper introduces the basic concept of graph theory, reviews the properties of Laplacian matrix and the traditional graph cuts method. Then, it fo­ cuses on four aspects of the realization process of spectral clustering, including the construction of similarity matrix, the establishment of Laplacian matrix, the selection of eigenvectors and the determination of the number of clusters. In addition, some successful applications of spectral clustering are summarized. In each aspect, the shortcomings of spectral clustering and some representative improved algorithms are emphatically analyzed. Finally, the paper comprehensively analyzes some research on spectral clustering that has not yet been in-depth, and gives prospects on some valuable research directions.
```

### Notes

Spectral clustering converts the data clustering problem to the graph cut problem.

4 aspects of the realization process of spectral clustering:
- The construction of similarity matrix
- The establishment of Laplacian matrix
- The selection of eigenvectors
- The determination of the number of clusters

### Spectral Clustering:
1. Define the similarity matrix $W$
2. Calculate Degree matrix: $$d_i = \sum W_{ij}, i = 1, ...,n$$
3. Calculate the unnormalize Laplacian matrix: $L = D - W$
4. Ues the properties of Laplacian matrix L to select the eigenvectors corresponding to the smallest c eigenvalues to form the feature matrix $F ∈ R^{n×c}$ 
5. Clustering each row in feature matrix F with k-means

### Laplacian matrix of Graph
Ref: https://mbernste.github.io/posts/laplacian_matrix/
Laplacian is the divergence of gradient -> If at some point, 𝑥, the steepness is not changing – that is, 𝑥 is located at a steady incline or a steady decline – then the divergence of the gradient at 𝑥 will be zero. That is, the Laplacian will be zero. On the other hand, if at 𝑥, the steepness is shrinking or growing – for example, because we’re close to a local maximum/minimum – then the divergence (i.e. Laplacian) will be either large or small (large if we’re approaching a minimum and small if we’re approaching a maximum).

![[Pasted image 20240615024821.png]]




---

