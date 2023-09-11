# README
---
#2 try
# Acute Myeloid Leukemia Heatmap

This code generates a clustering heatmap using RNA-seq data from an acute myeloid leukemia (AML) sample dataset. The data is pre-processed and quantile normalized using the [refine.bio](https://www.refine.bio/) platform.

## Purpose of the Analysis

The purpose of this analysis is to cluster and visualize gene expression patterns in AML samples. The code uses the `pheatmap` package for clustering and creating the heatmap.

## Set up Analysis Folders

Before running the code, ensure that the following folders exist in the project directory:

- `data`: To store input data files
- `plots`: To store output plots
- `results`: To store intermediate or final results

```R
if (!dir.exists("data")) {
  dir.create("data")
}

plots_dir <- "plots"
if (!dir.exists(plots_dir)) {
  dir.create(plots_dir)
}

results_dir <- "results"
if (!dir.exists(results_dir)) {
  dir.create(results_dir)
}
```

## Install Libraries

Make sure you have the `pheatmap` library installed. If not, run the following code to install it.

```R
if (!("pheatmap" %in% installed.packages())) {
  install.packages("pheatmap", update = FALSE)
}

library(pheatmap)
library(magrittr)
set.seed(12345)
```

## Import and Set up Data

The code imports metadata and gene expression data from TSV files. Make sure the TSV files are located in the `data` directory.

```R
metadata <- readr::read_tsv(metadata_file)
expression_df <- readr::read_tsv(data_file) %>%
  tibble::column_to_rownames("Gene")
```

## Choose Genes of Interest

The code selects genes based on their variance. By default, it selects genes in the upper quartile of variance.

```R
variances <- apply(expression_df, 1, var)
upper_var <- quantile(variances, 0.75)
df_by_var <- data.frame(expression_df) %>%
  dplyr::filter(variances > upper_var)
```

## Prepare Metadata for Annotation

The code prepares the metadata for annotating the heatmap. It extracts relevant information from the metadata, such as mutation type and treatment.

```R
annotation_df <- metadata %>%
  dplyr::mutate(
    mutation = dplyr::case_when(
      startsWith(refinebio_title, "TET2") ~ "TET2",
      startsWith(refinebio_title, "IDH2") ~ "IDH2",
      startsWith(refinebio_title, "WT") ~ "WT",
      TRUE ~ "unknown"
    )
  ) %>%
  dplyr::select(
    refinebio_accession_code,
    mutation,
    refinebio_treatment
  ) %>%
  tibble::column_to_rownames("refinebio_accession_code")
```

## Create the Clustering Heatmap

The code generates the clustering heatmap using the `pheatmap` function. It clusters both rows and columns and uses the prepared metadata for annotation.

```R
heatmap_annotated <- pheatmap(
  df_by_var,
  cluster_rows = TRUE,
  cluster_cols = TRUE,
  show_rownames = FALSE,
  annotation_col = annotation_df,
  main = "Annotated Heatmap",
  colorRampPalette(c("deepskyblue", "black", "yellow"))(25),
  scale = "row"
)
```

## Save the Heatmap as PNG

The code saves the annotated heatmap as a PNG image file in the `plots` directory.

```R
png(file.path(plots_dir, "aml_heatmap.png"))
heatmap_annotated
dev.off()
```

## Session Info

The code prints the session information to keep track of the packages and versions used.

```R
sessioninfo::session_info()
```

---

You can create a new file called `README.md` in your project directory and paste the above content into it. Make sure to update any file paths or other details based on your specific project setup.

This README provides an overview of the code, explains its purpose, and gives instructions on how to set up the analysis folders, install libraries, import and preprocess the data, choose genes of interest, prepare metadata, create the clustering heatmap, and save the output. It also includes the session information for reference.

Feel free to customize the README based on your specific needs and add any additional information that might be relevant to your project.

I hope this helps! Let me know if you have any further questions.
