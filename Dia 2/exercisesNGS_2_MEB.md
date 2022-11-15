
> ## Filtering and Trimming
  


Most software for the analysis of HTS data is freely available to users. Nonetheless, they often require the use of the command line in a Unix-like environment (seqtk is one such case). User-friendly desktop software such as [CLC](https://www.qiagenbioinformatics.com/products/clc-genomics-workbench/) or [Ugene](http://ugene.net/) is available, but given the quick pace of developmpent in this area, they are constantly outdated. Moreover, even with better algorithms, HTS analysis must often be run in external servers due to the heavy computational requirements. One popular tool is [Galaxy](https://galaxyproject.org/), which allows even non-expert users to execute many different HTS analysis programs through a simple web interface.

**TASK**: Let's use Galaxy to run some bioinformatic tools. Open the [url](https://usegalaxy.org/) on your browser. You should see the Galaxy interface on your web browser. Click on the upload icon on the top left of the interface. Upload into Galaxy the files MiSeq_76bp.fastq.gz and MiSeq_250bp.fastq.gz. You should now seem them on your history in the right panel. You can visualize their content by pressing the view data icon (the eye icon). After you have your data, you're ready to run some tools on your data. The tools are listed on the left panel. Search for fastqc on the tool search bar on the left panel. By clicking on the tool you should have in the middle the interface to run fastQC. To run fastc you just need to select the fastq file and press "Execute". Run fastqc on the fastq files you uploaded and see the result. Still in galaxy again, search for and run "seqtk trimfq" on the file MiSeq_250bp.fastq with the same parameters as you used in the command line. 


As we saw before, sequencing machines (namely, the illumina ones) require that you add specific sequences (adaptors) to your DNA so that it can be sequenced. For many different reasons, such sequences may end up in your read, and you need to remove these artifacts from your sequences.

**Exercise 7**: How can adaptors appear in your sequences? Take the sample MiSeq_76bp.fastq.gz as an example. 
<details><summary>Click Here to see the answer</summary>
	When the fragment being read is smaller than the number of bases the sequencing machine reads, then it will start reading the bases of the adaptor that is attached to all fragments so they can be read by the machines. In the case of MiSeq_76bp, the fragments were all 36bp, and since 76bp were being read, the remaining bases belong to the illumina adaptor that was used.
</details>
<br/>

There are many programs to remove adaptors from your sequences, such as [cutadapt](https://cutadapt.readthedocs.org/en/stable/). To use them you need to know the adaptors that were used in your library preparation (eg. Illumina TruSeq). For this you need to ask the sequencing center that generated your data.

**Exercise 8**: In Galaxy, use cutadapt to remove adaptors from MiSeq_76bp.fastq.gz. In this sample, we know that we used the illumina adaptor GTGACTGGAGTTCAGACGTGTGCTCTTCCGATCT, so try to remove this from the 3' end of reads and see the impact of the procedure using FastQC. For this, you need to insert a new adapter in 3', and in the source, select "Enter a custom sequence" (you don't need to add a name, just paste the sequence). 

**Exercise 9**: What happened? To answer, look at the report from cutadapt, and use FastQC on the fastq that is output by cutadapt. 
<details><summary>Click Here to see the answer</summary>
	Almost no read was affected. This is because what you get is a readthrough, so what is actually in the read is the reverse complement of the adaptor. Now, try the same procedure but with AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC (reverse complement of the previous). This time, most reads should have had the adaptor removed. This [tool](https://www.bioinformatics.org/sms/rev_comp.html) is very useful to obtain the reverse complement. 
</details>
<br/>




[Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) is a tool that performs both trimming of low quality reads, as well as adaptor removal. Moreover, it already contains a library of commonly used adaptors, so you don't need to know their sequence. Similar to FastQC, it is a java program, so you can use it in any operating system (such as Windows and Mac), although unlike FastQC it needs to be run only using the commandline. 


**Exercise 10**: Find and select the Trimmomatic tool in Galaxy. What different operations can you perform with Trimmomatic that use the base quality information? 
<details><summary>Click Here to see the answer</summary>
You can perform the following operations with Trimmomatic (either isolated, or in combination):

	* ILLUMINACLIP: Cut adapter and other illumina-specific sequences from the read
	
	* SLIDINGWINDOW: Perform a sliding window trimming, cutting once the average quality within the window falls below a threshold
        
	* MINLEN: Drop the read if it is below a specified length

	* LEADING: Cut bases off the start of a read, if below a threshold quality

	* TRAILING: Cut bases off the end of a read, if below a threshold quality

	* CROP: Cut the read to a specified length

	* HEADCROP: Cut the specified number of bases from the start of the read

	* AVGQUAL: Drop the read if the average quality is below a specified value

	* MAXINFO: Trim reads adaptively, balancing read length and error rate to maximise the value of each read
	
</details>
<br/>


**TASK**: Let's use Trimmomatic in Galaxy to remove low quality bases from MiSeq_250bp.fastq.gz, as well as the remainings of illumina Nextera adaptors that are still left in some of the reads. This fastq file is in the commonly used Phred score, so you can change its file type to 'fastqsanger'. Now Trimmomatic should find it. Let's perform the default operation "Sliding Window" of size 4 and average quality 20. Let's also remove the adaptors. For this, select 'Yes' on 'Perform initial ILLUMINACLIP step'. The select "Nextera (paired end)" and leave the rest of the parameters as they were. Finally, you can click on Execute.

**Exercise 11**: What happened? To answer, use FastQC on the fastq output by Trimmomatic. 
<details><summary>Click Here to see the answer</summary>
	The base quality distribution improved. Moreover, the few Nextera primers in the end of the reads also disappeared. Nonetheless, read length is now shortened, and we have fewer reads than before.
</details>
<br/>
<br/>


**Exercise 12**: Let's use Trimmomatic in Galaxy with a paired-end dataset. Upload the files paired_end_example_1.fastq.gz and paired_end_example_2.fastq.gz. Change their types to 'fastqsanger'. In Trimmomatic, select 'Paired-end (two separate input file)'. Perform the same operations as before.

<br/>

**Exercise 13**: Now, you get 4 files as output from Trimmomatic. Can you explain what these are? 
<details><summary>Click Here to see the answer</summary>
You get the following paired files: Trimmomatic on paired_end_example_1.fastq (R1 paired) and Trimmomatic on paired_end_example_2.fastq (R2 paired). These contain the paired reads that "survived" the quality operation from both the forward and the reverse and could therefore be kept as pairs. Then, you have the cases where just one of the pairs was removed because of low quality. In this case, it cannot be kept as pair, but in a separate "isolated" file, both for the forward (Trimmomatic on paired_end_example_1.fastq (R1 unpaired)) and the reverse (Trimmomatic on paired_end_example_2.fastq (R2 unpaired)).
</details>
<br/>

**Exercise 14**: From the "isolated" reads resulting from Trimmomatic, which one has more reads? Why is that?
<details><summary>Click Here to see the answer</summary>
The forward file (Trimmomatic on paired_end_example_1.fastq (R1 unpaired)) has more reads, because the reverse reads have usually less quality, and therefore are more likely to be removed in the filtering process.
</details>
<br/>
<br/>
<br/>


**NOTE**:  Assess how well you achieve the learning outcome. For this, see how well you responded to the different questions during the activities and also make the following questions to yourself.

* Could you manually remove low quality bases from the end of a read in the fastq format? 

* Did you broadly understand the challenges of removing bad quality bases from reads? 

* Could you use cutadapt to remove adaptors from reads in a fastq file? 

* Could you use trimmomatic to remove bad quality bases and remove adaptors from reads in a fastq file? 

* Did you understand the issue of manipulating paired-end fastq files? 
<br/>
<br/>


<br/>
<br/>

> ## Align HTS data against a genome

> ### Use the BWA aligner to align HTS data against a genome

One of the most common applications of NGS is resequencing, where we want to genotype an individual from a species whose genome has already been assembled (a reference genome), such as the human genome, often with the goal to identify mutations that can explain a phenotype of interest.

After obtaining millions of short reads, we need to align them to a (sometimes large) reference genome. To achieve this, novel, more efficient, alignment methods had to be developed. One popular method is based on the [burrows-wheeler transform](https://en.wikipedia.org/wiki/Burrows%E2%80%93Wheeler_transform) and the use of efficient data structures, of which [bwa](http://bio-bwa.sourceforge.net/) and [bowtie](http://bowtie-bio.sourceforge.net/index.shtml) are examples. They enable alignment of millions of reads in a few minutes, even in a laptop.

**NOTE:** Aligners based on the burrows-wheeler transform makes some assumptions to speed up the alignment process. Namely, they require the reference genome to be very similar to your sequenced DNA (less than 2-5% differences). Moreover, they are not optimal, and therefore sometimes make some mistakes.


> ###  The SAM/BAM alignment format

To store millions of alignments, researchers also had to develop new, more practical formats. The [Sequence Alignment/Map (SAM) format](https://samtools.github.io/hts-specs/SAMv1.pdf) is a tabular text file format, where each line contains information for one alignment.
 

**Exercise 15** Upload the reference genome "Escherichia coli str. K-12 substr. MG1655, complete genome" from NCBI repository as fasta file. Consider the Trimmomatic on Forward reads (R1 paired) and Trimmomatic on Reverse reads (R2 paired) that you obtained before. Next, perform an alignment with _bwa mem_ of the paired reads (you need to select the option of paired reads) against the reference genome (choose one from history). Set read groups information: Automatically assign ID.
Leave the rest of the settings on default  Next, download the bam file that was created. Also download the companion bai index file (you need to press on the download icon to have the option to download the bam and the bai files). 
<br/>
<br/>

**NOTE**:  Assess how well you achieve the learning outcome. For this, see how well you responded to the different questions during the activities and also make the following questions to yourself.

* Did you broadly understand the challenges of aligning millions of short reads to a genome? 

* Did you broadly understand the assumptions underlying the use of burrows-wheeler aligners?

* Could you use bwa to align reads to a reference genome? 

* Do you know what is the most common alignment format these aligners use? 

* Do you broadly understand the contents of a SAM/BAM file and the difference between SAM and BAM? 
<br/>
<br/>






