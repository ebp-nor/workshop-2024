# hifiasm tutorial

The kind of sequencing data you have determines what kind of assembler you will have to use. Most assemblers have a recommended set of inputs, and this basically guides what kind of sequencing strategy you would use for a particular species. As always, you should start from what you actually want from genome sequencing and assembly project, and then figure out what kind of data and assembly process you need to achieve this. We want to create [haplotype-resolved assemblies](https://lh3.github.io/2021/10/10/introducing-dual-assembly), and up until quite recently ([these](https://doi.org/10.1101/2023.02.21.529152) [two](https://www.biorxiv.org/content/10.1101/2024.03.15.585294v2) papers change the lay of the land a bit), the easiest  way to do this was to generate PacBio HiFi and Hi-C reads and use [hifiasm](https://github.com/chhylp123/hifiasm). It is quite quick, easy to use, and creates high quality assemblies with longer contigs.

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

We have set up this script for you. What you need to do is to create a run.sh in your working folder (`/cluster/projects/nn9984k/work/<username>/assembly/hifiasm`) with this content (with `nano` for instance): 
 
```
sbatch /cluster/projects/nn9984k/scripts/run_hifiasm.sh iyAthRosa \
/cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/hic/ERR6054981_1_50x.fastq.gz \
/cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/hic/ERR6054981_2_50x.fastq.gz \
/cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/pacbio/ERR6548410_22x.fastq.gz
```
This script contain the unfiltered HiFi reads. **You have to replace the reads with the filtered reads you created using HiFiAdapterFilt.** (Only replace the last line and keep the Hi-C reads lines as they are.)

When you have done this, you can submit to the cluster by typing `sh run.sh`.
 
This will run for a while. When testing it ran for 50 minutes. As this step takes a bit longer, we have provided the necessary output in case your analysis didn't finish yet when you need it. You can monitor the progress with `squeue -u <username>`.

For extra fun:
We have also subsampled the PacBio sequencing reads to 26x and 30x coverages (look in `/cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/pacbio/`). If you have time and/or if you want, you can assemble the genome using that data instead, or in addition. Please don't start all three different assemblies at the same time, but chose one or collaborate with someone else. You will get some different metrics out of these. 26x coverage is what is recommened by the developers of hifiasm, and 30x is what is recommended by [Vertebrate Genomes Project](https://www.nature.com/articles/s41587-023-02100-3). 

## Software versions used
```
eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 
conda activate hifiasm
conda list
```
hifiasm version 0.19.8


|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/03_HiFiAdapterFilt.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/05_YaHS.md)|
|---|---|
