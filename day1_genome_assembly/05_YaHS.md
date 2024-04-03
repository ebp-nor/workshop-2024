# YaHS tutorial

If Hifiasm has completetd, than congratulations: you have created your sawfly assembly! However, this is not the end of it. The next step is to combine the generated contigs into larger scaffolds. For this we will use **YaHS**. YaHS stands for “yet another Hi-C scaffolding tool”, and as the name implies, there are a lot of Hi-C scaffolders out there. However, we in EBP-Nor choose to use YaHS because it is fast, creates more contiguous scaffolds, with better genome statistics compared to other widely used scaffolders. To learn more about this, click [*here*](https://github.com/c-zhou/yahs), otherwise scroll down to start scaffolding your haplotype-resolved assemblies.

## Scaffolding with YaHS

```
#!/bin/bash
#SBATCH --job-name=yahs
#SBATCH --account=nn9984k
#SBATCH --time=4:0:0
#SBATCH --mem-per-cpu=20G
#SBATCH --ntasks-per-node=5

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate yahs

REF=$1

[ -s $REF.bwt ] || bwa index $REF

SAMPLE=$2

mkdir -p outs

[ -s hic_markdup.sort_n.bam ] || bwa mem -t 8 -R '@RG\tSM:$SAMPLE\tID:$SAMPLE' -5SPM $REF \
$3 $4 \
|samtools view -buS - |samtools sort -@1 -n -T tmp_n -O bam - \
|samtools fixmate -mr - -|samtools sort -@1 -T hic_tmp -O bam - |samtools markdup -rsS - -  2> hic_markdup.stats |samtools sort -n -@1 -n -T temp_n -O bam\
> hic_markdup.sort_n.bam

[ -s $REF.fai ] ||samtools faidx $REF

if [ -s $SAMPLE.bin ]; then
        yahs $REF $SAMPLE.bin -o $SAMPLE \
        1> yahs_"`date +\%y\%m\%d_\%H\%M\%S`".out 2> yahs_"`date +\%y\%m\%d_\%H\%M\%S`".err
else
        yahs $REF hic_markdup.sort_n.bam -o $SAMPLE \
        1> yahs_"`date +\%y\%m\%d_\%H\%M\%S`".out 2> yahs_"`date +\%y\%m\%d_\%H\%M\%S`".err
fi

```

Again, we have set up this script for you. Create a run.sh in your working folder (`/cluster/projects/nn9984k/work/<username>/assembly/yahs`) with this content (with `nano` for instance):

```
ln -s ../hifiasm/iyAthRosa.hic.hap1.p_ctg.fa .

#or link from /cluster/projects/nn9984k/data/iyAthRosa1/assembly/22x/iyAthRosa.hic.hap1.p_ctg.fa if you are not done yet with the assembly (or from /cluster/projects/nn9984k/data/iyAthRosa1/assembly/26x or /cluster/projects/nn9984k/data/iyAthRosa1/assembly/30x)

sbatch /cluster/projects/nn9984k/scripts/run_yahs.sh iyAthRosa.hic.hap1.p_ctg.fa \
iyAthRosa \
/cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/hic/ERR6054981_1_50x.fastq.gz \
/cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/hic/ERR6054981_2_50x.fastq.gz
```

Make sure to check that you link the correct file from the correct folder!
When you have done this, you can submit to the cluster by typing `sh run.sh`.

When testing, this ran for 1.6 hours. You can monitor the progress with `squeue -u <username>`.

Here we have simplified matters a bit and we will only scaffold one of the haplotype assemblies using all the Hi-C data. Usually, we would filter the Hi-C data based on unique k-mers from each haplotype assembly in order to only scaffold using reads from the proper haplotype.

## Software versions used
```
eval "$(/cluster/projects/nn9984k/miniconda3/bin/conda shell.bash hook)" 
conda activate yahs
conda list
```
bwa version 0.7.17 

samtools version 1.19.2

yahs version 1.2a.2

|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/04_hifiasm.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/06_gfastats.md)|
|---|---|
