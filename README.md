# Introduction to scRNA-seq Integration

## Overview

This project demonstrates a complete single-cell RNA-seq (scRNA-seq) integration pipeline applied to colorectal cancer immune cells, using the Harmony algorithm for batch correction. The analysis integrates data from two independent cohorts (Korean SMC and Belgian KUL datasets) to identify and annotate immune cell populations across 29 patient samples.
The analysis can be found here https://htmlpreview.github.io/?https://github.com/parulkuls26/scRNAseq-batch-integration-harmony/blob/main/session2_harmony_2026(1).html


## Dataset
Data is sourced from:
> Lee, HO., Hong, Y., Etlioglu, H.E. et al. *Lineage-dependent gene expression programs influence the immune landscape of colorectal cancer.* Nat Genet 52, 594–603 (2020). https://doi.org/10.1038/s41588-020-0636-z

- **28,888 immune cells** across 29 patient samples
- **2 cohorts:** SMC (Korean, n=22) and KUL (Belgian, n=7)
- Cell types include: T cells, NK cells, B cells, Plasma cells, Myeloid cells

## Analysis Workflow

### 1. Data Loading & QC
- Load and concatenate two `.h5ad` files
- Inspect QC metrics: `n_genes_by_counts`, `total_counts`, `pct_counts_mt`, `pct_counts_ribo`
- Data was pre-filtered (>300 genes detected, <10% mitochondrial reads)

### 2. Preprocessing
- Library size normalisation (CPM, target sum = 10,000)
- Log1p transformation
- Highly Variable Gene (HVG) selection (top 2,000 genes, per batch using `seurat_v3`)
- Regression of technical variables (`n_genes_by_counts`, `pct_counts_mt`)
- Scaling to unit variance (max value clipped at 10)

### 3. Dimensionality Reduction (Unintegrated)
- PCA (50 components, `svd_solver='arpack'`)
- Neighbourhood graph construction (k=15, 20 PCs)
- UMAP embedding as unintegrated baseline

### 4. Harmony Integration
- Batch correction across SMC and KUL cohorts
- Corrected PCs stored in `adata.obsm['X_pca_harmony']`
- Neighbourhood graph and UMAP recomputed using Harmony-corrected PCs
- Comparison of unintegrated vs integrated UMAP embeddings

### 5. Clustering & Major Cell Type Annotation
- Leiden clustering (resolution=0.4)
- Differential expression using Wilcoxon rank-sum test
- Manual annotation using canonical marker genes:
  - **B cells:** CD79A, BANK1
  - **Plasma cells:** IGHG3, JCHAIN, MZB1, IGHA1, IGKC
  - **Myeloid:** CD68, CD14, LYZ
  - **Neutrophils:** FCGR3B, SLC11A1
  - **T-NK-ILC:** TRAC, CD3D
  - **Cycling:** MKI67, TOP2A, PCNA

### 6. T-NK-ILC Subpopulation Analysis
- Subset 7,491 T-NK-ILC cells for deeper annotation
- Re-clustering at higher resolution (resolution=0.2)
- Subtype annotation using T/NK-specific markers:
  - **CD4+ T cells:** CD4
  - **CD8+ T cells:** CD8A, CD8B
  - **NK cells:** KLRC1, FCER1G
  - **Tregs:** FOXP3, CTLA4, IL2RA
  - **Cycling:** MKI67, TOP2A

## Tools & Dependencies

| Package | Purpose |
|---|---|
| scanpy | Core single-cell analysis |
| harmonypy | Batch integration |
| anndata | Data structure |
| leidenalg | Graph-based clustering |
| python-igraph | Graph computations |
| pandas / numpy | Data manipulation |
| matplotlib | Visualisation |
| scikit-misc | Required for seurat_v3 HVG |
| openpyxl | Export DE results to Excel |

Install dependencies:
```bash
conda create --name scRNA python=3.12 -y
conda activate scRNA
conda install -c conda-forge scanpy anndata leidenalg python-igraph jupyterlab -y
pip install harmonypy scikit-misc openpyxl
```

## Repository Structure
```
scRNAseq-harmony-integration/
├── README.md
├── notebooks/
│   └── session2_harmony_2026.ipynb
├── results/
│   └── DE_results_cellType.xlsx
└── figures/
    ├── umap_before_harmony.png
    ├── umap_after_harmony.png
    ├── marker_dotplot_all_cells.png
    └── marker_dotplot_TNKILC.png
```

## Key Results
- Successfully integrated 28,888 immune cells from 2 independent cohorts
- Harmony effectively removed batch effects between SMC and KUL datasets
- Identified 5+ major immune cell populations
- Resolved T-NK-ILC subpopulations including CD4+, CD8+ T cells, NK cells and Tregs
- DE analysis confirmed canonical marker genes for each cluster

## Notes
- Raw `.h5ad` data files are not included due to size — available via QMplus or the original publication
- Analysis was run locally on Apple Silicon (M-series) Mac using miniforge3
- Dataset is from colorectal cancer patient samples (tumour-infiltrating immune cells only)
