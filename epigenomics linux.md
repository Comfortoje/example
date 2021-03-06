---
title: "team epigenomics linux"
output: word_document
---
## **Dataset**

Download files from <https://zenodo.org/record/3862793#.YRze2XUvNH4> or use the wget command in the linux

- `wget https://zenodo.org/record/3862793/files/ENCFF933NTR.bed.gz`
- `wget https://zenodo.org/record/3862793/files/SRR891268_chr22_enriched_R1.fastq.gz`
- `wget https://zenodo.org/record/3862793/files/SRR891268_chr22_enriched_R2.fastq.gz`

* ### Obtaining Annotation for Hg38

Go to <http://genome.ucsc.edu/cgi-bin/hgTables> and set the parameters as-
- **“clade”**: `Mammal`
- **“genome”**: `Human`
- **“assembly”**: `Dec. 2013 (GRCh38/hg38)`
- **“group”**: `Genes and Gene Prediction`
- **“track”**: `All GENCODE V37`
- **“table”**: `Basic`
- **“region”**: `position chr22`
- **“output format”**: `all fields from selected table`
- **“output filename:”** `chr22`
- **“file type returned:”** `gzipped compressed`

And then select Get output 

Thus, chr22.gz file will be downloaded.

* ### **Converting chr22 file into a bed file**

1. Unzip the downloaded chr22.gz using `gunzip chr22.gz`  command
2. `awk -F "\t" 'OFS="\t" {print $3, $5, $6, $13, $12, $4 > ("chr22.bed")}' chr22` (to get only expected columns into a newly created chr22.bed file)
3. Output should be as follows-
 


