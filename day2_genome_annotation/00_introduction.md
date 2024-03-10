## The study organism
For the genome annotation exercise we will use [_Umbelopsis ramanniana_](https://en.wikipedia.org/wiki/Umbelopsis_ramanniana), a fungus common in soil. It is one of the species we are working on in the EBP-Nor project, and has a small genome which is perfect for this workshop, as everything will run quicker. This species is also one of the species use in the comparative genomics part of the worksop.

## How to use this material
The material for this workshop is written in a way that you can work through it independently of any other resource. The explanations should suffice understanding the material, but contains links to other resources (e.g. scientific papers and/or websites) where you can learn more if you are interested. These links are not necessary to complete the workshop, but are included as reference materaio that you can use to learn more about the software and concepts used during the workshop. 
Please read both the text and the scripts carefully. If you want to see what a certain program does, you can generally try running it without any arguments. This will in general show you information about the program, and its parameters. Please don't hesitate to ask questions if you want to know more, or if anything is unclear.

## Package management

Administrating the different programs that are needed in a project can be a hassle. Conda, especially [miniconda](https://docs.conda.io/en/latest/miniconda.html), is a very user-friendly package/software manager that makes it easy to install and update different packages. We have already set up the different environments (containing all the necessary software) necessary for this workshop. [Bioconda](https://bioconda.github.io) is a repository that contains a lot of different packages that are relevant for this workshop, and for genomics and bioinformatics in general.

To load conda, run the following code in your terminal:
```
eval "$(/fp/projects01/ec146/miniconda3/bin/conda shell.bash hook)" 
```

Note:
There are some programs that are not available through (bio)conda. For most of these we use [Singularity containers](https://docs.sylabs.io/guides/3.5/user-guide/introduction.html) instead. 

We mostly set up scripts and arranged the data so that it is ready to use for you, but in some steps you will be asked to modify them. We have backups of everything in case things go wrong, but please be careful so you don't delete something you shouldn't.

## Infrastructure

During this workshop, we will run our analyses on [Educloud](https://www.uio.no/english/services/it/research/platforms/edu-research/). To use it, you need an account which you can get here: [https://research.educloud.no/register](https://research.educloud.no/register). The project we are using in this course is *ec146*, so please ask for access to that one, and we will let you in. 

We will do the work in this workshop in the folder `/projects/ec146/work` on [Fox](https://www.uio.no/english/services/it/research/platforms/edu-research/help/fox/), which is the HPC part of Educloud. After creating an account, you can log in using `ssh <educloud-username>@fox.educloud.no`. You will be prompted for a One-Time Code for a 2-factor authenticator app (Microsoft Authenticator) and your Fox/Educloud password.

Fox uses SLURM as workload manager. This means we will need to submit jobs/analyses as job scripts. These job scripts provide instructions to the system on what kind of analysis we are running, or more concretely, how much memory and computing power we need. 

A generic job script might look like this (copied from [Job Scripts on Fox](https://www.uio.no/english/services/it/research/platforms/edu-research/help/fox/jobs/job-scripts.md)):
```
#!/bin/bash

# Job name:
#SBATCH --job-name=YourJobname
#
# Project:
#SBATCH --account=ecXXX
#
# Wall time limit:
#SBATCH --time=DD-HH:MM:SS
#
# Other parameters:
#SBATCH ...

## Set up job environment:
set -o errexit  # Exit the script on any error
set -o nounset  # Treat any unset variables as an error

module --quiet purge  # Reset the modules to the system default
module load SomeProgram/SomeVersion
module list

## Do some work:
YourCommands
```

|[Next](https://github.com/ebp-nor/genome_annotation_comparative_genomics_part1/blob/main/01_repeatmasking.md)|
|---|
