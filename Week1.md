# GWAS Tutorial Session - Week 1

## Environment

* First, connect to the GSDS cluster using lab account (You can find the credential information in Google Drive)
* Connect to the CPU node using the command `ssh cpu01`
* Activate the conda environment with R using the command `conda activate r3.6` or `r3` (shortcut)
* Make your own directory for the practice in `/media/leelabsg_storage01/GWAS_tutorial`

## GWAS for standing height

In this practice session, we will run GWAS on chromosome 2 only.

For an introduction to GWAS, please refer to the presentation material (210708).

#### Step 1. Build a phenotype (.fam) file

You can find the phenotype file in `/media/leelabsg_storage01/DATA/UKBB/Pheno/`

Run the following command in the above path:

```head -1 ukb42597.tab | tr '\t' '\n' | nl | head -30```

Then, you will see the result as follows.

```
     1  f.eid
     2  f.46.0.0
     3  f.46.1.0
     4  f.46.2.0
     5  f.46.3.0
     6  f.47.0.0
     7  f.47.1.0
     8  f.47.2.0
     9  f.47.3.0
    10  f.48.0.0
    11  f.48.1.0
    12  f.48.2.0
    13  f.48.3.0
    14  f.49.0.0
    15  f.49.1.0
    16  f.49.2.0
    17  f.49.3.0
    18  f.50.0.0
    19  f.50.1.0
    20  f.50.2.0
```

We need only column 1 and column 18, so we have to extract these two columns
using the following command.

```cat ukb42597.tab | awk '{print $1, $18}' > /media/leelabsg_storage01/GWAS_tutorial/test.txt```

Next, you will have to merge the height data to `.fam` file.

In this tutorial, we will use `dplyr` package in R.
But, you can also use Python, Excel, etc.

Our (called) genotype dataset is in `/media/leelabsg_storage01/DATA/UKBB/cal`

PLINK binary genotype data consists of 3 files.

* `.bed` file: Genotype data in binary format
* `.bim` file: Variants information
* `.fam` file: Sample information

`.fam` file looks like this:

```
3267751 3267751 0 0 1 Batch_b001
4085874 4085874 0 0 2 Batch_b001
1488844 1488844 0 0 2 Batch_b001
2299675 2299675 0 0 2 Batch_b001
4189123 4189123 0 0 2 Batch_b001
3600970 3600970 0 0 1 Batch_b001
1111417 1111417 0 0 2 Batch_b001
1414057 1414057 0 0 2 Batch_b001
2503961 2503961 0 0 1 Batch_b001
1802736 1802736 0 0 2 Batch_b001
```

The last column is for phenotype we want to analyze.
So, replace the last column with the height.

```
# R code for building a phenotype (.fam) file

library(data.table)
library(dplyr)

fam = fread("/media/leelabsg_storage01/DATA/UKBB/cal/ukb45227_cal_chr20_v2_s488264.fam", header=F)
height = fread("/media/leelabsg_storage01/GWAS_tutorial/test.txt", header=T)
join = left_join(fam, height, by=c("V2"="f.eid"))
out = join[,-c(6)]

out_path = "/media/leelabsg_storage01/GWAS_tutorial/height.fam"
write.table(out, out_path, row.names=F, col.names=F, quote=F)
```

Note: `fread` function in `data.table` package in R is a very useful function for
reading large data files.

As a result, you will see the output:

```
3267751 3267751 0 0 1 177
4085874 4085874 0 0 2 166
1488844 1488844 0 0 2 158
2299675 2299675 0 0 2 158
4189123 4189123 0 0 2 165.5
3600970 3600970 0 0 1 186
1111417 1111417 0 0 2 162.5
1414057 1414057 0 0 2 166
2503961 2503961 0 0 1 167
1802736 1802736 0 0 2 156.5
```

#### Step 2. Build a covariate file

We have pre-calculated principal components (PC) data in

```/media/leelabsg_storage01/DATA/UKBB/PC/PEDMASTER_UNRELATED_WhiteBritish_20180612_v2.txt```

**But, be aware that IID in PC file is different from our data**.
We have to convert the IID in PC file using `/media/leelabsg_storage01/DATA/UKBB/Mapping/mapping.csv`

**Exercise**: convert IIDs in our PC file using `mapping.csv`, and save this PC file in your practice directory.

The order of IIDs in the output file will be different from our genotype file.
So, we have to sort IIDs in the output file using the following command:

```
~/plink \
--bed /media/leelabsg_storage01/DATA/UKBB/cal/ukb_cal_chr2_v2.bed \
--bim /media/leelabsg_storage01/DATA/UKBB/cal/ukb_snp_chr2_v2.bim \
--fam ukb45227_cal_chr20_v2_s488264.fam \
--covar ourdata.txt \
--write-covar \
--out covar.cov
```

Then, you will get the covariate file with the same order as our genotype data (`.fam`).

#### Step 3. Run GWAS

Everything is ready. But we are solving a huge problem, it will take quite long time.

Before running GWAS, let's check if you understand our problem well.

*Q1*: How many linear regression equations need to be solved?

*Q2*: If we write our linear regression model as $y= X\beta + \epsilon$, what is the dimension of $X$?

Let's run the analysis using the following command:

```
~/plink \
--bed /media/leelabsg_storage01/DATA/UKBB/cal/ukb_cal_chr2_v2.bed \
--bim /media/leelabsg_storage01/DATA/UKBB/cal/ukb_snp_chr2_v2.bim \
--fam /media/leelabsg_storage01/GWAS_tutorial/height.fam \
--linear
--covar covar.cov
--out chr2
```

Then, after a while, you will obtain three result files:

* `.assoc.linear`: main result file
* `.log`: log for our analysis
* `.nosex`: IIDs of ambiguous sex

**Note**: If the connection to the server is lost, the work is stopped and you have to start over from the beginning. Therefore, `nohup` command will be of great help for your analysis.

## Homework

1. Draw a manhattan and QQ plot for your result (chromosome 2) using `qqman` package in R, and compare your results with the figure on page 38 in 210708 material.
2. If the above experiment was reproduced normally, try *Genome-wide association test* (using all 22 chromosomes) using `plink`. When you obtain the results, draw a manhattan and QQ plot.
3. Think of a trait (phenotype) you are interested in. We will try GWAS with that trait next week. You can find the list of phenotypes on the UK Biobank showcase website.

