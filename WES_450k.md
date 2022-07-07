# UKB WES 450k Pipeline Tutorial

This document describes how to run SAIGE on DNAnexus using UKB WES 450k data.

## Environment

DNAnexus offers web-based interface, but it is not suitable for submitting multiple job.\
I recommend using `dxpy` (https://github.com/dnanexus/dx-toolkit).\
It can also be installed using Anaconda.
```
conda install -c bioconda dxpy
```


## Data

* WES 450K genotype data: `UKB_Main:/Bulk/Exome sequences/Population level exome OQFE variants, PLINK format - interim 450k release`
* Pruned genotype data (for step 0 and step 1): `UKB_Main:/WES_450k/pruned`
* fam file of WES 450K data: `UKB_Main:/WES_450k/fam`
* group file: `UKB_Main:/WES_450k/group_files`
* sparse GRM (result of step 0): `UKB_Main:/WES_450k/sparseGRM`
* workflow files: `UKB_Main:/workflows`
* Docker images: `UKB_Main:/docker_images`

## Workflow

* **Workflow**: a set of apps or applets linked together
* **Workflow Description Language (WDL)**: a way to specify data processing workflows

We need to compile `WDL` file to create a Workflow.

### Compliling `WDL` file

Since I have already created workflows for SAIGE 1.0.9, it is **NOT necessary** to make a new workflow for running SAIGE 1.0.9 on DNAnexus.

If you want to make a workflow for another version of SAIGE (or other software), you need to create the workflow with `WDL` file.

You need `dxCompiler` (and Java) to compile `WDL` file.\
To compile `WDL` file, download `dxCompiler` on GSDS cluster.

```
wget https://github.com/dnanexus/dxCompiler/releases/download/2.10.2/dxCompiler-2.10.2.jar
```

We can check our project ID by `dx ls -l`

```
project-G53FX9QJ23G3bb2F4gJ09B6x
```

#### SAIGE step 1

```
java -jar dxCompiler-2.10.2.jar compile saige_step1_spGRMforNULLModel.wdl -project project-G53FX9QJ23G3bb2F4gJ09B6x -folder /workflows/ -f
```

#### SAIGE step 2


```
java -jar dxCompiler-2.10.2.jar compile saigegene_step2.wdl -project project-G53FX9QJ23G3bb2F4gJ09B6x -folder /workflows/ -f
```

We now have our workflows at `UKB_Main:/workflows/`, and if you want to change the SAIGE version, you can recompile with another `WDL` file (and corresponding docker image).

You can run the workflow (with inputs) using `dx run` command.


## SAIGE

### Step 0 (DON'T NEED TO RUN THIS AGAIN)

We only need to run step 0 once. (We can use the same sparse GRM for all phenotypes.)\
I ran step 0 on GSDS cluster (not on DNAnexus) using scripts in `/media/leelabsg-storage0/kisung/dnanexus/WES_450k/scripts/step0`.

After LD pruning, we obtained 215,514 variants. We use this PLINK file to construct a sparse GRM.

I have uploaded the sparse GRM file at `UKB_Main:/WES_450k/sparseGRM`


### Step 1

```
pheno="HDL LDL TG"

for p in ${pheno}
do
    instance_type="mem3_ssd1_v2_x8"
    traitType=quantitative
    invNormalize=TRUE
    phenoCol=${p}
    covariatesList=PC1,PC2,PC3,PC4,PC5,PC6,PC7,PC8,PC9,PC10,Age,Sex,WES_50k_batch
    qCovarColList=Sex
    sampleIDCol=FID
    pheno_file=UKB_WES_450k_Pheno_Unrelated_WhiteBritish.txt
    PLINK_for_vr=prune_all
    sparseGRMfile=UKB_sparseGRM_relatednessCutoff_0.05_5000_randomMarkersUsed.sparseGRM.mtx
    sparseGRM_sample_file=UKB_sparseGRM_relatednessCutoff_0.05_5000_randomMarkersUsed.sparseGRM.mtx.sampleIDs.txt
    jobname=WES_450k_${p}_step1
    outputPrefix=WES_450k_${p}_step1_UNR_WB

    workflow=workflow-GBfypP0J23GPjGfx50Y16zJy


    dx run ${workflow} \
        -istage-common.phenofile=UKB_Main:/WES_450k/pheno/${pheno_file} \
        -istage-common.bedfile=UKB_Main:/WES_450k/pruned/${PLINK_for_vr}.bed \
        -istage-common.bimfile=UKB_Main:/WES_450k/pruned/${PLINK_for_vr}.bim \
        -istage-common.famfile=UKB_Main:/WES_450k/pruned/${PLINK_for_vr}.fam \
        -istage-common.spGRMfile=UKB_Main:/WES_450k/sparseGRM/${sparseGRMfile} \
        -istage-common.spGRMSamplefile=UKB_Main:/WES_450k/sparseGRM/${sparseGRM_sample_file} \
        -istage-common.output_prefix=${outputPrefix} \
        -istage-common.phenoCol=${phenoCol} \
        -istage-common.traitType=${traitType} \
        -istage-common.covariatesList=${covariatesList} \
        -istage-common.qCovarColList=${qCovarColList} \
        -istage-common.sampleIDCol=${sampleIDCol} \
        -istage-common.invNormalize=${invNormalize} \
        --folder UKB_Main:/WES_450k/output/ \
        --yes \
        --name=${jobname} \
        --instance-type=${instance_type}
done
```


### Step 2

```
pheno="HDL LDL TG"

for p in ${pheno}
do
	for i in {1..22}
	do
	workflow=workflow-GBg7QQ0J23GPjGfx50Y17kqj

	dx run ${workflow} \
            -istage-common.GMMATmodelFile=UKB_Main:/WES_450k/output/WES_450k_${p}_step1_UNR_WB.rda \
            -istage-common.varianceRatioFile=UKB_Main:/WES_450k/output/WES_450k_${p}_step1_UNR_WB.varianceRatio.txt \
            -istage-common.spGRMfile=UKB_Main:/WES_450k/sparseGRM/UKB_sparseGRM_relatednessCutoff_0.05_5000_randomMarkersUsed.sparseGRM.mtx \
            -istage-common.spGRMSamplefile=UKB_Main:/WES_450k/sparseGRM/UKB_sparseGRM_relatednessCutoff_0.05_5000_randomMarkersUsed.sparseGRM.mtx.sampleIDs.txt \
            -istage-common.bed=UKB_Main:/Bulk/Exome\ sequences/Population\ level\ exome\ OQFE\ variants,\ PLINK\ format\ -\ interim\ 450k\ release/ukb23149_c${i}_b0_v1.bed \
            -istage-common.bim=UKB_Main:/Bulk/Exome\ sequences/Population\ level\ exome\ OQFE\ variants,\ PLINK\ format\ -\ interim\ 450k\ release/ukb23149_c${i}_b0_v1.bim \
            -istage-common.fam=UKB_Main:/Bulk/Exome\ sequences/Population\ level\ exome\ OQFE\ variants,\ PLINK\ format\ -\ interim\ 450k\ release/ukb23149_c${i}_b0_v1.fam \
            -istage-common.grpfile=UKB_Main:/WES_450k/group_files/UKBexomeOQFE_chr${i}.gene.anno.hg38_multianno.lMS.group.chrpos.SAIGEGENEplus.txt \
            -istage-common.chrom=${i} \
            -istage-common.annotation_in_groupTest=lof,missense:lof,missense:lof:synonymous \
            -istage-common.maxMAF_in_groupTest=0.01,0.001,0.0001 \
            -istage-common.output_prefix=WES_450k_${p}_chr${i}_UNR_WB.txt \
            -istage-common.isFast=TRUE \
            --folder=UKB_Main:/WES_450k/output/step2/ \
            --name=Step2_${p}_chr${i}_UNR_WB \
            --yes \
            --instance-type=mem3_ssd1_v2_x16
	done
done
```

