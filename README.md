
<!-- README.md is generated from README.Rmd. Please edit that file -->

# cyCombine <img src="cyCombine.png" width="200" align="right" />

<!-- badges: start -->
<!-- [![Lifecycle: experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental) -->

[![Lifecycle:
stable](https://img.shields.io/badge/lifecycle-stable-brightgreen.svg)](https://lifecycle.r-lib.org/articles/stages.html#stable)
<!-- badges: end -->

<!-- ## Clone github repository -->
<!-- ``` {sh, eval = FALSE} -->
<!-- # Run in terminal -->
<!-- git clone git@github.com:shdam/cyCombine.git -->
<!-- ``` -->
<!-- ## Restore renv library -->
<!-- ``` {r, eval = FALSE} -->
<!-- # Open project in Rstudio -->
<!-- # Install renv and restore library -->
<!-- install.packages("renv") -->
<!-- library(renv) -->
<!-- renv::restore() -->
<!-- ``` -->

## Install from github

``` r
# To ensure Rstudio looks up BioConductor packages run:
setRepositories(ind = c(1:6, 8))
# Then install package with
devtools::install_github("biosurf/cyCombine")
```

## Vignettes

Vignettes are available at [biosurf](https://biosurf.org/cyCombine).

There is a total of eight vignettes, which cover a range of different
topics:

1.  A [reference manual](https://biosurf.org/cyCombine_ref_manual.html)
    showing general commands for using cyCombine.
2.  A [performance benchmarking
    vignette](https://biosurf.org/cyCombine_benchmarking.html) that
    shows an example of batch correction using five different tools,
    including calculation of EMD reductions and MAD scores.
3.  A
    [vignette](https://biosurf.org/cyCombine_detect_batch_effects.html)
    covering how to detect batch effects.
4.  A [panel merging
    vignette](https://biosurf.org/cyCombine_panel_merging.html)
    including an extended discussion of how to use and evaluate merging.
5.  An [example
    vignette](https://biosurf.org/cyCombine_CyTOF_1panel.html) covering
    how to process a 1-panel mass cytometry (CyTOF) dataset.
6.  An [example
    vignette](https://biosurf.org/cyCombine_CyTOF_2panels.html) covering
    how to process a 2-panel mass cytometry (CyTOF) dataset.
7.  A [vignette](https://biosurf.org/cyCombine_Spectralflow_CyTOF.html)
    showing an example with integration of datasets from spectral flow
    cytometry (SFC) and mass cytometry (CyTOF).
8.  A
    [vignette](https://biosurf.org/cyCombine_CITEseq_Spectral_CyTOF.html)
    covering an example with integration of datasets from spectral flow
    cytometry (SFC), CITE-seq, and mass cytometry (CyTOF).

## Usage

### From a directory of uncorrected .fcs files

``` r
library(cyCombine)
library(magrittr)
# Directory containing .fcs files
data_dir <- "data/raw"
# Markers of interest
markers <- c("CD20", "CD3", "CD27", "CD45RA", "CD279", "CD5", "CD19", "CD14", "CD45RO", "GranzymeA", "GranzymeK", "FCRL6", "CD355", "CD152", "CD69", "CD33", "CD4", "CD337", "CD8", "CD197", "LAG3", "CD56", "CD137", "CD161", "FoxP3", "CD80", "CD270", "CD275", "CD134", "CD278", "CD127", "KLRG1", "CD25", "HLADR", "TBet", "XCL1")
# The list of markers can also be imported from a panel file (See the reference manual for an example)

# Compile fcs files, down-sample, and preprocess
uncorrected <- prepare_data(data_dir = data_dir,
                             markers = markers,
                             metadata = file.path(data_dir, "metadata.xlsx"), # Can also be .csv file or data.frame object
                             sample_ids = NULL,
                             batch_ids = "Batch",
                             filename_col = "FCS_name",
                             condition = "Set",
                             down_sample = TRUE,
                             sample_size = 500000,
                             seed = 473,
                             cofactor = 5) 
saveRDS(uncorrected, file = "_data/cycombine_raw_uncorrected.RDS")

# Run batch correction
corrected <- uncorrected %>%
  batch_correct(markers = markers,
                norm_method = "scale", # "rank" is recommended when combining data with heavy batch effects
                rlen = 10, # Consider a larger value, if results are not convincing (e.g. 100)
                covar = "condition")
saveRDS(corrected, file = "_data/cycombine_raw_corrected.RDS")
```

### The modular workflow

If your data is in another format than FCS files or a flowset, please
convert your data to a *tibble*, add the relevant columns (*sample*,
*batch*, *covar*/*condition*/*anchor*), and begin from
*transform_asinh()* (if your data is not yet transformed; otherwise,
skip that step as well).

``` r
library(cyCombine)
library(magrittr)
# Directory containing .fcs files
data_dir <- "data/raw"
# Markers of interest
markers <- c("CD20", "CD3", "CD27", "CD45RA", "CD279", "CD5", "CD19", "CD14", "CD45RO", "GranzymeA", "GranzymeK", "FCRL6", "CD355", "CD152", "CD69", "CD33", "CD4", "CD337", "CD8", "CD197", "LAG3", "CD56", "CD137", "CD161", "FoxP3", "CD80", "CD270", "CD275", "CD134", "CD278", "CD127", "KLRG1", "CD25", "HLADR", "TBet", "XCL1")

# Compile fcs files, down-sample, and preprocess
flowset <- compile_fcs(data_dir = data_dir,
                   pattern = "\\.fcs")

# Convert the generated flowset into a tibble
df <- convert_flowset(metadata = file.path(data_dir, "metadata.xlsx"),
                      sample_ids = NULL,
                      batch_ids = "Batch",
                      filename_col = "FCS_name",
                      condition = "Set",
                      down_sample = TRUE,
                      sample_size = 500000,
                      seed = 473)

# Transform data
uncorrected <- df %>% 
  transform_asinh(markers = markers)

saveRDS(uncorrected, file = "_data/cycombine_raw_uncorrected.RDS")

# Run batch correction
labels <- uncorrected %>%
  normalize(markers = markers,
            norm_method = "scale") %>%
  create_som(markers = markers,
             rlen = 10)

corrected <- uncorrected %>%
  correct_data(label = labels,
               covar = "condition")
saveRDS(corrected, file = "_data/cycombine_raw_corrected.RDS")
```

<!-- ### From a flowset -->
<!-- ```{r, eval = FALSE} -->
<!-- library(cyCombine) -->
<!-- library(magrittr) -->
<!-- # Load data -->
<!-- # Should contain the flowset, sample_ids, batch_ids, and markers of interest -->
<!-- load("data/flowset.Rdata") -->
<!-- # Convert flowset to workable datafram and transform data -->
<!-- uncorrected <- flowset %>% -->
<!--   convert_flowset(batch_ids = batch_ids, -->
<!--                   sample_ids = sample_ids, -->
<!--                   down_sample = TRUE, -->
<!--                   sample_size = 100000, -->
<!--                   seed = 473) %>%  -->
<!--   transform_asinh(markers = markers) -->
<!-- # Run batch correction -->
<!-- corrected <- uncorrected %>% -->
<!--   batch_correct(seed = 473) -->
<!-- ``` -->

## Plotting

``` r
# Full analysis - type ?run_analysis to see how you can modify the analysis
run_analysis(tool = "cycombine", data = "raw", data_dir = "_data", markers = markers)

# Otherwise, plots can be made like so:
density_plots(uncorrected = uncorrected,
              corrected = corrected,
              markers = markers,
              filename = 'figs/densities_withcovar.png')

# PCA plot uncorrected
pca1 <- uncorrected %>%
  plot_dimred('uncorrected', type = 'pca')
  
# PCA plot corrected
pca2 <- corrected %>%
  plot_dimred('corrected', type = 'pca')
save_two_plots(pca1, pca2, filename = 'figs/pca.png')

# UMAP
# UMAP plot uncorrected
set.seed(473)
sample <- sample(1:nrow(uncorrected), 20000)
plot1 <- plot_dimred(uncorrected[sample,], type = 'umap', name = 'Uncorrected')
plot2 <- plot_dimred(corrected[sample,], type = 'umap', name = 'Corrected')
save_two_plots(plot1, plot2, filename = 'figs/umap.png')
```
