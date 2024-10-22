# Running jobs on HPC scheduler

Objectives:
* Learn some basic Unix commands for navigating the filesytem in Hydra.
* Learn best practices for organizing your project directories.
* Learn how to submit a job to the HPC scheduler.
* Learn how to transfer results to your local computer.

## Logging In

You will need a terminal program in order to connect to, and interact with Hydra. Also, ensure that you are connected to the "si-staff" WiFi network.

### PC

For PC users, we recommend installing PuTTY, which you can download from [https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). Choose the 64-bit MSI installer.

When you have PuTTY installed, open it up and enter `{user}@hydra-login01.si.edu` in "Host Name (or IP address)" and 22 for Port. Click the "Open" button to connect, and you will be prompted for your Hydra password on another window.

### Mac

If you are on a Mac, open a Terminal window.

Enter the following command.

```console
$ ssh {user}@hydra-login01.si.edu
Password:
```

## Creating a project directory

Now that we are successfully logged in, you will see a little blurb about recent Hydra news. Make sure you read this.

You will also see a prompt that looks like this:

```console
[{user}@login-30-1 ~]$ _
```

By default, you are placed in your "home" directory when you first log in. You can see where you are at any point with the `pwd` command, which stands for "present working directory". Try entering it now.

```console
$ pwd
```

Another useful command for getting a feeling for the file system is `ls`, which is short for "list". It will list the files in your current working directory.

```console
$ ls
```

Most commands Unix commands let you add extra *parameters* that give you more control over what you want the command to do. Let's enter the same `ls` command again, and add the `-lh`, which stand for "long listing" and "human-readable".

```console
$ ls -lh
```

You can see a full listing of the options for `ls` in its help screen. All Linux command have a screen like this, which is slightly faster than Google-ing.

```console
$ ls --help
```

Your "/home" directory has a relatively small size limit, so you should only use it for basic configuration files, scripts and job files. Your "/pool" directory is where you should run your jobs from.

Let's move to the "/pool" directory with the `cd` command, which stands for "change directory". Feel free to use the `pwd` and `ls` commands again to see what has changed.

```console
$ cd /pool/genomics/{user}
```

Now that we're in the correct location, it is best practice to create a separate directory for each "project" you are working on. You can make a new directory with the `mkdir` command.

```console
$ mkdir hydra_workshop
```

And now enter the new project directory by using the `cd` command again.

```console
$ cd hydra_workshop
```

Now that we're in our project directory, it's also best practice to create an organizational structure so that we can come back in a few weeks and remember what we did. We recommend the following structure, but it will change based on your needs.

```console
$ mkdir jobs
```

```console
$ mkdir logs
```

```console
$ mkdir -p data/raw
$ mkdir -p data/results
```

Now that our scaffold is built, you can visualize it with the `tree` command.

```console
$ tree .
.
├── data
│   ├── raw
│   └── results
├── jobs
└── logs

5 directories, 0 files
```

## Modules

Modules are a way to set your system paths and environmental variables for use of a particular program or pipeline.

You can view available modules with the command:

```console
$ module avail
```

