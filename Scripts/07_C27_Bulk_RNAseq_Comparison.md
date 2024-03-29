Identification of differentially active transcriptional regulators
between adult and fetal hGPCs
================
John Mariani
03/06/23

``` r
library(ggplot2)
library(tximport)
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(plyr)
```

    ## ------------------------------------------------------------------------------

    ## You have loaded plyr after dplyr - this is likely to cause problems.
    ## If you need functions from both plyr and dplyr, please load plyr first, then dplyr:
    ## library(plyr); library(dplyr)

    ## ------------------------------------------------------------------------------

    ## 
    ## Attaching package: 'plyr'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     arrange, count, desc, failwith, id, mutate, rename, summarise,
    ##     summarize

``` r
library(tidyr)
```

\#Load prior data

``` r
#txi.rsem <- readRDS("RDS/txi.rsem.rds")
#highTPM <- readRDS("RDS/highTPM.rds")
#sampleTableFull <- read.csv("data_for_import/sampleTableFull.csv")
ensemblGeneListH <- read.csv("data_for_import/ensemblGeneList.csv")
de_intersect <- read.delim("output/de_Adult_vs_Fetal_Intersect.txt")
```

### Comparison to iPSC line C27

``` r
#Read in RSEM gene output
sampleTableC27 <- read.csv("data_for_import/sampleTableC27.csv")
nrow(sampleTableC27)
```

    ## [1] 24

``` r
temp = list.files(path = "./data_for_import/genes", pattern="genes.results")
length(temp)
```

    ## [1] 29

``` r
names(temp) <- substr(temp, 1, nchar(temp)-19)
temp <- temp[names(temp) %in% sampleTableC27$sample]



txi.rsem.c27 <- tximport(paste0("./data_for_import/genes/",temp), type = "rsem")
```

    ## It looks like you are importing RSEM genes.results files, setting txIn=FALSE

    ## reading in files with read_tsv

    ## 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24

``` r
for(i in 1:3){
  colnames(txi.rsem.c27[[i]]) <- names(temp)
}


sampleTableC27 <- sampleTableC27[match(names(temp), sampleTableC27$sample),]
```

### C27 Heatmap

``` r
markerHMgenes <- c("MKI67", "CDK1", "TOP2A", "PDGFRA", "PTPRZ1", "LMNB1", "MYC", "TEAD2", "NFIB", "HMGA2", "HDAC2", "EZH2", "BCL11A", "MBP", "MAG", "MOG", "TF", "AHR", "IL1A","STAT3", "E2F6", "MAX", "ZNF274", "IKZF3")



markerTPM <- merge(txi.rsem.c27$abundance, ensemblGeneListH, by.x = 0, by.y = "ensembl_gene_id")
markerTPM <- markerTPM[markerTPM$external_gene_name %in% markerHMgenes,]


markerTPM <- markerTPM[,1:26] %>%
  pivot_longer(-c(Row.names, external_gene_name), names_to = "Sample", values_to = "TPM")

markerTPM$group <- mapvalues(markerTPM$Sample,sampleTableC27$sample, as.character(sampleTableC27$label))

markerTPM$TPM <- log2(markerTPM$TPM + .1)

markerTPM$group <- factor(markerTPM$group, levels =c("iPSC CD140a", "Fetal CD140a", "Fetal A2B5", "Adult A2B5"))
markerTPM$external_gene_name <- factor(markerTPM$external_gene_name, levels = rev(c(markerHMgenes)))


###

c27HMgg <- ggplot(markerTPM, aes(Sample, external_gene_name)) + geom_tile(aes(fill = TPM)) + 
  theme(panel.spacing.y = unit(.5, "lines"), 
        panel.spacing.x = unit(.25,"lines"), 
        axis.title.x = element_blank(), 
        axis.title.y = element_blank(), 
        axis.text.x = element_blank(), 
        axis.ticks.y = element_blank(), 
        axis.text.y = element_text(colour = ifelse(levels(markerTPM$external_gene_name) %in% de_intersect[de_intersect$log2FoldChange > 0,]$external_gene_name, "#FF2020", "#2E30FF")))  + 
  scale_fill_gradientn(colours = c("#009900","#fffcbd","#ff2020"), guide = guide_colourbar(direction = "horizontal", title = "Log2(TPM + .1)", title.position = "top")) + scale_x_discrete(expand = c(0, 0))  + theme(axis.ticks.x = element_blank(), legend.position = "bottom", legend.direction = "horizontal") + facet_grid(cols = vars(group), scales = "free", space  = "free")


c27HMgg
```

![](07_C27_Bulk_RNAseq_Comparison_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
#ggsave("c27HMgg.pdf", units = "in", dpi = 300, width = 16, height = 8, device = NULL)
```

``` r
sessionInfo()
```

    ## R version 4.2.3 (2023-03-15)
    ## Platform: aarch64-apple-darwin20 (64-bit)
    ## Running under: macOS Ventura 13.2.1
    ## 
    ## Matrix products: default
    ## BLAS:   /Library/Frameworks/R.framework/Versions/4.2-arm64/Resources/lib/libRblas.0.dylib
    ## LAPACK: /Library/Frameworks/R.framework/Versions/4.2-arm64/Resources/lib/libRlapack.dylib
    ## 
    ## locale:
    ## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] tidyr_1.3.0     plyr_1.8.8      dplyr_1.1.1     tximport_1.26.1
    ## [5] ggplot2_3.4.2  
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_1.0.10      highr_0.10       pillar_1.9.0     compiler_4.2.3  
    ##  [5] tools_4.2.3      bit_4.0.5        digest_0.6.31    evaluate_0.20   
    ##  [9] lifecycle_1.0.3  tibble_3.2.1     gtable_0.3.3     pkgconfig_2.0.3 
    ## [13] rlang_1.1.0      cli_3.6.1        rstudioapi_0.14  parallel_4.2.3  
    ## [17] yaml_2.3.7       xfun_0.38        fastmap_1.1.1    withr_2.5.0     
    ## [21] knitr_1.42       hms_1.1.3        generics_0.1.3   vctrs_0.6.1     
    ## [25] bit64_4.0.5      rprojroot_2.0.3  grid_4.2.3       tidyselect_1.2.0
    ## [29] glue_1.6.2       R6_2.5.1         fansi_1.0.4      vroom_1.6.1     
    ## [33] rmarkdown_2.21   farver_2.1.1     tzdb_0.3.0       purrr_1.0.1     
    ## [37] readr_2.1.4      magrittr_2.0.3   scales_1.2.1     htmltools_0.5.5 
    ## [41] colorspace_2.1-0 labeling_0.4.2   utf8_1.2.3       munsell_0.5.0   
    ## [45] crayon_1.5.2
