# Documentation_Genetics_Globant

# 01_Trimmomatic

Trimmomatic V0.32 is a fast, multithreaded command line tool that can be used to trim and crop Illumina (FASTQ) data as well as to remove adapters.

The current trimming steps/methods are:

**ILLUMINACLIP**: Cut adapter and other illumina-specific **sequences** from the read.

**SLIDINGWINDOW**: Performs a sliding window trimming approach. It starts
scanning at the 5‟ end and clips the read once the average quality within the window
falls below a threshold.

**MAXINFO**: An adaptive quality trimmer which balances read length and error rate to
maximise the value of each read.

**LEADING**: Cut bases off the start of a read, if below a threshold quality.

**TRAILING**: Cut bases off the end of a read, if below a threshold quality.

**CROP**: Cut the read to a specified length by removing bases from the end.

**HEADCROP**: Cut the specified number of bases from the start of the read.

**MINLEN**: Drop the read if it is below a specified length.

**AVGQUAL**: Drop the read if the average quality is below the specified level.

**TOPHRED33**: Convert quality scores to Phred-33.

**TOPHRED64**: Convert quality scores to Phred-64.


### ILUMINACLIP

Identifying adapter or other contaminant sequences within a dataset is inherently a trade off between sensitivity (ensuring all contaminant sequences are removed) and specificity (leaving all non-contaminant sequence data intact)

the most common cause of adapter contamination is sequencing of a DNA fragment which is shorter than the read length. This is known as “adapter read-through”.

Trimmomatic combines two approaches to detect technical sequences.  The first, referred to as ‘simple mode’, conducts a local alignment of technical sequences (adapter sequences are specified in an adapter sequence file provided to the module) against a read.  If the alignment score exceeds a user-defined threshold (adapter.clip.simple.clip.threshold parameter), the portion of the read that aligns to the technical sequence plus the remainder of the read after the alignment (towards the 3’ direction) are trimmed from the read.

Trimmomatic’s second approach to technical sequence detection, referred to as “palindrome mode”, Palindrome mode can only be used with paired-end data. When a read-through occurs, both reads in a pair will contain an equal number of valid bases (i.e., not from adapter sequences) followed by contaminating sequence from opposite adapters.  The valid sequence in each of the pair’s reads will be reverse complements.  Trimmomatic’s palindrome mode uses these characteristics to identify contaminating technical sequences Operating in the palindrome mode, Trimmomatic prepends the Illumina adapter sequences to their respective reads in the paired-end data. The resulting sequences are then globally aligned against one another.  A high scoring alignment (greater than adapter.clip.palindrome.threshold) indicates that the first parts of each read are reverse complements of one another and the remaining parts of the reads match their respective adapters.  Read bases matching the adapters are removed.

Trimmomatic uses a “seed and extend” method for alignment detection and scoring in both the simple and palindrome modes.  Initial sequence comparisons are done using 16 base fragments from each sequence.  If the number of mismatches between seeds from the two sequences are less than or equal to a specified threshold (see seed adapter.clip.seed.mismatches in Parameters section), the full alignment scoring algorithm is run.

Parameters relevant to this operation are:

`adapter.clip.sequence.file`

`adapter.clip.seed.mismatches`

`adapter.clip.palindrome.clip.threshold`

`adapter.clip.simple.clip.threshold`

`adapter.clip.min.length`

`adapter.clip.keep.both.reads`



```
ILLUMINACLIP:<fastaWithAdaptersEtc>:<seed mismatches>:<palindrome clip
threshold>:<simple clip threshold>
```
__fastaWithAdaptersEtc__: specifies the path to a fasta file containing all the adapters,
PCR sequences etc. The naming of the various sequences within this file determines
how they are used. See the section below or use one of the provided adapter files.

__seedMismatches__: specifies the maximum mismatch count which will still allow a full
match to be performed.

__palindromeClipThreshold__: specifies how accurate the match between the two 'adapter
ligated' reads must be for PE palindrome read alignment.

__simpleClipThreshold__: specifies how accurate the match between any adapter etc.
sequence must be against a read.

ILLUMINACLIP also supports two additional optional parameters, which affect palindrome
mode only.

