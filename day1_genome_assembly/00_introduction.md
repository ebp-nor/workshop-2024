# Genome assembly, curation and validation workshop.

Today you'll learn how to assemble a whole genome.

After attending the workshop you will:
- know about the most-used approaches for genome assembly
- be able to assess information contained in sequencing reads
- be able to understand and run a genome assembly
- be able to validate genome assemblies
- know about manual curation of assemblies

## How to use this material
The material for this workshop is written in a way that you can work through it independently of any other resource. The explanations should suffice to understand the material, but contains links to other resources (e.g. scientific papers and/or websites) where you can learn more if you are interested. These links are not necessary to complete the workshop, but are included as reference material that you can use to learn more about the software and concepts used during the workshop. 
Please read both the text and the scripts carefully. If you want to see what a certain program does, you can generally try running it without any arguments. This will in general show you information about the program, and its parameters. Please don't hesitate to ask questions if you want to know more, or if anything is unclear.

## Our dataset

The coleseed or turnip sawfly, *Athalia rosae*, is a sawfly found in Europe, Asia, North America and Africa. It is often considered a pest, feeding on plants of the brassica family, such as rapeseed, turnip, mustard and cabbage. 

[Darwin Tree of Life](https://www.darwintreeoflife.org) has sequenced and assembled the coleseed sawfly genome. It is published [here](https://wellcomeopenresearch.org/articles/8-87). All the data is open access, allowing us to play around with the data. DToL has some interesting webpages where they list several quality measures for the sequencing (some of which we will do in this workshop) [here](https://tolqc.cog.sanger.ac.uk/darwin/insects/Athalia_rosae/). It can be worth taking a look. We downloaded the data from [ENA](https://www.ebi.ac.uk/ena/browser/view/GCA_917208135) and subsampled it to get it to the coverages we expect/plan for. 

The genome itself is around 170 Mbp, and the PacBio data is about 18 Gbp, which means we have more than 100x coverage. The data was downsampled using [seqtk](https://github.com/lh3/seqtk) like this:
```
seqtk sample ERR6548410.fastq.gz  300000 |gzip > ERR6548410_22x.fastq.gz
```
to get about 22x coverage with PacBio data. We could have used higher coverages, but did not to prevent long running times. We also subsampled the Hi-C data like this:
```
seqtk sample ERR6054981_1.fastq.gz 25000000 |gzip > ERR6054981_1_50x.fastq.gz
seqtk sample ERR6054981_2.fastq.gz 25000000 |gzip > ERR6054981_2_50x.fastq.gz
```
If you want the full set of reads, or if you want to try to subsample it yourself, you can grab the files by these commands:
```
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR605/001/ERR6054981/ERR6054981_1.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR605/001/ERR6054981/ERR6054981_2.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR654/000/ERR6548410/ERR6548410.fastq.gz
```
We have also uploaded [subsampled reads at Zenodo](https://doi.org/10.5281/zenodo.10867805) so that you can play around with this on your own.

## Why do we use a combination of HiFi and Hi-C reads? 

HiFi sequencing creates highly accurate circularized consensus reads. How are these reads generated? By ligating hairpin adapters, the DNA fragment that is being sequenced becomes a circle. This means that the sequencing machine can do multiple passes over the same DNA sequence, to weed out any misread nucleotides. This is how HiFi reads can be relatively long, while remaining over 99.9% accurate. 

Hi-C sequencing is done to capture how the chromatin is folded within the cell nucleus. By ligating the folded DNA strands, we can capture which loci are found in close proximity, and thus which parts of the DNA are found within the same chromosomes.

When combining both technologies, we can create haplotype resolved assemblies, meaning we can separate reads by maternal and paternal origin, without having access to parental data. In diploid (or polyploid) organisms this allows better separation of haplotypes and creates more accurate assemblies than a primary and alternate assembly created from only long-read sequencing data. 

Experience and testing (by various genome projects such as Darwin Tree of Life, Vertebrate Project, and the Earth Biogenome Project Norway) has shown that the combination of HiFi and Hi-C, both using appropriate coverages, usually generates assemblies that fulfill the Earth Biogenome Project's criteria for [assembly standards](https://www.earthbiogenome.org/assembly-standards). Here we use [hifiasm](https://github.com/chhylp123/hifiasm) as the main tool to manage this. Hifiasm has [been designed](https://doi.org/10.1038/s41587-022-01261-x) to take advantage of the combination of Hi-C and HiFi reads. There are other ways to get to these standards, for instance by using combinations of Oxford Nanopore Technologies sequencing data (Pore-C, Duplex and Simplex). A [preprint](https://www.biorxiv.org/content/10.1101/2024.03.15.585294v1) from 17th March 2024 outlines how to do this. The next step would be to benchmark these two approaches in multiple aspects such as material and computational cost and time, and assembly accuracy and quality to figure out which method is preferable under which conditions.

For an up-to-date general overview of the status of genome assembly today (2024), [this paper](https://arxiv.org/abs/2308.07877) by Heng Li and Richard Durbin (two legends in the field) is essential reading.

## Package management

Administrating and installing the different programs that are needed in a project can be a hassle. We like conda/mamba to deal with software installation and dependencay manageing, especially [miniforge](https://github.com/conda-forge/miniforge), and have set up different environments we will use for the different analyses. [Bioconda](https://bioconda.github.io) contain a lot of different packages that are relevant for us, and genomics and bioinformatics in general.

To load the conda installation that we have provided for the workshop, run the following line:

```
eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 
```

Some of the programs we will be using are not available through conda. For most of these we use [Singularity containers](https://docs.sylabs.io/guides/3.5/user-guide/introduction.html). 

For the live workshop, we have set up most scripts and arranged data so it is ready to run, but we ask you to modify them in some cases. We have backups of everything, but please be careful so you don't delete something you shouldn't.

## Infrastructure

For the different analyses we are doing, we will use [Saga](https://documentation.sigma2.no/hpc_machines/saga.html). Everything should be set up properly by now. The project we have at Saga is called nn9984k, and the working folder is `/cluster/projects/nn9984k`. You should set up and do stuff in `/cluster/projects/nn9984k/work/$USERNAME`, but we'll come back to that in the next section.

On Saga we will submit jobs/analyses as job scripts. This is for a system called SLURM. Basically, this is instructions to the system for what kind of analysis we are running, or more concretely, how much memory and computing power we need. SLURM manages all the submitted jobs from all users and assigns them to the computing nodes for execution.

A generic job script might look like this (copied from [Saga](https://documentation.sigma2.no/hpc_machines/saga.html)):
```
#SBATCH --account=MyProject
#SBATCH --job-name=MyJob
#SBATCH --time=1-0:0:0
#SBATCH --mem-per-cpu=3G
#SBATCH --ntasks=16

First command
Second command
Another command
...
```
## Other resources about assembly

We are not the first to create a workshop or tutorials about genome assembly. Here are a couple of good sources for more information.

Vertebrate Genomes Project has a workflow they have implemented in Galaxy, but with enough detail that you can run it yourself outside of Galaxy. They also link to a bit of background material. You can reach it here: 
[https://training.galaxyproject.org/training-material//topics/assembly/tutorials/vgp_genome_assembly/tutorial.html](https://training.galaxyproject.org/training-material//topics/assembly/tutorials/vgp_genome_assembly/tutorial.html)

UC Davis Bioinformatics Core has a genome assembly workshop they held in 2020, which also contains a lot of useful information if you want to deep-dive into this subject. Read it here: [https://ucdavis-bioinformatics-training.github.io/2020-Genome_Assembly_Workshop/](https://ucdavis-bioinformatics-training.github.io/2020-Genome_Assembly_Workshop/)

We also try to give links to relevant GitHub repositories and publications/preprints, at least the most essential, across these GitHub pages. This is meant for your own enjoyment and interest. 


|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/01_GenomeScope2.md)|
|---|
