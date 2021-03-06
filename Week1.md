# Biobank Data Analysis Tutorial Session - Week 1

## Environment

* First, connect to the GSDS cluster using lab account (You can find the credential information in Google Drive)
* Connect to the CPU node (ex. `ssh leelabsg01`)
* Make your own directory for the practice in `/media/leelabsg-storage0/GWAS_tutorial`

## GWAS for standing height

In this practice session, we will run GWAS on chromosome 2 only. \
For an introduction to GWAS, please refer to the presentation material.

### Step 1. Build a phenotype (.fam) file

You can find the phenotype file in `/media/leelabsg-storage0/DATA/UKBB/Pheno/`

Run the following command in the above path:

```head -1 ukb42597.tab | tr '\t' '\n' | nl | head -20```

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

```cat ukb42597.tab | awk '{print $1, $18}' > /media/leelabsg-storage0/GWAS_tutorial/test.txt```

Next, you will have to merge the height data to `.fam` file.

In this tutorial, we will use `dplyr` package in R.
But, you can also use Python, Excel, etc.

Our (called) genotype dataset is in `/media/leelabsg-storage0/DATA/UKBB/cal`

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

fam = fread("/media/leelabsg-storage0/DATA/UKBB/cal/ukb45227_cal_chr20_v2_s488264.fam", header=F)
height = fread("/media/leelabsg-storage0/GWAS_tutorial/test.txt", header=T)
join = left_join(fam, height, by=c("V2"="f.eid"))
out = join[,-c(6)]

out_path = "/media/leelabsg-storage0/GWAS_tutorial/height.fam"
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

### Step 2. Build a covariate file

We have pre-calculated principal components (PC) data in

```/media/leelabsg-storage0/DATA/UKBB/PC/PEDMASTER_UNRELATED_WhiteBritish_20180612_v2.txt```

**But, be aware that IID in PC file is different from our data**.
We have to convert the IID in PC file using `/media/leelabsg-storage0/DATA/UKBB/Mapping/mapping.csv`

**Exercise**: convert IIDs in our PC file using `mapping.csv`, and save this PC file in your practice directory.
Your output should look like this:

```
FID IID Sex birthYear PC1 PC2 PC3 PC4 PC5 PC6 PC7 PC8 PC9 PC10 X153 X193 X365 X411 Age
1265060 1265060 1 1956 -16.8861 3.38776 -0.496867 0.617104 -3.90062 -2.19504 1.94369 -2.57336 2.39074 -5.53032 0 0 0 0 62
4279687 4279687 1 1938 -14.1724 4.57744 -4.61973 2.05025 2.0004 -2.85618 2.4178 -2.58268 2.78382 1.20632 0 0 0 0 80
4608614 4608614 1 1944 -10.5828 1.87179 -2.91263 -1.04566 -5.07404 0.0109573 -0.205352 -3.64898 -0.346414 -1.05994 0 0 0 1 74
2087882 2087882 1 1949 -12.5769 4.26968 -3.1915 2.89076 -7.3564 2.36535 -1.49095 2.14493 3.6423 3.03614 0 0 0 0 69
1526004 1526004 1 1941 -10.8651 2.52371 -2.79921 -2.98971 -5.64579 2.53417 0.56558 0.422705 -0.424998 -5.14622 0 0 0 1 77
2530063 2530063 1 1943 -15.3836 4.45268 -2.205 1.19426 -6.6835 0.30293 0.225121 -0.69315 3.78101 1.42883 0 0 0 0 75
1347805 1347805 2 1960 -12.9173 3.34111 -1.74563 -0.10012 -8.09168 1.13279 -2.03264 -1.08073 1.35118 3.82198 0 0 0 0 58
3278616 3278616 2 1956 -11.9514 4.58447 0.618089 4.84009 8.15947 0.594526 -1.3289 0.75907 -3.17168 2.44387 0 0 0 0 62
2438170 2438170 2 1942 -13.2883 3.21761 -1.30171 1.36956 4.88423 0.542946 0.730243 -2.53709 -14.293 0.268081 0 0 0 0 76
```

The order of IIDs in the output file will be different from our genotype file.
So, we have to sort IIDs in the output file using the following command:

