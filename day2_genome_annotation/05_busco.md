## BUSCO

By now, you have created several GFF files with different content, and even merged several of these together with [EvidenceModeler](04_evm.md). However, it is not trivial to actually assess what you got from this. For a genome assembly you can look at [length metrics](https://github.com/ebp-nor/genome-assembly-workshop-2023/blob/main/06_gfastats.md) and [BUSCO scores](https://github.com/ebp-nor/genome-assembly-workshop-2023/blob/main/07_BUSCO.md). You usually know how large the genome assembly should be, and you know that having more complete BUSCO genes is better. However, for annotation it is more difficult to assess the quality. We don't necessary know how many genes we expect a species to have, and how long they should be. Even for the most studied genome, the human genome, We don't even know properly as two different annotated sets of genes for a human reference genome contain [20,054 and 19,817 protein coding genes](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-018-1590-2). Looking at the statistics of the annotated genes might not be informative, apart from having a ballpark number of genes you expect the species to have. However, we can at look at the amount of complete [BUSCO genes](https://busco.ezlab.org/) compared to a reference database. BUSCO ([Manni et al (2021)](https://currentprotocols.onlinelibrary.wiley.com/doi/full/10.1002/cpz1.323) is used extensively for this purpose, although it tends to be more used to assess a genome assembly as mentioned above. 

Here we will set up a script you can run on the different `*.proteins.fa` files that have been created in the different folders so far. We have set up two reference databases, fungi for genes that are expected to be found in all fungi, and mucoromycota for more clade-specific genes. We have set up this script:
```
#!/bin/bash
#SBATCH --job-name=busco
#SBATCH --account=ec146
#SBATCH --time=1:0:0
#SBATCH --mem-per-cpu=1G
#SBATCH --ntasks-per-node=5

eval "$(/fp/projects01/ec146/miniconda3/bin/conda shell.bash hook)" 

conda activate busco

prefix=${1%.*}

mkdir -p busco5_${2}_${prefix}
origdir=$PWD

cd busco5_${2}_${prefix}

busco -c 10 -i ${origdir}/${1} -l /projects/ec146/opt/busco/lineages/${2}_odb10 \
-o assembly -m proteins --offline --tar > busco.out 2> busco.err
```

As you see, there are two parameters here (`$1` and `$2`), the first is the set of proteins you want to compare against the database, and the second is the BUSCO lineage (fungi or mucoromycota in this case). 

We leave writing a `run_busco.sh` file to you as an exercise. You should run both lineages on at least two of the predicted protein sets. The path to the BUSCO script is `/projects/ec146/scripts/annotation/run_busco_lineage.sh`. 

|[Previous](https://github.com/ebp-nor/genome_annotation_comparative_genomics_part1/blob/main/04_evm.md)|[Next](https://github.com/ebp-nor/genome_annotation_comparative_genomics_part1/blob/main/06_filtering.md)|
|---|---|
