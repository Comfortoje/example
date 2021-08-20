## **Dataset**

Download files from *[here](https://zenodo.org/record/3862793#.YRze2XUvNH4) or use the wget command in the linux

- `wget https://zenodo.org/record/3862793/files/ENCFF933NTR.bed.gz`
- `wget https://zenodo.org/record/3862793/files/SRR891268_chr22_enriched_R1.fastq.gz`
- `wget https://zenodo.org/record/3862793/files/SRR891268_chr22_enriched_R2.fastq.gz`

* ### Obtaining Annotation for Hg38

Go to <http://genome.ucsc.edu/cgi-bin/hgTables> and set the parameters as-
- **"clade"**: `Mammal`
- **"genome"**: `Human`
- **"assembly"**: `Dec. 2013 (GRCh38/hg38)`
- **"group"**: `Genes and Gene Prediction`
- **"track"**: `All GENCODE V37`
- **"table"**: `Basic`
- **"region"**: `position chr22`
- **"output format"**: `all fields from selected table`
- **"output filename:"** `chr22`
- **"file type returned:"** `gzipped compressed`

And then select Get output 

Thus, chr22.gz file will be downloaded.

* ### **Converting chr22 file into a bed file**

1. Unzip the downloaded chr22.gz using `gunzip chr22.gz`  command
2. `awk -F "\t" 'OFS="\t" {print $3, $5, $6, $13, $12, $4 > ("chr22.bed")}' chr22` (to get only expected columns into a newly created chr22.bed file)
3. Output should be as follows-

## **Data preprocessing**

You can unzip the sequence files with gunzip
- `$ gunzip SRR891268_chr22_enriched_R1.fastq.gz`
- `$ gunzip SRR891268_chr22_enriched_R2.fastq.gz`

## **Quality Control with FASTQC**

* Download the FastQC module

*Note: FASTQC requires java and javac installed for implementation and you need to run the fastqc file from the folder (using the relative/absolute links to the sequence reads)*

`$ sudo apt install default-jre`
`$ sudo apt install default-jdk`
* Make the "fastqc" an executable file
`$ chmod 755 fastqc`
* Run the fastqc on all sequenced reads from its folder
`$ fastqc SRR891268_chr22_enriched_R1.fastq SRR891268_chr22_enriched_R2.fastq`  
* The report for each file is generated as an html file and a zip file containing more files that can be customised for reports. Look into the html files

## **Adapter Trimming**

The fastqc report  indicates the presence of an overrepresented sequence and fastqc identifies it as "Nextera Transposase Sequence ''. This sequence is similar to but longer than the one given in the tutorial.

`## SRR891268_chr22_enriched_R1 = CTGTCTCTTATACACATCTCCGAGCCCACGAGACTAAGGCGAATCTCGTA (fastqc)`
`## SRR891268_chr22_enriched_R1 = CTGTCTCTTATACACATCTCCGAGCCCACGAGAC (Galaxy tutorial)`
`## SRR891268_chr22_enriched_R2 = CTGTCTCTTATACACATCTGACGCTGCCGACGAGTGTAGATCTCGGTGGT (fastqc)`
`## SRR891268_chr22_enriched_R2 = CTGTCTCTTATACACATCTGACGCTGCCGACGA (Galaxy tutorial)`

## **Adapter Trimming with Cutadapt** 

* Install cutadapt running- 
`$ sudo apt install cutadapt`
* For paired end trimming- 
`$ cutadapt -a CTGTCTCTTATACACATCTCCGAGCCCACGAGAC -A CTGTCTCTTATACACATCTGACGCTGCCGACGA --minimum-length 20 -q 20 -o trimmed_1.fastq`
`-p trimmed_2.fastq SRR891268_chr22_enriched_R1.fastq` 
`SRR891268_chr22_enriched_R2.fastq`

## **Mapping and Alignment**

* Pulling the sequence for chromosome 22 for indexing and mapping
`$ wget --timestamping` 
`'ftp://hgdownload.cse.ucsc.edu/goldenPath/hg38/chromosomes/chr22.fa.gz' -O chr22.fa.gz`
* For mapping to chr22-`
1. install bowtie2
2. Create index for Chromosome 22: `bowtie2-build chr22.fa.gz indexed_chr22`
3. Start mapping for the parameters specified by Galaxy: `bowtie2 --very-sensitive --maxins 1000 --dovetail -x indexed_chr22 -1 trimmed_1.fastq -2 trimmed_2.fastq -S Aligned_output.sam` 
4. Output should be as follows-

## **Filtering Mapped Reads**  

1. Install samtools
2. `samtools view -q 30 -f 0x2 -b -h Aligned_output.sam > Filtered_output.bam` 
  *Note:This will filter out uninformative reads (Mapping quality >= 30 & Properly Paired)*

## **Mark Duplicate Reads**

* Download picard.jar in your working folder from *[here](https://github.com/broadinstitute/picard/releases/download/2.26.0/picard.jar) 
* From that directory, run `java -jar picard.jar -h` to check whether it works *(you can skip this step*)
* For sorting the output file from last step use- `samtools sort -T temp -O bam -o filtered_output_sorted.bam Filtered_output.bam`
* Finally, run *java -jar picard.jar MarkDuplicates I=filtered_output_sorted.bam O=marked_dup.bam M=marked_dup.metrics.txt`  for marking duplicates      
* If you can have a look into the metrics in the metrics.txt file  

## **Check Insert Sizes**

Check Insert Size tells us the size of the DNA fragment the read pairs came from. For this step we have to make a plot of the frequencies of the reads in the bam file to observe the peaks around where there are likely Tn5 transposase activities into nucleosome-free regions.
`$ sudo apt install r-base`
`$ java -jar picard.jar CollectInsertSizeMetrics I=marked_dup.bam O=chart.txt` `H=insertSizePlot.pdf M=0.5`






