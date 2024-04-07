# PretextView tutorial

There are many ways to view a Hi-C contact map, but today we are going to use PretextView. To download the PretextView desktop application, click [here](https://github.com/wtsi-hpag/PretextView/releases), and pick a release that is suitable for your laptop. Some of the text below might be a bit outdated, but we have tried to address it where we have come across it. In addition to the text, take a look at [https://github.com/Nadolina/Rapid-curation-2.0](https://github.com/Nadolina/Rapid-curation-2.0), and make sure to tag your chromosomes.

If you are unaware of how many chromosomes you do expect, you can search up your species on [https://goat.genomehubs.org/](https://goat.genomehubs.org/). This is a really useful website also for genome size and other characteristics of the genome. It might not always be clear exactly how many chromosomes you have based on the Hi-C contact map. Ideally, it is quite clear, but the real world is not always ideal.

## How to curate your assemblies

### Step 1: Tweaking your settings

 To start the curation process, open the PretextView desktop application, click **load map**, and navigate to where you saved your pretext files. 
 
 When you load your Hi-C contact map, this is what it should look like, depending on which colour scheme you have chosen. For this tutorial we will be using **"Blue-Orange Divergent"**.  
 
 <img width="962" alt="Screenshot 2023-02-06 at 13 43 38" src="https://user-images.githubusercontent.com/110542053/216974988-510ca53d-a3f9-4d84-8a0f-77ce8d53057a.png">

 
Here each square represents a scaffold (which after curation will hopefully all be of chromosome length). The red diagonal line shows where the strongest contact signals are between the DNA sequences. 

Go ahead and look at the extensions for your contact maps. These overlays makes it easier to figure out where the unplaced and wrongly oriented scaffolds are supposed to go. 

For this tutorial, turn on the **Gaps** extension, and turn the **Gamma Min** and **Gamma Mid** sliders down to zero, and the **Gamma Max** slider all the way up. This will make it easier to see where there are gaps in the assembly, and increase the contrast so the contact signals will be easier to interpret. 

 ### Step 2: Moving scaffolds in PretextView

For this step you´ll need a computer mouse. Before you do your first edit, try to orient yourself in the Hi-C contact map. Pressing **U** removes the main menu. Don´t worry, pressing **U** again brings it right back if you want to change any settings. 

Try to move around in the contact map. Scrolling the mouse wheel zooms you in and out. You´ll notice that the zoom is not affected by where your cursor is, so how do we zoom in on one particular scaffold? By dragging the map and placing the area you want to look at in the centre of the screen. To drag the map, click and hold the right mouse button, and move the mouse around. Move around the map for a bit, and look at the scaffolds. Do some look fragmented? Move to the far right bottom corner of the contact map. Are there many smaller scaffolds there?

When your comfortable with this movement, let's try to bring up some of the other menus! Pressing **E** activates "Edit mode". When this mode is active, the cursor changes. Do you see that the further away from the red diagonal you move, the larger an area of the scaffold is marked by green? This green indicator shows which part of the scaffold you are picking up. Try to make an edit! Cut out a chunk of a scaffold and move it someplace else in the contact map. Don´t worry about whether the edit is correct, we'll delete all the test edits before we start the proper curation process. 

Move to the far right of the contact map. Are there any smaller, unplaced scaffolds with clear, red contact signals that you think would go well in any of the larger scaffolds? Hover over them, press the space bar, and move the cursor without clicking the left button. When you have oriented yourself to where you want to place the unplaced scaffold, click the left cursor. If the piece fits best on the end of one of the larger squares, press **S** while in Edit mode to toggle the Snap function. Do you notice that the unplaced scaffold "travels" differently across the contact map, in a skipping motion? This lets you "snap" the unplaced scaffold in place at the end of the larger scaffolds. 

Now that you know how to move around and make edits in PretextView, delete all your edits with **Q** while in Edit mode, and press **U** to check to see that all your test edits are gone. You are now ready to edit the assembly for real!

Here are some different scenarios that you may encounter:

#### "I want to add an unplaced scaffold to the end of one of the larger, chromosome sized scaffolds"

No worries, just enter Edit mode, toggle the snap function, and snap the unplaced scaffold onto the end of the larger chromosome sized scaffold. 

#### "Some of my unplaced scaffolds have ambiguous contact signals, what do I do?"

Ask us for help! You can also read more about these ambiguous signals in GRIT´s documentation [here](https://gitlab.com/wtsi-grit/rapid-curation/-/blob/main/PretextView%20-%20Tutorial.pdf). 

#### "I want to make edits within one of the larger scaffolds, what do I do"

In some instances you'll want to invert segments in your chromosome sized scaffolds. If there are gaps in the breakpoints where you want to make the inversion, just cut where the gaps are and invert. However, if there are no gaps, do not make the edit. To invert segments, press spacebar again after you have picked up the segment you want to invert. 

### Step 3: Painting your scaffolds

You have made your edits, and now you hopefully have eight large scaffolds matching *Athalia rosae's* karyotype. Before finishing your assembly, you need to "paint" them. What does this mean? You need to mark which scaffolds are part of the same "super-scaffolds" or chromosomes, so they´ll all have the same name in the final FASTA. 

Before painting the scaffolds, tag them with either Hap_1 or Hap_2 as mention on [https://github.com/Nadolina/Rapid-curation-2.0](https://github.com/Nadolina/Rapid-curation-2.0). Enter tagging mode by pressing **M**, and scroll through the tags by right and left arrows.

Enter the "Scaffold Edit Mode" by pressing **S**. Go to the bottom right corner, and left click on the smallest scaffold you want to include as a chromosome. 

While clicking, hold **A**, and drag in a diagonal line (following the contact signal) till you reach the end of the chromosome. Repeat until you have reached the end in the left top corner, and have painted all the scaffolds. 

### Step 4: Finishing your assembly

To finish your assembly you need to:

#### 1. Create an AGP file

Press **U** to bring up the main menu. Press the "Generate AGP" button, and create a out.pretext.agp file. 

#### 2. Transfer your out.pretext.agp file to Saga

Bring the AGP file back to your Saga working directory by opening another terminal window. When you are in the right location, modify the command below to copy your file:

```
scp -r out.pretext.agp <username>@saga.sigma2.no:/cluster/projects/nn9984k/work/<USERNAME>/assembly/curation
```

Enter your password and the file will be transferred to the directory. 

#### 3. Create new FASTA files from the original fasta file and AGP file

Create a script like the one below (you could call it `process_agp.sh` or similiar). You might have to change some text depending on what you used as input files.

```
cat iyAthRosa_30x.pretext.agp_1 |sed "s/H1_/H1\./g" |sed "s/H2_/H2\./g" > out.pretext.agp_1.fix

cat data/ref.fa |sed "s/H1_/H1\./g" |sed "s/H2_/H2\./g" > ref.fix.fa

eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate curation

sh /cluster/projects/nn9984k/opt/Rapid-curation-2.0/curation_2.0_pipe.sh -f ref.fix.fa -a out.pretext.agp_1.fix
```
You can then run mashmap to map the two haps towards each other, and rename the entries in hap_2 (you could add this to the script above):

```
eval "$(/cluster/projects/nn9984k/miniforge3/bin/conda shell.bash hook)" 

conda activate curation

sh /cluster/projects/nn9984k/opt/Rapid-curation-2.0/hap2_hap1_ID_mapping.sh Hap_1/hap.chr_level.fa Hap_2/hap.chr_level.fa

# Update names in haplotype 2 to match haplotype 1
ruby /cluster/projects/nn9984k/opt/Rapid-curation-2.0/update_mapping.rb --fasta Hap_2/hap.chr_level.fa \
--mashmap_table hap2_hap1.tsv > Hap_2/hap.chr_level_renamed.fa

```

And now you are left with a complete, curated, haplotype resolved whole-genome assembly! Congratulations, you champ!

![leo_gif](https://user-images.githubusercontent.com/110542053/206199166-141c2f3b-2f9c-42a4-913f-62e3913511fa.gif)


|[Previous](https://github.com/ebp-nor/workshop-2024/blob/main/day1_genome_assembly/10_Rapid_curation.md)|
|---|
