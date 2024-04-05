## Miniprot
[Miniprot](https://github.com/lh3/miniprot) is one of the most exciting things to have happened in the genome annotation world since many years. Miniprot ([Li 2023](https://academic.oup.com/bioinformatics/article/39/1/btad014/6989621), a nice paper defenitely worth a read) is a software to map proteins to a (nucleotide) genome sequence, and can thus create gene models based on the mapped proteins. It is very fast, allowing us to quickly map different sets of proteins to a genome and get an impression of its genetic content. For closely related species, mapping proteins from one (well-assembled and well-annotated) genome from a closey related species to another could suffice to have a decent annotation. However, since Miniprot is so fast to run, different datasets can be easily mapped to the genome, which can then be merged into one annotation using [EvidenceModeler](04_evidencemodeler) (more On that later). We usally use proteins from a well-annotated and well-assembled relatively closely related species, for instance we can use the human set of proteins to annotate most mammal species. In addition, [UniProtKB/Swiss-Prot](https://academic.oup.com/nar/article/51/D1/D523/6835362) is a database with curated protein models which could contain protein evidence that is lacking in the protein set of the closely related species. Lastly, we often also align the relevant set of proteins from [OrthoDB v11](https://academic.oup.com/nar/article/51/D1/D445/6814468). However, these proteins sets are generally very large, and it can take substantial time and computational power to align these to a genome. Therefore, we will not align OrthoDB proteins in this workshop.

For our fungus we will align the UniProtKB/Swiss-Prot protein database in additon to proteins from a related fungus to the genome. 
  
We suggest that you create a subfolder called `miniprot` at `/cluster/projects/nn9984k/work/$USER/annotation`. Enter the new folder. Again, We have created the two relevant scripts to run miniprot, one for UniProtKB/Swiss-Prot and one for the related fungus. 

This is how the first one looks like: 
```
#!/bin/bash
#SBATCH --job-name=miniprot
#SBATCH --account=nn9984k
#SBATCH --time=1:0:0
#SBATCH --mem-per-cpu=2G
#SBATCH --ntasks-per-node=10

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate anno_pipeline

miniprot -ut5 --gff $1 /cluster/projects/nn9984k/opt/funannotate_db/uniprot_sprot.fasta > mp_uniprot_sprot.gff 2> miniprot_"`date +\%y\%m\%d_\%H\%M\%S`".err

agat_sp_extract_sequences.pl --gff mp_uniprot_sprot.gff -f $1 -t cds -p -o uniprot_sprot.proteins.fa \
1> agat_proteins_"`date +\%y\%m\%d_\%H\%M\%S`".out 2> agat_proteins_"`date +\%y\%m\%d_\%H\%M\%S`".err

python /cluster/projects/nn9984k/scripts/annotation/miniprot_GFF_2_EVM_GFF3.py mp_uniprot_sprot.gff > mp_uniprot_sprot_evm.gff
```
Again, you have to create a `run.sh` script containing the following lines:
```
ln -s /cluster/projects/nn9984k/data/annotation/gzUmbRama1.contigs.fasta .
ln -s /cluster/projects/nn9984k/data/annotation/GCF_025201355.1_Halrad1_protein.faa.gz .
sbatch /cluster/projects/nn9984k/scripts/annotation/run_miniprot_swissprot.sh gzUmbRama1.contigs.fasta
sbatch /cluster/projects/nn9984k/scripts/annotation/run_miniprot_model.sh gzUmbRama1.contigs.fasta GCF_025201355.1_Halrad1_protein.faa.gz
```
Then, you can submit the script, in a similar fashion to what we did for the repeatmasking (run `sh run.sh`)
This `run.sh` script will submit both miniprot jobs. If you look at the script itself, you'll see in addition to miniprot two other programs, `agat_sp_extract_sequences.pl` and `miniprot_GFF_2_EVM_GFF3.py`.  `agat_sp_extract_sequences.pl` is from [AGAT](https://github.com/NBISweden/AGAT) (short for Another Gtf/Gff Analysis Toolkit). It is a really useful set of tools if you need to convert between different formats, create the predicted proteins from the annotation/alignments (as used here) or for adding functional annotation information to the annotation (shown later). `miniprot_GFF_2_EVM_GFF3.py` is an utility that is provided together with EvidenceModeler to convert the output of miniprot into something EvindenceModeler understands (but is not found in the current release yet).   

You should also investigate what the different options to the different programs mean. Usually you will get this information by running the program without any options, or by running it with only the `-h` option. Some options used here might not be relevant for your own purpuse, or we might have gotten something wrong, so it is always good to check!

The alignment of UniProtKB/Swiss-Prot proteins took around 50 minutes when we ran it, but alignment of the related fungus proteins only took a couple of minutes. These jobs can be started independently of the softmasking, so you don't need to wait for the repeatmasking to complete to run these jobs.

|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day2_genome_annotation/01_repeatmasking.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day2_genome_annotation/03_galba.md)|
|---|---|

