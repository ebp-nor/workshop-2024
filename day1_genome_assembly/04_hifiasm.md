# hifiasm tutorial

The kind of sequencing data you have sets some limit for what kine of assembler you can use. Most assemblers have a recommended set of inputs, and this basically guide what kind of sequencing strategy you would use for a particular species. As always, you should start from what you actually want from genome sequencing and assembly project, and then figure out what kind of data and assembly process you need to achieve this. We want to create [haplotype-resolved assemblies](https://lh3.github.io/2021/10/10/introducing-dual-assembly), and up until quite recently ([these](https://doi.org/10.1101/2023.02.21.529152) [two](https://www.biorxiv.org/content/10.1101/2024.03.15.585294v2) papers changes the lay of the land a bit), the easiest  way to do this was to generate PacBio HiFi and Hi-C reads and use [hifiasm](https://github.com/chhylp123/hifiasm). It is quite quick, easy to use, and creates high quality assemblies with longer contigs.

## Assembling with hifiasm

```
#!/bin/bash
#SBATCH --job-name=hifiasm
#SBATCH --account=nn9984k
#SBATCH --time=4:0:0
#SBATCH --mem-per-cpu=5G
#SBATCH --ntasks-per-node=10

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate hifiasm

hifiasm -o $1 -t10  \
--h1 $2 \
--h2 $3 \
$4 \
1> hifiasm_"`date +\%y\%m\%d_\%H\%M\%S`".out 2> hifiasm_"`date +\%y\%m\%d_\%H\%M\%S`".err

awk '/^S/{print ">"$2"\n"$3}' $1.hic.hap1.p_ctg.gfa | fold > $1.hic.hap1.p_ctg.fa
awk '/^S/{print ">"$2"\n"$3}' $1.hic.hap2.p_ctg.gfa | fold > $1.hic.hap2.p_ctg.fa
```

We have set up this script for you. What you need to do is to create a run.sh in your working folder (`/cluster/projects/nn9984k/work/<username>/hifiasm`) with this content (with nano for instance): 
 
```
sbatch /cluster/projects/nn9984k/scripts/run_hifiasm.sh iyAthRosa \
/cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/hic/ERR6054981_1_50x.fastq.gz \
/cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/hic/ERR6054981_2_50x.fastq.gz \
/cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/pacbio/ERR6548410_22x.fastq.gz
```
This script contain the unfiltered HiFi reads. Please replace the reads with the filtered reads you created with HiFiAdapterFilt.

When you have done this, you can submit to the cluster by typing `sh run.sh`.
 
This will run for a while. When testing it ran for 3.2 hours. When next needing the assembly, we will use one which is already done. You can monitor the progress with `squeue -u <username>`.

## Software versions used
```
eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 
conda activate hifiasm
conda list
```
hifiasm version 0.19.8


|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/03_HiFiAdapterFilt.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/05_YaHS.md)|
|---|---|
