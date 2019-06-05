# Running Blast and Blast2GO
After running AUGUSTUS

###Steps:
1. Convert Augustus GFF to GFF3
2. Extract CDS annotations
3. Get fasta from GFF3
4. Split fasta into sub-files with 100 sequences
5. Create blast jobs
6. Merge the xml files
7. Running Blast2GO
8. Processing Blast2GO output (optional)


###Before starting:

#####Folder structure:

	BLAST
		|_ blast2go		b2go results 
		|_ fa			split fasta files
		|_ input_files 	input files before starting the analysis
		|_ jobs			job files
		|_ logs			log files
		|_ results		FINAL results
		|_ xml			xml outputs




###1. Convert AUGUSTUS GFF TO GFF3

(This is the last step in the Genome Annotation Guide. If you have already done it, just skip to Step 2.)

Here we will use an EVM script to convert from AUGUSTUS GFF to a linear GFF# (without the sequences and comments).

	/PATH/TO/EVM/EvmUtils/misc/augustus_GFF3_to_EVM_GFF3.pl augustus_all.gff3 > augustus_final.gff3


*Save/copy the output in the blast/input_files folder*

###2. Extract CDS annotations


	grep "CDS" augustus_final.gff3 > augustus_CDS.gff3

*Save the output to the folder blast/input_files*

###3. Get fasta from GFF3
This step can be submitted as a job or ran it from the interactive queue. 


	# /bin/sh 
	# ----------------Parameters---------------------- #
	#$  -S /bin/sh
	#$ -q sThC.q
	#$ -l mres=6G,h_data=6G,h_vmem=6G
	#$ -cwd
	#$ -j y
	#$ -N getfasta
	#$ -o ../logs/getfasta.log
	#
	# ----------------Modules------------------------- #
	module load bioinformatics/bedtools
	#
	# ----------------Your Commands------------------- #
	#
	echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
	echo + NSLOTS = $NSLOTS
	#
	bedtools getfasta -fi assembly.fa -bed augustus_CDS.gff3 -fo augustus_CDS.fasta
	#
	echo = `date` job $JOB_NAME done

**Notes:**

- Running from the folder `blast/input_files`
- I prefer to create a symlink of the assembly in this folder.