```
ILLUMINACLIP:<fastaWithAdaptersEtc>:<seed mismatches>:<palindrome clip
threshold>:<simple clip threshold>:<minAdapterLength>
```
```
ILLUMINACLIP:<fastaWithAdaptersEtc>:<seed mismatches>:<palindrome clip
threshold>:<simple clip threshold>:<minAdapterLength>:<keepBothReads>
```
__minAdapterLength__: In addition to the alignment score, palindrome mode can verify
that a minimum length of adapter has been detected. If unspecified, this defaults to 8 bases,
for historical reasons. However, since palindrome mode has a very low false positive rate, this
can be safely reduced, even down to 1, to allow shorter adapter fragments to be removed.

__keepBothReads__: After read-though has been detected by palindrome mode, and the
adapter sequence removed, the reverse read contains the same sequence information as the
forward read, albeit in reverse complement. For this reason, the default behaviour is to
entirely drop the reverse read. By specifying „true‟ for this parameter, the reverse read will
also be retained, which may be useful e.g. if the downstream tools cannot handle a
combination of paired and unpaired reads.

### SLIDINGWINDOW

Perform a sliding window trimming, cutting once the average quality within the window falls
below a threshold. By considering multiple bases, a single poor quality base will not cause the
removal of high quality data later in the read.

```
SLIDINGWINDOW:<windowSize>:<requiredQuality>
```
**windowSize**: specifies the number of bases to average across.

**requiredQuality**: specifies the average quality required.

### MAXINFO

Performs an adaptive quality trim, balancing the benefits of retaining longer reads against the
costs of retaining bases with errors.

For many applications, the “value” of a read is a balance between three factors:

Minimal read length, Additional read length, Error sensitivity (see manual)

The motivation for the adaptive approach is that, in many scenarios, the incremental value of retaining a red's additional bases is related to the read length.  Very short reads are of little value since they are likely to align to multiple locations in a reference sequence; thus, it is beneficial to retain lower-quality reads early in a read so that the trimmed read is long enough to be informative,  However, beyond a certain length, retaining additional bases is less beneficial and could even be detrimental if the retention of low-quality reads leads to the read becoming unmappable.

The combined score is calculated for each possible read length, and the optimal score is used
to determine where the read should be trimmed. In practice, the different factors combine as
follows:

- At very short read lengths, the minimal read length factor dominates. This will heavily
penalize reads which are too short to be useful.

- Once the target read length has been achieved, the minimal read length factor penalty
becomes a modest bonus. However, once the read is significantly longer than the
target length, further bonuses from the minimal read length factor are limited, due to
the logistic function.

- The additional read length factor then provides a modest benefit as additional bases
are retained. This is countered by the increasing penalty as the “error free‟ probability
drops with increasing read length. The balance between these two factors is controlled
by the “strictness” parameter.

- For most reads, depending on the quality of the read and the “strictness” setting, the
increasing penalty from the likelihood of error exceeds the bonus of retaining
additional bases at some point, and the read is trimmed accordingly.

```
MAXINFO:<targetLength>:<strictness>
```

**targetLength**: This specifies the read length which is likely to allow the location of the read within the target sequence to be determined.

**strictness**: This value, which should be set between 0 and 1, specifies the balance between preserving as much read length as possible vs. removal of incorrect bases. A low value of this parameter (<0.2) favours longer reads, while a high value (>0.8) favours read correctness. 

**_The use of sliding.window at the same time as max.info is not recommended._**

###LEADING

Remove low quality bases from the beginning. As long as a base has a value below this
threshold the base is removed and the next base will be investigated.

```	
LEADING:<quality>
```

**quality**: Specifies the minimum quality required to keep a base.

### HEADCROP

Removes the specified number of bases, regardless of quality, from the beginning of the read.

```
HEADCROP:<length>
```

length: The number of bases to remove from the start of the read.

### MINLEN

This module removes reads that fall below the specified minimal length. If required, it should
normally be after all other processing steps. Reads removed by this step will be counted and
included in the „dropped reads‟ count presented in the trimmomatic summary.

```
MINLEN:<length>
```

