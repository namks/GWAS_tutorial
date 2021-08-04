# Biobank Data Analysis Tutorial Session - Week 3

## Check your GWAS results

How long did your GWAS take? Have a question?

## Genotype Files

### PLINK binary format (.bed, .bim, .fam)

You're probably familiar with PLINK binary format now.
* `.bed` file: Genotype data in binary format
* `.bim` file: Variants information
* `.fam` file: Sample information

PLINK binary format is a very efficient format to store the **called** genotype data.

**BUT**, there is a big limitation: cannot store the probability information (from imputation).
To store the probability information, a different format is required.
* Variant Call Format (`.vcf`)
* Oxford Format (`.bgen`)
* Other formats: `.bcf` (binary version of VCF), `.sav` (Sparse Allele Vectors)

#### Imputation and dosage

Imputation is the statistical inference of unobserved genotypes. It is achieved by using known haplotypes in a population. (*Wikipedia*)
By imputation, we obtain the probability for each genotype: P(G = 0), P(G = 1) and P(G = 2)

Dosage is the expectation of genotype. For example, assuming that your genotype probability of a certain SNP is
* P(G = 0) = 0.2
* P(G = 1) = 0.5
* P(G = 2) = 0.3

Then the dosage for this SNP is `0 * 0.2 + 1 * 0.5 + 2 * 0.3 = 1.1`.
(Therefore, the minimum dosage we can get is 0 and maximum 2.)

In this tutorial, we will cover two formats: `.vcf` and `.bgen`.

### Variant Call Format (.vcf)

Imputed KoGES (KBN) data are distributed in `.vcf` format.
You can find the genotype file in `/media/leelabsg_storage01/KBN/유전정보/KCHIP_72298`.

We can check the contents of the file with the following command:

`zcat CHR1_annoINFO_filINFO0.8_72K.vcf.gz | head -27 | cut -f 1-10 -d$'\t'`

The output is as follows:

```
##fileformat=VCFv4.1
##FILTER=<ID=PASS,Description="All filters passed">
##fileDate=20190511
##FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype">
##FORMAT=<ID=DS,Number=1,Type=Float,Description="Genotype Dosage">
##FORMAT=<ID=GP,Number=G,Type=Float,Description="Genotype Probability">
##INFO=<ID=IQS,Number=1,Type=Float,Description="Imputation Quality Score">
##contig=<ID=1>
##bcftools_viewVersion=1.3+htslib-1.3
##bcftools_viewCommand=view -i 'IQS >= 0.8 & MAF >= 0.01' /jdata/scratch/myhwang/KCHIP_160K/02_IMPUTATION/RESULTs/annoINFO/chr1_10178_5000000_annoINFO.vcf.gz
##bcftools_concatVersion=1.3+htslib-1.3
##bcftools_concatCommand=concat --file-list /jdata/scratch/myhwang/KCHIP_160K/02_IMPUTATION/INPUTs/mergeList/chr1_mergeList.txt
##INFO=<ID=AC,Number=A,Type=Integer,Description="Allele count in genotypes">
##INFO=<ID=AN,Number=1,Type=Integer,Description="Total number of alleles in called genotypes">
##bcftools_viewCommand=view --samples-file /jdata/scratch/myhwang/KCHIP_160K/02_IMPUTATION/INPUTs/KCHIP_78K_20190514.txt /jdata/scratch/myhwang/KCHIP_160K/02_IMPUTATION/RESULTs/mergeIMPUTE4/chr1_annoINFO_filINFO0.8.vcf.gz
##bcftools_viewCommand=view --samples-file ^/jdata/scratch/myhwang/KCHIP_160K/02_IMPUTATION/INPUTs/withdraw_20200206.txt /jdata/scratch/myhwang/KCHIP_160K/02_IMPUTATION/RESULTs/sampleExtract/chr1_annoINFO_filINFO0.8_Open_78K_20190514.vcf.gz
#CHROM  POS     ID                      REF             ALT     QUAL    FILTER  INFO                            FORMAT          NIH20O7486348
1       714646  1:714646_TTTGAAAC/T     TTTGAAAC        T       .       PASS    IQS=0.849878;AC=3792;AN=144596  GT:DS:GP        0/0:0.004:0.996,0.004,0
1       725256  1:725256_C/T            C               T       .       PASS    IQS=0.848369;AC=3829;AN=144596  GT:DS:GP        0/0:0.004:0.996,0.004,0
1       726805  1:726805_G/GA           G               GA      .       PASS    IQS=0.824008;AC=3714;AN=144596  GT:DS:GP        0/0:0.004:0.996,0.004,0
1       726807  1:726807_TTG/T          TTG             T       .       PASS    IQS=0.824008;AC=3714;AN=144596  GT:DS:GP        0/0:0.004:0.996,0.004,0
1       726812  1:726812_A/AG           A               AG      .       PASS    IQS=0.824008;AC=3714;AN=144596  GT:DS:GP        0/0:0.004:0.996,0.004,0
1       746898  1:746898_G/A            G               A       .       PASS    IQS=0.816327;AC=1646;AN=144596  GT:DS:GP        0/0:0.001:0.999,0.001,0
1       749825  1:749825_G/A            G               A       .       PASS    IQS=0.895944;AC=3039;AN=144596  GT:DS:GP        0/0:0:1,0,0
1       751343  1:751343_T/A            T               A       .       PASS    IQS=0.888948;AC=21654;AN=144596 GT:DS:GP        0/0:0.023:0.977,0.023,0
1       751488  1:751488_G/GA           G               GA      .       PASS    IQS=0.81448;AC=21580;AN=144596  GT:DS:GP        0/0:0.005:0.995,0.005,0
1       751599  1:751599_T/C            T               C       .       PASS    IQS=0.960483;AC=3273;AN=144596  GT:DS:GP        0/0:0:1,0,0
```

