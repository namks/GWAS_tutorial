# Biobank Data Analysis Tutorial Session - Week 2

## Review of last week's homework

### Manhattan plot

**Manhattan plot** is a type of scatter plot, usually used to display data with a large number of data-points, many of non-zero amplitude, and with a distribution of higher-magnitude values. *(Wikipedia)*

Have you successfully replicated the manhattan plot below?

![GWAS Height](https://i.imgur.com/BB6rHea.png)
*(Source: https://pheweb.org/UKB-Neale/pheno/50)*


### QQ Plot

**QQ plot** is a probability plot, which is a graphical method for comparing two probability distributions by plotting their quantiles against each other. *(Wikipedia)*

*Q1*: When we draw a QQ plot, which probability distribution are we comparing to?

*Q2*: Why do we draw a QQ plot?

## Some tips for your analysis

### Drawing plots in command line interface (CLI)

Drawing plots with PLINK output files (`.assoc.linear`) would not have been easy for beginners.
One reason is that there is no way to visualize the result on the server with command line interface (CLI), so we have to copy the result files to the local machine. But the size of the GWAS result file is large, therefore reading and processing  this on a typical laptop can be time consuming.

The R code below is the code that saves our results to the `example.jpeg` file:

```
library(qqman)

jpeg(file="example.jpeg", height=300, width=1000)
manhattan(df, chr='CHR', bp='BP', snp='SNP', p='P', main='GWAS Result')
dev.off()
```

Then, all we need to do is copy this plot (`.jpeg`) to our local machine!

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


## GWAS for the phenotype your choice

If you chose a continuous phenotype, then you can repeat the same procedure
as last week. In this section, we will cover the GWAS of binary phenotypes.

### Step 1. Build a phenotype (.fam) file

You can find the phenotype information in ```/media/leelabsg_storage01/DATA/UKBB/PheCode/UKB_Phenome```

Since the phenotype file is compressed, you can see the contents of the file by:

```zcat PEDMASTER_ALL_20180514.txt.gz | head```

Note that IID in this file is different from our genotype data. We have to convert the IID in PC file using
```/media/leelabsg_storage01/DATA/UKBB/Mapping/mapping.csv```

**Exercise**: Make a phenotype (.fam) file using the phenotype of your choice.

### Step 2. Build a covariate file

You don't need to re-build the covariate file. (You can reuse the covariate file you created last week.)

### Step 3. Run GWAS

Let's run the analysis using the following command:

```
~/plink \
--bed /media/leelabsg_storage01/DATA/UKBB/cal/ukb_cal_chr2_v2.bed \
--bim /media/leelabsg_storage01/DATA/UKBB/cal/ukb_snp_chr2_v2.bim \
--fam /media/leelabsg_storage01/GWAS_tutorial/T2D.fam \
--logistic \
--1 \
--covar covar.cov \
--covar-name Sex,PC1-PC10,Age \
--out chr2
```

In PLINK, affection status, by default, should be coded:

```
-9    missing
 0    missing
 1    unaffected (control)
 2    affected (case)
```

But our data is coded:
```
-9    missing
 0    unaffected (control)
 1    affected (case)
```

Therefore, **don't forget to add `--1` flag** in your command.

You can check your results by comparing them with those of SAIGE Pheweb.
(https://pheweb.org/UKB-SAIGE/)
