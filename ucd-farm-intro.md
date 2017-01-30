
Submitting scripts as jobs on HPC clusters:
===========================================

* Many people use hpc cluster systems to analyze genomics-y type data.
* There are several hpc cluster systems at UC Davis. I use [farm](https://wiki.cse.ucdavis.edu/support/systems/farm) (College of Agricultural and Environmental Sciences), so we will use that today. 
* farm uses the [slurm workload management scheduling system](https://slurm.schedmd.com/sbatch.html)
* Modules are pre-installed software programs for you to use
* Try not to run programs on the head node (where everyone is logged in), especially if it might take more than 5-10 min or use a lot of memory. This slows things for other users and someone might get mad. We don't want that to happen.

So, let's log in to a node just for us:

```
srun -p high -t 24:00:00 --mem=20000 --pty bash
```

This starts an interactively-running job with 20 GB RAM for 24 hrs.

To see all the software modules available on the cluster:

```
module avail
```

Let's load the `bio` module with commonly-used genomics programs pre-installed:

```
module load bio/1.0
```

Use this to see what programs were loaded with this module (which appears to be a py2.7 [conda](https://conda.io/docs/using/using.html) environment):

```
conda list
```

Submit scripts as slurm jobs so that they will run while you can walk away from the computer. To get coffee! Or whatever. The scrolling output you would normally see on the screen will be automatically saved to slurm output files for you to review later.

With nano (or your editor of choice), make a new file `count_reads.sh` and use this as a template script. 

```
#!/bin/bash
#SBATCH -D /home/ljcohen/adv-unix-workshop/data/MiSeq/farm/
#SBATCH -J insert_descriptive_job_name_here
#SBATCH -t 2:00:00
#SBATCH -N 1
#SBATCH -n 1
#SBATCH -p high
#SBATCH -c 4
#SBATCH --mem=16000

# load modules 
module load bio/1.0

# change directory to where the files are
cd ../

# counts the number of reads in each file
for i in *.fq
do
  echo $i
  grep -c "^@M00967" $i
  fastqc $i
done
```

This script tells the slurm management system to request 16 GB RAM on 1 node with 4 processors for 2 hrs at high priority.

To run this script, submit it to slurm with this command:

```
sbatch count_reads.sh
```

A confirmation message like this should appear, telling you that the job was submitted: 

```
Submitted batch job 10670199
```

After the job finished, it will produce an output file named with the job ID, e.g. `slurm-10670199.out`. To inspect the status of the job, type this:

```
squeue -u ljcohen
```

or to monitor jobs for a long time you can use this:

```
watch squeue -u ljcohen
```

Check the output, what do you see?

```
cat slurm-10670198.out 
```

## Other info:
* Send Terri Knight or Bill Broadly email for help/questions: [help@cse.ucdavis.edu](help@cse.ucdavis.edu)
* Additional software programs can be installed in your home directory without root privaleges. (Or you can ask admin to install a new module for you.)
* The `~/.bashrc` file contains [$PATH](http://unix.stackexchange.com/questions/26047/how-to-correctly-add-a-path-to-path) variables. If you installed software and it's not running the next time you log in, try activating it with this:

```
source ~/.bashrc
```

## References:

* [Node info for farm](http://stats.cse.ucdavis.edu/ganglia/?c=Agri&m=load_one&r=hour&s=descending&hc=4&mc=2)
* https://wiki.cse.ucdavis.edu/support/systems/farm
* https://github.com/WhiteheadLab/Lab_Wiki/wiki/Using-the-farm-cluster
* https://github.com/RILAB/lab-docs/wiki/Using-Farm