###4. Split fasta into sub-files with 100 sequences
To make BLAST more efficient, we will split the fasta file generated in the previous step (augustus_CDS.fasta) into 100 sequences. We can split into less than that (let's say, 25 or 50 sequences).

	awk 'BEGIN {n_seq=0;} /^>/ \
	{if(n_seq%100==0){file=sprintf("prefix_%d.fa",n_seq);} \
	print >> file; n_seq++; next;} { print >> file; }' < augustus_CDS.fasta
	
(Source: https://www.biostars.org/p/13270/)

Some notes about this script: 

- On the second line, the number correspond to the number of sequences in each file (100).
- The first file id will be zero. Which means it can be used as a TASK_ID. (FIX IT)
- For now, we can just submit a separate job for the file that was numbered zero.
- The prefix is the name of your new files, followed by their number ID.

*Move all new fasta (.fa) to the folder blast/fa*

###5. Create blast jobs

Use the script `create_blast_jobs.sh` to generate the job files. Run the script from the `blast/jobs` folder. The script is below:

```
#!/bin/sh
#the numbers after seq correspond to: start, increment, max number
for start in `seq -f "%.0f" 100 1000 386000`
	do 
	echo '# /bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -q mThC.q
#$ -l mres=6G,h_data=6G,h_vmem=6G
#$ -t '$start'-'$(($start+900))':100 -tc 100
#$ -cwd
#$ -j y
#$ -N blast
#$ -o ../blast_$TASK_ID.log 
#
# ----------------Modules------------------------- #
module load bioinformatics/blast
#
# ----------------Your Commands------------------- #
#
echo + `date` $JOB_NAME started on $HOSTNAME in $QUEUE with jobID=$JOB_ID and taskID=$TASK_ID
#
blastx -query ../fa/prefix_$SGE_TASK_ID.fa -db nr -outfmt 5 -max_target_seqs 10 -evalue 1e-4 -out ../xml/prefix_$SGE_TASK_ID.xml
#
echo = `date` $JOB_NAME for taskID=$TASK_ID done.
#' &> "blast_prefix_$start-$(($start+900)).job"
done
```

This script will create multiple blast jobs with tasks. In this case, I ended up with 386 jobs (remember that each file was 100 sequences, so each task has 100 sequences, and each job will cover 1000 sequences). You can adjust the numbers to anything that makes sense to your dataset.

###6. Merge the xml files

After all the jobs are done, you need to merge all xml files into one single file. You can use cat for that:
From the xml folder, do:

	cat $(find . -name "*.xml" | sort -V) > ../results/all.xml
	
###7. Running Blast2GO

**Before submitting the job:** from the folder blast2go, you need to load the blast2go module and run the command hydracliprop:

	module load bioinformatics/blast2go
	hydracliprop

This command will create the file `cli.prop`, which contains the properties to run blast2go on Hydra. 

Job template:

```
# /bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -q lTb2g.q
#$ -l b2g,mres=6G,h_data=6G,h_vmem=6G
#$ -cwd
#$ -j y
#$ -N b2g
#$ -o ../logs/b2g.log
#
# ----------------Modules------------------------- #
#
module load bioinformatics/blast2go
#
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
#
#
runblast2go \
  -properties cli.prop \
  -loadfasta input_CDS.fasta \
  -loadblast input_final.xml \
  -mapping \
  -annotation \
  -nameprefix changeme \
  -saveb2g -saveannot -savereport -savelorf -saveseqtable
  -statistics all
  -exportgeneric
 #
echo = `date` job $JOB_NAME done

```

Explanation: 

- -properties: path to properties file (cli.prop)<sup>1</sup>
- -loadfasta: CDS fasta file (from step 3)
- -loadblast: concatenated blast xml output
- -mapping: Run the Gene Ontology mapping<sup>1</sup>
- -annotation: Run the Blast2GO annotation algorithm<sup>1</sup>
- -nameprefix: species name, assembly, etc.
- -saveb2go: Save the project as .b2g<sup>1</sup>
- -saveannot: Save the functional annotations (Gene Ontolgoy terms and Enzymes) as .annot<sup>1</sup>
- -savereport: Create .pdf report<sup>1</sup>
- -savelorf: Convert nucleotide sequences into aa sequences. Useful for interproscan run<sup>1</sup>
- -saveseqtable: Save your data as it would be shown in the Blast2GO GUI<sup>1</sup>
- -statistics all: Provide a comma-separated list of all statistical charts<sup>1</sup>
- -export generic: Export sequence data in tabular format for post processing<sup>1</sup><sup>,2</sup>

<sup>1</sup> From the [Blast2go Cli Manual](<http://cli.docs.blast2go.com/user-manual/command-line-parameters/>). See additional [case examples](http://cli.docs.blast2go.com/user-manual/use-case-examples/).

<sup>2</sup> More info about the [generic export](http://cli.docs.blast2go.com/user-manual/generic-export/).

More information about running Blast2GO on Hydra from the [Wiki](https://confluence.si.edu/display/HPC/Running+BLAST2GO+on+Hydra)


###8. Processing Blast2GO output (optional)
I wrote a script that takes the Blast2GO results and outputs (to the screen) the following info:

- Number of annotated scaffolds
- Extracts each GO term and saves them into the corresponding category (C: celullar component, P:biological process or F: molecular function). It also saves a file with all GO terms from all categories.

You need to have the output of a single blast2go run in the folder (if you have more than one run, organize them into different folders and then run the script).

```
#!/bin/sh
# Created by MTNTsuchiya on 27 Nov 2018
#
# Extracting GO information from B2GO files
#
species=${PWD##*/}
printf "\nProcessing the folder $species\n"
#
# ANNOT FILES
#
# Variables
annot_file=$(ls *.annot)
annot_filtered=$annot_file.fltr
#
printf "\nExtracting GO information from .annot files.\n\n"
echo File: "$annot_file"
#
printf "Step 1/3: Filtering only lines with GO annotation\n"
awk '{ if ($3) print ($0) }' < $annot_file > $annot_filtered
printf "Step 1/3: Done. Saving file $annot_filtered\n"
printf "Total number of lines with annotation: "
wc -l $annot_filtered | cut -f1 -d' '
#
#
printf "\n\nStep 2/3: Extracting list of scaffolds from $annot_filtered\n"
cut -f1 $annot_filtered | sort | uniq > $annot_filtered.scaffs
printf "Step 2/3: Done. Saving file $annot_filtered.scaffs\n"
printf "Total number of unique filtered scaffolds (filtered file): "
wc -l $annot_filtered.scaffs | cut -f1 -d' '
#
#
printf "\n\nStep 3/3: Comparing the number and list of unique scaffolds from the original and filtered files\n\n"
#
printf "Getting list of unique scaffolds from the original annot file\n"
cut -f1 $annot_file | sort | uniq > $annot_file.scaffs
printf "Total number of unique scaffolds (original file): "
wc -l $annot_file.scaffs | cut -f1 -d' '
#
printf "Number of unique scaffolds to original file: "
comm -13 <(sort "$species".annot.scaffs) <(sort "$species".annot.fltr.scaffs) | wc -l
#
printf "Number of unique scaffolds to filtered file: "
comm -23 <(sort "$species".annot.scaffs) <(sort "$species".annot.fltr.scaffs) | wc -l
#
printf "Number of scaffolds common to both files: "
comm -12 <(sort "$species".annot.scaffs) <(sort "$species".annot.fltr.scaffs) | wc -l
printf "\n"
#
#
printf "\nNow processing txt files\n"
#
# Variables
txt_file=$species.txt
singleGO=$species.singleGO.txt
GOlist_single=$species.GOlist_single.txt
GOlist_total=$species.GOlist.total.txt
#
printf "File: $txt_file\n"
#
printf "Total number of lines (original txt): "
wc -l $txt_file | cut -f1 -d' '
#
printf "\nExtracting list of single GO terms...\n"
sed 's/ *;.*//' $txt_file > $singleGO
printf "Total number of lines $singleGO: "
wc -l $singleGO | cut -f1 -d' '
grep -o '[CPF]:GO:[0-9]\{7\}' $singleGO > $GOlist_single
printf "Total number of lines $GOlist_single: "
wc -l $GOlist_single | cut -f1 -d' '
#
printf "\nExtracting each GO category from $GOlist_single\n"
go_cat=(C F P)
for go in "${go_cat[@]}"; do
	pattern="$go":GO:'[0-9]\{7\}'
	out_single="$species"_"$go"_single.txt
	printf "\nProcessing category $go...\n"
	grep -o "$pattern" "$singleGO" > "$out_single"
	printf "Saved the file $out_single\nTotal number of $go terms: "
	wc -l "$out_single" | cut -f1 -d' '
done
#
grep -o '[CPF]:GO:[0-9]\{7\}' $txt_file > $GOlist_total
printf "\nTotal number of lines $GOlist_total: "
wc -l $GOlist_total | cut -f1 -d' '
#
printf "\nExtracting each GO category from $GOlist_total\n"
go_cat=(C F P)
for go in "${go_cat[@]}"; do
	pattern="$go":GO:'[0-9]\{7\}'
	out_total="$species"_"$go"_total.txt
	printf "\nProcessing category $go...\n"
	grep -o "$pattern" "$txt_file" > "$out_total"
	printf "Saved the file $out_total\nTotal number of $go terms: "
	wc -l "$out_total" | cut -f1 -d' '
done
#
printf "\n\nProcessing complete\n\n"
#
# Modified on 27 Nov 2018
```

You can use the F,C and P outputs to create Venn Diagrams and compare each category between species, different assembly methods, etc. 