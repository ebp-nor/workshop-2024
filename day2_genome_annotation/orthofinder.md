
# Orthofinder

[OrthoFinder](https://github.com/davidemms/OrthoFinder) is an easy, fast and great tool to study the evolutionary relationship between genes, and identify gene duplications. It also provides a species tree, as well as gene trees. 

![image](https://user-images.githubusercontent.com/46928237/191227132-3227f638-abd4-4804-9b91-fbc080c905d9.png)


## What are orthologs? Paralogs? Homologs? 
Homology is similarity due to shared ancestry between a pair of structures or genes in different taxa. If two genes share ancestry following a speciation event, they are known as orthologs. If two genes trace their ancestry back to a gene duplication event, they are known as paralogs.

![image](https://user-images.githubusercontent.com/46928237/193239122-33223055-afc8-4f47-91a1-3d341e18535f.png)

OrthoFinder identifies something they call orthogroups. An orthogroup is the set of genes derived from a single gene in the last common ancestor of all the species under consideration.

## How to run Orthofinder on a cluster

We will go through how to run OrthoFinder on a cluster. The work flow is based on this: [Step-by-Step OrthoFinder Tutorials](https://davidemms.github.io/menu/tutorials.html). This is a series of tutorials that go through how to find data and use it to run OrthoFinder. 

They show you how to manually downliad data from different databases. To do this step on the command line, look at tips and tricks in the databases section. 

# Getting the input data for Orthofinder

We will use our own data to run Orthofinder. All you need is a fasta file of all the protein sequences for each species. It is also good to have an outgroup species for your analsys. We have chosen to use Aspergillus.

Create a folder at your work directory:
```
cd /cluster/projects/nn9984k/work/$USER
mkdir -p orthofinder
cd orthofinder
```

# Setting up a conda environment
To set up the conda environment, we did this:
```
eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 
conda create -n orthofinder orthofinder
```
You do not have to do this yourself (please don't, you might mess up something).


# Filtering the data



# Running OrthoFinder

We will first run OrthoFinder on the protein sequences. Create a subfolder called proteins and navigate to it. 

To get the data, copy them like this:
```
rsync -ravz /cluster/projects/nn9984k/data/proteomes/primary_transcripts/*.fa .
```
Then you can submit a job like this:
```
sbatch /cluster/projects/nn9984k/scripts/orthofinder/run_orthofinder_proteins.sh
```

A previous version of the SLURM scripts had specified running `muscle` and `iqtree` (`-M msa -A muscle -T iqtree`). By default, OrthoFinder calls `muscle` like this:
```
muscle -in INPUT -out OUTPUT
```
However, the version of `muscle` we have installed (5.1) requires 
```
muscle -align INPUT -output OUTPUT
```

Luckily, this is easy to change. We just modified this file to the correct command: `/cluster/projects/nn9984k/miniconda3/envs/orthofinder/bin/scripts_of/config.json`. If you run into similar issues when doing this yourself, be aware that it might be easily addressed. Running with the the `muscle` and `iqtree` options took a lot of time, so we did not enable them. However, some of you can try it, but please do not create a lot of jobs. Each OrthoFinder job might create tens of thousands of files, and overload the filesystem we are working on.

## Example script
```
#!/bin/bash

#SBATCH --job-name=orthofinder
#SBATCH --account=nn9984k
#SBATCH --time=4:0:0 ## increase if the job doesn't finish
#SBATCH --mem-per-cpu=4500M
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=5 ## increase if the job doesn't finish

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate orthofinder

#increase -a and -t if the job needs more time
orthofinder -a 5 \
-t 5 \
-f proteins > orthofinder.out 2> orthofinder.err
```
