## Single Cell Atlas - PDAC 
### Integrated Analysis of the Tumor Microenvironment (TME)
***
### Overview of Repository
#### 1). **PDAC Atlas Data**   
   * #####  CellType1 Subset Seurat Objects (rds)  
    *   CellType2 Pan-TME Secretome Expression (csv)   
    *   CellType2 Pan-TME Cell-Contact Expression (csv)      
    *   CellType2 Differential Gene Markers (csv)  
    *   CellType2 Gene Set Enrichment Output (csv)     
     
#### 2). **Subtype Dependent Secretome Expression Data**
   * #####  Stromal Subtype DEG (csv)    
   * #####  Tumor Subtype DEG (csv)  

#### 3). **Single Cell Classifier**
   * #####  Trained Classifier (rds)    
   * #####  Tutorial 
***   

#### **Required Installation**
```r
install.packages("singlecellnet")
```

#### **Quick Start** 
```r
library(singleCellNet)
library(dplyr)
## change Seurat into dgCMatrix.

# Load Classifier
Classifier <- readRDS("~/data/PDAC_Tier1_Classifier_MouseOverlap5000.40pairs.rds")

# set Query data
## this is how to grab Large dgCMatrix from Seurat Object
stMouse <- pseudo_GEMMs@meta.data
stMouse$sample_name <- rownames(stMouse)
expMouse <- pseudo_GEMMs@assays$RNA[]
```



```r
## add random 
# More detail description is on SingelCellNet github: https://github.com/pcahan1/singleCellNet  
nqRand = 0
system.time( crParkall <- scn_predict(Classifier[['cnProc']], expMouse, nrand = nqRand))
```


#### **Visualize Classifier Results**
```r
sgrp = as.vector(stMouse$CellType)
names(sgrp) = as.vector(stMouse$sample_name)
grpRand =rep("rand", nqRand)
names(grpRand) = paste("rand_", 1:nqRand, sep='')
sgrp = append(sgrp, grpRand)

# heatmap classification result
sc_hmClass(crParkall, sgrp, max=5000, isBig=TRUE, cCol=F, font=8)
```

#### **Join the result back to Original Seurat object**
```r
result = crParkall[,!grepl("^rand_",colnames(crParkall))]
## drop rand row : Classifier has six cell types and random will be at 6th row, but should double check before drop it. 
result <- result[-6, ]

## Transpose the matrix and make it into dataframe
result_classi <- t(result)
result_classi <- as.data.frame(result_classi)

## Pick maximum result as the result of the Classifier. 
result_classi$CellType_cs <- colnames(result_classi)[max.col(result_classi,ties.method="first")]

result_classi$sample_name <- row.names(result_classi)

## make new meta data to left join new classified cell type
mouse_cell <- pancreas_merged@meta.data
mouse_cell$sample_name <- rownames(mouse_cell)

# meta_mouse <- GEMMs@meta.data
# meta_mouse$sample_name <- rownames(meta_mouse)

expMouse_2 <- merge(x = mouse_cell, y = result_classi, by = "sample_name", all.x = TRUE)

rownames(expMouse_2) <- expMouse_2$sample_name
# drop Sample_name
expMouse_2 <- expMouse_2[,-1]
```

#### **Add result back to the original mouse cell with mouse genes** 
```r
pancreas_merged@meta.data <- expMouse_2

```

