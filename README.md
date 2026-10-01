# Discriminant Feature Analysis in Lipids: Mouse vs NMR

This repository presents an analysis of the features that discriminate lipid membranes of two species, Mouse and NMR (naked mole-rat), using g₃ maps derived from molecular dynamics simulations. Classification models, feature importance and ROC AUC, dimensionality-reduction embeddings and clustering are used to identify which structural descriptors best separate the two groups, and to explore subpopulations within them.

## Project description

The project focuses on two types of lipids:

- **CHOL**: Cholesterol. Characterized by a `head` and a `body`.
- **DPSM**: Sphingomyelin. Characterized by `heads` and `tails`.

For each lipid type, an independent model is trained to correctly classify which group each molecule belongs to.

## Raw data

The raw data were generated from molecular dynamics simulations of lipid membranes.

**Credits**: [Naikary Paloma Martínez Velázquez](https://www.linkedin.com/in/naikary-mart%C3%ADnez/) | Thesis Project - CIMAT Monterrey

### Process:

1. **Simulation**: Lipid bilayers of 512 lipids per membrane
   - NMR (naked mole-rat): 342 cholesterol + 170 DPSM
   - Mouse: 170 cholesterol + 342 DPSM
   - Duration: 10 µs per membrane at 310.15 K (NPT)

2. **Extraction**: Bead positions extracted from `.gro` and `.trr` trajectories

3. **Computation**: g₃ function (radial probability density) over 100 consecutive frames
   - Captures local structure: distance (r) and angle (θ)
   - Cutoff: 10 Å (13 Å for cholesterol heads)

4. **Format**: 2D maps per lipid
   - NumPy arrays (n_lipids, 401 × 201 bins)
   - One `.npy` file per region × species
   - Residue IDs in `_resids.npy` files

## Project structure

The analysis is divided into five main stages, each implemented in a Jupyter notebook:

### 1. Exploratory data analysis (`01_analysis.ipynb`)

In this stage, the data are initially loaded and fundamental descriptive statistics are obtained:

- Loading of `.npy` arrays from the `Mouse` and `NMR` directories
- Data integrity checks (null values, infinities, data types and shapes)
- Descriptive statistics of the three-dimensional data
- Analysis of the data distribution by group

**Input**: `.npy` files with raw molecular data.

**Output**: Descriptive statistical characteristics.

### 2. Feature engineering (`02_feat_engineering.ipynb`)

In this stage, the three-dimensional data are transformed into a set of numerical features that serve as input to the models:

For each molecule, the following features are extracted:

- **Residual**: Molecule identifier index (`resid`)
- **Magnitude**: Mean and standard deviation of the values
- **Ratios**: Quotient between the means and standard deviations of the components (head/body for CHOL; heads/tails for DPSM)
- **Correlation**: Pearson coefficient between components
- **Amplitude**: Range, minimum and maximum values

Two independent datasets are generated:

- `chol.csv`: Features of CHOL molecules (512 samples)
- `dpsm.csv`: Features of DPSM molecules (512 samples)

**Input**: Raw three-dimensional data (Mouse and NMR).

**Output**: Two CSV files prepared for the models/algorithms of the following stages (`chol.csv`, `dpsm.csv`).

### 3. Model building and validation (`03_training.ipynb`)

In this stage, independent classification models are trained for each lipid type:

- Loading of the generated feature sets
- Data preparation (normalization, train/validation split)
- Training of multiple classification algorithms
- Cross-validation and evaluation of metrics (precision, recall, F1-score)
- Comparative performance analysis

Cross-validation techniques are used to assess the generalization capability of the models.

**Input**: CSV files with features (`chol.csv`, `dpsm.csv`).

**Output**: Trained and evaluated models; performance metrics.

### 4. Embedding generation (`04_embeddings.ipynb`)

In this stage, lower-dimensional representations (embeddings) of the data are generated using the extracted features:

- Dropping of the minimum and maximum value features because they provide no information (see `03_training.ipynb`), as well as `lipid_type` and `target`
- Feature standardization (`StandardScaler`) before fitting the algorithms
- Dimensionality reduction using `t-SNE` and `UMAP`
- Visualization of the embeddings in low-dimensional spaces
- Analysis of the separability between groups
- Characterization of patterns and similarities in the embedding spaces
- Re-execution of the same procedure using only scale-invariant features (`ratio_*`, `corr_*`) to compare against the full feature set
- Feature evaluation by ROC AUC to identify discriminants

**Input**: Features of both lipid types.

**Output**: Embeddings and dimensionality reduction visualizations, for the full feature set and for the subset of invariant features.

### 5. Subpopulation identification (`05_nmr_chol_subpops.ipynb`)

In `01_analysis.ipynb`, `CHOL body` was found to have the highest coefficient of variation among all regions (~41-20x for **Mouse** and **NMR** respectively). Based on this, an analysis of the **NMR** group is proposed to identify possible subpopulations:

- Loading of the raw `CHOL body` data for the **NMR** group and computation of `mean_body`, `std_body`, `range_body` and a new feature: coefficient of variation (`cv_body`)
- Feature standardization (`StandardScaler`)
- Selection of the number of clusters using the elbow method, `silhouette_score` and BIC/AIC, comparing `KMeans`, `AgglomerativeClustering` (`ward` linkage) and `GaussianMixture`
- Comparison of candidate values of *k* against the original baseline `clusters_CHOL_BODY.npy`
- Visualization of the clusters projected with `t-SNE` and `UMAP`
- Final fit of the three algorithms with the optimal *k* and comparison of agreement between methods using the **Adjusted Rand Index**
- Analysis of the outlier microcluster found with the optimal *k*, with descriptive statistics and visualization of the g3 map compared against the mean of `CHOL body`

**Input**: Raw `CHOL body` data for the NMR group.

**Output**: Cluster labels per algorithm, agreement comparison between methods, and subpopulation visualizations.

## Workflow

The analysis flow follows this order:

```
Raw data (Mouse/NMR)
        |
        v
01_analysis.ipynb
(Exploration and statistics)
        |
        v
02_feat_engineering.ipynb
(Feature extraction)
        |
        +----> CHOL features
        |
        +----> DPSM features
        |
        v
03_training.ipynb
(Model training)
        |
        v
04_embeddings.ipynb
(Embedding generation and visualization)

Raw data (NMR, CHOL body)
        |
        v
05_nmr_chol_subpops.ipynb
(Subpopulation identification)
```

## Features by lipid type

### CHOL (Cholesterol)

Columns in `chol.csv`:

- `lipid_type`: Lipid type (CHOL)
- `mean_head`, `mean_body`: Mean of values
- `std_head`, `std_body`: Standard deviation
- `ratio_head_body`: Ratio of means
- `ratio_std_head_body`: Ratio of standard deviations
- `corr_head_body`: Pearson correlation
- `min_head`, `max_head`, `range_head`: Head amplitude
- `min_body`, `max_body`, `range_body`: Body amplitude
- `target`: Classification label (0=Mouse, 1=NMR)

### DPSM (Sphingomyelin)

Columns in `dpsm.csv`:

- `lipid_type`: Lipid type (DPSM)
- `mean_heads`, `mean_tails`: Mean of values
- `std_heads`, `std_tails`: Standard deviation
- `ratio_heads_tails`: Ratio of means
- `ratio_std_heads_tails`: Ratio of standard deviations
- `corr_heads_tails`: Pearson correlation
- `min_heads`, `max_heads`, `range_heads`: Heads amplitude
- `min_tails`, `max_tails`, `range_tails`: Tails amplitude
- `target`: Classification label (0=Mouse, 1=NMR)

## Main results
- Exploratory analysis presented the data distribution (mean and standard deviation) and radial and angular analysis; a high correlation was found between `DPSM_heads` and `DPSM_tails` (0.923 and 0.764) and independence (-0.004 and 0.009) between `CHOL_head` and `CHOL_body` for the **Mouse** and **NMR** groups respectively, using Pearson's $r$.
- The models trained during cross-validation achieved F1-score values of approximately 1.0, indicating a clear separation between the two groups (Mouse and NMR) based on the extracted molecular features.
- `XGBOOST` stands out, as it found `std_head` and `corr_heads_tails` to be the main discriminating features for the groups.
- The models were retrained, encapsulated in a `Pipeline` (`StandardScaler` + `SMOTE` + classifier), using only scale-invariant features (`ratio_*`, `corr_*`), maintaining an F1-score ≈ 1.0 in all three models. This rules out that the separation between groups depends solely on scale (box volume, inverted lipid composition between **Mouse** and **NMR**). With these features, `XGBoost` identifies `ratio_head_body` (`CHOL`) and `corr_heads_tails` (`DPSM`) as the main discriminants.
- The embeddings created with `t-SNE` and `UMAP`, now standardized with `StandardScaler` and without the minimum and maximum features, show very good results in the plots and in **Silhouette** ((`CHOL`: `t-SNE`=0.709, `UMAP`=0.751), (`DPSM`: `t-SNE`=0.671, `UMAP`=0.810)) and **Davies-Bouldin** ((`CHOL`: `t-SNE`=0.403, `UMAP`=0.324), (`DPSM`: `t-SNE`=0.481, `UMAP`=0.274)) for both lipid types, with `UMAP` obtaining the best score. This demonstrates cohesion and compactness among the resulting clusters.
- When repeating the embedding using only scale-invariant features (`ratio_*`, `corr_*`), `CHOL` improved in both `t-SNE` (Silhouette=0.739, Davies-Bouldin=0.350) and `UMAP` (Silhouette=0.778, Davies-Bouldin=0.307); in contrast, `DPSM` worsened in both algorithms (`t-SNE`: Silhouette=0.581, Davies-Bouldin=0.631; `UMAP`: Silhouette=0.710, Davies-Bouldin=0.399).
- The subpopulation analysis of `CHOL body` for **NMR** identified *k*=3 as the optimal number of clusters (strongest signal in `silhouette_score`), with high agreement between `AgglomerativeClustering` and `GaussianMixture` (**Adjusted Rand Index**=0.932). The clusters differ mainly in `mean_body` and `cv_body`.
- A microcluster of 4 outliers, agreed upon by all three methods (`KMeans`, `AgglomerativeClustering` and `GaussianMixture`), was obtained; it contains the lowest values of the `CHOL body` population for `std_body` (~0.3% - ~1.2%) and `range_body` (~0.3% - ~2.0%).

## Suggestion for a future project
A **Convolutional Autoencoder** could be built to compress the maps (raw data) into a dense embedding and automatically capture the molecular structure.

## Requirements

- Python 3.8 or higher
- NumPy
- Pandas
- Scikit-learn
- Matplotlib/Seaborn (for visualizations)
- UMAP or similar (for advanced dimensionality reduction)

## Execution

To run the complete analysis:

1. Make sure the raw data are available in the `Mouse/` and `NMR/` directories
2. Run the notebooks in the specified order
3. Intermediate outputs (CSV files and models) will be generated automatically

## Expected directory structure

```
mouse-nmr/
├── data/
│   ├── chol.csv
│   └── dpsm.csv
├── Mouse/
│   ├── g3_CHOL_head.npy
│   ├── g3_CHOL_body.npy
│   ├── g3_DPSM_heads.npy
│   ├── g3_DPSM_tails.npy
│   └── *_resids.npy
├── NMR/
│   ├── g3_CHOL_head.npy
│   ├── g3_CHOL_body.npy
│   ├── g3_DPSM_heads.npy
│   ├── g3_DPSM_tails.npy
│   └── *_resids.npy
└── notebooks/
    ├── 01_analysis.ipynb
    ├── 02_feat_engineering.ipynb
    ├── 03_training.ipynb
    ├── 04_embeddings.ipynb
    └── 05_nmr_chol_subpops.ipynb
```

## Author
Name: Dylan Hernández Rojas

GitHub: https://github.com/dylanhrojas

## Acknowledgments

Special thanks to:

- [Dr. Ángel David Reyes Figueroa](https://www.linkedin.com/in/adreyesf/)
- [Naikary Paloma Martínez Velázquez](https://www.linkedin.com/in/naikary-mart%C3%ADnez/), Master's candidate

## License

This project is licensed under the MIT license.

## Notes

- All provided data are normalized by box volume (`artificial_volume`)
- Residue indices (`resids`) are unique identifiers for each molecule
- The analysis assumes that the separability between groups is sufficient to obtain good classification results

## AI usage

Claude was used for the following tasks:

- Methodology recommendations
- Understanding the organization of the raw data
- Spelling and writing corrections for the notebooks
- Conceptual questions about library usage
- Autocompletion of trivial code (statistics, plots, data loading, etc.)
- Creation of `README.md`
