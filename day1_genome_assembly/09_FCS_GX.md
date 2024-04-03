# FCS-GX tutorial

Contaminants can end up in your assemblies in various different ways. Maybe someone touched samples without gloves, adding foreign DNA to the sample? Maybe there were symbionts living on the organism when it was sampled? Or maybe the sample was contaminated during the sequencing run? Luckily, there are several genomic decontamination tools available, and the one we use in EBP-Nor is the **NCBI Foreign Contamination Screen (FCS)** tool suite (click [here](https://github.com/ncbi/fcs) to read more, it is [published](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-024-03198-7)). This program suite can identify and remove contaminant sequences, whether it is adaptor sequences, vector contamination or foreign organisms. In today's workshop, we are going to focus on the latter, and below you can find the code to run your own decontamination script. 

## Decontaminating the sawfly assembly

```
#!/bin/bash
#SBATCH --job-name=fcsgx
#SBATCH --account=nn9984k
#SBATCH --partition=bigmem
#SBATCH --time=4:0:0
#SBATCH --mem-per-cpu=50G
#SBATCH --ntasks-per-node=10

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

export SHM_LOC=/cluster/projects/nn9984k/opt/fcs/
export GXDB_LOC=/cluster/projects/nn9984k/opt/fcs/

echo "GX_NUM_CORES=10" > env.txt

PREFIX=${1%.*}

python3 /cluster/projects/nn9984k/opt/fcs/fcs.py \
--image /cluster/projects/nn9984k/opt/fcs/fcs-gx.sif \
screen genome --fasta $1 \
--tax-id $2 \
--out-dir ./gx_out/ \
--gx-db  "${GXDB_LOC}/gxdb"

cat $1 | python3 /cluster/projects/nn9984k/opt/fcs/fcs.py \
--image /cluster/projects/nn9984k/opt/fcs/fcs-gx.sif \
clean genome \
--action-report ./gx_out/*.fcs_gx_report.txt --output ${PREFIX}.decon.fasta --contam-fasta-out contam.fasta

```

As we have done earlier, we have set up this script for you. Create a run.sh in your working folder (`/projects/ec146/work/<username>/assembly/fcsgx`) with this content (with `nano` for instance):

```
sbatch /cluster/projects/nn9984k/scripts/run_fcsgx.sh assembly.fasta \
taxonomy_id
```

You have to modify the run.sh script based on your assembly file and you have to find the NCBI taxonomy ID for *Athalia rosae* and input that. You also have to copy the fasta file, because FCS-GX runs in a Singularity container, and cannot follow soft-linked files.

Unfortunately this program requires a lot of memory to run (["approximately 470 GiB"](https://github.com/ncbi/fcs/wiki/FCS-GX)). If it is given unsufficient memory, the running time can increase by a factor of 10000x. On Saga, there are not that [many nodes](https://documentation.sigma2.no/hpc_machines/saga.html) with a lot of memory. There are 8 so-called bigmem nodes, which should be able to handle multiple jobs each of the script above. However, these are quite heavily used, so it is not certain that we will be able to run our jobs here. If configured properly, it is quite quick (1-30 minutes when testing). 

Due to the limited number of bigmem nodes, we should coordinate this step so only a couple people submit to the cluster. Please let us know when you reached this point before submitting the job.

After running the decontamination script, which foreign contaminants did you find?

If you were unable to run it, you can take a look at the results of a previous run at ` /cluster/projects/nn9984k/data/iyAthRosa1/fcsgx/iyAthRosa_scaffolds_final.37344.taxonomy_22x_h1.rpt` and `/cluster/projects/nn9984k/data/iyAthRosa1/fcsgx/iyAthRosa_scaffolds_final.37344.fcs_gx_report_22x_h1.txt `. 



|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/08_Merqury.md)|[Next](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/10_Rapid_curation.md)|
|---|---|
