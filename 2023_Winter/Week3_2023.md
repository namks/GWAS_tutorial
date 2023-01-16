# Biobank Data Analysis Tutorial Session - Week 3

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

* `seqminer` package in R (Refer [link](https://github.com/namks/GWAS_tutorial/blob/main/Week3_2021.md) for details)
* `bgen_reader` package in Python

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
It contains 6,180 publications and 458,152 associations (as of December 2022).

* GWAS Catalog website: https://www.ebi.ac.uk/gwas/

You can also use [gwasrapidd](https://github.com/ramiromagno/gwasrapidd) package to use API of GWAS Catalog in R

### dbSNP

dbSNP contains human single nucleotide variations, microsatellites, and small-scale insertions and deletions along with publication, population frequency, molecular consequence, and genomic and RefSeq mapping information for both common variations and clinical mutations.

* dbSNP website: https://www.ncbi.nlm.nih.gov/snp/