This will output all modules installed on Hydra. A complete list can also be found at [https://www.cfa.harvard.edu/~sylvain/hpc/module-avail.html](https://www.cfa.harvard.edu/~sylvain/hpc/module-avail.html).

There are additional module commands to find out more about a particular module. Let's use the bioinformatics program *IQ-TREE* to test these out.

```console
$ module whatis bioinformatics/iqtree
bioinformatics/iqtree: System paths to run IQTREE 1.6.10
```

You can also see module-specific help for any module with the `module help` command. These are written as part of the Hydra installation process, and should not be mistaken for the official documentation for the software.

```console
$ module help bioinformatics/iqtree

----------- Module Specific Help for 'bioinformatics/iqtree/1.6.10' ---------------------------


Purpose
-------
This module file defines the system paths for IQTREE 1.6.1
The compiled binary that you can now call are:
iqtree
To run threaded use -nt flag, see documentation for more information.

Documentation
-------------
http://www.cibiv.at/software/iqtree/

<- Last updated: Mon Feb 25 10:36:42 EST 2019, VLG ->
```

Ok, now let's actually load IQ-TREE.

```console
$ module load bioinformatics/iqtree
```

Nothing happens, but let's run a quick command to show that we have IQ-TREE loaded properly.

```console
$ iqtree -h
IQ-TREE multicore version 1.6.10 for Linux 64-bit built Feb 19 2019
Developed by Bui Quang Minh, Nguyen Lam Tung, Olga Chernomor,
Heiko Schmidt, Dominik Schrempf, Michael Woodhams.

Usage: iqtree -s <alignment> [OPTIONS]

GENERAL OPTIONS:
  -? or -h             Print this help dialog
  -version             Display version number
  -s <alignment>       Input alignment in PHYLIP/FASTA/NEXUS/CLUSTAL/MSF format
...
```

## Submitting a job

We will now be creating and submitting an IQ-TREE job to the compute cluster.

First of all, let's copy a sample input .phy file into our `data/raw` directory. We will use the `cp` command, which takes in 2 *arguments*: the file you want to copy, and then the new location.

```console
$ cp /data/genomics/data/primates.dna.phy data/raw
```

Now that the input file is in place, we'll need to generate a job submission file.

We will be using the Hydra QSub Generation Utility to create this file. Go to [https://hydra-4.si.edu/tools/QSubGen/](https://hydra-4.si.edu/tools/QSubGen/) and fill in the following values:

* *CPU time*: short
* *Memory*: 1 GB
* *Type of PE*: multi-thread
* *Number of CPUs*: 4
* *Select the job's shell*: sh
* *Select which modules to add*: bioinformatics/iqtree
* *Job specific commands*:
iqtree -s ../data/raw/primates.dna.phy -nt $NSLOTS -pre ../data/results/primates.dna

* *Job Name*: iqtree-{user}
* *Log File Name*: ../logs/iqtree
* *Err File Name*: [blank]
* *Change to CWD*: Y
* *Join output & error files*: Y
* *Send email notifications*: Y
* *Email*: [your email]

This will generate the contents of your "job file" at the bottom, but now we need to get this into a file on Hydra. Change directories into the "/jobs" directory.

```xonsole
$ cd jobs
```

And now we'll open up the `nano` text editor to edit text. You can use the nano command to create a new file by entering a filename as an argument.

```console
$ nano iqtree.job
```

This will put you into a text editor of a blank document. You can copy and paste the gray contents from the QSub generator webpage directly into here. Once you have pasted in the commands, press `Ctrl-X` to exit, enter `Y` to save changes, and then `↵` to save to "iqtree.job".

You're finally ready to submit your job to the compute nodes, using the `qsub` command.

```console
$ qsub iqtree.job
```

## Monitoring your job

If everything worked correctly, your job should now be running on the compute nodes. You can use `qstat` to check the status of your jobs. If nothing appears here, then your job is already finished.

```console
$ qstat
```

When `qstat` shows that your job is finished running, you can now `cd` into your `data/results` directory to see if iqtree produced output files.

**If your job finishes without producing output files (or anything else unexpected occurs), your first instinct should be to check the log file.**

To check the log file, `cd` to the `/log` directory and read the entire contents with the `cat` command.

```console
$ cat iqtree.log
```

Note the job id that is listed in here, and you can now use it to look into the amount of resources used during your run. This is useful for troubleshooting when memory limits are hit.

```console
$ qacct -j {job ID}
```

## Transferring files

If you haven't already, go ahead and install a file transfer client. We recommend FileZilla, which you can download from [https://filezilla-project.org/download.php?show_all=1](https://filezilla-project.org/download.php?show_all=1). DO NOT USE THE SKETCHY INSTALLER AT [https://filezilla-project.org/download.php?type=client](https://filezilla-project.org/download.php?type=client), which comes with "bundled offers".

In the Quickconnect toolbar at the top of the window enter:

* Host: hydra-login01.si.edu or hydra-login02.si.edu
* Username: your Hydra username
* Password: your Hydra password
* Port: 22 

Press the "Quickconnect" button to start the connection.

The files listed on the left side of the window are on your local computer, those on the right are on Hydra.

Enter your `/pool/genomics/{user}` filepath in the "Remote site" entry on the right side. You can than use the file tree on the right to navigate to your result files.

Drag the ".treefile" file to the left side in an appropriate directory on your local computer.

Now, go to [https://icytree.org/](https://icytree.org/) in a web browser, and open up the ".treefile" file to view the tree.

## Interactive queue

Hydra has another way of running jobs on the compute nodes, without using the job scheduler. Let's try to run the exact same iqtree command using this technique.

First, you need to use the `qrsh` command to enter this environment. The main parameter to know is how many threads you will be using.

```console
$ qrsh -pe mthread 2
```

An important point to note with the interactive queue is that is always places you back in your `/home` directory, which can be confusing.

```console
$ pwd
/home/{user}
```

So let's go back to our "hydra_workshop" directory.

```console
$ cd /pool/genomics/{}/hydra_workshop
```

And now we can directly run the same commands that we listed out in our job file.

```console
module load bioinformatics/iqtree
```

And then the same iqtree command -- but changing the data paths, since we're in the main project directory now.

```console
iqtree -s data/raw/primates.dna.phy -nt $NSLOTS -pre data/results/primates.dna
```

