# Biobank Data Analysis Tutorial Session - Day 3

## Check your GWAS results

Any questions?

## Useful tools

### Extract SNPs and samples of interest from `.bed` files  using `PLINK`

Sometimes you will need to extract a part of genotype data for several reasons.

* Extract SNPs of interest
* Extract samples (individuals) of interest

In this tutorial, we will use files in `/media/leelabsg-storage0/GWAS_tutorial/data`.

There are 250,857 variants in `UKB_step1.bim` file, and 408,946 samples in `UKB_step1.fam` file.

Suppose we want to extract 1,000 SNPs and 5,000 samples.

* `snps.txt` contains SNP IDs of 1,000 SNPs of interest
* `samples.txt` contains FIDs and IIDs of 5,000 samples of interest.

We can extract the genotype data file set using the following command:

```
plink2 \
--bfile /media/leelabsg-storage0/GWAS_tutorial/data/UKB_step1 \
--extract /media/leelabsg-storage0/GWAS_tutorial/data/snps.txt \
--keep /media/leelabsg-storage0/GWAS_tutorial/data/samples.txt \
--make-bed \
--out /media/leelabsg-storage0/GWAS_tutorial/output
```

We can verify that the filtering (extraction) is done well based on the SNPs and samples of interest, and the size of the `.bed` file is greatly reduced.

### Read genotype from `.bed` or `.bgen` file

* `seqminer` package in R
* `bgen_reader` package in Python

In this tutorial, we will cover the following:

* Read the genotype data from PLINK binary format (`.bed`) files using `seqminer`
* Read the genotype data from Oxford format (`.bgen`) files using `seqminer`
* Extract genotype data of SNPs and samples of interest from PLINK binary format (`.bed`) files using `PLINK`

#### Read `.bed` files using `seqminer`

```
# INPUT
library(seqminer)
fname = "/media/leelabsg-storage0/KBN_WORK/plinkfile/by_chr/KBN_CHR22"
plinkObj = openPlink(fname)

str(plinkObj)
```

Note that all three files should have the same file name: `KBN_CHR22.bed`, `KBN_CHR22.bim` and `KBN_CHR22.fam`.
Then we can see the contents of `.bim` file and `.fam` file.

```
# OUTPUT
List of 3
 $ prefix: chr "/media/leelabsg-storage0/KBN_WORK/plinkfile/by_chr/KBN_CHR22"
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

#### Read `.bgen` files using `seqminer`

You can also read `.bgen` files using `seqminer` in R.

```
# INPUT
library(seqminer)
fname = "/media/leelabsg-storage0/KBN/Genomics/KCHIP_72298/chr22.bgen"
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

### PheWeb

PheWeb is a tool for exploring and visualizing large-scale genetic associations. \
We can host a web server using PheWeb.

* PheWeb paper (*Nature Genetics*, 2020): https://www.nature.com/articles/s41588-020-0622-5
* PheWeb GitHub: https://github.com/statgen/pheweb

#### Several pages using PheWeb

* KoGES PheWeb: https://koges.leelabsg.org/
* SAIGE-UKB PheWeb (1,403 binary phenotypes in UKB): https://pheweb.org/UKB-SAIGE/
* UKB WES 200k PheWeb (Gene-based test for 171 phenotypes in UKB): https://ukb-200kexome.leelabsg.org/
* Neale lab's GWAS PheWeb (2,419 phenotypes in UKB): https://pheweb.org/UKB-Neale/
* Biobank Japan PheWeb: https://pheweb.jp/
* Taiwan Biobank PheWeb: http://pheweb.twbiobank.org.tw:5038/
 
### LocusZoom

LocusZoom is a tool to provide fast visualization of GWAS results for research and publication. \
You can easily visualize (Manhattan and QQ plot) your analysis by uploading the GWAS summary statistics.

* LocusZoom website: http://locuszoom.org/

### FUMA

FUMA is a platform that can be used to annotate, prioritize, visualize and interpret GWAS results. \
FUMA provides several useful functions including gene mapping.

* FUMA website: https://fuma.ctglab.nl/

### GWAS Catalog

GWAS Catalog is a database of published genetic associations. \
It contains 6,688 publications and 569,163 associations (as of January 2024).

* GWAS Catalog website: https://www.ebi.ac.uk/gwas/

You can also use [gwasrapidd](https://github.com/ramiromagno/gwasrapidd) package to use API of GWAS Catalog in R

### dbSNP

dbSNP contains human single nucleotide variations, microsatellites, and small-scale insertions and deletions along with publication, population frequency, molecular consequence, and genomic and RefSeq mapping information for both common variations and clinical mutations.

* dbSNP website: https://www.ncbi.nlm.nih.gov/snp/
