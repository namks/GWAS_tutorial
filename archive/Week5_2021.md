# Biobank Data Analysis Tutorial Session - Week 5

## Slurm

### What is Slurm?

* **Slurm** (Simple Linux Utility for Resource Management) is a free and open-source job scheduler for Linux (*Wikipedia*)
* **Job scheduler** is a computer application for controlling unattended background program execution of jobs (*Wikipedia*)


### Why Slurm?

* Can process multiple tasks in parallel using multiple nodes
* Can allocate exclusive resource for specific tasks
* Can compare methods under the same condition
* Prevents errors from occurring due to lack of resources

### Usage

Slurm is installed on `cpu01-04` nodes in GSDS cluster. (As of Aug 25, 2021)
You can submit jobs on any node where Slurm is installed.

Slurm commands:

`sbatch`: Submit slurm jobs

`squeue`: Check the status of running/pending jobs

`sinfo`: Check the status of nodes

`scancel`: Cancel slurm jobs

#### Job submission

Two examples for job submission:

1. Directly run a batch file using `sbatch` command:

First, we have to make a script file (`run.sh`) with Slurm options (`#SBATCH`)

```
#!/bin/bash

#SBATCH --job-name=example 
#SBATCH --nodes=1
#SBATCH --gpus=1
#SBATCH --time=0-00:02:00 
#SBATCH --mem=16000MB
#SBATCH --cpus-per-task=8

source /home/${USER}/.bashrc 
conda activate <your torch env>

srun python <your python script> <args...>
```


You can run this script file using
```
sbatch run.sh
```


2. Create batch files (using `while` or `for` loop) and run them with `bash` command:

```
#!/bin/bash

job_directory=$PWD/.job

for((i=1;i<=26;i++))
do
        job_file="${job_directory}/${i}.job"

        echo "#!/bin/bash
        #SBATCH --job-name=${i}.job
        #SBATCH --output=.out/${i}.out
        #SBATCH --error=.out/${i}.err
        #SBATCH --ntasks=1
        #SBATCH --cpus-per-task=16
        srun Rscript ~/SAIGE/extdata/step1_fitNULLGLMM.R \
        --plinkFile=/media/leelabsg_storage01/KBN_WORK/plinkfile/IQS1/KBN_IQS1_Pruned_all \
        --phenoFile=/media/leelabsg_storage01/KBN_WORK/plinkfile/pheno/KBN_SAIGE_${i}_CA2.txt \
        --phenoCol=y_quant \
        --covarColList=B_SEX,B_AGE,PC1,PC2,PC3,PC4,PC5,PC6,PC7,PC8,PC9,PC10,B_DATA_CLASS_A01,B_DATA_CLASS_A02,B_DATA_CLASS_D04,B_DATA_CLASS_D05,B_DATA_CLASS_D06,B_DATA_CLASS_D07,B_DATA_CLASS_D08,B_DATA_CLASS_D09,B_DATA_CLASS_D10,B_DATA_CLASS_D11,B_DATA_CLASS_D12,B_DATA_CLASS_D13,B_DATA_CLASS_D14,B_DATA_CLASS_N01,B_DATA_CLASS_N02,B_DATA_CLASS_N03,B_DATA_CLASS_N04,B_DATA_CLASS_N05,B_DATA_CLASS_N06,B_DATA_CLASS_N08,B_DATA_CLASS_N09,B_DATA_CLASS_N10,B_DATA_CLASS_N11,B_DATA_CLASS_N12,B_DATA_CLASS_N13,B_DATA_CLASS_N14,B_DATA_CLASS_N15,B_DATA_CLASS_N16,B_DATA_CLASS_N17,B_DATA_CLASS_N18 \
        --sampleIDColinphenoFile=V2 \
        --traitType=quantitative \
        --invNormalize=TRUE \
        --outputPrefix=/media/leelabsg_storage01/KBN_WORK/plinkfile/pheno/out_saige/out_saige_${i}_loco \
        --nThreads=4 \
        --LOCO=TRUE \
        --tauInit=1,0" > $job_file
        sbatch $job_file
done
```

You can run this script using
```
bash run.sh
```

You can also specify the node (eg. `--nodelist=cpu01`).

#### Job monitoring (Check status)

You can check the status of running/pending jobs using
```
squeue
```

#### Cancel jobs

If you want to cancel a running job, you can terminate the job using
```
scancel JOB_ID
```

You can also cancel multiple jobs with sequential IDs using `bash` script (`cancel.sh`):
```
#!/bin/bash

for i in {1..10}
do
    scancel ${i}
done
```

```
bash cancel.sh
```


## What we have learned

* Exploring UK Biobank data

    1. Phenotype (`.tab`) files
    2. Covariate (PC) files
    3. PLINK binary (`.bed`, `.bim`, `.fam`) files

* GWAS using PLINK: Quantitative (eg. height) and Binary (eg. disease)
* Drawing manhattan and QQ plot with GWAS results (summary statistics)
* Linux commands
* Various format of genetic data

    1. PLINK binary format
    2. Variant Call format (`.vcf`)
    3. Oxford format (`.bgen`)

* Handling genetic data files using `seqminer` and `PLINK`
* Introduction to SAIGE
* GWAS using SAIGE with various environments:

    1. Anaconda
    2. Docker

* Running jobs using Slurm
