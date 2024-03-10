Understanding the impact of adaptive pressures on genome evolution is a fundamental goal in biology. Natural Selection primarily operates on the phenotype, but its traces can be discerned at the genetic level, manifesting as signatures of selection. One widely employed approach to identify these signatures is the dN/dS ratio analysis.

dN measures the substitution rate for nonsynonymous mutations, which can directly alter protein function, while dS quantifies the rate for synonymous mutations, which do not affect amino acid sequence. Departures from the expected neutral dN/dS ratio are considered indicative of selection.

A ratio greater than 1 suggests positive selection, where nonsynonymous substitutions accumulate at a faster rate than synonymous ones, implying adaptive evolution. A ratio close to 1 indicates neutral evolution, where both types of mutations evolve at similar rates. Conversely, a ratio less than 1 implies purifying selection, where nonsynonymous substitutions accumulate more slowly than synonymous ones, suggesting the removal of deleterious mutations.

For our analyses we will utilize HyPhy, a comprehensive tool that employs phylogenetic trees and alignments to unravel signatures at the sequence-level. Within HyPhy, we will employ abSREL, a branch-site method specifically designed to detect lineage-specific evolution.

abSREL investigates whether a particular lineage or lineages have undergone selection, categorizing them as "branch models." It assesses whether a fraction of sites along specific branches or lineages exhibit positive selection. abSREL is particularly well-suited for:
Exploratory testing for lineage-specific positive selection in sequences up to 100.
Evaluating branches selected a priori.

Note that abSREL does not explicitly test for purifying selection, as the null model and the alternative model provide identical performance under purifying conditions. Consequently, the resulting P-value is 1.

To identify lineage-specific diversifying selection, abSREL compares the full model to a nested null model, utilizing a likelihood ratio test to determine statistical significance. Furthermore, it employs Bonferroni-Holm procedures to correct P-values obtained from individual tests for multiple comparisons, mitigating the risk of false positives.


### 00 - Let's begin by setting up our environment:
```
eval "$(/fp/projects01/ec146/miniconda3/bin/conda shell.bash hook)" 
conda activate selection

# How to create an environment:
# conda create -n selection
# conda activate selection
# conda install -c bioconda hyphy
# conda install -c bioconda zorro
# conda install -c bioconda prank

```

### 01 - first let's move to the right folder and make a proper folder structure
```
### Copy and paste the following: We make an environment at our work space (first command), move there (second command) and create a folder structure (third command)
mkdir -p /cluster/work/projects/ec146/dnds/$USER
cd /cluster/work/projects/ec146/dnds/$USER
mkdir 00_data  01_alignments  02_zorro  03_tree  04_absrel
```

### 02 - Now, let's fetch the data we need.
```
cp /projects/ec146/data/dnds/*fa ./00_data

# You can take a look at it:
less ./00_data/*fa
```

### 03 - Let's make an alignment with Prank
```
# This command uses prank to align, moves the file to the right location, and removes the tmp folder
prank -d=00_data/OG0000246.fa -o=OG0000246 -codon -verbose; mv OG0000246.best.fas 01_alignments/OG0000246.fai; rm -r tmp*

# Notice we have another OG. "OG0000251.fa". Run prank with the same command but with the 2nd OG
prank -d=00_data/OG0000251.fa -o=OG0000251 -codon -verbose; mv OG0000251.best.fas 01_alignments/OG0000251.fai; rm -r tmp*
```

### 04 - We need to certify the alignments are high-quality. We use Zorro for this.
```
# This command runs zorro on both alingment and estimates a alignment quality threshold. From the abstract: "Here we describe ZORRO, a probabilistic masking program that accounts for alignment uncertainty by assigning confidence scores to each alignment position.".
for i in 01_alignments/*fai; do zorro $i > 02_zorro/${i#01_alignments/}.zorro; done

# To see an average score (I usually go with > 5)
for i in 02_zorro/*zorro; do echo $i; awk '{ sum += $1 } END { if (NR > 0) print sum / NR }' $i; done
```

### 05 - We need a tree for hyphy.
```
# The second command just moves things.
for i in 01_alignments/*fai; do iqtree2 -s $i -B 1000 -T 1; mv ${i}.* ./03_tree; done
```

### 06 - One of the sequences has a remove codon (OG0000251.fai)
```
cd 01_alignments
hyphy rmv
# Now you'll get a screen. You'll have to click "1" for Universal code, copy and paste the file name "OG0000251.fai", and specify we only want to remove stop codons "1", and save the file "OG0000251.fai"
cd ..
```
## 07 - We are finally ready for hyphy
```
for i in OG0000246.fai  OG0000251.fai; do hyphy absrel --alignment 01_alignments/$i --tree 03_tree/${i}.treefile > 04_absrel/${i%.fai}.hyphy; echo "$i done"; mv 01_alignments/${i}.ABSREL.json 04_absrel/${i}.ABSREL.json; done

# Explore the output and we will discuss it later.
```
