---
layout: lesson
root: .
title: mothur tutorial
date: 2016-07-09
---


### Logistics
Instructor:
Tracy Teal <tkteal@datacarpentry.org> @tracykteal

Real-time notes for the class.  These will stay there after the workshop too.
<br>Etherpad:  [https://public.etherpad-mozilla.org/p/mothur-stamps2016](https://public.etherpad-mozilla.org/p/mothur-stamps2016)

shell cheat sheets:<br>
[http://fosswire.com/post/2007/08/unixlinux-command-cheat-sheet/](http://fosswire.com/post/2007/08/unixlinux-command-cheat-sheet/)

[https://github.com/swcarpentry/boot-camps/blob/master/shell/shell_cheatsheet.md](https://github.com/swcarpentry/boot-camps/blob/master/shell/shell_cheatsheet.md)

### Overview of mothur
What you can use it for and what is the workflow in slides

[http://tracykteal.github.io/mothur-tutorial/mothur_stamps_presentation.pdf](http://tracykteal.github.io/mothur-tutorial/mothur_stamps_presentation.pdf)


#### Running mothur
You can run mothur on your own computer or an a remote computer. For this 
exercise we'll be running it on Amazon EC2 instances. I've set up the instances for 
this course, but there are great instructions for setting up your own instances.

[mothur AMI](http://mothur.org/wiki/Mothur_AMI)

The biggest constrainst with mothur when working with large data files is memory, so
when you work with your own data, you might need to get more memory. Most analyses 
will need more computational power than your own computer, so you'll need something
like Amazon instances or your local HPC. 

#### Using and connecting to Amazon cloud computing for mothur

While you can run mothur on your own computer, in practice, you'll need 
to run it on a computer with more computational power, like on Amazon EC2
or computers at your university. The advantage of this is that it's not taking 
up computational resources on your own computer.
You could even start something and then close your computer and go home, or I mean 
go do some lab work. Also a lot of these analyses are just too computationally 
intensive to run on your own computer.

[Connecting to mothur Amazon AMI](https://github.com/tracykteal/mothur-tutorial/blob/gh-pages/cloud-genomics/index.md)

#### Starting mothur

Type

   `mothur`  

Now you're in mothur, and we can start!  
You should get the command prompt  

   `mothur >`

Green sticky note up if you have the mothur command prompt

Once you have it up the first thing we'll do is quit mothur, so type  

   `quit()`  

If you're in mothur, or really any command line environment, you can type Ctrl^C to quit that command

**Getting help**  
mothur manual - [http://wiki.mothur.org/wiki/Mothur_manual](http://wiki.mothur.org/wiki/Mothur_manual)  
mothur user forum - [http://www.mothur.org/forum/](http://www.mothur.org/forum/)

### Set up your work environment

How are you going to keep track of all the commands you're running.  
Be nice to the you of 6 months or even a week from now!

Computational processes need lab notebooks just like lab work.

- Evernote
- OneNote
- Word
- a text editor
- IPython notebook
- photographic memory

In mothur if you run an analysis with different parameters in the same directory,
it will write over your old analysis.

mothur does always write out logfiles, so you can keep track with those some. We'll 
look at those after we've run some commands.

### Exploring the data

Let's take a look at the data we have and how it's organized.

We have three directories: 'code', 'data' and 'R'. 

The 'R' directory has the information for running R on these instances. 
We aren't going to do anything with that. 

The 'code' directory has the code for running the batch file. We'll talk
about that later.

The 'data' directory has the data we'll use for this tutorial. Let's go into
that directory.

   ```
   cd data
   ls
   ```

Now, we see we have 'mothur', 'process', 'raw' and 'references'. 'raw' are all our 
raw data. 'process' will be where we do this analysis. 'references' are the 
reference files. 'mothur' are files for running mothur. We can see that we have
a bunch of gzipped FASTQ files in the 'raw' directory. We're going to get started
by getting this data in the format we need.

### Starting to work with mothur

Go into the 'process' directory.

   cd
   cd data/process

Start mothur

   mothur

First we'll set the directory where we want mothur to look for data by default
and set the output directory using the [set.dir command](http://www.mothur.org/wiki/Set.dir)

    set.dir(input=../raw)
    set.dir(output=.)
    
Now we'll make the file that has the information for all of our reads

    make.file(inputdir=../raw, type=gz)

We're going to rename that file, so it has the same name as in the 
mothur MiSeq SOP, for convenience. The command 'system' in mothur let's us 
do the normal things we would do at the command line.

    system(mv fileList.paired.file stability.files)

The file we just generated looks like this:

![stablility.files](img/stability.jpg)

You can see that the format is a tab delimited file with the sample name in the first column,
the forward read in the second column and the paired end read in the third column.  

(Optional: close the tmux window and look at the stablity.files file with 'less')

Now we'll take the unpaired reads and pair them, using the information 
in the file we just generated.

    make.contigs(file=stability.files, processors=8)

You'll wait a bit, and get some output.  
Four files will be created:  
* stability.trim.contigs.fasta - the FASTA file of the assembled paired end sequences
stability.contigs.report  - a report for each assembled contig of the overlap and number of Ns
stability.scrap.contigs.fasta - sequences that didn't pair
* stability.contigs.groups - a file with information on what sequence belongs to what sample

* files are ones that will be used in downstream analysis.  
In general, the *.fasta and *.groups files are the ones you'll need in the next steps.

### BREAK

We're going to pause here and when we come back, we'll start going through the workflow


### The mothur MiSeq SOP

Go to the mothur MiSeq SOP. This is what we're going to go through.  

[http://www.mothur.org/wiki/MiSeq_SOP](http://www.mothur.org/wiki/MiSeq_SOP)

Now let's start going through the protocol  
** Often the summary.seqs() commands take awhile.  Just wait and eventually you'll get the result

**Notes along the workflow**

- The pcr.seqs command is different than in the SOP because of updated databases

    pcr.seqs(fasta=../references/silva.seed_v123.align, start=11894, end=25319, keepdots=F, processors=8)
    

The classify.seqs command is different than the SOP because there are updated databases

    classify.seqs(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.count_table, reference=../references/trainset14_032015.pds.fasta, taxonomy=../references/trainset14_032015.pds.tax, cutoff=80

- We'll skip the Assessing error rates section

- We'll skip the Phylotypes and Phylogenetic sections

- We'll stop after the Alpha diversity section



   align.seqs(fasta=stability.trim.contigs.good.unique.fasta, reference=silva.bacteria.pcr.fasta)

At the remove.seqs command, the command should be

   `remove.seqs(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.fasta, accnos=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.accnos)`

The remove.lineage line should be

   `remove.lineage(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.count_table, taxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pds.wang.taxonomy, taxon=Chloroplast-Mitochondria-unknown-Archaea-Eukaryota)`

Then

   `get.groups(count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.pick.count_table, fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.fasta, groups=Mock)`

remove.groups

   `remove.groups(count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.pick.count_table, fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.fasta, taxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pds.wang.pick.taxonomy, groups=Mock)`

cluster

   `cluster(column=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.dist, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.pick.pick.count_table)`

cluster.split

   `cluster.split(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.pick.pick.count_table, taxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pds.wang.pick.pick.taxonomy, splitmethod=classify, taxlevel=4, cutoff=0.15)`


Creating the distance matrix and doing the clustering are the time and memory intensive steps.

Once we create the shared file, there are several things we can do.
Let's take a look at the shared file though. Let's open it in Excel.
[stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.shared](img/stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.shared)

Let's also look at the list file  
[stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.list](img/stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.list)

There's been a lot of discussion of what to do with the raw counts:  
- subsample
- normalize by sample size
- mixed model
- something else

For this lesson we're going to subsample

See how many sequences we have  
`mothur > count.groups()`

Lowest is 2441, so we'll subsample to that size  
`mothur > sub.sample(shared=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.shared, size=2441)`  
Now we have a new shared file that has been subsampled. If we did the subsampling again, we'd actually get a different new shared file.  

** Exercise

Go ahead and try it. Open up your new shared file. Rerun the subsampling and see if the re-subsampled one looks the same.
This is easiest if you're running it on your own computer. Pair up with someone who is if you're not.

### Others things to do with mothur (if you're really working with it)

#### Exploring parameter space

Let's say that instead of using the 0.03 cutoff when you make your shared file, you want to use 0.05?

By default if you run the same command with a different parameter it just overwrites the previous files. 
Try running 

   `mothur > make.shared(list=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.list, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.uchime.pick.pick.pick.count_table, label=0.05)`

and see.

Rerun it again with 0.03 to get it back to where it was.

Instead you can redirect the output to a new folder with set.dir. Let's try it. We'll tell mothur
to use the input directory 'analyses/first' where we have all the files that have been 
generated already, then we'll set a new output directory.

   `set.dir(input=../analyses/first, output=../analyses/second)`

Then run

   `mothur > make.shared(list=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.list, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.uchime.pick.pick.pick.count_table, label=0.05)`

Ta da, the 0.05 file is now in 'second'. You can do this sort of thing anywhere along the path. Instead
of explicitly setting the output and input, you can also point to an input file directly.

   `mothur > make.shared(list=../analyses/first/stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.list, count=../analyses/first/stability.trim.contigs.good.unique.good.filter.unique.precluster.uchime.pick.pick.pick.count_table, label=0.05)`

### Running mothur in batch mode

Once you've run through mothur with your shiny new data, you're get to a point where 
you're happy with a certain set of parameters. Now you might want to set mothur up in batch mode.
That's where you make a file with a bunch of commands and then just run that file.

We have one of those files already in our data directory: stability.batch

When you're doing this the fact that mothur remembers what files you used last is great, because
you don't have to type it all out.

Let's try making our own. On Mac you can use nano or emacs. On PC you can use something in MobaXterm
or SublimeText.

Put the first couple of commands we run in to a file, along with comments about what we are doing.


    #set input and output directory  
    set.dir(input=~/Desktop/mothur-project/data/MiSeq_SOP, output=~/Desktop/mothur-project/analyses/first)  
    #make contigs  
    make.contigs(file=stability.files, processors=8)


We're using the 'full path' of the file names here so there's no confusion.

### Looking at some of this data

PAST - a GUI statistical package developed by Oyvind Hammer  
[http://folk.uio.no/ohammer/past/](http://folk.uio.no/ohammer/past/)




### Running mothur in batch mode


### Creating your own 'stability.file' for your data




### Past the shared file

- Classify
- Alpha and beta diversity
- Data visualization
