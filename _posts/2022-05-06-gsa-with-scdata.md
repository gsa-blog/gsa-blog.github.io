---
layout: post
title: "GSA Analysis with scRNA-seq Data"
author: "Xie Zihong"
header-style: text
lang: en
tags:
  - GSA
  - Statistics
  - Single-cell RNA sequencing
  - R
---

Single-cell RNA-seq data, which always has high levels of technical noise and intrinsic biological stochasticity, sometimes may not be suitable for previous differential expression analysis approaches developed for bulk RNA-seq data. 

Gene set or pathway enrichment analysis is a computational approach that determines whether an prior defined gene sets such as a pathway shows statistically significant of a dataset or between two biological states. It is powerful when our single-cell RNA-seq data does not have obvious meaningful high-level expressed genes or a statistically significant difference between two biological states.

Here, we briefly introduce some tools for GSA analysis with scRNA-seq data using R or web platform.



## 1. Liger

This R package is for bulk RNA gene set enrichment analysis, but we can apply it to single-cell RNA data if we have normalized expression matrix as well as cell type and condition annotation.

Here is an example using processed sub-sample data *(min.cells = 50, 1000 cells from donor patients and 1000 cells from IPF patients, **macrophage only**)* from GEO (GSE122960).

Data download: https://zenodo.org/record/6523552/files/sample.rds?download=1



##### 1. Installation

```R
install.packages("liger")
```

##### 2. Loading example data and gene set data

```R
library(liger)
library(Seurat)
sample <- read.RDS("sample.rds")
data("org.Hs.GO2Symbol.list")
head(org.Hs.GO2Symbol.list)
## $`GO:0000002`
## [1] "AKT3" "C10orf2" "DNA2" "LIG3" "MEF2A" "MGME1"
## [7] "MPV17" "OPA1" "PID1" "PRIMPOL" "SLC25A33" "SLC25A36"
## [13] "SLC25A4" "STOML2" "TYMP"
...
```

##### 3. Prepare matrix and annotation data for DEG analysis

```R
NormalizeData(sample)
mtx <- sample@assays$RNA@data #	normalized data
ann <- sample$Class
names(ann) <- colnames(mtx)
```

##### 4. Run differential expression analysis using simple t-test

```R
vals.info <- lapply(1:nrow(mtx), function(i) {
  pv <- t.test(
    mtx[i, ann == "Donor"],
    mtx[i, ann == "IPF"]
  )
  return(list(val=pv$statistic, p=pv$p.value))
})
vals <- unlist(lapply(vals.info, function(x) x$val))
p <- unlist(lapply(vals.info, function(x) x$p))
names(p) <- names(vals) <- rownames(mtx)
                   
p.adj <- p.adjust(p, method="bonferroni") # multiple-testing correction
names(p.adj) <- rownames(mtx)
                   
barplot(sort(-log10(p.adj), decreasing=TRUE), ylim=c(0, 100), las=2)
abline(h = -log10(0.05), col="red")
```

