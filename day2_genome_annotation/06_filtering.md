## Filtering based on protein lenght and similarity to repeat proteins

EvidenceModeler creates a set of genes that is the consensus of the input sets of genes we provided it. However, as mentioned before, [GALBA](03_galba.md) does not take into account masking in a softmasked genome assembly. Since we mapped all of [UniProtKB/Swiss-prot](02_miniprot.md) against genome, and UniProtKB/Swiss-prot contain all kinds of known proteins including proteins that are found in transposable elements, we likely picked up some genes we don't want to annotate here. Additionally, softmasking with Red should find all (or most at least) repetitive parts of the genome, but there might be transposable elements that only occur once and might therefore not be masked. To address this, we compare all predicted proteins against a set of repeat proteins that are available through the Funannotate pipeline. Furthermore, we might have predicted several short proteins that are possibly not valid proteins, so we will apply a filter on protein length also.

All this can be done with this script:

```
#!/bin/bash
#SBATCH --job-name=filter
#SBATCH --account=nn9984k
#SBATCH --time=1:0:0
#SBATCH --mem-per-cpu=1G
#SBATCH --ntasks-per-node=5

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate anno_pipeline

MIN_AA_SIZE=50

diamond blastp --sensitive --query $1 --threads 5 --out repeats.tsv --db /cluster/projects/nn9984k/opt/funannotate_db/repeats.dmnd --evalue 1e-10 --max-target-seqs 1 --outfmt 6

cut -f 1 repeats.tsv |sort -u > kill.list

rm -rf removed_repeats*
agat_sp_filter_feature_from_kill_list.pl --gff $2 --kill_list kill.list -o removed_repeats.gff

rm -rf filtered_genes*
agat_sp_filter_by_ORF_size.pl --gff removed_repeats.gff -s $MIN_AA_SIZE -o filtered_genes.gff

agat_sp_extract_sequences.pl --gff filtered_genes_sup${MIN_AA_SIZE}.gff -f $3 -t cds -p -o filtered.proteins.fa 1> agat_proteins_"`date +\%y\%m\%d_\%H\%M\%S`".out 2> agat_proteins_"`date +\%y\%m\%d_\%H\%M\%S`".err
```
You can run this script by this `run.sh` script:
```
sbatch /cluster/projects/nn9984k/scripts/annotation/run_filter.sh \
../evm/evm.proteins.fa \
../evm/evm.gff3 \
../softmask/gzUmbRama1.softmasked.fa
```

(There are backups for all input files at `/cluster/projects/nn9984k/data/annotation/` if some of them were not generated for some reason.)

This should only take a minute or so.

|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day2_genome_annotation/05_busco.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day2_genome_annotation/07_functional.md)|
|---|---|