The first few lines are for metadata, and followed by genotype data.

### Oxford format (.bgen)

Imputed UKB data are distributed in `.bgen` format.

You can find the genotype file in `/media/leelabsg_storage01/DATA/UKBB/imp`, but you cannot read this file in command line interface.

There are several tools for handling `.bgen` files.

* `seqminer` in R
* `bgen_reader` in Python
* PLINK (https://www.cog-genomics.org/plink/1.9/input#oxford)
* QCTOOLS (https://www.well.ox.ac.uk/~gav/qctool_v2/)


## Handling Genotype Files

In this tutorial, we will cover the following:

* Read the genotype data from PLINK binary format (`.bed`) files using `seqminer`
* Read the genotype data from Oxford format (`.bgen`) files using `seqminer`
* Extract genotype data of SNPs and samples of interest from PLINK binary format (`.bed`) files using `PLINK`

### Read `.bed` files using `seqminer`

```
# INPUT
library(seqminer)
fname = "/media/leelabsg_storage01/KBN_WORK/plinkfile/by_chr/KBN_CHR22"
plinkObj = openPlink(fname)

str(plinkObj)
```

Note that all three files should have the same file name: `KBN_CHR22.bed`, `KBN_CHR22.bim` and `KBN_CHR22.fam`.
Then we can see the contents of `.bim` file and `.fam` file.

```
# OUTPUT
List of 3
 $ prefix: chr "/media/leelabsg_storage01/KBN_WORK/plinkfile/by_chr/KBN_CHR22"
 $ fam   :'data.frame':	72298 obs. of  6 variables:
  ..$ V1: int [1:72298] 0 0 0 0 0 0 0 0 0 0 ...
  ..$ V2: chr [1:72298] "NIH20O7486348" "NIH20O7673779" "NIH20O7662312" "NIH20O7330271" ...
  ..$ V3: int [1:72298] 0 0 0 0 0 0 0 0 0 0 ...
  ..$ V4: int [1:72298] 0 0 0 0 0 0 0 0 0 0 ...
  ..$ V5: int [1:72298] 0 0 0 0 0 0 0 0 0 0 ...
  ..$ V6: int [1:72298] -9 -9 -9 -9 -9 -9 -9 -9 -9 -9 ...
 $ bim   :'data.frame':	108762 obs. of  6 variables:
  ..$ V1: int [1:108762] 22 22 22 22 22 22 22 22 22 22 ...
  ..$ V2: chr [1:108762] "22:16439593_G/A" "22:16440500_T/C" "22:16441330_G/C" "22:16441441_A/G" ...
  ..$ V3: int [1:108762] 0 0 0 0 0 0 0 0 0 0 ...
  ..$ V4: int [1:108762] 16439593 16440500 16441330 16441441 16442005 16444235 16458883 16459520 16460679 16462088 ...
  ..$ V5: chr [1:108762] "A" "C" "C" "G" ...
  ..$ V6: chr [1:108762] "G" "T" "G" "A" ...
 - attr(*, "class")= chr "PlinkFile"
```

Next, we want to see the genotypes of SNPs and samples of interest.

```
# INPUT
sampleIndex = seq(10)    # 1 2 3 ... 10
markerIndex = seq(15)    # 1 2 3 ... 15
readPlinkToMatrixByIndex(fname, sampleIndex, markerIndex)
```

Then the genotype matrix of given SNPs and samples will be returned.
```
# OUTPUT
              22:16439593 22:16440500 22:16441330 22:16441441 22:16442005
NIH20O7486348           2           2           2           2           2
NIH20O7673779           2           2           2           2           2
NIH20O7662312           2           2           2           2           2
NIH20O7330271           2           2           2           2           2
NIH20O7807305           1           1           1           1           1
NIH20O7589097           2           2           2           2           2
NIH20O7475800           2           2           2           2           2
NIH20O7888377           2           2           2           2           2
NIH20O7471566           2           2           2           2           2
NIH20O7062415           2           2           2           2           2
              22:16444235 22:16458883 22:16459520 22:16460679 22:16462088
NIH20O7486348           2           2           2           2           2
NIH20O7673779           2           2           2           2           2
NIH20O7662312           2           2           2           2           2
NIH20O7330271           2           2           2           2           2
NIH20O7807305           1           1           1           1           1
NIH20O7589097           2           2           2           2           2
NIH20O7475800           2           2           2           2           2
NIH20O7888377           2           2           2           2           2
NIH20O7471566           2           2           2           2           2
NIH20O7062415           2           2           2           2           2
              22:16464274 22:16464422 22:16464502 22:16464716 22:16474473
NIH20O7486348           2           0           2           0           2
NIH20O7673779           2           2           2           2           2
NIH20O7662312           2           0           2           0           2
NIH20O7330271           1           1           2           1           2
NIH20O7807305           2           1           1           1           1
NIH20O7589097           1           1           2           1           2
NIH20O7475800           1           0           2           0           2
NIH20O7888377           1           1           2           1           2
NIH20O7471566           1           0           2           0           2
NIH20O7062415           2           0           2           0           2
```

**Exercise**

Find the genotype of position `22:41418229_T/G` of the individual with IID `NIH20O7911808`.
Hint: `match` function will help you find the index.

### Read `.bgen` files using `seqminer`

You can also read `.bgen` files using `seqminer` in R.

```
# INPUT
library(seqminer)
fname = "/media/leelabsg_storage01/KBN/유전정보/KCHIP_72298/chr22.bgen"
range = "22:16440000-16460000"
bgenObj = readBGENToListByRange(fname, range)
str(bgenObj)
```

Then, You can see what kind of data is there.
```
# OUTPUT
List of 8
 $ chrom      : chr [1:7] "22" "22" "22" "22" ...
 $ pos        : int [1:7] 16440500 16441330 16441441 16442005 16444235 16458883 16459520
 $ varid      : chr [1:7] "22:16440500_T/C" "22:16441330_G/C" "22:16441441_A/G" "22:16442005_A/G" ...
 $ rsid       : chr [1:7] "22:16440500_T/C" "22:16441330_G/C" "22:16441441_A/G" "22:16442005_A/G" ...
 $ alleles    : chr [1:7] "T,C" "G,C" "A,G" "A,G" ...
 $ isPhased   : logi [1:7] FALSE FALSE FALSE FALSE FALSE FALSE ...
 $ probability:List of 7
  ..$ : num [1:3, 1:72298] 9.67e-01 3.30e-02 -2.61e-08 9.69e-01 3.10e-02 ...
  ..$ : num [1:3, 1:72298] 9.67e-01 3.30e-02 -2.61e-08 9.67e-01 3.30e-02 ...
  ..$ : num [1:3, 1:72298] 9.67e-01 3.30e-02 -2.61e-08 9.67e-01 3.30e-02 ...
  ..$ : num [1:3, 1:72298] 9.67e-01 3.30e-02 -2.61e-08 9.67e-01 3.30e-02 ...
  ..$ : num [1:3, 1:72298] 9.67e-01 3.30e-02 -2.61e-08 9.67e-01 3.30e-02 ...
  ..$ : num [1:3, 1:72298] 9.68e-01 3.20e-02 -1.12e-08 9.68e-01 3.20e-02 ...
  ..$ : num [1:3, 1:72298] 9.72e-01 2.80e-02 -9.31e-09 9.68e-01 3.20e-02 ...
 $ sampleId   : chr [1:72298] "sample_0" "sample_1" "sample_2" "sample_3" ...
```

There are 7 SNPs in a given range, and there are three probability values per SNP.

For example, suppose that we want to see the genotypes at the first SNP in `bgenObj` (`22:16440500_T/C`) of first 5 individuals.
```
# INPUT
bgenObj$probability[[1]][, 1:5]
```

```
# OUTPUT
              [,1]         [,2]          [,3]          [,4]        [,5]
[1,]  9.669948e-01 9.689937e-01  9.920043e-01  9.920043e-01 0.001998932
[2,]  3.300526e-02 3.100633e-02  7.995727e-03  7.995727e-03 0.998001039
[3,] -2.607703e-08 3.725290e-09 -2.793968e-09 -2.793968e-09 0.000000000
```

This gives you three probability values.

Alternatively, you can use `readBGENToMatrixByRange` function which returns dosage values.

```
# INPUT
bgenObj = readBGENToMatrixByRange(fname, range)
bgenObj$`22:16440000-16460000`[,1:5]
```

```
# OUTPUT
              sample_0   sample_1    sample_2    sample_3 sample_4
22:16440500 0.03300521 0.03100634 0.007995722 0.007995722 0.998001
22:16441330 0.03300521 0.03300521 0.007995722 0.007995722 0.998001
22:16441441 0.03300521 0.03300521 0.007995722 0.007995722 0.998001
22:16442005 0.03300521 0.03300521 0.007995722 0.007995722 0.998001
22:16444235 0.03300521 0.03300521 0.007995722 0.007995722 0.998001
22:16458883 0.03199815 0.03199815 0.005996851 0.007995722 0.998001
22:16459520 0.02800029 0.03199815 0.005996851 0.007995722 0.998001
```

See the documentation for more information on `seqminer`:
(https://cran.r-project.org/web/packages/seqminer/seqminer.pdf)

### Extract SNPs and samples of interest from `.bed` files  using `PLINK`

Sometimes you will need to extract a part of genotype data for several reasons.

* Extract SNPs of interest
* Extract samples (individuals) of interest

In this tutorial, we will use files in `/media/leelabsg_storage01/GWAS_tutorial/test`.

There are 327,540 variants in `test.bim` file, and 72,298 samples in `test.fam` file.

Suppose we want to extract 1,000 SNPs and 5,000 samples.

* `snps.txt` contains SNP IDs of 1,000 SNPs of interest
* `samples.txt` contains FIDs and IIDs of 5,000 samples of interest.

We can extract the genotype data file set using the following command:

```
~/plink \
--bfile /media/leelabsg_storage01/GWAS_tutorial/test/test \
--extract /media/leelabsg_storage01/GWAS_tutorial/test/snps.txt \
--keep /media/leelabsg_storage01/GWAS_tutorial/test/samples.txt \
--make-bed \
--out test_filtered
```

We can verify that the filtering (extraction) is done well based on the SNPs and samples of interest, and the size of the `.bed` file is greatly reduced.