![](https://github.com/gsa-blog/gsa-blog.github.io/raw/master/img/gsa_scdata1.png)

*Fig. Differential expression analysis results for genes. Barplot shows -log10(p-value) for each gene. Red line shows the p = 0.05 significant threshold. Only few genes passes the significant threshold among all 7484 genes*



##### 5. Run interative bulk GSEA on 100 gene sets as test

```R
gseaVals <- iterative.bulk.gsea(
  values = vals,
  set.list = org.Hs.GO2Symbol.list[1:100], # we choose only 100 gene sets
  rank=TRUE)
head(gseaVals)
##                 p.val     q.val     sscore       edge
## GO:0000002 0.64356436 0.6530285  0.4553371  1.1104947
## GO:0000003 0.63366337 0.6525787 -0.4260054  3.1379212
## GO:0000012 0.07192807 0.2919434  1.0577473  1.8439200
## GO:0000014 0.25742574 0.4511489 -0.7475910  2.1944566
## GO:0000018 0.04695305 0.2492123 -1.1409113 -0.5427542
## GO:0000028 0.04495504 0.2492123  1.1303700  8.8042035

# identify significantly enriched gene sets
gseaSig <- rownames(gseaVals[gseaVals$q.val < 0.05,])
print(gseaSig)
## [1] "GO:0000184"

# look at plot of significantly enriched gene sets
for(i in seq_along(gseaSig)) {
  gs <- org.Hs.GO2Symbol.list[[gseaSig[i]]]
  gsea(values=vals, geneset=gs, mc.cores=1, plot=TRUE, rank=TRUE)
}
```

![](https://github.com/gsa-blog/gsa-blog.github.io/raw/master/img/gsa_scdata2.png)

*Fig. Gene set enrichment plot for gene set GO:0000184 demonstrates significant enrichment of our single-cell RNA data*







## 2. testSctpa

This R package provide six portable interfaces (*"AUCell", "Vision", "GSVA", "ssGSEA", "plage", "zscore"*) for PAS (*Pathway activity score*) calculation tools and abundant pathway databases in human and mouse.

Here is an example using the processed sub-sample data from GEO (GSE122960).

Download link: https://zenodo.org/record/6497091/files/GSE122960.rds?download=1



##### 1. Installation

```R
# install.packages("GSVA")
# devtools::install_github("YosefLab/VISION")
BiocManager::install("AUCell")
devtools::install_github('zgyaru/testSctpa')
```

##### 2. Load example data

```R
library(testSctpa)
library(Seurat)
library(dplyr)
se_oj <- readRDS("GSE122960.rds")
```

##### 3. Calculate pathway activity score

```R
se_oj <- cal_PAS(seurat_object = se_oj,
          tool = 'AUCell',   ## GSVA, ssGSEA, plage, zscore or Vision
          normalize = 'log',
          species = 'human', 
          pathway='kegg') ## more pathway, see https://github.com/zgyaru/testSctpa
```

After calculation, another assay named PAS with ***pathway x cell*** information would be created.

Next, all the process would use PAS assay.

##### 4. Process cells using PAS by Seurat

```R
se_oj <- FindVariableFeatures(se_oj, verbose = FALSE)
se_oj <- ScaleData(se_oj)
## RunPCA(), RunUMAP(), FindNeibeighbors(), FindClusters() are not needed here because the data has cell type annotation already
```

##### 5. Cell-type specific pathways

```R
markers <- FindAllMarkers(se_oj,logfc.threshold = 0)
pathways <- markers %>% group_by(cluster) %>% top_n(n=5,wt=avg_log2FC)
DoHeatmap(se_oj,features=pathways$gene)
```

![](https://github.com/gsa-blog/gsa-blog.github.io/raw/master/img/gsa_scdata3.png)

*Fig. Heatmap of cell-type specific pathways*









## 3. scTPA

(http://sctpa.bio-data.cn/sctpa/)

Also, the author of testSctpa provides with a web platform for analysis without coding. However, **it cannot process files more than 150 MB**.

![](https://github.com/gsa-blog/gsa-blog.github.io/raw/master/img/gsa_scdata4.png)



Here, we can try Example 1 from the website with all default setting (KEGG pathway). 

After running the job, we can download tables or plots of different information. (pathway activity score, cell-type-specific activation pathways, heatmap, PCA reduction, etc.)

![](https://github.com/gsa-blog/gsa-blog.github.io/raw/master/img/gsa_scdata5.png)

![](https://github.com/gsa-blog/gsa-blog.github.io/raw/master/img/gsa_scdata6.png)



​                     



## 4. VISION

VISION is an R tool that simply interpret scRNA-seq data with only an expression matrix and a signature library. A gene signature is a set of genes involved in some biological process. More detailed information: https://yoseflab.github.io/VISION/articles/Signatures.html .

Here we perform a simple example from https://yoseflab.github.io/VISION/articles/VISION-vignette.html .

##### 1. Installation

```R
require(devtools)
install_github("YosefLab/VISION")
```

##### 2. Create a VISION object

```R
# Load VISION
library(VISION)
# Read in expression counts (Genes X Cells)
counts <- read.table("data/expression_matrix.txt.gz",
                     header = TRUE,
                     sep = '\t',
                     row.names = 1)
# Scale counts within a sample
n.umi <- colSums(counts)
scaled_counts <- t(t(counts) / n.umi) * median(n.umi)
# Read in meta data (Cells x Vars)
meta = read.table("data/glio_meta.txt.gz", sep='\t', header=T, row.names=1)
# Create the VISION object
vis <- Vision(scaled_counts,
              signatures = c("data/h.all.v5.2.symbols.gmt"),
              meta = meta)
```

##### 3. Run the analysis

```R
# Set the number of threads when running parallel computations
# On Windows, this must either be omitted or set to 1
options(mc.cores = 2)
vis <- analyze(vis)
```

##### 4. View results

After analysis, a dynamic web report can be generated.

```R
viewResults(vis)
```

Then, a browser will be launched for the interactive report.

![](https://github.com/gsa-blog/gsa-blog.github.io/raw/master/img/gsa_scdata7.png)

*Fig.  Web report of VISION (Signature scores are computed using the expression matrix)*







## Reference

[1] Fan J. (2019) Differential Pathway Analysis. In: Yuan GC. (eds)  Computational Methods for Single-Cell Data Analysis. Methods in  Molecular Biology, vol 1935. Humana Press, New York, NY.

[2] testSctpa. https://github.com/zgyaru/testSctpa.

[3] Zhang Y, Zhang Y, Hu J, Zhang J, Guo F, Zhou M, Zhang G, Yu F, Su J. scTPA: A web tool for single-cell transcriptome analysis of pathway activation signatures. Bioinformatics. 2020.                            
[4] Zhang Y, Ma Y, Huang Y, Zhang Y, Jiang Q,  Zhou M, Su J. Benchmarking algorithms for pathway activity transformation of  single-cell RNA-seq data. Computational and Structural Biotechnology  Journal (Accepted).

[5] DeTomaso, D., Jones, M.G., Subramaniam, M. et al. Functional interpretation of single cell similarity maps. Nat Commun 10, 4376 (2019).
