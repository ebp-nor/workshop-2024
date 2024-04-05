## EvidenceModeler

If everything so far has gone well, you should have three different annotations/files with gene models. One from GALBA, and two from the miniprot alignments. This is as much as we can do in a workshop like this, but remember that if you were to "properly" annotate your own genome assembly, additional _ab initio_ gene predictions and/or more aligned protein data and/or some kind of transcriptome evidence such as RNA-seq or IsoSeq is higly recommended. 

With that many files/annotations, it can be hard to know what to do. You could of course pick one based on some kind of criteria (maybe number of complete BUSCO genes). Some tools, such as BRAKER1/2/3, usually create one set of genes. However, in our case we want to combine the different annotations/files we created into one, non-redudant, gene set. We will do this using [EvidenceModeler](https://github.com/EVidenceModeler/EVidenceModeler) ([Haas et al (2008)](https://pubmed.ncbi.nlm.nih.gov/18190707/)).

We have installed [Funannotate](https://github.com/nextgenusfs/funannotate), an all-in-one genome annotation pipeline, on Fox for this workshop. It is a nice pipeline, but for you to get a better understanding of what actually happens when annotating a genome, we do this workshop by running different programs independently. If you are looking for something more comprehensive you should take a look at Funannotate. In any case, it has a some features that we will take direct advantage of here, namely a script that runs EvidenceModeler for us. This can be run like so:

```
#!/bin/bash
#SBATCH --job-name=evm
#SBATCH --account=nn9984k
#SBATCH --time=1:0:0
#SBATCH --mem-per-cpu=1G
#SBATCH --ntasks-per-node=10

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate anno_pipeline

export EVM_HOME=/cluster/projects/nn9984k/miniforge3/envs/anno_pipeline/opt/evidencemodeler-1.1.1/

/cluster/projects/nn9984k/miniforge3/envs/anno_pipeline/opt/evidencemodeler-1.1.1/EvmUtils/misc/augustus_GTF_to_EVM_GFF3.pl $2 > galba.evm.gff3

cat galba.evm.gff3 > gene_predictions.gff3
cat $3 >> gene_predictions.gff3
cat $4 >> gene_predictions.gff3

printf "ABINITIO_PREDICTION\tAugustus\t1\n" > weights.evm.txt 
printf "OTHER_PREDICTION\tminiprot\t7\n" >> weights.evm.txt

python  /cluster/projects/nn9984k/miniforge3/envs/anno_pipeline/lib/python3.8/site-packages/funannotate/aux_scripts/funannotate-runEVM.py \
-w weights.evm.txt \
-c 10 \
-d . \
-g gene_predictions.gff3 \
-f $1 \
-l evm.log \
-m 10 \
-i 1500 \
-o evm.gff3 \
--EVM_HOME /cluster/projects/nn9984k/miniforge3/envs/anno_pipeline/opt/evidencemodeler-1.1.1/ \
1> evm_"`date +\%y\%m\%d_\%H\%M\%S`".out 2> evm_"`date +\%y\%m\%d_\%H\%M\%S`".err 

agat_sp_extract_sequences.pl --gff evm.gff3 -f $1 -t cds -p -o evm.proteins.fa 1> agat_proteins_"`date +\%y\%m\%d_\%H\%M\%S`".out 2> agat_proteins_"`date +\%y\%m\%d_\%H\%M\%S`".err
agat_sp_extract_sequences.pl --gff evm.gff3 -f $1 -t exon --merge -o evm.mrna.fa 1> agat_exons_"`date +\%y\%m\%d_\%H\%M\%S`".out 2> agat_exons_"`date +\%y\%m\%d_\%H\%M\%S`".err
```

There are several things going on here. The first is running `augustus_GTF_to_EVM_GFF3.pl` on the GALBA output to create a GFF file that EvidenceModeler accepts. Then we concatenate the resulting file together with the two files from [miniprot](02_miniprot.md). `weights.evm.txt` is created, and contains two lines that determine how much weight EvidenceModeler should put on the _ab initio_ gene predictions versus the protein alignments. You should try to get an impression of what the different options are meant for here. In the end we create an `evm.proteins.fa` file with all the predicted proteins as output of EvidenceModeler.

Create a subfolder called `evm`, enter it and create a `run.sh` with this content:
```
sbatch /cluster/projects/nn9984k/scripts/annotation/run_evm.sh \
../softmask/gzUmbRama1.softmasked.fa \
../galba/galba.gtf \
../miniprot/mp_uniprot_sprot_evm.gff \
../miniprot/mp_model_evm.gff
```

(There are backups for all input files at `/cluster/projects/nn9984k/data/annotation` if some of them were not generated for some reason.)

Submit the job.
This ran for 80 minutes when testing.

|[Previous](https://github.com/ebp-nor/gworkshop-2024/blob/main/day2_genome_annotation/03_galba.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day2_genome_annotation/05_busco.md)|
|---|---|