**length**: Specifies the minimum length of reads to be kept.

### TOPHRED33

This (re)encodes the quality part of the FASTQ file to base 33.

```
TOPHRED33 (no further parameters)
```

### TOPHRED64

This (re)encodes the quality part of the FASTQ file to base 64.

```
TOPHRED64 (no further parameters)
```

### USING TRIMMOMATIC

Trimmomatic expects user to specify single ends (SE) or paired ends (PE) trimming operation.

Trimmomatic expects user to specify single ends (SE) or paired ends (PE) trimming operation.

$ trimmomatic

Usage paired end:

```
PE [-version] [-threads <threads>] [-phred33|-phred64] [-trimlog <trimLogFile>] [-summary <statsSummaryFile>] [-quiet] [-validatePairs] [-basein <inputBase> | <inputFile1> <inputFile2>] [-baseout <outputBase> | <outputFile1P> <outputFile1U> <outputFile2P> <outputFile2U>] <step1>...
```

Or single end: 

```
SE [-version] [-threads <threads>] [-phred33|-phred64] [-trimlog <trimLogFile>] [-summary <statsSummaryFile>] [-quiet] <inputFile> <outputFile> <step1>...
```

Trimmomatic expects the two input files, and then the names of the output files. See table below:

![001](https://user-images.githubusercontent.com/103220850/173130044-969ac3f9-4b7b-454c-ab09-16a584b459a4.jpg)

Trimmomatic main arguments and methods are:

<img width="499" alt="002" src="https://user-images.githubusercontent.com/103220850/173130713-208ff278-0d11-46aa-a085-82fedc7804f0.png">

An overview of the sliding window trimming is presented below.

![003](https://user-images.githubusercontent.com/103220850/173133082-91b7a8b3-1d16-425b-9429-19f324921b76.jpg)

A code example:

```
$ trimmomatic PE -threads 4 SRR_1056_1.fastq SRR_1056_2.fastq\
              SRR_1056_1.trimmed.fastq SRR_1056_1un.trimmed.fastq\
              SRR_1056_2.trimmed.fastq SRR_1056_2un.trimmed.fastq\
              ILLUMINACLIP:SRR_adapters.fa SLIDINGWINDOW:4:20
```

Read the table below to check arguments and methods and their meaning in the script above: 

![005](https://user-images.githubusercontent.com/103220850/173133339-be35efd9-ecc9-4c8b-b162-900273beeda1.jpg)

On the pages above you may have seen a lot of scores used for indicate quality, most of this scores are on the phred scale.

The Phred scale was originally used to represent base quality scores emitted by the Phred program in the early days of the Human Genome Project. Now they are widely used to represent probabilities and confidence scores in other contexts of genome science.

Most useable Phred-scaled base quality scores range from 2 to 40, However, Phred-scaled quality scores in general can range anywhere from 0 to infinity. A higher score indicates a higher probability that a particular decision is correct, while conversely, a lower score indicates a higher probability that the decision is incorrect.

For many purposes, a Phred Score of 20 or above is acceptable, because this means that whatever it qualifies is 99% accurate, with a 1% chance of error.

![006](https://user-images.githubusercontent.com/103220850/173133496-cb7f5def-555b-406b-b772-c17c2c6be439.jpg)

# 02_Reference_Genome_Indexing

Indexing is widely used in bioinformatics workflows to improve performance. Typically it is applied to large data files with many records to improve the ability of tools to rapidly access random locations of the file. 

The most common example of this is the reference genome. Each alignment algorithm (and sometimes even different versions of the same algorithm) requires its own distinctive index. Having the right index for each tool is important, and trying to use an incorrect one is a common error encountered in this kind of analysis.

# 03_BWA_Alignment

BWA is a software package for mapping low-divergent sequences against a large reference genome, such as the human genome.

Depending on read length, BWA has different modes optimized for different sequence lengths:

**BWA-backtrack**: designed for Illumina sequence reads up to 100bp (3-step)

**BWA-SW**: designed for longer sequences ranging from 70bp to 1Mbp, long-read support and split alignment

**BWA-MEM**: shares similar features to BWA-SW, but BWA-MEM is the latest, and is generally recommended for high-quality queries as it is faster and more accurate. BWA-MEM also has better performance than BWA-backtrack for 70-100bp Illumina reads.

### Creating BWA-MEM index

The first step in the BWA alignment is to create an index for the reference genome. Similar to Bowtie2, BWA indexes the genome with an FM Index based on the Burrows-Wheeler Transform to keep memory requirements low for the alignment process.

The basic options for indexing the genome using BWA are:

_-p: prefix for all index files_

```
$ bwa index -p chr20 chr20.fa
```

**_Genome index is necesary for using the algorithms._**

### Using BWA 

Ejm:

```
	bwa mem 
	-M 
	-t 1 
	-R '@RG\tID:{FLOWCELL}.{LANE}\tPU:{FLOWCELL_BARCODE}.{LANE}.{SAMPLE}\tSM:{SAMPLE}\tPL:{PLATFORM}\tLB{LIBRARY}' 
	<genome_prefix> 
	<reads_1.fq> 
	<reads_2.fq> > 
	<samplename_bwa.sam>
```
Arguments and their respective meaning are shown in the table below:

| Argument        | Meaning     
| ------------- |:-------------:
| -M      | Mark shorter split hits as secondary (for Picard compatibility)
| -t INT	      | Number of threads      
| -R STR | Complete read group header line. ’\t’ can be used in STR and will be converted 				to a TAB in the output SAM. The read group ID will be attached to every read in 			the output. An example is ’@RG\tID:foo\tSM:bar’
| genome_prefix	      | Index for the reference genome generated from bwa index 
| <reads_1.fq>	      | Input reads pair ends 1 (files generated from Trimommatic)  
| <reads_2.fq>	      | Input reads pair ends 2 (files generated from Trimommatic)
| > <samplename_bwa.sam>	      | Output file

# 04_SAM BAM files conversion (samtools)

BAM (binary alignment and map) files contain the same information as SAM (sequence alignment and map) files, except they are in binary file format which is not readable by humans. On the other hand, BAM files are smaller and more efficient for software to work with than SAM files, saving time and reducing costs of computation and storage.

Conversion to BAM files is carried on with the samtools view comand (must have samtools installed), important flags are marked below:

ejm.

```
samtools view -S -b sample.sam > sample.bam

	-s Input in SAM format.
	-b Output in BAM format. 
```

Alternatively SAM to BAM conversion may be done with picard:

```
SamFormatConverter

	-I input file
	-O output file
```
# 05_Sort BAM

After conversion to a BAM file, the next steps are to sort and index the file. Indexing is crucial to save computing time and perform other steps more rapidly. Sorting is merely a rearrangement of the aligments in the BAM/SAM file  according to coordinates (position) or by name (ID). Sorting may be done with samtools or picard:

`samtools sort`

```
samtools sort sample.bam -o sample.sorted.bam

	-o output file
	-n sort reads by name
```

`SortSam (picard)`
```
java -jar picard.jar SortSam \
	INPUT=input.bam \
     	OUTPUT=sorted.bam \
     	SORT_ORDER=coordinate
```

The --SORT_ORDER argument is an enumerated type (SortOrder), which can have one of the following values:

* `queryname`
* `coordinate`
* `duplicate`

# 06_Alignement metrics

Several tools can be used in order to evaluate quality in BAM files, Samtools, Picard and Sambamba are some of the most useful and common:

Samtools

```samtools stats``` 

Picard

```CollectAlignmentSummaryMetrics```

Sambamba

```sambamba-flagstat``` 
	
Several tools can be used in order to evaluate quality in BAM files, Samtools, Picard and Sambamba are some of the most useful and common:

![007](https://user-images.githubusercontent.com/103220850/173136775-8d696ea5-488c-4d60-b9b5-f8a0a06c5408.jpg)

# 07_Mark and remove duplicates



```
```
### References

Trimmomatic Manual: V0.32 

https://gatk.broadinstitute.org/hc/en-us/articles/360035531872-Phred-scaled-quality-scores

http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf

https://carpentries-incubator.github.io/metagenomics/03-trimming-filtering/index.html

