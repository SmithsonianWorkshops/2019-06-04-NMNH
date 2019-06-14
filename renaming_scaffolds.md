## Rename the scaffolds sequentially

We will use `bioawk` to rename all scaffolds sequentially. Here, the `filtered fasta` corresponds to your fasta file after removing the small scaffolds, and `ordered_fasta` will be your final output. 

```
bioawk -c fastx '{ print ">scaffold_" ++i"\n"$seq }' < filtered.fasta > ordered.fasta
```

**Why are we doing this?**
For the AUGUSTUS run, we will create a *job array* to make our annotation more efficient. Job arrays consist on multiple tasks inside the same job, and those tasks have to be sequential (or not random at least). If we had just filtered out the small scaffolds and did not perform this step, we would end up with lots of of "empty" tasks. Like this:

- Imagine that we kept scaffolds 1 until 10, then 50 until 60. if we don't do the renaming step, the job array would look for the tasks 11 until 49 and would find nothing.

- If we had only 100 scaffolds, that would probably be a just a minor inconvenience. But since most projects will have thousands, (or dozens of thousands) scaffolds, that mess can become difficult to clean and filter out - and it also wastes computer resources.

## Create a name map of the renaming step. 
In case you had already run other analyses, or you want to be able to revert to the original scaffold names, you can obtain the name map using this command:

`paste <(grep ">" filtered.fasta) <(grep ">" ordered.fasta) | sed 's/>//g' > name_mapping.txt`

The name map file will look like this:

```
scaffold_1	scaffold_1
scaffold_2	scaffold_2
scaffold_4	scaffold_3
scaffold_6	scaffold_4
scaffold_7	scaffold_5
scaffold_10	scaffold_6
scaffold_11	scaffold_7
scaffold_13	scaffold_8
scaffold_14	scaffold_9
scaffold_18	scaffold_10

```

First column shows the original scaffold name (from the filtered file) and the second the new scaffold id (after the renaming).
