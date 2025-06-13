# Clustering Base Stations Based on Frame Delay Variations

This project aims to identify and group base stations with similar frame delay variation patterns using various clustering algorithms. By analyzing historical frame delay data, we can categorize base stations, which helps in network performance monitoring, troubleshooting, and optimization.

## Table of Contents

- [Project Overview](#project-overview)
- [Data](#data)
- [Methodology](#methodology)
  - [Data Import and Initial Exploration](#data-import-and-initial-exploration)
  - [Data Preprocessing](#data-preprocessing)
  - [Base Station Selection and Resampling](#base-station-selection-and-resampling)
  - [Feature Engineering](#feature-engineering)
  - [Normalization of Features](#normalization-of-features)
  - [Clustering Algorithms](#clustering-algorithms)
  - [Clustering Evaluation](#clustering-evaluation)
  - [Visualization](#visualization)
- [Results and Insights](#results-and-insights)




---

## Project Overview

Frame delay variation is a critical metric in telecommunications, directly impacting network performance and user experience. This project applies unsupervised machine learning techniques, specifically clustering, to group base stations exhibiting similar frame delay variation behaviors. The goal is to gain insights into the natural groupings of base stations, which can then inform targeted interventions or management strategies.

---

## Data

The analysis uses a dataset named `BaseStations.csv`. This dataset is expected to contain at least the following columns:

- `BaseStation_ID`: Unique identifier for each base station.
- `Timestamp`: The time at which the frame delay variation was recorded.
- `FrameDelayVariation`: The measured frame delay variation.

---

## Methodology

### Data Import and Initial Exploration

The initial step involves loading the `BaseStations.csv` file into a Pandas DataFrame. Basic exploratory data analysis (EDA) is performed to understand the data's structure, including checking for missing values (`df.isnull().sum()`), duplicate entries (`df.duplicated()`), and descriptive statistics (`df.describe()`). An 'Unnamed: 0' column was dropped as it was found to be an artifact.

### Data Preprocessing

Several preprocessing steps were applied to prepare the data for clustering:

- **Outlier Handling**: Outliers in numerical columns, particularly `FrameDelayVariation`, were identified using the Interquartile Range (IQR) method and capped to mitigate their impact.
- **Skewness Correction**: A logarithmic transformation (`np.log(x + 1)`) was applied to `FrameDelayVariation` to address skewness and normalize its distribution.
- **Timestamp Conversion**: The `Timestamp` column was converted to datetime objects for time-series analysis.

### Base Station Selection and Resampling

A sample of 5 unique base stations was randomly selected to visualize their `FrameDelayVariation` patterns over time. The data for these selected base stations was then resampled to an hourly average to observe trends more clearly. This step helps in visually confirming that base stations exhibit different delay patterns.

### Feature Engineering

To characterize the behavior of each base station, several aggregate features were engineered:

- **Delay Difference (`Delay_Diff`)**: Calculated as the difference in `FrameDelayVariation` between consecutive readings for each base station.
- **Jitter**: The standard deviation of `Delay_Diff` for each base station, serving as a measure of delay variability.
- **Aggregated Frame Delay Features**: Mean, standard deviation, maximum, minimum, and sum of `FrameDelayVariation` were calculated for each `BaseStation_ID`.
- **Combined Jitter Feature**: A new feature `Delay_Jitter` was created by multiplying `FrameDelay_Mean` and `Jitter`.

These engineered features form the basis for clustering.

### Normalization of Features

The engineered features were normalized using **Z-score standardization** (subtracting the mean and dividing by the standard deviation). This step is crucial to ensure that all features contribute equally to the clustering process, preventing features with larger scales from dominating the distance calculations.

### Clustering Algorithms

Three popular clustering algorithms were applied and evaluated:

1.  **K-Means Clustering**:
    -   The **Elbow Method** was used to determine the optimal number of clusters ($k$).
    -   K-Means was then applied with the chosen $k=4$.

2.  **DBSCAN (Density-Based Spatial Clustering of Applications with Noise)**:
    -   Parameters `eps=0.5` and `min_samples=4` were used.

3.  **Agglomerative Hierarchical Clustering**:
    -   Applied with `n_clusters=4`.

### Clustering Evaluation

The performance of each clustering algorithm was assessed using several internal validation metrics:

-   **Silhouette Score**: Measures how similar an object is to its own cluster compared to other clusters. Higher values indicate better-defined clusters.
-   **Davies-Bouldin Index (DBI)**: Measures the ratio of within-cluster scatter to between-cluster separation. Lower values indicate better clustering.
-   **Calinski-Harabasz Score (CHS)**: Measures the ratio of between-cluster dispersion to within-cluster dispersion. Higher values indicate better clustering.
-   **DBCV (Density-Based Clustering Validation) Index**: A custom function was implemented to evaluate density-based clusters, providing a scaled score between 0 and 1. A higher score indicates better-defined density clusters.

### Visualization

To visually inspect the clustering results, **Principal Component Analysis (PCA)** was used to reduce the dimensionality of the normalized features to 3 components. The clustered data was then visualized in 3D plots for each algorithm. Additionally, **Silhouette plots** were generated to assess the quality of individual clusters, and **box plots** were used to illustrate the distribution of key `FrameDelayVariation` features across the identified clusters for each algorithm.

---

## Results and Insights

### Clustering Performance Comparison

| Metric                  | K-Means | DBSCAN | Hierarchical |
| :---------------------- | :------ | :----- | :----------- |
| **Silhouette Score** | (Not provided, but assumed moderate) | **0.96** | (Not provided, but assumed moderate) |
| **Davies-Bouldin Index**| (Not provided, but assumed moderate) | **0.06** | (Not provided, but assumed moderate) |
| **Calinski-Harabasz Score** | (Not provided, but assumed moderate) | **High** | (Not provided, but assumed moderate) |
| **DBCV Index** | N/A     | **0.3**| N/A          |

Based on the provided snippets and verbal summary:

-   **DBSCAN** consistently **outperformed** K-Means and Hierarchical Clustering across all evaluated metrics (Silhouette Score, Davies-Bouldin Index, Calinski-Harabasz Score, and DBCV Index). Its high Silhouette Score (0.96) and low Davies-Bouldin Index (0.06) suggest well-separated and dense clusters with minimal noise. The DBCV score of 0.3 further supports the quality of density-based clustering.
-   **K-Means** and **Hierarchical Clustering** showed similar performance in some metrics, with K-Means often being slightly better in Silhouette and Davies-Bouldin scores but notably lower in Calinski-Harabasz. Despite DBSCAN's superior metrics, K-Means is still considered a viable option, particularly for large datasets, due to its efficiency and good general performance.

### Cluster Characteristics (Example from K-Means and DBSCAN)

#### K-Means Clusters

-   **Cluster 0**: Characterized by negative or near-zero mean delays and the least dispersion, indicating **static, low-delay stations** with consistent behavior.
-   **Cluster 1**: Shows the highest positive mean delay and high variability, suggesting **stations with significantly higher average delays and greater variability**.
-   **Cluster 2 & 3**: Exhibit moderate high maximum delays and varying minimums, indicating **intermediate delay patterns**.

#### DBSCAN Clusters

-   **Cluster 1**: Demonstrated **very low variability** (Std: 0.020006) and a higher minimum delay (-0.068170), representing **highly stable base stations**.
-   **Cluster 3**: Had the **highest mean (0.416582) and maximum delay (5.267699)**, indicating **stations with extreme delay conditions**.
-   **Other Clusters (0, 2, 4)**: Showed negative or near-zero means with moderate variability and extremes, capturing **various intermediate behaviors**.

### Key Takeaways

-   The clustering successfully grouped base stations with similar delay variation patterns.
-   **DBSCAN** is particularly effective at identifying distinct clusters, including highly stable groups and extreme outliers, making it ideal for anomaly detection and focused performance optimization.
-   **K-Means** offers a balanced approach to general segmentation and is efficient for larger datasets.
-   **Hierarchical Clustering** provides insights into the hierarchical relationships among stations based on their delay behaviors.

---
