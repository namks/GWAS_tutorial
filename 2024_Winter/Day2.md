# Biobank Data Analysis Tutorial Session - Week 2

## Review of last week's homework

### Manhattan plot

**Manhattan plot** is a type of scatter plot, usually used to display data with a large number of data-points, many of non-zero amplitude, and with a distribution of higher-magnitude values. *(Wikipedia)*

Have you successfully replicated the manhattan plot?


### QQ Plot

**QQ plot** is a probability plot, which is a graphical method for comparing two probability distributions by plotting their quantiles against each other. *(Wikipedia)*

*Q1*: When we draw a QQ plot, which probability distribution are we comparing to?

*Q2*: Why do we draw a QQ plot?

### Linux command for parallel processing

What command did you use when trying genome-wide association test? (using all 22 chromosomes)
Parallel processing is a big help for the analysis. In other words, it would be better
to run the tasks in the background (using `&`).

*Q3*: Which of the following commands will you use for parallel processing?

```
# Choice 1
nohup for i in $(seq 1 22)
do
    plink command 
done &
```
```
# Choice 2
for i in $(seq 1 22)
do
    nohup plink command &
done
```

You can find the answer through the following toy example:
```
# Toy version of choice 1
for i in $(seq 1 5)
do        
    echo ${i} 
    sleep 3   
done &    
```
```
# Toy version of choice 2
for i in $(seq 1 5)
do        
    echo ${i} 
    sleep 3 &
done
```

### Linux command to cancel jobs

Sometimes, there may be situations when you have to cancel a lot of jobs.

You can check the status of the node by `htop` or `ps aux` command.
But, it returns all processes of the node, so you need to filter with a certain condition.
For example, suppose we want to kill all processes containing the string `plink`.

The following command returns processes containing the string `plink`:
```
ps aux | grep plink
```

Among them, we need to extract process IDs (which are in the second column) and save into a text file with the following command:
```
ps aux | grep plink | awk '{print $2}' > pid.txt
```

With this file (with PIDs), you can kill them all by:
```
while read p
do
    kill -9 ${p}
done < pid.txt
```

The `-9` flag means `SIGKILL` (to force quit).


### Linux command for file transfer

We can transfer (copy) files using `scp` command.

Example:
```
scp -rP 22555 'leelabguest@147.47.200.131:PATH_OF_SOURCE' PATH_OF_DESTINATION
```


## GWAS for the phenotype of your choice

We will use **SAIGE** to run GWAS for the phenotype of your choice.
* SAIGE Paper (*Nature Genetics*, 2018): https://www.nature.com/articles/s41588-018-0184-y
* SAIGE GitHub: https://github.com/saigegit/SAIGE
* SAIGE Documentation: https://saigegit.github.io//SAIGE-doc/

### GWAS using PLINK

* Linear or logistic regression-based GWAS
* Wald test to compute test statistics

### What is SAIGE?

* Mixed effect model based method
* Score test to compute test statistics
* Several techniques for fast computation

### Why SAIGE?

* Can account for related individuals
* Fast computation
* Prevent type I error inflation in case of unbalanced case-control ratio


### Docker

* We will run SAIGE using Docker (https://www.docker.com/)

#### Container and Image

* A **container** is an isolated place where an application runs without affecting the rest of the system and without the system impacting the application.
* **Images** are read-only templates containing instructions for creating a container. A Docker image creates containers to run on the Docker platform.

#### Docker commands

```
docker image ls
docker container ls
docker logs [CONTAINER_ID]
docker kill [CONTAINER_ID]
```

### SAIGE Workflow

#### Step 1 (Fitting the null model)

* Binary phenotype

```
docker run -dv /media/leelabsg-storage0:/media/leelabsg-storage0 wzhou88/saige:1.1.3 step1_fitNULLGLMM.R \
    --plinkFile=/media/leelabsg-storage0/GWAS_tutorial/data/UKB_step1 \
    --phenoFile=/media/leelabsg-storage0/GWAS_tutorial/YOUR_OWN_FILE \
    --phenoCol={NAME_OF_PHENOTYPE_COLUMN} \
    --covarColList={NAMES_OF_COVARIATE_COLUMNS} \
    --sampleIDColinphenoFile=IID \
    --traitType=binary \
    --outputPrefix=/media/leelabsg-storage0/GWAS_tutorial/{OUTPUT_PATH_step1}/{OUTPUT_PREFIX_step1} \
    --nThreads=4 \
    --LOCO=FALSE \
    --IsOverwriteVarianceRatioFile=TRUE
```

* Quantitative phenotype

```
docker run -dv /media/leelabsg-storage0:/media/leelabsg-storage0 wzhou88/saige:1.1.3 step1_fitNULLGLMM.R \
    --plinkFile=/media/leelabsg-storage0/GWAS_tutorial/data/UKB_step1 \
    --phenoFile=/media/leelabsg-storage0/GWAS_tutorial/{YOUR_OWN_FILE} \
    --phenoCol={NAME_OF_PHENOTYPE_COLUMN} \
    --covarColList={NAMES_OF_COVARIATE_COLUMNS} \
    --sampleIDColinphenoFile=IID \
    --traitType=quantitative \
    --invNormalize=TRUE \
    --outputPrefix=/media/leelabsg-storage0/GWAS_tutorial/{OUTPUT_PATH_step1}/{OUTPUT_PREFIX_step1} \
    --nThreads=4 \
    --LOCO=FALSE \
    --IsOverwriteVarianceRatioFile=TRUE
```


#### Step 2 (Single-variant association test)

```
docker run -dv /media/leelabsg-storage0:/media/leelabsg-storage0 wzhou88/saige:1.1.3 step2_SPAtests.R \
    --bedFile=/media/leelabsg-storage0/DATA/UKBB/cal/ukb_cal_chr1_v2.bed \
    --bimFile=/media/leelabsg-storage0/DATA/UKBB/cal/ukb_snp_chr1_v2.bim \
    --famFile=/media/leelabsg-storage0/DATA/UKBB/cal/ukb45227_cal_chr20_v2_s488264.fam \
    --AlleleOrder=alt-first \
    --SAIGEOutputFile=/media/leelabsg-storage0/GWAS_tutorial/{OUTPUT_PATH_step2}/{OUTPUT_PREFIX_step2} \
    --chrom=1 \
    --minMAF=0 \
    --minMAC=20 \
    --LOCO=FALSE \
    --GMMATmodelFile=/media/leelabsg-storage0/GWAS_tutorial/{OUTPUT_PATH_step1}/{OUTPUT_PREFIX_step1}.rda \
    --varianceRatioFile=//media/leelabsg-storage0/GWAS_tutorial/{OUTPUT_PATH_step1}/{OUTPUT_PREFIX_step1}.varianceRatio.txt \
    --is_output_moreDetails=TRUE
```

You can check your results by comparing them with those of SAIGE Pheweb.
(https://pheweb.org/UKB-SAIGE/)
