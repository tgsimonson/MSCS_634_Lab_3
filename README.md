# MSCS 634 Lab 3: Clustering with K-Means and K-Medoids

**Todd Simonson**
MSCS-634: Advanced Big Data and Data Mining
Dr. Satish Penmatsa

## Purpose

This lab applies two partitioning clustering algorithms, K-Means and K-Medoids, to the Wine dataset from scikit-learn: 178 wines from three cultivars, described by 13 chemical measurements. The features were standardized with z-score normalization, and each algorithm was run with k = 3 without using the class labels. The results were then evaluated two ways: the **Silhouette Score** measures how well-defined the clusters are, and the **Adjusted Rand Index (ARI)** measures how closely the clusters match the true cultivars. The clusters were visualized side by side with their centroids and medoids marked.

## Repository Contents

| File | Description |
|---|---|
| `MSCS_634_Lab_3.ipynb` | Notebook with data exploration, standardization, both algorithms, metrics, plots, and analysis |
| `requirements.txt` | Python packages needed to run the notebook |

The Wine dataset ships with scikit-learn (`sklearn.datasets.load_wine`), so no data file is needed.

## How to Run

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook MSCS_634_Lab_3.ipynb
```

## Results

| | K-Means | K-Medoids |
|---|---|---|
| Silhouette score | **0.2849** | 0.2676 |
| Adjusted Rand Index | **0.8975** | 0.7411 |
| Cluster sizes | 65, 51, 62 | 55, 74, 49 |
| Wines in a cluster matched to another class | **6** | 16 |

## Key Insights

- **K-Means produced better-defined clusters on both metrics.** The silhouette gap is small (0.017), so both found similarly compact groups, but the ARI gap is large (0.156): K-Means' clusters match the real cultivars much more closely.
- **Both recovered class_0 and class_2 perfectly.** Every disagreement involves class_1, which sits between the other two cultivars, as the PCA plot shows: the three groups form an arc with class_1 in the middle.
- **Why K-Medoids scored lower.** A medoid must be a real wine. The class_0 medoid (wine 35) lies on the edge of its group facing class_1 instead of in the center, which pushes that cluster's boundary outward. As a result, it absorbed a band of 15 class_1 wines, visible as negative silhouette values in the K-Medoids silhouette plot.
- **Silhouette scores near 0.27 to 0.28 reflect real overlap between the cultivars,** not a failure of either algorithm. k = 3 gave the highest silhouette for both algorithms among k = 2 to 6, which supports matching k to the three cultivars.
- **Stability:** K-Medoids gave the identical result for all 10 random seeds tested. K-Means varied slightly (ARI 0.8975 or 0.9149).
- **When to use each:**
  - **K-Means** suits clean, numeric, standardized data with roughly round clusters, and it scales to large datasets.
  - **K-Medoids** suits data with outliers, non-Euclidean or custom distance measures (it accepts any distance, and Manhattan distance improved its ARI here to 0.7694), and cases where a real, interpretable example for each cluster is needed.

## Challenges and Decisions

1. **The usual K-Medoids library does not install on current Python.** The `KMedoids` class in `scikit-learn-extra` predates NumPy 2 and fails to import on Python 3.12 and later (it depends on `distutils`, which Python removed). The notebook uses the maintained [`kmedoids`](https://pypi.org/project/kmedoids/) package instead, which implements FasterPAM (Schubert & Rousseeuw, 2021), an optimized version of the classic PAM algorithm with the same goal. It was configured with Euclidean distance so its results compare directly with K-Means.
2. **Plotting 13-dimensional clusters.** Clustering ran in all 13 standardized dimensions. For the side-by-side plots, the data was projected onto its first two principal components, which capture 55.4% of the variance. The K-Means centroids were projected with the same transformation, and the medoids were plotted at the positions of their actual wines.
3. **Matching colors across plots.** Cluster numbers are arbitrary, so the same group can be labeled 0 by one algorithm and 2 by the other. Each cluster was matched to the true class most of its wines belong to, for coloring only. ARI does not depend on cluster numbering, so the metrics were unaffected.
4. **Checking the result was not a fluke.** Supplementary checks were added: silhouette scores across k = 2 to 6, results across 10 random seeds, a Manhattan-distance version of K-Medoids, and the distance from each K-Means centroid to its nearest real wine. That last check showed the centroids already sit inside dense regions of the data (1.08 to 1.55 units from the nearest wine, compared with a typical neighbor spacing of 1.74), which explains why restricting centers to real wines did not help here.

## Reference

Schubert, E., & Rousseeuw, P. J. (2021). Fast and eager k-medoids clustering: O(k) runtime improvement of the PAM, CLARA, and CLARANS algorithms. *Information Systems, 101*, 101804. https://doi.org/10.1016/j.is.2021.101804
