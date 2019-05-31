
## Summary of steps:
 <!--ts-->
 * [1. Quick Unix tutorial](#1-quick-unix-tutorial)  
 * [2. Folder structure](#2-folder-structure)
 * [3. Get the assembly stats with assembly-stats](#3-Get-the-assembly-stats-with-assembly-stats)
 * [4. Filter out small scaffolds](#4-Filter-out-small-scaffolds)
 * [5. Run BUSCO using the --long flag](#5-Run-BUSCO-using-the---long-flag)
 * [6. Run RepeatMasker](#6-Run-RepeatMasker)
 * [7. Run BLAT](#7-Run-BLAT)
 * [8. Create Augustus hints](#8-Create-Augustus-hints)
 * [9. Edit the extrinsic file to add only the evidence you have](#9-Edit-the-extrinsic-file-to-add-only-the-evidence-you-have)
 * [10. A brief introduction to LOOPS](#10-A-brief-introduction-to-LOOPS)
 * [11. Run AUGUSTUS](#11-Run-AUGUSTUS)
 * [12. Create a genome browser with JBrowse](#12-Create-a-genome-browser-with-JBrowse)
<!--te-->

## DAY 1

### Quick Unix tutorial
For this workshop, we will need to know some UNIX commands. You might be familiar with them, but let's just do a quick refresher so we are all on the same page. If you have additional questions about a command and its options, you can use `man` to access the manual. Try the command `man ls` to access the manual for the list command. 

- To figure out where you are: `pwd`. This command will print the full path of your current directory.
- To list all all files and directories: `ls`
- To **C**hange **D**irectories: `cd destination`. Like this: `cd /pool/genomics/your_username`
- To create a new folder: `mkdir folder_name`
	- `mkdir` can take multiple arguments, which means it can create multiple folders at the same time. You can type: `mkdir folder1 folder2 folder3` and three folders will be created.
- To copy files: `cp file_to_copy destination`. For example: `cp ref.fasta /pool/genomics/tsuchiyam`
	- To copy directories, you need to use the flag `-r`, like this: `cp -r assembly/ /pool/genomics/tsuchiyam/`  
- To move files or directories: `mv file_to_copy destination`. For example: `mv ref.fasta ./assembly/`. 
	- `mv` can also be used to **rename** files and folders. So, if you want to rename the file `assembly.fa` to `dog_assebmly.fasta`, you would use this command: `mv assembly.fa dog_assembly.fasta`.

**Be careful not to overwritte anything**. If you want to be extra safe, use the flag `-i` (interactive) to prevent you from inadvertently overwritting any files. This flag can be used both for `cp` and `mv`.


- To create jobs: use the Qsub Generation Utility, available [here](https://hydra-4.si.edu/tools/QSubGen/).


### 2. Folder structure

Now let's create our folder structure for this workshop. It's easier to troubleshoot any issues if we are all working within the same framework. First, we will create a folder called `GAworkshop` in your `/pool/genomics/username` folder. 

```
cd /pool/genomics/your_username

mkdir GAworkshop

```

Next, we will change directories to the GAworkshop folder and we will create a several folders that we will use today and tomorrow. Here's the list of folders:

- assembly
- busco
- repeatmasker
- blat
- augustus
- jbrowse
- logs
- jobs



<details><summary>SOLUTION</summary>
<p>

```
mkdir assembly busco repeatmasker blat augustus jbrowse logs jobs

```
If you type the command `tree`, this is what you should see:

```
.  
|__ assembly  
|__ busco  
|__ repeatmasker  
|__ blat  
|__ augustus  
|__ jbrowse  
|__ jobs  
|__ logs  

```

</p>
</details>


If we follow this folder structure, we will have all jobs in the same folder, as well as all log files. Also, the results of each job will be organized by software, which will facilitate finding everything later. 

**Submitting jobs**: With this folder structure, we will save all the job files in the `jobs` folder, but they will be submitted from the software folder.  Like this: 

- If I'm submitting the BUSCO job, I will:   
	a. Create the jobfile in the `jobs` folder. I'll save it as `busco.job`  
	b. `cd` to the folder `busco` (from `jobs`, I would do `cd ../busco`)    
	b. Submit the job:  `qsub ../jobs/busco.job`


### 3. Get the assembly stats with `assembly-stats`

For this workshop, we will use the genome assembly of *Drosophila hydei*, which size is 147 Mb. You can copy the assembly file from `/data/workshops/GAworkshop/Dhydei_genome.fa` (FOR NOW, THE FILE IS IN `/scratch/genomics/tsuchiyam/workshops/GAworkshop/assembly/Dhydei_genome.fa`). You should copy it to your `assembly` folder.

<details><summary>SOLUTION</summary>
<p>

`cp /data/workshops/GAworkshop/Dhydei_genome.fa ./assembly/`

</p>
</details>


The first step will be to get some basic stats about this assembly. We will use the module `assembly_stats` for that, we will run it from the interactive queue (we don't run anything from the login queue).  

To access the interactive queue, type `qrsh`. Now you are logged to a compute node.


---
**Important**   
If you type `pwd`, you will see that you are back to your home directory `/home/username/`. You need to `cd` to the `assembly` folder in the `GAworkshop`. 

<details><summary>SOLUTION</summary>
<p>

`cd /pool/genomics/username/GAworkshop/assembly`

</p>
</details>

---

Now we will load the `assembly_stats` module, and run the `assembly_stats.py` script. 


	module load bioinformatics/assembly_stats/0.1
	assembly_stats.py Dhydei_genome.fa

Your results should look like this:

```
{
  "Contig Stats": {
    "L10": 0,
    "L20": 2,
    "L30": 3,
    "L40": 5,
    "L50": 8,
    "N10": 15867853,
    "N20": 10617722,
    "N30": 10355786,
    "N40": 7178142,
    "N50": 3367132,
    "gc_content": 30.981828480036405,
    "longest": 15867853,
    "mean": 708392.66359447,
    "median": 63369.0,
    "sequence_count": 217,
    "shortest": 3135,
    "total_bps": 153721208
  },
  "Scaffold Stats": {
    "L10": 0,
    "L20": 2,
    "L30": 3,
    "L40": 5,
    "L50": 8,
    "N10": 15867853,
    "N20": 10617722,
    "N30": 10355786,
    "N40": 7178142,
    "N50": 3367132,
    "gc_content": 30.981828480036405,
    "longest": 15867853,
    "mean": 708392.66359447,
    "median": 63369.0,
    "sequence_count": 217,
    "shortest": 3135,
    "total_bps": 153721208
  }
}
```

Alternatively, you can save your results to a file, instead of having them printed to the screen. To do so, you use the following command:

	assembly_stats.py Dhydei_genome.fa > Dhydei_genome.stats
	
If you use the command`cat`, you will see that the output file has the same results:

	cat Dhydei_genome.stats

---
****Important****

From now on, we will run all the other steps as jobs, but it is not possible to submit jobs from the interactive queue. To exit back to the login node, type `exit`	

---

### 4. Filter out small scaffolds (optional)

It's common for assemblies to have very short scaffolds ( <500 bp or 1000 bp) that won't be useful for annotation. This is not our case, since the shortest scaffold we have is 3135 bp. But if you are working on your own assemblies and you see that the shortest scaffold is smaller than 500 bp, you can filter those small scaffolds out. It will speed up your genome annotation process, and in my experience, did not affect the genome size and other stats.

To do so, we will use `bioawk`, which is available as module. This command also can be run on the interactive queue.



```
module load bioinformatics/bioawk
bioawk -c fastx '{ if(length($seq) > 499) { print ">"$name; print $seq }}' input.fasta > filtered.fasta
```

**Observation**: this bioawk command will remove any scaffolds of the specified size an below. Since I want to keep the 500bp scaffolds, I used 499 instead of 500.  
  


### 5. Run BUSCO using the --long flag

BUSCO (Simão et al. 2015; Waterhouse et al. 2017) assesses completeness by searching the genome for a selected set of single copy orthologous genes. There are several databases that can be used with BUSCO and they can be downloaded from here: [https://buscos.ezlab.org](). Let's `cd` to the directory `busco`

**Database:**

For this workshop, we will use the Diptera database. Download it to the folder `02busco` using the command `wget` and extract it.

	wget https://buscos.ezlab.org/datasets/diptera_odb9.tar.gz
	tar -zxf diptera_odb9.tar.gz


According to the BUSCO manual, the `--long` flag turns on Augustus optimization mode for self-training. It can be used as a training set for AUGUSTUS.


##### *** Before running BUSCO, copy the file augustus/config folder to a place where you have writing privileges***
The AUGUSTUS config folder can be found here:  `/share/apps/bioinformatics/augustus/gcc/4.9.2/3.3/config/`. Copy it to the folder `augustus` inside your `GAworkshop` folder.

<details><summary>SOLUTION</summary>
<p>
Assuming you are in the folder GAworkshop:  

`cp -r /share/apps/bioinformatics/augustus/gcc/4.9.2/3.3/config/ augustus/`

</p>
</details>



#### Job file: busco.job

- PE: multi-thread
- Number of CPUs: 4
- Memory: 6G
- Module: `module load bioinformatics/busco/3.0`
- Commands:

```
export AUGUSTUS_CONFIG_PATH="/pool/genomics/username/GAworkshop/augustus/config"
#
run_BUSCO.py --long -o Dhydei -i ../assembly/Dhydei_genome.fa -l diptera_odb9 -c $NSLOTS -m genome
```

##### Explanation:
```
--long: turn on Augustus optimization mode for self-training.
-o: name of the output folder and files
-i: input file (FASTA)
-l: path to the folder containing the database of BUSCOs (the one you downloaded from the BUSCO website).
-c: number of CPUs
-m: mode (options are genome, transcriptome, proteins

```


##### 5a. Using BUSCO output for the AUGUSTUS run (MUST UPDATE THE NAMES)
Copy the folder `run_Dhydei/augustus_output/retraining_parameters` to your folder `augustus/config/species`. Rename the folder with the name of the run (you can find it by looking at the file prefix inside the folder). In this case, we will rename the folder `BUSCO_Dhydei_2188842729`

```
BUSCO_Dhydei_2188842729_exon_probs.pbl
BUSCO_Dhydei_2188842729_igenic_probs.pbl
BUSCO_Dhydei_2188842729_intron_probs.pbl
BUSCO_Dhydei_2188842729_metapars.cfg
BUSCO_Dhydei_2188842729_metapars.cgp.cfg
BUSCO_Dhydei_2188842729_metapars.utr.cfg
BUSCO_Dhydei_2188842729_parameters.cfg
BUSCO_Dhydei_2188842729_weightmatrix.txt

```

<details><summary>HINT</summary>
<p>

---
**From the busco folder**  

```
cp run_Dhydei/augustus_output/retraining_parameters ../augustus/config/species/
cd ../augustus/config/species/
ls retraining_parameters
mv retraining_parameters BUSCO_Dhydei_2188842729
```


---

</p>
</details>


<details><summary>ADDITIONAL INFO</summary>
<p>

---
You can also modify all filenames to match just the species ("Dhydei"). But in this case, you need to rename all files AND replace the current species id `BUSCO_Dhydei_2188842729` by "Dhydei" in all files using *sed*.

In this case, these are the steps we would follow:

- Rename the folder `retraining_parameters` to `Dhydei`  
	`mv retraining_parameters Dhydei`
	
- Rename all files in the folder  
	```
	for f in *; do 
		mv $f ${f/BUSCO_Dhydei_2188842729/Dhydei};
	done
	```
	
- Replace the names in the files using `sed`  
	`sed -i 's/BUSCO_Dhydei_2188842729/Dhydei/g' *`

---

</p>
</details>


### 7. Masking and annotating repetitive elements

From the RepeatMasker manual (Smit, 2013-2015):
> Repeatmasker is a program that screens DNA sequences for interspersed repeats and low complexity DNA sequences.The output of the program is a detailed annotation of the repeats that are present in the query sequence as well as a modified version of the query sequence in which all the annotated repeats have been masked. 

**Important**: `cd` to the `repeatmasker` folder before submitting the job.

We will create a link to our assembly `Dhydei_genome.fa` in the `repeatmasker` folder. Here's the command:

`ln -s ../assembly/Dhydei_genome.fa .`

If you `ls -l` in the `repeatmasker` folder, you should see something like this:

```
total 0
lrwxrwxrwx 1 tsuchiyam tsuchiyam 28 May 31 09:19 Dhydei_genome.fa -> ../assembly/Dhydei_genome.fa
```

#### Job file: repeatmasker.job
- Queue: medium
- PE: multi-thread
- Number of CPUs: 10
- Memory: 4G
- Module: `module load bioinformatics/repeatmasker`
- Commands:

```
RepeatMasker -species drosophila -pa $NSLOTS -xsmall -dir . Dhydei_genome.fa

```
##### Explanation:
```
-species: species/taxonomic group repbase database (browse available species here: 
 https://www.girinst.org/repbase/update/browse.php)
-pa: number of cpus
-xsmall: softmasking (instead of hardmasking with N)
-dir: writes the output to the current directory

```

##### Output files:
- Dhydei_genome.fa.tbl: summary information about the repetitive elements
- Dhydei_genome.fa.masked: hardmasked assembly
- Dhydei_genome.fa.softmasked: softmasked assembly
- Dhydei_genome.fa.out: detailed information about the repetitive elements, including coordinates, repeat type and size.

##### About the species:
- You can use the script `queryTaxonomyDatabase.pl` from the RepeatMasker module to search for your species of interest. 

	`queryTaxonomyDatabase.pl -species cat`


### 7. Run BLAT

BLAT (BLAST-like Alignment Tool, Kent 2002) is a tool that aligns DNA (as well as 6-frame translated DNA or proteins) to DNA, RNA and proteins across different species. The output of this program will also be used as hints for AUGUSTUS.

Since we don't have a transcriptome for this species, we will use one from *Drosophila melanogaster*. Link [here](ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/215/GCF_000001215.4_Release_6_plus_ISO1_MT/GCF_000001215.4_Release_6_plus_ISO1_MT_rna.fna.gz)

<details><summary>HINT</summary>
<p>

**HINT**: Use `wget` to download the file to the `blat` folder.
`wget ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/215/GCF_000001215.4_Release_6_plus_ISO1_MT/GCF_000001215.4_Release_6_plus_ISO1_MT_rna.fna.gz`

</p>
</details>


Also, extract the file using gunzip: 

`gunzip GCF_000001215.4_Release_6_plus_ISO1_MT_rna.fna.gz`

#### Job file: blat.job
- Queue: medium
- PE: serial
- Memory: 2G
- Module: `module load bioinformatics/blat`
- Commands:

```
blat -t=dna -q=rna ../assembly/Dhydei_genome.fa \
GCF_000001215.4_Release_6_plus_ISO1_MT_rna.fna \
Dhydei_blat.psl
```

##### Explanation:
```
-t: target database type (DNA, 6-frame translated DNA or protein)
-q: query database type (DNA, RNA, protein, 6-frame translated DNA or RNA).

```


## DAY 2 (maybe)

Now we will integrate the information from the previous jobs into our AUGUSTUS job. To do that, we first need to create hint files from both RepeatMasker and BLAT. We will run the commands from this step using the **interative** queue (`qrsh`)

Also, don't forget to `cd` to your AUGUSTUS folder after logging to the interactive queue. 

For the sake of keeping things organized, let's create three folders inside your`augustus` folder: `hints`, `output` and `scaffolds`. 

<details><summary>HINT</summary>
<p>
`mkdir hints output scaffolds`

</p>
</details>
 
### 8. Create Augustus hints

#### Creating hints files from RepeatMasker
In this step, we will use the .out file from the RepeatMasker use it to create a hint file for AUGUSTUS. 

- **Use the script `rmOutToGFF3.pl` to convert your .out file into GFF3**

	```
	module load bioinformatics/repeatmasker
	rmOutToGFF3.pl ../repeatmasker/Dhydei_genome.fa.out > hints/Dhydei_RM.gff3
	```

- **Use the script `gff2hints.pl` convert the gff3 into a hints file**
This script can be found here: [http://iubio.bio.indiana.edu/gmod/genogrid/scripts/gff2hints.pl]()


	<details><summary>HINT</summary>
	<p>
	
	---
	**Use `wget` to download it.**
	```wget http://iubio.bio.indiana.edu/gmod/genogrid/scripts/gff2hints.pl```
	
	---
	
	
	</p>
	</details>
	
	
	
	Now run the script on the gff3 file you just created:
	
	`perl gff2hints.pl --in=hints/Dhydei_RM.gff3 --source=RM --out=hints/Dhydei_RM_hints.out`


#### Creating hints files from BLAT
In this step, we will use the .psl file from the BLAT run to create a hint file for AUGUSTUS.

- **Sort the .psl file**
	`cat ../blat/Dhydei_blat.psl | sort -n -k 16,16 | sort -s -k 14,14 > hints/Dhydei_blat_srt.psl`

- **Use the script `blat2hints.pl` from the Augustus 3.3 module**

	```
	module load bioinformatics/augustus/3.3
	blat2hints.pl --in=hints/Dhydei_blat_srt.psl --out=hints/Dhydei_blat_hints.out
	```

#### Merging the hints files:

To merge the hints files from RepeatMasker and BLAT, use the following command: 

`cat Dhydei_RM_hints.out Dhydei_blat_hints.out | sort -n -k 4,4 | sort -n -k 5,5 > Dhydei_hints_RM_E.gff3`


### 9. Augustus extrinsic file

The AUGUSTUS extrinsic file has the information about the sources of evidence that will be used. In our case, we have evidence about repetitive elements (from RepeatMasker) and transcriptome (from BLAT). 

We have created a custom extrinsic file with those two sources of evidence. Copy it from `/data/workshops/GAworkshop/extrinsic.M.RM.E.cfg` to YOUR augustus/config/extrinsic folder. Like this:

`cp /data/workshops/GAworkshop/extrinsic.M.RM.E.cfg config/extrinsic`


<details><summary>EXAMPLE</summary>
<p>

##### Example of extrinsic file with RepeatMasker (RM) and BLAT (E) evidence.

```
# extrinsic information configuration file for AUGUSTUS
# 
# include with --extrinsicCfgFile=filename
# date: 15.4.2015
# Mario Stanke (mario.stanke@uni-greifswald.de)


# source of extrinsic information:
# M manual anchor (required)
# P protein database hit
# E EST/cDNA database hit
# C combined est/protein database hit
# D Dialign
# R retroposed genes
# T transMapped refSeqs
# W wiggle track coverage info from RNA-Seq

[SOURCES]
M RM E

#
# individual_liability: Only unsatisfiable hints are disregarded. By default this flag is not set
# and the whole hint group is disregarded when one hint in it is unsatisfiable.
# 1group1gene: Try to predict a single gene that covers all hints of a given group. This is relevant for
# hint groups with gaps, e.g. when two ESTs, say 5' and 3', from the same clone align nearby.
#
[SOURCE-PARAMETERS]


#   feature        bonus         malus   gradelevelcolumns
#		r+/r-
#
# the gradelevel colums have the following format for each source
# sourcecharacter numscoreclasses boundary    ...  boundary    gradequot  ...  gradequot
# 

[GENERAL]
      start      1          1  M    1  1e+100  RM  1     1    E 1    1
       stop      1          1  M    1  1e+100  RM  1     1    E 1    1
        tss      1          1  M    1  1e+100  RM  1     1    E 1    1
        tts      1          1  M    1  1e+100  RM  1     1    E 1    1
        ass      1      1 0.1  M    1  1e+100  RM  1     1    E 1    1
        dss      1      1 0.1  M    1  1e+100  RM  1     1    E 1    1
   exonpart      1  .992 .985  M    1  1e+100  RM  1     1    E 1  1e2
       exon      1          1  M    1  1e+100  RM  1     1    E 1  1e4
 intronpart      1          1  M    1  1e+100  RM  1     1    E 1    1
     intron      1        .34  M    1  1e+100  RM  1     1    E 1  1e6
    CDSpart      1     1 .985  M    1  1e+100  RM  1     1    E 1    1
        CDS      1          1  M    1  1e+100  RM  1     1    E 1    1
    UTRpart      1     1 .985  M    1  1e+100  RM  1     1    E 1    1
        UTR      1          1  M    1  1e+100  RM  1     1    E 1    1
     irpart      1          1  M    1  1e+100  RM  1     1    E 1    1
nonexonpart      1          1  M    1  1e+100  RM  1     1.15 E 1    1
  genicpart      1          1  M    1  1e+100  RM  1     1    E 1    1

#
# Explanation: see original extrinsic.cfg file
#
```

</p>
</details>


### 10. A brief introduction to LOOPS

We will take a "break" from this pipeline to talk about loops. 
Loops allow you to execute repetitive tasks multiple times with a single command. For example:

Imagine that I have a folder with the following files: 

```
01.txt	02.txt	03.txt	04.txt	01.dat	02.dat
```

And let's say that I want to make a copy of all files with the extension `.txt`
and add the word backup in from of it. What are the options? 

1. I can type each command individually

	```
	cp 01.txt backup_01.txt
	cp 02.txt backup_02.txt
	cp 03.txt backup_03.txt
	cp 04.txt backup_04.txt
	```
	
2. Or I can write a loop, that will iterate over all `txt` files:

	```
	for f in *.txt; do
		cp ${f} backup_${f};
	done
	```

**What does this loop mean?**

- `for`: starts a for loop  
- `f in *.txt`: this statement takes all files that have the extension `.txt` and assign them to the variable of name `f`. You can call your variable anything you want (really, anything). The for loop will iterate over all files, one at a time.
- `do`: before the command, you need to add the word `do`.
- `cp ${f} backup_${f}`: command to be executed. Here we are copying all `txt` files and adding the preffix backup_.
- `done`: closes the loop.

**Why is it important?**

For loops are very useful when you have multiple files to process. We will use a for loop to submit our augustus jobs. 

### 11.AUGUSTUS


For the AUGUSTUS job, we need the following input files:

1. assembly (masked)
2. merged RM and E hints file
3. extrinsic file
4. retraining parameters (from BUSCO)


AUGUSTUS will run serially, one scaffold at a time. In order to speed up the process, we can break the assembly into scaffolds and process them in paralel. To do so, we will use a script from EVM (EVidenceModeller) to split the assembly and the hints file, and create job arrays for AUGUSTUS.

**Partition the assembly into scaffolds** 

EVM (EVidence Modeller, Haas et al. 2008) is a program that combines *ab initio* gene predictions and protein and transcript alignments into weighted consensus gene structures. We will use an EVM script that splits the assembly into folders, with one scaffold per folder plus its corresponding hints file (in gff).

We don't have EVM installed as a module on Hydra, but you can download ([https://github.com/EVidenceModeler/EVidenceModeler/archive/v1.1.1.tar.gz]()) and extract it in your `augustus` folder. The script is in the folder EVmutils. This script runs fast, so we can continue on the interactive queue to run it. 

<details><summary>HINT</summary>
<p>

---

**From your `augustus` folder**  

```
wget https://github.com/EVidenceModeler/EVidenceModeler/archive/v1.1.1.tar.gz
tar -zxf v1.1.1.tar.gz

```
---


</p>
</details>


Now `cd` to `scaffolds` and run the following command from it:

```
module load bioinformatics/bioperl

perl ../EVidenceModeler-1.1.1/EvmUtils/partition_EVM_inputs.pl --genome ../../repeatmasker/Dhydei_genome.fa.softmasked \
     --gene_predictions ../hints/Dhydei_hints_RM_E.gff3 \
     --segmentSize 100000 --overlapSize 10000 \
     --partition_listing partitions_list.out
```

Now you should have many (217 to be exact) folders, each one with one scaffold and its corresponding hints file. They all retained the same name or the original file, and the folders are identified as `scaffold_1`, `scaffold_2`... `scaffold_217`.


Now, let's create our augustus job file.

#### Job file: augustus.job
- Queue: short
- PE: serial
- Memory: 20G
- Module: `module load bioinformatics/augustus/3.3`
- Commands:
	
```
	export AUGUSTUS_CONFIG_PATH="/pool/genomics/username/GAworkshop/augustus/config"
	#
	augustus --strand=both --singlestrand=true \
	--hintsfile=${1}/Dhydei_hints_RM_E.gff3 \
	--extrinsicCfgFile=extrinsic.M.RM.E.cfg \
	--alternatives-from-evidence=true \
	--gff3=on \
	--uniqueGeneId=true \
	--softmasking=1 \
	--species= BUSCO_Dhydei_2188842729 \
	${1}/Dhydei_genome.fa.softmasked > ../output/Dhydei_augustus_${1}.gff
	#
	echo = `date` job $JOB_NAME done
	#
```

Before we submit the job, let's exit from the interactive queue back to the login node by typing `exit`

Now, let's make sure we are in the correct folder. We will submit this job from the folder `scaffolds` (the one that has all the 217 folders).

To submit the job, use the following command (a for loop)

```
for dir in scaffold-*/; do 
 out=${dir/\/}; qsub -N augustus_${out} -o ../../logs/augustus_${out}.log ../../jobs/augustus.job ${out};
done
```


##### Understanding the commands:

- `for dir in scaffold-*/;`: This loop will iterate over all folders that correspond to the pattern (in this case, name starts with scaffold-)
- `do out=${dir/\/};`: this command will create a new variable called `out` (you can give any name you want). This new variable will be based on the variable `$dir`, without the trailing slash "/".
- `qsub -N test_${out} -o ../../logs/augustus_${out}.log ../../jobs/augustus.job ${out}`. Here we will finally submit the job, and some of the job parameters will be overwritten:
	- `-N`: job name. We will give a name that includes the variable that contains the scaffold name.
	- `-o`: log file. Same as the job name, The goal here is to one log file per scaffold.
	- `../../jobs/augustus.job ${out};`: in this part, we are calling on the job file `jobs/augustus.job` to be submitted. We need to include the variable `${out}` for the job to run. In the job file, we did not provide any specific paths (remember that we used the variable `${1}`?). The variable `${1}` in the job corresponds to the variable that comes after the job file (in this case, `${out}`).
- `done`: closes the loop.
- `--hintsfile=${1}/hints_RM.E.gff3`: The variable `${1}` is used to identify each folder.
	-  The same applies to `${1}/Dhydei_genome.fa.softmasked` and the output file `Dhydei_augustus_${1}.gff`.


The job is submitted from the directory where all the `scaffold_n` folders are located, and not from inside each folder.
Also, I'm saving all output files in a separate directory `output` to facilitate post-processing.


#### Combining the results.

Use the script `join_aug_pred.pl` from AUGUSTUS (use the interactive queue `qrsh` and run the commands from the `output` folder).

1. Concatenate all output files from augustus in numerical order:
`cat $(find . name "Dhydei_augustus_*.gff" | sort -V) > Dhydei_augustus.concat`

2. Join the results using the `join_aug_pred.pl`
`cat Dhydei_augustus.concat < join_aug_pred.pl > Dhydei_augustus_all.gff3`

3. Convert the Augustus GFF3 to EVM GFF3 (linear, without the sequences):
`perl ../EVidenceModeler-1.1.1/EvmUtils/misc/augustus_GFF3_to_EVM_GFF3.pl augustus_all.gff3 > Dhydei_augustus_final.gff3 `

### 12. Create a genome browser with JBrowse

Now that we have an annotated genome, we can visualize the assembly and annotations using a genome browser. Today we will show you how to setup a genome browser using JBrowse, and those same files can be used with WebApollo for manual annotation.

1. **prepare-refseqs.pl**: formats the reference sequence to be used with JBrowse
	
	```
	prepare-refseqs.pl \  
	--fasta ../assembly/Dhydei_genome.fa \
	--out ./Dhydei
	```
	
	Obs: fasta can be gzipped.  
	
2. **flatfile-to-json.pl**: format data into JBrowse JSON format from an annotation file

	```
	flatfile-to-json.pl --gff ../augustus/output/Dhydei_augustus_final.gff3 \
	--type CDS \
	--tracklabel CDS \
	--nameAttributes "name,alias,id,gene,product"
	--out ./Dhydei
	
	```

	Observations:  
	`--gff` can't be gzipped, and must be GFF3. In addition, this script will accept `--bed` and `--gbk` (genbank) files as input.  
	`--type` is the 3rd column of the GFF file. Option are: cDNA\_match, CDS, exon, gene, guide\_RNA, lnc\_RNA, mRNA, pseudogene, rRNA, snoRNA, snRNA, transcript, tRNA, V\_gene\_segment.  
	`--tracklabel` should be informative
	`--nameAttributes` are the list of attributes to be saved for each feature. This is important if you want to be able to search by gene name, for example. Default: `"name, alias,id"`

3. **generate-names.pl**: builds a global index of feature names.
	
	```
	generate-names.pl --out ./Dhydei
	```
	Obs: `--out` is the directory to be processed. 

4. **Zip your results**
	
	```
	tar -zcf Dhydei.tar.gz ./Dhydei
	```	

To visualize the results, you need to install JBrowse locally on your laptop. To make things easier, save the JBrowse folder on your Desktop. The simplified steps are listed below; you might need to install extra dependencies. 

- **From GitHub**
	
	```
	git clone https://github.com/GMOD/jbrowse
	cd jbrowse
	./setup.sh
	```

- **From the JBrowse blog**: visit [http://jbrowse.org/blog](http://jbrowse.org/blog) and download one of the available zip files. Extract it and rename the folder to `jbrowse` to simplify things. The run the following command:

	```
	cd jbrowse
	./setup.sh
	```
	
After that, you need to start jbrowse by running the command `npm run start`. This will start a local jbrowse instance, and the address to access it is listed on your terminal:

```
@gmod/jbrowse@1.16.4 start /Users/mtsuchiya/Desktop/jbrowse
> utils/jb_run.js -p 8082

JBrowse is running on port 8082
Point your browser to http://localhost:8082
```

In my case, I opened a web browser (Chrome) and pasted `http://localhost:8082` on the address bar. This should start JBrowse with one of the sample projects available with the installation.

- **To visualize your data**
	- Copy the zipped file from Hydra to your machine using `scp` or Filezilla.  

		`scp username@hydra-login01.si.edu:/pool/genomics/username/GAworkshop/jbrowse/file.tar.gz ./Desktop/jbrowse`
	- Extract the file  
		`tar -zxf file.tar.gz`
	- Add the folder name to the address bar. In my case, this is the address to display my JBrowse files: 
		`http://localhost:8082/index.html?data=Dhydei`
		
		Let me explain what this address mean:  
			- `localhost:8082`: this is the port in your computer that's being used by JBrowse.   
			- `index.html`: you will find this file in your local `jbrowse` folder. This is a HTML page, formatted to display the JBrowse files correctly.  
			- `data=Dhydei`: this is the path to your data file inside the `jbrowse` folder. The examples provided with the installation are inside `sample_data/json/volvox`. If you replace your folder name by this, you will be able to see the sample data for *Volvox*. 
