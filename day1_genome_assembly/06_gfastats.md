# gfastats tutorial

Now it is time to evaluate the assemblies you have created, starting with **gfastats**. This tool summarises some important assembly metrics such as number of contigs/scaffolds/, contig/scaffold N50, and the number of gaps in the assembly. If you want to learn more about gfastats, click [here.](https://github.com/vgl-hub/gfastats). When you are ready, run the code and try to answer the questions below.

## Evaluating the gfastats results

Run this code in the terminal command line on the hifiasm assemblies (iyAthRosa.hic.hap1.p_ctg.fa and iyAthRosa.hic.hap2.p_ctg.fa), on the scaffolded assembly (iyAthRosa_scaffolds_final), and fill out the table below. If your are missing any of the files, or your assemblies ar not done running yet, you can find a copy of the necessary files at `/cluster/projects/nn9984k/data/iyAthRosa1/assembly/` (in subfolders called 22x, 26x, 30x and 22x_scaffolded and 30x_scaffolded. 26x was not scaffolded in time).

```
eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate gfastats

gfastats <your_assembly.fa>
```
The two first lines enable access to the conda system, and the gfastats environment in particular. 


Metric | Value
-------|-------
Number of scaffolds |
Total scaffold length |
Average scaffold length |
Scaffold N50 |
Largest scaffold |
Number of contigs |
Total contig length |
Average contig length |
Contig N50 |
Number of gaps |
Total gap length | 
Average gap length |
Gap N50 |


Why do you think each of these metrics are important for evaluating the quality of your assemblies? Discuss with the person sitting next to you.

Compare the metrics of the different assemblies to each other, and discuss with your neighbour. 

The sequencing data and approach used in this tutorial was chosen to be able to fulfill the criteria/standards for the Earth Biogenome Project. You can review these standards [here](https://www.earthbiogenome.org/assembly-standards). Does this assembly or these assemblies fulfill the N50 contig length criterium?

## Software versions used
```
eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 
conda activate gfastats
conda list
```
gfastats version 1.3.6

|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/05_YaHS.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/07_BUSCO.md)|
|---|---|
