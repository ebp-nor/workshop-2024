## Functional annotation

After filtering the EvidenceModeler genes we have a set of genes we (hopefully) trust, and which are (hopefully) correct. However, we don't actually know what each gene does, nor what it is named. Not all genes have a known function, nor name, but we have some tools that can help us with this situation. Proteins usually contain domains that can be informative with regards to their function. [InterProScan](https://github.com/ebi-pf-team/interproscan) ([Jones et al (2014)](https://academic.oup.com/bioinformatics/article/30/9/1236/237988)) is a tool that annotates known domains in proteins by comparisons to several different databases, such as PFAM and Panther. 

In addition to annotating domains, it is useful to attach gene names to the proteins. This can be done by a simple comparison to known genes with known names using [DIAMOND](https://github.com/bbuchfink/diamond), similar to what we did for [filtering](06_filtering.md). 

Comparisons against UniProtKB/Swiss-Prot can be done with this script:
```
#!/bin/bash
#SBATCH --job-name=uniprot
#SBATCH --account=nn9984k
#SBATCH --time=1:0:0
#SBATCH --mem-per-cpu=1G
#SBATCH --ntasks-per-node=5

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate anno_pipeline

mkdir -p uniprot

diamond blastp \
--query $1 \
--db /cluster/projects/nn9984k/opt/funannotate_db/uniprot_sprot.fasta \
--outfmt 6 \
--sensitive \
--max-target-seqs 1 \
--evalue 1e-25 \
--threads 5 \
> uniprot/diamond.blastp.out
```
You can run this by 
```
sbatch /cluster/projects/nn9984k/scripts/annotation/run_uniprot.sh ../filter/filtered.proteins.fa
```
Please put that command into a `run.sh` script as usual to keep track of what has been done.

If we have time, we can also add different kinds of annotations such as domains and [Gene Ontology](https://geneontology.org/) via [InterProScan](https://github.com/ebi-pf-team/interproscan).

```
#!/bin/bash
#SBATCH --job-name=interproscan
#SBATCH --account=nn9984k
#SBATCH --time=48:0:0
##SBATCH --partition=bigmem
#SBATCH --mem-per-cpu=4500M
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=20

module --force purge

module load StdEnv

module load InterProScan/5.62-94.0-foss-2022a

mkdir -p $USERWORK/tmp
mkdir -p ipr

cat $1 |sed 's/*//g' > ipr/$1

cd ipr

interproscan.sh --cpu 20 -dp -i $1  \
-f XML,GFF3,TSV -goterms -iprlookup -b $2 -T $USERWORK/tmp/ 1> interproscan.out 2> interproscan.err 
```

You can run this by 
```
sbatch /cluster/projects/nn9984k/scripts/annotation/run_ipr.sh ../filter/filtered.proteins.fa ipr
```

The next script creates a new GFF where the gene names and GOs and such have been added:

```
#!/bin/bash
#SBATCH --job-name=functional
#SBATCH --account=nn9984k
#SBATCH --time=1:0:0
#SBATCH --mem-per-cpu=1G
#SBATCH --ntasks-per-node=5

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate anno_pipeline

MIN_AA_SIZE=50

agat_sp_manage_functional_annotation.pl --gff $1  -b $2 -i $3 \
--ID FUNC \
-o fun \
-db /cluster/projects/nn9984k/opt/funannotate_db/uniprot_sprot.fasta \
1> manage_functional_annotation.out 2> manage_functional_annotation.err

agat_sp_statistics.pl --gff fun/filtered_genes_sup${MIN_AA_SIZE}.gff -o gff_stats 1> gff_stats.out 2> gff_stats.err

cp fun/filtered_genes_sup${MIN_AA_SIZE}.gff $4.gff 

agat_sp_extract_sequences.pl --gff $4.gff -f $5 -t cds -p -o $4.proteins.fa 1> agat_proteins_"`date +\%y\%m\%d_\%H\%M\%S`".out 2> agat_proteins_"`date +\%y\%m\%d_\%H\%M\%S`".err
agat_sp_extract_sequences.pl --gff $4.gff -f $5 -t exon --merge -o $4.mrna.fa 1> agat_mrna_"`date +\%y\%m\%d_\%H\%M\%S`".out 2> agat_mrna_"`date +\%y\%m\%d_\%H\%M\%S`".err
```

You can run this as
```
sbatch /projects/ec146/scripts/annotation/run_functional.sh \
../filter/filtered_genes_sup50.gff  \
uniprot/diamond.blastp.out \
/cluster/projects/nn9984k/data/annotation/ipr.tsv \
awesome_fun_guy \
../softmask/gzUmbRama1.softmasked.fa
```
If you managed to create a `ipr.tsv` yourself, you can add that instead of the previously generated.

(There are backups for all input files at `/cluster/projects/nn9984k/data/annotation/` if some of them were not generated for some reason.)


Congratulations! You have now both structurally and functionally annotated a genome.

How many genes did you get? How many complete BUSCO genes?

The file `/cluster/projects/nn9984k/data/orthofinder/gzUmbIsab1.proteins.fa` is the result of another annotation effort. How many genes are found in that file? How many complete BUSCO genes? What might the difference be attributed to?

|[Previous](https://github.com/ebp-nor/gworkshop-2024/blob/main/day2_genome_annotation/06_filtering.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day2_genome_annotation/orthofinder.md)|
|---|---|


