# Rapid curation (2.0) tutorial

Although hifiasm is a pretty good assembler and YaHS an equally good scaffolder, assembly errors do occur. Whether there are contigs that are misassembled, or scaffolds that are harder for the software to place, sometimes we need to manually curate the assemblies in order to reach the [desired standards](https://www.earthbiogenome.org/report-on-assembly-standards). To do this, we use the [**Rapid curation** suite](https://gitlab.com/wtsi-grit/rapid-curation/-/blob/main/README_software.md), developed by the GRIT team at the Wellcome Sanger Institute, and [**PretextView**](https://github.com/sanger-tol/PretextView), which you'll learn more about in the last tutorial, in combination with [Rapid curation 2.0](https://github.com/Nadolina/Rapid-curation-2.0). This last repository enable us to avoid the tedious TPF (Tile Path Format) editing the manuals and tutorials from the first [Rapid curation](https://gitlab.com/wtsi-grit/rapid-curation/-/blob/main/README_software.md) repository discusses. It is worth taking a look at them anyway, especially for how to interpret the Hi-C contact maps we will work with in the next stage. 

The Rapid curation 2.0 approach involves looking at both haplotypes simultaneously. This usually allows easier determination of sex chromosomes and small autosomal chromosomes (like the microchromosomes in birds). To read more about manual curation in general, you can take a look at [this publication](https://academic.oup.com/gigascience/article/10/1/giaa153/6072294).

Furthermore, when preparing this teaching material we noticed that Sanger has two new pipelines out that replaces the one above, [https://pipelines.tol.sanger.ac.uk/treeval](https://pipelines.tol.sanger.ac.uk/treeval) and [https://pipelines.tol.sanger.ac.uk/curationpretext](https://pipelines.tol.sanger.ac.uk/curationpretext). We were not able to test these yet, but some of the Singularity containers we use below are no longer available online, so if you want to do curation now, you should take a closer look at those pipelines instead.

## Running the Rapid curation suite

### Run the suite

As with the other programs in the assembly pipeline, we have set up a script for you (see the code chunk below):

```
#!/bin/bash
#SBATCH --job-name=curation
#SBATCH --account=nn9984k
#SBATCH --time=4:0:0
#SBATCH --mem-per-cpu=2G
#SBATCH --ntasks-per-node=5

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate curation

WORKDIR=$PWD/data
DESTDIR=$PWD/out
HICDIR=$2

export SINGULARITY_BIND="
$WORKDIR:/data,\
$HICDIR:/hic,\
$DESTDIR:/output,\
$TMP_DIR:/tmp
"

#hic
[ -f data/ref.fa.bwt ] || bwa index data/ref.fa

bwa mem -t 5 -5SPM data/ref.fa \
$2 $3 \
|samtools view -buS - | samtools sort -@2 -n -T tmp_n -O bam - \
|samtools fixmate -mr - -|samtools sort -@2 -T hic_tmp -O bam - | samtools markdup -rsS - -  2> hic_markdup.stats |samtools sort -@2  -T temp_n -O bam > hic_markdup.sort.bam

samtools view -h hic_markdup.sort.bam | PretextMap -o bwa_map.pretext --sortby length --sortorder descend --mapq 0

#coverage
minimap2 -ax map-hifi \
         -t 5 data/ref.fa \
	$4 \
| samtools sort -@2 -O BAM -o coverage.bam

samtools view -b -F 256 coverage.bam > coverage_pri.bam

samtools index coverage_pri.bam

bamCoverage -b coverage_pri.bam -o coverage.bw

#gaps
singularity run /cluster/projects/nn9984k/opt/rapid-curation/rapid_hic_software/runGap.sif -t $1

#repeats
singularity run /cluster/projects/nn9984k/opt/rapid-curation/rapid_hic_software/runRepeat.sif -t $1

#telomers
singularity run /cluster/projects/nn9984k/opt/rapid-curation/rapid_hic_software/runTelo.sif -t $1 -s TTAGG

cp bwa_map.pretext $1.pretext

#put it together
bigWigToBedGraph coverage.bw  /dev/stdout |PretextGraph -i $1.pretext -n "PB coverage"

cat out/*_gap.bedgraph  | PretextGraph -i $1.pretext -n "gaps"

cat out/*_telomere.bedgraph |awk -v OFS="\t" '{$4 *= 1000; print}' | PretextGraph -i $1.pretext -n "telomers"

bigWigToBedGraph  out/*_repeat_density.bw /dev/stdout | PretextGraph -i $1.pretext -n "repeat density"
```

This script creates both the Hi-C contact map that we'll use in PretextView, and the overlays we'll use to inform our edits during curation. 

### Starting the script

To run the script above, create a `run.sh` file in a new `curation` directory, and run the code using `sh run.sh`. 

```
mkdir -p data
mkdir -p out

#or use your own
cat /cluster/projects/nn9984k/data/iyAthRosa1/fcsgx/iyAthRosa_scaffolds_final_22x_h1.decon.fasta |sed "s/>/>H1_/g" > data/ref.fa 
cat /cluster/projects/nn9984k/data/iyAthRosa1/fcsgx/iyAthRosa_scaffolds_final_22x_h2.decon.fasta |sed "s/>/>H2_/g" > data/ref.fa 

sbatch /cluster/projects/nn9984k/scripts/run_rapidcuration.sh iyAthRosa \
/cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/hic/ERR6054981_1_50x.fastq.gz \
/cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/hic/ERR6054981_2_50x.fastq.gz \
/cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/pacbio/ERR6548410_22x.fastq.gz
```

After this is finished, you should have an iyAthRosa.pretext file which can be used for manual curation. 

If you don´t want to wait for your scripts to finish, and you want to start curating right away, we have provided the file you need to do so. To download the PRETEXT file to your local computer, open a new terminal window, navigate to where you want to place the file, and use the code below:

```
scp -r <username>@saga.sigma2.no:/cluster/projects/nn9984k/data/iyAthRosa1/pretext/iyAthRosa_30x.pretext .

```

There is also a version based on 22x PacBio reads in the same location. One difference is that there is more scaffolds/contigs in the 22x version.

## Software versions used
```
eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 
conda activate curation
conda list
```

deeptools version 3.5.5 

pretextgraph version 0.0.6

pretextsnapshot version 0.0.4

pretextmap version 0.1.9 

ucsc-bigwigtobedgraph version 448

bwa version 0.7.17 

minimap2 version 2.27 

samtools version 1.19.2

biopython version 1.83

gfastats version 1.3.6

pandas version 2.2.1

mashmap 3.1.3



|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/09_FCS_GX.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/11_PretextView.md)|
|---|---|
