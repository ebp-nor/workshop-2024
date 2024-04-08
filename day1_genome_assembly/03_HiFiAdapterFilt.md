# HiFiAdapterFilt tutorial

Now that you have learned a bit more about your dataset you are *almost* ready to start the assembly process. Before you can start, you need to remove any remaining adapter sequences that may still be attached to your HiFi reads. To do this, we in EBP-Nor use **HiFiAdapterFilt**. To learn more about how this software works, click [*here*](https://github.com/sheinasim/HiFiAdapterFilt), and to do it yourself, follow the tutorial below.

## Filtering adapter sequences with HiFiAdapterFilt

```
#!/bin/bash
#SBATCH --job-name=hifiadaptfilt
#SBATCH --account=nn9984k
#SBATCH --time=1:0:0
#SBATCH --mem-per-cpu=1G
#SBATCH --ntasks-per-node=5

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate hifiadapterfilt

export PATH=/cluster/projects/nn9984k/opt/HiFiAdapterFilt/:$PATH
export PATH=/cluster/projects/nn9984k/opt/HiFiAdapterFilt/DB:$PATH

pbadapterfilt.sh -t 5
```

We have set up this script for you. What you need to do is to create a run.sh in your working folder (`cluster/projects/nn9984k/work/<username>/assembly/hifiadaptfilt`) with this content (with `nano` for instance):

```
ln -s /cluster/projects/nn9984k/data/iyAthRosa1/genomic_data/pacbio/ERR6548410_22x.fastq.gz .
sbatch /cluster/projects/nn9984k/scripts/run_hifiadaptfilt.sh
```  
When you have done this, you can submit to the cluster by typing `sh run.sh`.

This should finish in a handful of minutes (when testing it ran for about 9 minutes). You can monitor the progress with `squeue -u <username>`.

HiFiAdapterFilt creates several files, for instance the filtered file and a statistics file: 

```
Started on Mon Mar 25 09:19:20 AM CET 2024
For the ERR6548410_22x dataset:
Removing reads containing adapters a minimum of 44 bp in length and 97% match.

Number of ccs reads: 300000
Number of adapter contaminated ccs reads: 146 (0.0486667% of total)
Number of ccs reads retained: 299854 (99.9513% of total)

Finished on Mon Mar 25 09:27:53 AM CET 2024
```

You should have similar content.

We also have a set of backup files at `/cluster/projects/nn9984k/data/iyAthRosa1/hifiadaptfilt` if anything went wrong with your job. 

## Software versions used
```
eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 
conda activate hifiadapterfilt
conda list
```
HiFiAdapterFilt version Second release

blast version 2.15.0

bamtools version 2.5.2

|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/02_Smudgeplot.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/04_hifiasm.md)|
|---|---|
