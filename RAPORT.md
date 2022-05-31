


Data used for the experiment was downloaded from https://trace.ncbi.nlm.nih.gov/Traces/sra/?study=SRP003355.  
There were mentioned three different yeast strains: wild type VAC22 to demonstrate the detection of structural variants, VAC6 - wild type before mutation and VAC6 mutated.  
In the research from which data was taken the goal of next generation sequencing was to determine functional mutations from a large excess of polymorphisms, incidental mutations and other sequencing errors.  
The informations were stored in pair-end fasta files.
Instrument: Illumina Genome Analyzer
Strategy: WGS
Source: GENOMIC
Selection: RANDOM
Layout: PAIRED


# Analysis’s steps:

1. Downloading data for the research from SRA repository
To download the files from SRA database fastdump was used, because it has an option for determining how much reads we want to get. Flag split-3 was added, so in the result we got two files from both strands.  
Command used for downloading 1M of records from SRR064545, SRR064546, SRR064547 runs:  
*fastq-dump --split-3 -X 1000000 SRR064545*  

3. Checking the quality of the files with fastqc
On fasta files freshly downloaded from SRA database I performed quality tests using fastqc, from which I could read quality scores. Per base sequence quality showed that many bases of the sequences had quality lower than 20, so they needed to be cutted.  
Command used for quality analysis:  
*fastqc SRR064545_1.fastq SRR064545_2.fastq  SRR064546_1.fastq SRR064546_2.fastq  SRR064547_1.fastq SRR064547_2.fastq*

5. Trimming the sequences by using trimmomatic
The output of trimmomatic program, which was used to trim bases with low quality, was two fastq files for each sequence. The encoding was Illumina 1.9 what we needed to mention in the command line, as well with information how big we want our sliding window to be and what was the minimal lenght of the sequences which we wanted to spare after cutting. First file, paired gave us info about which sequences from both strands survived and unpaired where the ones that was disqualified. For further operations only the paired files were used. 
Command used for trimming low quality bases (for each record):  
**java -jar** */trimmomatic-0.40-rc1.jar PE SRR064547_1.fastq SRR064547_2.fastq SRR064547_1_paired.fastq SRR064547_1_unpaired.fastq SRR064547_2_paired.fastq SRR064547_2_unpaired.fastq* **ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:22**



7. Checking the quality after trimming by fastqc
*/ensemblgenomes/pub/release-53/fungi/fasta/saccharomyces_cerevisiae/dna/*



9. Aquiring the reference genome from ensemble fungi database
Reference genome was downloaded in parts, separate file for each chromosome. I used *cat > /* command to concatenate them all into one, so it could be used for the aligment.
Database used for aquiring reference genome:  
*/ensemblgenomes/pub/release-53/fungi/fasta/saccharomyces_cerevisiae/dna/* 


11. Aligment to reference genome with Burrows-Wheeler Aligner
**bwa index** *Saccharomyces_cerevisiae.R64-1-1.dna.chromosom.fa*.  
**bwa mem** *Saccharomyces_cerevisiae.R64-1-1.dna.chromosom.fa SRR064545_1_paired.fastq SRR064545_2_paired.fastq > aln2_4545.sam*
I aligned the paired sequences to reference genome using bwa program. The sam format is a bit too big for being stored, so it was immediately transformed into BAM format, which is more sufficient for next operations. For next tools, bam file needed to be indexed, sorted and had marked duplicates, so it could be used for the detection of polymorphism.
**samtools view** *-S -b aln_4545.sam > aln_4545.bam*



13. Post-aligment processing with samtools package
In txt format I have saved statistics after aligment, which were performed with samtools flagstat. I could read that the percentage of primary mapped sequences was around 100% and 60-80% was properly paired, so I decided to use those files for further operations.
**samtools fixmate** *-r -m aln_4545.bam aln_4545_fix.bam*  
**samtools sort** *aln_4545_fix.bam -o aln_4545_sorted.bam* 
**samtools markdup** *-r aln_4545_sorted.bam aln_4545_marked.bam* 
**samtools index** *aln_4545_marked.bam*   


15. Polymorphism detection with bcftools
Bcftools was used to detect polymorphisms in the given sequences. Even if bcf format is smaller, the next tool required vcf format, so we transformed it in the same command line. The results were needed to be visualised, so I could interprete them properly.
**bcftools mpileup -f** */genome_reference/Saccharomyces_cerevisiae.R64-1-1.dna.chromosom.fa aln_4545_marked.bam* | **bcftools call** *-mv -Ob -o polymorphism_4545.bcf*  
**bcftools convert** *polymorphism_4547.bcf -o polymorphism4547.vcf*


17. Generating results in VPC format with Variant Effect Predictor.



19. Analysis of results 
