## Masking repeats
When annotating a genome assembly, the first step is usually to mask the repeats. This can be thought of as a two-step process. First you need to find the repeats, and then you need to "mask" them. 'Repeats' in this context means all different kinds of sequences that are repeated across a genome. This might be interspered repeats such as transposable elements which spread themselves across a genome, or tandem repeats, such as short tandem repeats which consists of a motif repeated in tandem (for instance 'AC' repeated six times: ACACACACACAC). There are multiple reasons for masking repeats, but the two main ones are to reduce the search space for downstream analysis (as in general we're not interested in what is in the repeated sequences), and to avoid annotating genes found in transposable elements as the genes of the host. Arguably, it could be interesting to actually annotate the genes in transposable elements properly, so to better characterize them, but this is usually outside the scope of a genome annotation project. 

The actual masking itself can be done either "hard" or "soft". Hard masking entails replacing the detected repeats with either 'X' or 'N', thereby hiding the sequence itself and making it unavailable for downstream analyses. Soft masking means replacing the nucleotides of detected repeats by their lowercase equivalant (e.g. replace "A" with "a"), as nucleotides in fasta files are usually all in capital letters (ACTG). This makes it possible for different tools to recognise repeats (because they are in lowercase) and avoid creating alignments in these repeats, but alignments can be extended into the repeats if necessary, because the actual sequence is still accessible. However, some tools, such as [miniprot](https://github.com/lh3/miniprot), ignore soft masked bases and treat them as normal. Thus it is important to check how the software you're using handles repeat masking!

[RepeatMasker](https://www.repeatmasker.org/) is a widely used tool for masking repeats. First, a species/genome-specific repeat library is created using [RepeatModeler](http://www.repeatmasker.org/RepeatModeler/) ([Flynn et al (2020)](https://doi.org/10.1073/pnas.1921046117)), and afterwards classifying the repeats, for example into different classes of transposable elements. While an excellent tool, Repeatmasker has the downside of being quite slow.

Recently, faster repeatmasking software has been developed. However, these only identify and mask repeats, but do not classify them. Thus, they are only useful in case you are not interested in what the repeats are, and just in where there are. Here, we go for a program that uses a lot less resources, called Red ([Giris 2015](https://doi.org/10.1186/s12859-015-0654-5)). Red and an associated Python script [redmask.py](https://github.com/nextgenusfs/redmask) are already set up on Saga using conda and by downloading the redmask GitHub repository. You do not need to do anything yourself.  We will also run RepeatModeler and RepeatMasker. 

In order to keep your analyses tidy, and to make it easier to help you out if things go wrong, have to set up a user-specific working directory in `/cluster/projects/nn9984k/work/`. This can be done by running the following command:
```
mkdir -p /cluster/projects/nn9984k/work/$USER
```

Then, navigate to this location (`cd /cluster/projects/nn9984k/work/$USER`) and create a subfolder called 'annotation' and enter that. This will be our working area for now. 

Create a subfolder called 'softmask', and enter it. 

To run repeatmasking using Red, we created a script that looks like this:
```
#!/bin/bash
#SBATCH --job-name=red
#SBATCH --account=nn9984k
#SBATCH --time=1:0:0
#SBATCH --mem-per-cpu=10G
#SBATCH --ntasks-per-node=1

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate anno_pipeline

python /cluster/projects/nn9984k/opt/redmask/redmask.py -i ${1} -o ${2}
```
To set up the necessary data, and to submit the script to the server, you have to create a file called `run.sh` in your current folder (`/projects/ec146/work/$USER/annotation/softmask` presumably). In this file, you need to store the following lines of code: (for example using `nano` or `cat` for instance):
```
/cluster/projects/nn9984k/data/annotation/gzUmbRama1.contigs.fasta
sbatch /cluster/projects/nn9984k/scripts/annotation/run_red.sh gzUmbRama1.contigs.fasta gzUmbRama1
```

Then, you can run this script by typing `sh run.sh`. This will execute the lines of code in the file, and submit the Redmaks job to the cluster.

This should finish in a handful of minutes (when testing it ran for 0.5 minutes). You can monitor the progress with `squeue -u <username>`, or using `sacct`.

|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day2_genome_annotation/00_introduction.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day2_genome_annotation/02_miniprot.md)|
|---|---|
