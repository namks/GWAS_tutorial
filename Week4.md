# Biobank Data Analysis Tutorial Session - Week 4

## SAIGE

### GWAS using PLINK

* Linear or logistic regression-based GWAS
* Wald test to compute test statistics

### What is SAIGE?

* Mixed effect model based method
* Score test to compute test statistics
(Please refer to 200904 lab meeting material for "Wald test vs. Score test")
* Several techniques for fast computation

### Why SAIGE?

* Can account for related individuals
* Fast computation
* Prevent type I error inflation in case of unbalanced case-control ratio

### Outline

* Step 1: Fitting the null linear/logistic mixed model
* Step 2: Performing the single-variant association tests


## Running SAIGE

### Anaconda

We installed SAIGE on the GSDS cluster (lab account).
You can use the virtual environment with the following command:

```
# SAIGE 0.44.5
saige
conda activate saige

# SAIGE 0.43.3
rr
conda activate RSAIGE8
```

#### Checking Version

In R, you can check the version of SAIGE by the following command:
```
library(SAIGE)
sessionInfo()
```

```
# Output
R version 3.6.1 (2019-07-05)
Platform: x86_64-conda_cos6-linux-gnu (64-bit)
Running under: Ubuntu 18.04.5 LTS

Matrix products: default
BLAS/LAPACK: /home/leelabsg/anaconda3/envs/RSAIGE8/lib/R/lib/libRblas.so

locale:
 [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C
 [3] LC_TIME=en_US.UTF-8        LC_COLLATE=en_US.UTF-8
 [5] LC_MONETARY=en_US.UTF-8    LC_MESSAGES=en_US.UTF-8
 [7] LC_PAPER=en_US.UTF-8       LC_NAME=C
 [9] LC_ADDRESS=C               LC_TELEPHONE=C
[11] LC_MEASUREMENT=en_US.UTF-8 LC_IDENTIFICATION=C

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base

other attached packages:
[1] SAIGE_0.43.3
```

#### Step 1 (SAIGE 0.44.5 or `saige` environment)

```
nohup step1_fitNULLGLMM.R \
--plinkFile=/media/leelabsg_storage01/KOGO_workshop/SAIGE/saige_example \
--phenoFile=/media/leelabsg_storage01/KOGO_workshop/SAIGE/saige_pheno.txt \
--phenoCol=y_binary \
--covarColList=x1,x2 \
--sampleIDColinphenoFile=IID \
--traitType=binary \
--outputPrefix=./step1_result --nThreads=4 \
--LOCO=FALSE \
--IsOverwriteVarianceRatioFile=TRUE &
```
* Why do we need the genotype data in Step 1 (fitting the **null** model)?

#### Step 2 (SAIGE 0.43.3 or `RSAIGE8` environment)

```
nohup Rscript /home/leelabsg/SAIGE/extdata/step2_SPAtests.R \
--vcfFile=/media/leelabsg_storage01/KOGO_workshop/SAIGE/saige_example.vcf.gz \
--vcfFileIndex=/media/leelabsg_storage01/KOGO_workshop/SAIGE/saige_example.vcf.gz.tbi \
--vcfField=GT \
--chrom=1 \
--minMAF=0.0001 \
--minMAC=1 \
--sampleFile=/media/leelabsg_storage01/KOGO_workshop/SAIGE/sampleIDindosage.txt \
--GMMATmodelFile=step1_result.rda \
--varianceRatioFile=step1_result.varianceRatio.txt \
--SAIGEOutputFile=finalresult.txt \
--numLinesOutput=2 \
--IsOutputAFinCaseCtrl=TRUE \
--LOCO=FALSE &
```

#### Drawing plots (Manhattan and QQ) (`r3.6` environment)

```
library(qqman)

gwas<-read.table('finalresult.txt',h=T)

colnames(gwas)[c(2,3,14)]<-c('BP','SNP','P')
png(file='manhattan_plot.png',width=1000, height=1000)
manhattan(gwas, main='Chromosome 1 Manhattan Plot')
dev.off()

png(file='qq_plot.png',width=1000, height=1000)
qq(gwas$P, main='Q-Q plot of chromosome 1')
dev.off()
```

### Docker

You can also use Docker in `cpu02` node. We installed 4 different version of SAIGE:

* `0.43.2`
* `0.44.2`
* `0.44.3_scorercpp`
* `0.44.5`

You can make a Docker container with the following command:

```
docker run --rm -it -v /media/leelabsg_storage01:/docker --name SAIGE wzhou88/saige:0.44.5 /bin/bash
```

This command means:

* `/media/leelabsg_storage01` in local (GSDS cluster) is linked to `/docker` in the Docker container
* The name of container is `SAIGE`
* The image we are going to run is `wzhou88/saige:0.44.5`

Docker images work similarly to virtual machines.
You can use the command as you use on a local machine. (But, pay attention to the `path`.)
