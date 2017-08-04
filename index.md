---
layout: lesson
root: .
title: mothur tutorial
date: 2017-08-04
---


### Logistics
Instructor:
Tracy Teal <tkteal@datacarpentry.org> @tracykteal


### Overview of mothur
What you can use it for and what is the workflow in slides:

[http://tracykteal.github.io/mothur-tutorial/mothur_stamps_presentation.pdf](http://tracykteal.github.io/mothur-tutorial/mothur_stamps_presentation.pdf)


#### Running mothur
You can run mothur on your own computer or an a remote computer. For this 
exercise we'll be running it on the MBL servers.

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


   ```
   cd mothhur_data
   ls
   ```

We have all our files in 'MiSeq_SOP'. There are the FASTQ files from the experiment, as well as some reference files
we need for the analysis. We will end up with a lot of files in this one directory. We'll organize things a bit at the end.

The file that you need to create before any analysis, is a list of the samples and the sequences associated with those samples. Here it's called **stability.files** but you can call it anything. It looks like this.

![stability.files](img/stability.jpg)

You can see that the format is a tab delimited file with the sample name in the first column,
the forward read in the second column and the paired end read in the third column.  

(Optional: look at the stability.files file with 'less')

### Starting to work with mothur

Working with mothur, we're going to follow the [mothur MiSeq SOP](https://www.mothur.org/wiki/MiSeq_SOP).

The command we'll use on the MBL server are here.

**Follow along on this document**: [MBL mothur MiSeq SOP](https://hackmd.io/EwVgnAxmYMwGwFo4HYBGEEBYQBMOuR2C2QA4AGZaAU2GTSA=?view)


### Working with the OTU table (the shared file)

Once we create the shared file, there are several things we can do. First we can download this file to our local computers.

Now let's take a look at the shared file. Let's open it in Excel.
[stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.shared](img/stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.shared)

Let's also look at the list file  
[stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.list](img/stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.list)

There's been a lot of discussion of what to do with the raw counts:  
- subsample
- normalize by sample size
- mixed model
- something else

For this lesson we have subsampled. When we subsample, things will look different each time. Compare your 'shared' file with that of your neighbors. Are they the same?

### Others things to do with mothur (if you're really working with it)

#### Exploring parameter space

Let's say that instead of using the 0.03 cutoff when you make your shared file, you want to use 0.05?

By default if you run the same command with a different parameter it just overwrites the previous files. 
Try running 

   `make.shared(list=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.list, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.uchime.pick.pick.pick.count_table, label=0.05)`

and see.

Rerun it again with 0.03 to get it back to where it was.

Instead you can redirect the output to a new folder with set.dir. Let's try it. We'll tell mothur
to use the input directory 'analyses/first' where we have all the files that have been 
generated already, then we'll set a new output directory.
   
   ```
   system(mkdir ~/data/second)
   set.dir(input=~/data/process, output=../data/second)
   ```

Then run

   `make.shared(list=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.list, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.uchime.pick.pick.pick.count_table, label=0.05)`

Ta da, the 0.05 file is now in 'second'. You can do this sort of thing anywhere along the path. Instead
of explicitly setting the output and input, you can also point to an input file directly.

   `make.shared(list=~/data/process/stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.list, count=~/data/process/stability.trim.contigs.good.unique.good.filter.unique.precluster.uchime.pick.pick.pick.count_table, label=0.05)`

### Running mothur in batch mode

Once you've run through mothur with your shiny new data, you're get to a point where 
you're happy with a certain set of parameters. Now you might want to set mothur up in batch mode.
That's where you make a file with a bunch of commands and then just run that file.

We have one of those files already in our data directory: stability.batch

When you're doing this the fact that mothur remembers what files you used last is great, because
you don't have to type it all out.

Let's try making our own. We'll go back on the server and use **nano**.

Put the first couple of commands we run in to a file, along with comments about what we are doing.

```
    #set input and output directory  
    set.dir(input=~/Desktop/mothur-project/data/MiSeq_SOP, output=~/Desktop/mothur-project/analyses/first)  
    #make contigs  
    make.contigs(file=stability.files, processors=8)
```

We're using the 'full path' of the file names here so there's no confusion.

### Looking at some of this data

You have been working in R already, so there's a lot of ways you can work with the data there. 

There is also PAST - a GUI statistical package developed by Oyvind Hammer.  
[http://folk.uio.no/ohammer/past/](http://folk.uio.no/ohammer/past/)




