# STAMPS mothur commands

**This document is the commands that we'll 
run in mothur during the tutorial**

1. Log on to the MBL campus machines
```
ssh username@class.mbl.edu
ssh username@class-host
```

2. Create a directory for this tutorial and go into that directory.
```
mkdir mothur_tutorial
cd mothur_tutorial
```

3. Copy the data needed for the tutorial into this directory.
```
cp -r ~tteal/mothur_data .
```

4. Unzip the files and move a few files around
```
unzip *

# move the silva.bacteria.fasta file to this directory
mv silva.bacteria/silva.bacteria.fasta MiSeq_SOP/

# move the training set data into the MiSeq directory
mv trainset9_032012.pds.* MiSeq_SOP/
```

5. Go into the MiSeq_SOP directory
```
cd MiSeq_SOP
```

6. Start mothur version 
```
module load mothur/1.37.3
mothur
```

### mothur SOP
Now we'll generally follow the mothur MiSeq SOP  
[https://www.mothur.org/wiki/MiSeq_SOP](https://www.mothur.org/wiki/MiSeq_SOP)

The following commands will be at the 'mothur' command line

```
mothur >
```

1. We have paired end data from this sequencing, so the first thing we'll do is assemble the pairs together to make contigs. The file **stability.files** is the file that has the information about what pairs go together and what the sample name is. We use the infromation in that file to do the assembling. If you have all your data in one directory and the beginning of your sequence name is something meaningful, you can use the mothur command ```make.file``` to create that file. Or you can create it by hand or another script. Here we'll start with that file to make the contigs.
```
make.contigs(file=stability.files, processors=8)
```

2. Get information about the files
```
summary.seqs(fasta=stability.trim.contigs.fasta)
```

3. Sequences should be about 250 base pairs, so we want to get rid of any sequence that are too long because they are likely errors. Also, work here at the Bay Paul Center showed that if there are any ambiguous base pairs in the sequence it's likely to be poor quality, so we'll remove sequences that have ambiguous base pairs.
```
screen.seqs(fasta=stability.trim.contigs.fasta, group=stability.contigs.groups, maxambig=0, maxlength=275)
```

4. Here we'll type out the names of the files in the commands, but mothur remembers that last time you defined a particular variable, like the fasta file. You can see the current state of that memory with ```get.current```.
```
get.current()
```

5. To make things easier to process, we want to reduce the number of sequences we're working with. There are a lot of repeated sequenes, so, we can just keep the unique sequences in the FASTA file we're working with, and create a new file where we keep track of how many times we see these unique sequences in each sample. We'll use this information at the end when we're calculating the number of times we see each OTU in each sample.
```
unique.seqs(fasta=stability.trim.contigs.good.fasta)

count.seqs(name=stability.trim.contigs.good.names, group=stability.contigs.good.groups)
```

6. See a summary
```
summary.seqs(count=stability.trim.contigs.good.count_table)
```

7. The next step is to align these sequences to a reference. We have the reference from Silva, but it's the full length of the 16S gene, and we don't need all that. So we're going to create a shorter version of the reference to use for the next steps. Because we know the primers we used, we know our sequence is between positions 11894 and 25319.
```
pcr.seqs(fasta=silva.bacteria.fasta, start=11894, end=25319, keepdots=F, processors=8)
```

8. Now align our sequences to this reference and look at summary.seqs. We see that now our sequences have different start and end points, because they're aligned onto this reference.
```
align.seqs(fasta=stability.trim.contigs.good.unique.fasta, reference=silva.bacteria.pcr.fasta)

summary.seqs(fasta=stability.trim.contigs.good.unique.align, count=stability.trim.contigs.good.count_table)
```

9. We're just going to keep the sequences that meet our expectations with alignment and aren't too short. We're also going to eliminate any sequences that have homopolymers longer than 8 because they are likely erroneous. 
```
screen.seqs(fasta=stability.trim.contigs.good.unique.align, count=stability.trim.contigs.good.count_table, summary=stability.trim.contigs.good.unique.summary, start=1968, end=11550, maxhomop=8)

summary.seqs(fasta=current, count=current)
```

10. Filter the sequences to remove any overhangs at the end and any columns that only contain a gap in all the sequences.
```
filter.seqs(fasta=stability.trim.contigs.good.unique.good.align, vertical=T, trump=.)
```

11. Now that things are aligned, we might have more identical sequences, so run unique.seqs again.
```
unique.seqs(fasta=stability.trim.contigs.good.unique.good.filter.fasta, count=stability.trim.contigs.good.good.count_table)
```
12. Pre-cluster the sequences. Basically identifying sequences that are only 2 or fewer base pairs different. Speeds things up since clustering has more than this many differences anyway.
```
pre.cluster(fasta=stability.trim.contigs.good.unique.good.filter.unique.fasta, count=stability.trim.contigs.good.unique.good.filter.count_table, diffs=2)
```

13. Look for chimeric sequences. Here's we'll use 'uchime' for the chimera algorithm, but there are several. The mothur SOP uses 'vsearch'.
```
chimera.uchime(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.count_table, dereplicate=t)
```

14. Remove the sequences identified as chimeric
```
remove.seqs(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.fasta, accnos=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.accnos)

summary.seqs(fasta=current, count=current)
```

15. Classify the sequences. We want to identify any that are classifying outside bacteria, since that's not what the primers are supposed to amplify, so those sequences should be removed.
```
classify.seqs(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.count_table, reference=trainset9_032012.pds.fasta, taxonomy=trainset9_032012.pds.tax, cutoff=80)
```

16. Remove those undesirable sequences
```
remove.lineage(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.count_table, taxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pds.wang.taxonomy, taxon=Chloroplast-Mitochondria-unknown-Archaea-Eukaryota)

summary.tax(taxonomy=current, count=current)
```

17. **A bit of a side note:** Assessing error rates. We can use the Mock community data to assess error rates, because we know how many unique sequences we should expect. First, pull out just the Mock community sequences.
```
get.groups(count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.pick.count_table, fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.fasta, groups=Mock)
```

18. Check the error rate
```
seq.error(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.pick.pick.count_table, reference=HMP_MOCK.v35.fasta, aligned=F)
```

19. We can also do our OTU clustering for just these sequences to see how many OTUs we have found in the mock community. (We expect 20)
```
dist.seqs(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.fasta, cutoff=0.03)

cluster(column=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.dist, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.pick.pick.count_table)

make.shared(list=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.opti_mcc.list, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.pick.pick.count_table, label=0.03)

rarefaction.single(shared=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.opti_mcc.shared)
```

20. **Back to our regular workflow** Remove the Mock community data, because we don't want that in our community analysis.
```
remove.groups(count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.pick.count_table, fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.fasta, taxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pds.wang.pick.taxonomy, groups=Mock)
```

21. Create a distance matrix. This will be used for clustering.
```
dist.seqs(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.fasta, cutoff=0.03)
```

22. Cluster the sequences. We'll use 'cluster-split'.
```
cluster.split(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.pick.pick.count_table, taxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pds.wang.pick.pick.taxonomy, splitmethod=classify, taxlevel=4, cutoff=0.03)
```

23. Create a file with that has the OTUs in each group.
```
make.shared(list=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.opti_mcc.unique_list.list, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.pick.pick.count_table, label=0.03)
```

24. We likely want to know who these OTUs are , so we can classify the sequences
```
classify.otu(list=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.opti_mcc.unique_list.list, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.pick.pick.count_table, taxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pds.wang.pick.pick.taxonomy, label=0.03)
```

25. Now we want to know how many sequences we have in each sample.
```
count.groups(shared=current)
```

26. We use this information to do sub-sampling. *Insert discussion on sub-sampling*. Create a sub-sampled file for our further OTU analyses.
```
sub.sample(shared=current, size=2392)
```

27. Now we've created a 'shared' file which is the name of each sample as rows and each OTU as a column. In each cell is the count of the number of times that OTU was found in that sample (after sub-sampling). We can use this file for further statistical analysis.











