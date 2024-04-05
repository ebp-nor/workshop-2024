## The study organism
For the genome annotation exercise we will use [_Umbelopsis ramanniana_](https://en.wikipedia.org/wiki/Umbelopsis_ramanniana), a fungus common in soil. It is one of the species we are working on in the EBP-Nor project, and has a small genome which is perfect for this workshop, as everything will run quicker. This species is also one of the species used in the comparative genomics part of the worksop.

The approach you will use here over the next pages is derived from the [Snakemake pipeline we use at EBP-Nor](https://github.com/ebp-nor/GenomeAnnotation). It is a bit simplified and has been split up into a series of scripts instead of one large Snakemake file.

## How to use this material
The material for this workshop is written in a way that you can work through it independently of any other resource. The explanations should suffice to understand the material, but contains links to other resources (e.g. scientific papers and/or websites) where you can learn more if you are interested. These links are not necessary to complete the workshop, but are included as reference material that you can use to learn more about the software and concepts used during the workshop. 
Please read both the text and the scripts carefully. If you want to see what a certain program does, you can generally try running it without any arguments. This will in general show you information about the program, and its parameters. Please don't hesitate to ask questions if you want to know more, or if anything is unclear.

## Package management

Administrating the different programs that are needed in a project can be a hassle. Conda/Mamba, especially [miniforge](https://github.com/conda-forge/miniforge), is a very user-friendly package/software manager that makes it easy to install and update different packages. We have already set up the different environments (containing all the necessary software) necessary for this workshop. [Bioconda](https://bioconda.github.io) is a repository that contains a lot of different packages that are relevant for this workshop, and for genomics and bioinformatics in general.

To load conda, run the following code in your terminal:
```
eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 
```

Note:
There are some programs that are not available through (bio)conda. For most of these we use [Singularity containers](https://docs.sylabs.io/guides/3.5/user-guide/introduction.html) instead. 

We mostly set up scripts and arranged the data so that it is ready to use for you, but in some steps you will be asked to modify them. We have backups of everything in case things go wrong, but please be careful so you don't delete something you shouldn't.

## Infrastructure

For the different analyses we are doing, we will use [Saga](https://documentation.sigma2.no/hpc_machines/saga.html). Everything should be set up properly by now. The project we have at Saga is called nn9984k, and the working folder is `/cluster/projects/nn9984k`. You should set up and do stuff in `/cluster/projects/nn9984k/work/$USERNAME`, but we'll come back to that in the next subject.

On Saga we will submit jobs/analyses as job scripts. This is for a system called SLURM. Basically, this is instructions to the system for what kind of analysis we are running, or more concretely, how much memory and computing power we need. 

A generic job script might look like this (copied from [Saga](https://documentation.sigma2.no/hpc_machines/saga.html)):
```
#SBATCH --account=MyProject
#SBATCH --job-name=MyJob
#SBATCH --time=1-0:0:0
#SBATCH --mem-per-cpu=3G
#SBATCH --ntasks=16

Some commands.
```


|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day2_genome_annotation/01_repeatmasking.md)|
|---|
