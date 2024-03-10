
## Infrastructure

For the different analyses we are doing, we will use [Educloud](https://www.uio.no/english/services/it/research/platforms/edu-research/). To use it, you need an account which you can get here: [https://research.educloud.no/register](https://research.educloud.no/register). The project we are using in this course is ec146, so please ask for access to that one, and we will let you in. 

We will do the work in this course at `/projects/ec146/work` on [Fox](https://www.uio.no/english/services/it/research/platforms/edu-research/help/fox/) which is the HPC part of Educloud. After creating an account, you can log in using `ssh <educloud-username>@fox.educloud.no`. You will be prompted for a One-Time Code for a 2-factor authenticator app (Microsoft Authenticator) and your Fox/Educloud password.

On Fox we will submit jobs/analyses as job scripts. This is for a system called SLURM. Basically, this is instructions to the system for what kind of analysis we are running, or more concretely, how much memory and computing power we need. 

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
