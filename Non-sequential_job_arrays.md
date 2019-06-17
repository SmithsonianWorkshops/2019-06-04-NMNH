# How to create a job array if you have non-sequential numbers in your files or folders

This mini-tutorial is heavily based on our [Hydra wiki page](https://confluence.si.edu/display/HPC/High+Performance+Computing). 
Check out [this page](https://confluence.si.edu/display/HPC/Job+Arrays) for additional info.

In this example, I'm creating a job array to run AUGUSTUS, but this idea can be applied to any other program. If your files/folders are in sequential order, you can use the script below to generate all the job files you need. 

### Script to create job files with tasks (sequential numbers)

```
#!/bin/sh
#the numbers after seq correspond to: start, increment, max number
for start in `seq -f "%.0f" 1 4999 79000`
    do
    echo '# /bin/sh 
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -q mThM.q
#$ -l mres=20G,h_data=20G,h_vmem=20G,himem
#$ -t '$start'-'$(($start+4999))' -tc 100 
#$ -cwd
#$ -j y
#$ -N augustus
#$ -o augustus_$TASK_ID.log
#
# ----------------Modules------------------------- #
module load bioinformatics/augustus/3.3
module load bioinformatics/bioperl
#
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
echo + NSLOTS = $NSLOTS
#
export AUGUSTUS_CONFIG_PATH="/pool/genomics/username/augustus/config"
#
augustus --strand=both --singlestrand=true \
--hintsfile=scaffold_$SGE_TASK_ID/hints_RM.E_renamed.gff \
--extrinsicCfgFile=extrinsic.M.RM.E.cfg \
--alternatives-from-evidence=true \
--gff3=on \
--uniqueGeneId=true \
--softmasking=1 \
--species=myspecies \
scaffold_$SGE_TASK_ID/assembly.fa.masked > augustus_output/augustus_$SGE_TASK_ID.gff
#
echo = `date` job $JOB_NAME done
#' &> "augustus_$start-$(($start+4999)).job"
done
```

This will create 16 job files, with 5000 tasks each (except for the last one, which will have 4900 tasks). One hundred tasks will be run at a time (`-tc 100`). 

**Important points:**  
- The max number of tasks you can run in each job is 5000 (that's our system configuration). 


### How about non-sequential numbers?

Let's say you had to trim your assembly to get rid of small scaffolds before annotation, and now your scaffolds are not in sequential order anymore. If you use the method described above, the system will try to run every task, even those that don't really exist. 

There are two potential alternatives:

1. Rename the scaffolds of your trimmed assembly so that their names are in sequential order again. We posted about this [here](https://github.com/SmithsonianWorkshops/2019-06-04-NMNH/blob/master/renaming_scaffolds.md).
2. But what if you already moved on with the other analyses (BUSCO, RepeatMasker, etc)? 


Here's how to use job arrays if that's you case.

#### 1. Create a list of files/directories

Here I'm assuming your folder names start with scaffold (in reference to the AUGUSTUS portion in the [genome annotation tutorial](https://github.com/SmithsonianWorkshops/2019-06-04-NMNH/blob/master/GenomeAnnotationGuide.md))

**If you have a small (<10,000) number of files**. You can restrict the number of lines using sed and tail. In this case, we had “only” 8311 folders, and we divided them into two lists, one with 4000 lines (folders) and other with 4311. 

```
ls -v -d scaffold_* | sed -n -e '1,4000p' > list1.txt
ls -v -d scaffold_* | tail -n +4001 > list2.txt
``` 
 
**If you have tens of thousands of scaffolds**. Use the `split` command to automatically generate lists with up to 5000 lines each. The lists will be named `xaa`, `xab`, `xac`… You can rename them if you want, but it’s not necessary.

```
ls -v -d scaffold_* | split -l 5000
``` 
 

#### 2. Create a file with program information

This file looks like the bottom part of a job file, without the qsub parameters. We will create later another file with that "missing" part. The file below is pulling information from the list `xaa` and using that as input for the run. 
Also, there's an additional line before listing the modules to load. 

```
#!/bin/sh
#
# ----------------Modules------------------------- #
source /etc/profile.d/modules.sh
module load bioinformatics/augustus/3.3
#
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
#
export AUGUSTUS_CONFIG_PATH="/pool/genomics/username/GAworkshop/augustus/config"
#
i=$SGE_TASK_ID
P=`awk "NR==$i" xaa`
#
echo $P
#
augustus --strand=both --singlestrand=true \
--hintsfile=${P}/hints_RM_E.gff3 \
--extrinsicCfgFile=extrinsic.M.RM.E.cfg \
--alternatives-from-evidence=true \
--gff3=on \
--uniqueGeneId=true \
--softmasking=1 \
--species=myspecies \
${P}/genome.fa.masked > ../output/augustus_${P}.gff
#
echo = `date` job $JOB_NAME done
#
```

Create the file and save is as a script (`.sh`). You also need to make this file executable using `chmod`.

`chmod +x augustus.sh`


#### 3. Create a file with qsub parameters.

```
qsub \
-q mThC.q \
-l mres=2G,h_data=2G,h_vmem=2G \
-cwd -j y -N augustus \
-t 1-4000 -tc 250 \
-o 'augustus_$TASK_ID.log' \
-b y $PWD/augustus.sh
```
I saved this file with the name `qsub.job`.

This job will run a total of 4000 tasks, 250 at a time. We are using the `-b` flag, which specifies the script to be used. In this case, we are placing the script and the qsub parameter file in the same folder.  

**To submit the jobs:** you will use a different command.

`source qsub.job`



#### Shortcut:

If you have many file lists, you can use the script below to create the script and job configuration files. Save as a file, and run it from the folder where you have your file lists.

(I know it isn't pretty, but it gets the job done. Please feel free to add suggestions/improvements! :) )

```
#!/bin/sh
for f in xa*
    do
    echo '#!/bin/sh
#
# ----------------Modules------------------------- #
source /etc/profile.d/modules.sh
module load bioinformatics/augustus/3.3
#
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
#
export AUGUSTUS_CONFIG_PATH="/pool/genomics/username/GAworkshop/augustus/config"
#
i=$SGE_TASK_ID
P=`awk "NR==$i" '$f'`
#
echo $P
#
augustus --strand=both --singlestrand=true \
--hintsfile=${P}/hints_RM_E.gff3 \
--extrinsicCfgFile=extrinsic.M.RM.E.cfg \
--alternatives-from-evidence=true \
--gff3=on \
--uniqueGeneId=true \
--softmasking=1 \
--species=myspecies \
${P}/genome.fa.masked > ../output/augustus_${P}.gff
#
echo = `date` job $JOB_NAME done
#' &> "augustus_$f.sh"
done

for f in xa*
	do 
	echo 'qsub \
-q mThC.q \
-l mres=2G,h_data=2G,h_vmem=2G \
-cwd -j y -N augustus \
-t 1-20 -tc 20 \
-o '\''augustus_$TASK_ID.log'\'' \
-b y $PWD/augustus_'$f'.sh' &> qsub_$f.job
done
```

To submit multiple files, you would do a simple loop:

`for f in qsub_xa*.job; do source $f; done`

Don't forget to make all scripts executable before submitting the jobs:

`chmod +x *.sh`

#### Things that can be improved:

- Save the scripts and job conf files in the folder jobs and run everything from there. 
- Output the log files to the logs folder

 
