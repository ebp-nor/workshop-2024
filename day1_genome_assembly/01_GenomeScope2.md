# GenomeScope2 tutorial

Before we start assembling the genome, it is good to know some basic statistics about our genome. What is the ploidy of the organism that we are assembling? What is the predicted size of the genome? What is the heterozygosity rate and how many repeats are there? All these parameters, and more, can be determined by running **GenomeScope2** on the sequencing reads. For this tutorial we will be running the code below, but for more information about the software, you can go [*here*.](https://github.com/tbenavi1/genomescope2.0) 

## Creating a k-mer profile plot

```
#!/bin/bash
#SBATCH --job-name=genomescope
#SBATCH --account=nn9984k
#SBATCH --time=1:0:0
#SBATCH --mem-per-cpu=10G
#SBATCH --ntasks-per-node=5

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

mamba activate smudgescope

k=32

mkdir -p tmp
echo $1 > FILES
[ -s reads.kmc_suf ] || kmc -k$k -t10 -m38 -ci1 -cs10000 @FILES reads tmp/

[ -s reads.histo ] || kmc_tools transform reads histogram reads.histo -cx10000

genomescope2 -i reads.histo -o output_ploidy1 -k $k -p 1 1> genomescope_ploidy1.out 2> genomescope_ploidy1.err
genomescope2 -i reads.histo -o output_ploidy2 -k $k -p 2 1> genomescope_ploidy2.out 2> genomescope_ploidy2.err
genomescope2 -i reads.histo -o output_ploidy4 -k $k -p 4 1> genomescope_ploidy4.out 2> genomescope_ploidy4.err

```

We have set up this script for you, so you don't need to copy the previous block of code.

In order to keep your analyses tidy, and to make it easier to help you out if things go wrong, you have to first set up a user-specific working directory in `/cluster/projects/nn9984k/work/`. This can be done by running the following command:
```
mkdir -p /cluster/projects/nn9984k/work/$USER
```

Then, navigate to this location (`cd /cluster/projects/nn9984k/work/$USER`) and create a subfolder called 'assembly' (`mkdir assembly`) and enter that folder using the `cd` command: `cd assembly`. Then, create a subfolder called 'genomescope'.

What you need to do is to create a `run.sh` in your working folder (`/cluster/projects/nn9984k/work/$USER/assembly/genomescope`), containing the following content: (you can use `nano` or `cat` for instance):

```
sbatch /cluster/projects/nn9984k/scripts/run_genomescope.sh  /cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/pacbio/ERR6548410_22x.fastq.gz
````

When you have done this, you can run the script, which will submit the genomescope script to the cluster by typing `sh run.sh`.

This anayslis should finish in a handful of minutes (when testing it ran for 5 minutes). You can monitor the progress with `squeue -u <username>`.

## Interpreting your k-mer profile plot

A typical k-mer profile plot will look like this (this is the k-mer profile plot for the river lamprey (*Lampetra fluviatilis*) from EBP-Nor): 

![genomescope_plot](https://user-images.githubusercontent.com/110542053/206213929-8a46e185-2f85-40e0-8331-ba090d3b0c3e.png)

The peak on the extreme left are all the kmers that are the result of sequencing errors. The largest peak to the right indicates the homozygous parts of the genomes that account for identical k-mers between the two haplotypes. The smaller peak to the left are the heterozygous kmers, that differentiate the two haplotypes. The smaller bumps on the right are k-mers derived from repeats.

What have you learned about your reads from the k-mer profile plot you created? To look at the plots you created, open a new terminal window, and navigate to a directory where you want to place your files. When you have found the place where you want to save them, use this command to copy them to your local computer:

```
scp -r <username>@saga.sigma2.no:/cluster/projects/nn9984k/work/<username>/assembly/genomescope/output_ploidy2/*.png .
#in some shells you might need to do this:
#scp -r <username>@saga.sigma2.no:/cluster/projects/nn9984k/work/<username>/assembly/genomescope/output_ploidy2/"*.png" .
```

We also have a set of backups at `/cluster/projects/nn9984k/data/iyAthRosa1/genomescope` which you can look at instead if there were some issues with your job.

(And we have uploaded them here: [linear_plot.png](linear_plot.png), [log_plot.png](log_plot.png), [transformed_linear_plot.png](transformed_linear_plot.png) and [transformed_log_plot.png](transformed_log_plot.png).)

What is the estimated genome size of *Athalia rosae*? 

You can look at the plot [here](genomescope2_linear_plot.png).

## Software versions used
```
eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 
conda activate smudgescope
conda list
```
kmc version 3.2.1

genomescope2 version 2.0

smudgeplot version 0.2.5

|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/00_introduction.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/02_Smudgeplot.md)|
|---|---|
