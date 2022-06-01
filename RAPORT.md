# RAPORT

Source: ttps://trace.ncbi.nlm.nih.gov/Traces/sra/?study=SRP003355  
Instrument: Illumina Genome Analyzer  
Strategy: WGS  
Source: GENOMIC  
Selection: RANDOM  
Layout: PAIRED  
  
Research was performed on three strains. They were two wild types - VAC22 and VAC6, the second one was mutated to create another stain. The goal of the research was to find functional mutations from wide range of polymorphisms, incidental mutations and other errors.

# Analysis’s steps:

### 1. Downloading data  
Data was downloaded from SRA database in fasta format. The files were pair-ended. The tool which was used was fastdump with flag split-3, which allowed to split results into two files. The amount of reads were limited to 1000 000.  
  
Downloaded records' names: SRR064545, SRR064546, SRR064547
Command: *fastq-dump --split-3 -X 1000000 SRR064545*  
  
### 2. Quality Check  
Quality tests were performed using fastqc. Some of the results indicated the quality of bases lower than 20. The sequences were determined for cleaning.  
  
Command: *fastqc SRR064545_1.fastq SRR064545_2.fastq  SRR064546_1.fastq SRR064546_2.fastq  SRR064547_1.fastq SRR064547_2.fastq*

### 3. Sample trimming
Method chosen for increasing the quality was trimming of low quality fragments. Program used for this task was trimmomatic. The action was performed based on Illumina 1.9 encoding.The minimal lenght of the sequences was established as 22 bases with the sliding window of 4. As the result it produced four fastq files for each sequence - two of them described as paired were used for further analyses. Unpaired files were eliminated from analyses. 
  
Command: **java -jar** */trimmomatic-0.40-rc1.jar PE SRR064545_1.fastq SRR064545_2.fastq SRR064545_1_paired.fastq SRR064545_1_unpaired.fastq SRR064545_2_paired.fastq SRR064545_2_unpaired.fastq* **ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:22**

### 4. Second quality check
Second quality check was performed as previous one. The results were satisfying. The files were determined for further analysis.  
  
Command: *fastqc SRR064545_paired_1.fastq SRR064545_paired_2.fastq  SRR064546_paired_1.fastq SRR064546_paired_2.fastq  SRR064547_paired_1.fastq SRR064547_paired_2.fastq*

### 5. Reference genome
Reference genome was downloaded from ensemble fungi database (/ensemblgenomes/pub/release-53/fungi/fasta/saccharomyces_cerevisiae/dna/). The genome was splitted between files for each chromosome. For further operations they were joined together in one file.

### 6. Aligment to reference genome 
Paired sequences were aligned to reference genome using Burrows-Wheeler Aligner. As the result it produced sam file, which was transformed to bam file to save computer's memory. This format was necessary for further operations.  
  
Commands: **bwa index** *Saccharomyces_cerevisiae.R64-1-1.dna.chromosom.fa*      
**bwa mem** *Saccharomyces_cerevisiae.R64-1-1.dna.chromosom.fa SRR064545_1_paired.fastq SRR064545_2_paired.fastq > aln2_4545.sam* | *samtools view** *-S -b aln_4545.sam > aln_4545.bam*

### 7. Post-aligment processing    
Several operations were performed on bam files, including sorting, marking and indexing. This was needed for the detection of polymorphism.
  
Commands:  
**samtools fixmate** *-r -m aln_4545.bam aln_4545_fix.bam*  
**samtools sort** *aln_4545_fix.bam -o aln_4545_sorted.bam* 
**samtools markdup** *-r aln_4545_sorted.bam aln_4545_marked.bam*   
**samtools index** *aln_4545_marked.bam*  

### 8. Third quality check
Third quality check was performed using samtools flagstat. Results were saved in txt files.

In txt format I have saved statistics after aligment, which were performed with samtools flagstat. I could read that the percentage of primary mapped sequences was around 100% and 60-80% was properly paired, so I decided to use those files for further operations.
  
Command: **samtools flagstat** *aln_4545_marked.bam > 4545_stats.txt*

### 9. Polymorphism detection
To detect various polymorphisms bfctools were used. The output was transformed into vcf format. 
  
Commands:  
**bcftools mpileup -f** */genome_reference/Saccharomyces_cerevisiae.R64-1-1.dna.chromosom.fa aln_4545_marked.bam* | **bcftools call** *-mv -Ob -o polymorphism_4545.bcf*  
**bcftools convert** *polymorphism_4547.bcf -o polymorphism4547.vcf*

### 10. Visualisation


# Results