```
plink \
--bed /media/leelabsg-storage0/DATA/UKBB/cal/ukb_cal_chr2_v2.bed \
--bim /media/leelabsg-storage0/DATA/UKBB/cal/ukb_snp_chr2_v2.bim \
--fam /media/leelabsg-storage0/DATA/UKBB/cal/ukb45227_cal_chr20_v2_s488264.fam \
--covar /media/leelabsg-storage0/DATA/UKBB/PC/PEDMASTER_UNRELATED_WhiteBritish_20180612_v2_MAPPED.txt \
--write-covar \
--out covar
```

Then, you will get the covariate file with the same order as our genotype data (`.fam`).

### Step 3. Run GWAS

Everything is ready. But we are solving a huge problem, it will take quite long time. \
Before running GWAS, let's check if you understand our problem well.

*Q1*: How many linear regression equations need to be solved?

*Q2*: If we write our linear regression model as **y = Xb + e**, what is the dimension of **X**?

Let's run the analysis using the following command:

```
plink \
--bed /media/leelabsg-storage0/DATA/UKBB/cal/ukb_cal_chr2_v2.bed \
--bim /media/leelabsg-storage0/DATA/UKBB/cal/ukb_snp_chr2_v2.bim \
--fam /media/leelabsg-storage0/GWAS_tutorial/height.fam \
--linear \
--covar covar.cov \
--covar-name Sex,PC1-PC10,Age \
--out chr2
```

Then, after a while, you will obtain three result files:

* `.assoc.linear`: main result file
* `.log`: log for our analysis
* `.nosex`: IIDs of ambiguous sex

And our result file (`.assoc.linear`) looks like

```
 CHR             SNP         BP   A1       TEST    NMISS       BETA         STAT            P 
   2      rs10172629      11944    T        ADD   342681   -0.07482       -1.071       0.2844
   2      rs10172629      11944    T        Sex   342681      -13.3       -613.6            0
   2      rs10172629      11944    T        PC1   342681    0.02394        3.389    0.0007006
   2      rs10172629      11944    T        PC2   342681   -0.00733      -0.9983       0.3181
   2      rs10172629      11944    T        PC3   342681    0.01767        2.493      0.01267
   2      rs10172629      11944    T        PC4   342681   -0.02114       -3.999    6.368e-05
   2      rs10172629      11944    T        PC5   342681   -0.08749       -37.81            0
   2      rs10172629      11944    T        PC6   342681    0.01756        2.598     0.009376
   2      rs10172629      11944    T        PC7   342681   -0.04646       -7.671    1.718e-14
   2      rs10172629      11944    T        PC8   342681    0.06326        10.53    6.559e-26
   2      rs10172629      11944    T        PC9   342681  -0.000604      -0.2491       0.8033
   2      rs10172629      11944    T       PC10   342681   -0.05583       -10.62    2.546e-26
   2      rs10172629      11944    T        Age   342681    -0.1596         -118            0
   2       rs7595668      16937    A        ADD   335808   -0.02245      -0.4165       0.6771
   2       rs7595668      16937    A        Sex   335808      -13.3       -607.6            0
   2       rs7595668      16937    A        PC1   335808     0.0239        3.349    0.0008097
   2       rs7595668      16937    A        PC2   335808  -0.006317      -0.8519       0.3943
   2       rs7595668      16937    A        PC3   335808     0.0173        2.416      0.01568
   2       rs7595668      16937    A        PC4   335808   -0.01938        -3.63    0.0002833
   2       rs7595668      16937    A        PC5   335808   -0.08857       -37.89            0
   2       rs7595668      16937    A        PC6   335808    0.01747        2.558      0.01052
   2       rs7595668      16937    A        PC7   335808   -0.04662       -7.624    2.474e-14
   2       rs7595668      16937    A        PC8   335808    0.06389        10.53    6.503e-26
   2       rs7595668      16937    A        PC9   335808 -0.0004022      -0.1643       0.8695
   2       rs7595668      16937    A       PC10   335808   -0.05544       -10.44    1.718e-25
   2       rs7595668      16937    A        Age   335808    -0.1594       -116.8            0
```

**Note**: If the connection to the server is lost, the work will be stopped and you have to start again from the beginning. Therefore, `nohup` command will be of great help for your analysis.

## Homework

1. Draw a manhattan and QQ plot for your result (chromosome 2) using `qqman` package in R, and compare your results with the figure on page 38 in 220715 presentation slide.
2. If the above experiment was reproduced normally, try *Genome-wide association test* (using all 22 chromosomes) using `plink`. When you obtain the results, draw a manhattan and QQ plot.
3. Think of a trait (phenotype) you are interested in. We will try GWAS with that trait next week. You can find the list of phenotypes on the UK Biobank showcase website.

