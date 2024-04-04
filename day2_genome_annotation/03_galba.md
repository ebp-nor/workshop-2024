## GALBA
[Miniprot](02_miniprot) will only output alignments based on the proteins it aligns. It cannot provide any gene models not supported by the input proteins. To cover our bases (nucleotides) we also want to run an _ab initio_ gene predictor. Not much has happened in the genome annotation world the last 10 years or so, as you might have gathered from the Miniprot paper if you have read it. There have been (continued) development of some _ab initio_ gene predictors, mainly [AUGUSTUS](https://github.com/Gaius-Augustus/Augustus) and [GeneMark](http://exon.gatech.edu/GeneMark/), but apart from those very few new programs managed to establish itself as far as we know. GeneMark has quite a restrictive license for usage, making it a bit harder to use in many cases (althoug it was recently announced they would change the license). AUGUSTUS on the other hand is free and open, making it very popular tool for genome annotation. 

The developers of AUGUSTUS have through the years developed different programs that allow one to easily train AUGUSTUS using different datasets (BRAKER1 with RNA-seq data, BRAKER2 with protein data, and recently BRAKER3 incorporating both, available from https://github.com/Gaius-Augustus/BRAKER), and use that to predict genes. Another recent AUGUSTUS wrapper that was developped is called [GALBA](https://github.com/Gaius-Augustus/GALBA). GALBA uses Miniprot to align a set of proteins your genome, and uses that to train AUGUSTUS to predict genes _de novo_.

Here, we will run GALBA with the proteins from a related fungus to generate a set up _ab initio_ predicted genes.

Our pre-prepared jobscript looks like this: 
```
#!/bin/bash
#SBATCH --job-name=galba
#SBATCH --account=nn9984k
#SBATCH --time=1:0:0
#SBATCH --mem-per-cpu=1G
#SBATCH --ntasks-per-node=10

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate anno_pipeline

#singularity exec -B $PWD:/data /cluster/projects/nn9984k/opt/galba/galba.sif cp -rf /usr/share/augustus/config /data/
singularity exec -B $PWD:/data /cluster/projects/nn9984k/opt/galba/galba.sif cp -rf /opt/Augustus/config /data/
singularity exec -B $PWD:/data /cluster/projects/nn9984k/opt/galba/galba.sif galba.pl --version > galba.version
singularity exec -B $PWD:/data /cluster/projects/nn9984k/opt/galba/galba.sif galba.pl --species=$2 --threads=10 \
--genome=/data/$1 \
--verbosity=4 --prot_seq=/data/$3  --workingdir=/data \
--augustus_args="--stopCodonExcludedFromCDS=False" --gff3 \
--AUGUSTUS_CONFIG_PATH=/data/config \
1> galba_"`date +\%y\%m\%d_\%H\%M\%S`".out 2> galba_"`date +\%y\%m\%d_\%H\%M\%S`".err

agat_sp_extract_sequences.pl --gff galba.gtf -f $1  -t cds -p -o galba.proteins.fa
```
Here we use a [Singularity container](https://docs.sylabs.io/guides/3.5/user-guide/introduction.html), because sometimes it can be a hassle to set up everything to run GALBA (and other AUGUSTUS-related software) properly, even with Conda. 

Remeber to create a new folder for GALBA in your working direcotry, and then you can use this jobscript for running GALBA by creating the following `run.sh` script:
```
cp -rf /cluster/projects/nn9984k/data/annotation/GCF_025201355.1_Halrad1_protein.faa  .
cp -rf ../softmask/gzUmbRama1.softmasked.fa .

sbatch /cluster/projects/nn9984k/scripts/annotation/run_galba.sh gzUmbRama1.softmasked.fa gzUmbRama1 GCF_025201355.1_Halrad1_protein.faa
```

(If for some reason `gzUmbRama1.softmasked.fa` was not created, there is a backup located at `/cluster/projects/nn9984k/data/annotation` you can use.)

Why do we need to copy both the genome assembly and the protein sequences to the current folder?

When we ran this, it took about 50 minutes. This job can be started at the same time as the miniprot jobs, but have to be run after the softmasking.

|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day2_genome_annotation/02_miniprot.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day2_genome_annotation/04_evm.md)|
|---|---|

