# Biobank Data Analysis - Appendix

## Converting Genotype Data Formats

### Genotype Data Formats

There are several types of genomics data file formats. (**Bold**: readable format)

* **Variant call format**: `.vcf`
    - may be paired with an index file (`.tbi` or `.csi`)
* Binary variant call format: `.bcf`
* **PLINK format**: `.ped` and `.map`
* PLINK binary format: `.bed`, `.bim`, and `.fam`
* Oxford format: `.bgen`
    - may be paired with an index file (`.bgi`)
* Sparse allele vectors: `.sav`
    - may be paired with an index file (`.s1r`)


### Softwares for Converting Genotype Data

* PLINK
* Tabix
* qctool
* bcftools
* bgenix
* and many other softwares...


### Examples for Converting Genotype Data

Today's genomics data contains tens (or hundreds) of thousands of samples.
Therefore, files with *binary* format are becoming popular.
However, some data (such as KoGES genotype data) are still distributed in the form of `.vcf`.

#### VCF to PLINK binary (`.bed`, `.bim`, `.fam`)

If we want to convert a *single* `.vcf` file to a PLINK binary dataset, we can use the command:

```
plink2 \
--vcf /media/leelabsg-storage0/KBN/유전정보/KCHIP_72298/CHR1_annoINFO_filINFO0.8_72K.vcf.gz \
--make-bed \
--out /media/leelabsg-storage0/GWAS_tutorial/convert_test/CHR1_test
```

We can use a shell script to convert multiple files with certain patterns.

```
#!/bin/bash

for (( i=1; i<=22; i++ ))
do
    nohup plink2 \
        --vcf /media/leelabsg-storage0/KBN/유전정보/KCHIP_72298/CHR${i}_annoINFO_filINFO0.8_72K.vcf.gz \
        --make-bed \
        --out /media/leelabsg-storage0/GWAS_tutorial/convert_test/CHR${i}_test &
done
```

#### VCF to Oxford (`.bgen`)

If we want to convert our genotype data to Oxford formatted files (`.bgen`), we can use the command:

```
#!/bin/bash

for (( i=1; i<=22; i++ ))
do
    nohup plink2 \
        --vcf /media/leelabsg-storage0/KBN/유전정보/KCHIP_72298/CHR${i}_annoINFO_filINFO0.8_72K.vcf.gz dosage=DS \
        --export bgen-1.2 ref-first bits=8 \
        --out /media/leelabsg-storage0/GWAS_tutorial/convert_test/CHR${i}_test_bgen &
done
```

### Generating Index Files

Sometimes the index file for corresponding genotype data file is missing.
We can generate the index files using softwares.

