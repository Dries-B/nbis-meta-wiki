## 1. Preprocessing
This workflow can perform preprocessing of paired- and/or single-end whole-genome shotgun metagenomic data (in fastq-format) using *e.g.*:
- Trimmomatic (adapter/quality trimming)
- Cutadapt (adapter trimming)
- SortMeRNA (rRNA filtering)
- Fastuniq (de-duplication)
- FastQC and MultiQC (read QC and report generation)

## 2a. Read-based classification
Preprocessed reads can be used for taxonomic classification and profiling using tools such as:
- Kraken2
- Centrifuge
- MetaPhlAn3

producing taxonomic profiles of the samples, as well as interactive krona plots.

## 2b. Assembly-based analysis
Preprocessed reads can also be assembled and analyzed further using tools such as:
- Megahit/MetaSPADES (for metagenomic assembly)
- prodigal (gene calling)
- pfam_scan, eggnog-mapper, Resistance Gene Identifier (protein level annotations)
- bowtie2 (mapping reads to contigs)
- featureCounts (assigning and counting mapped reads)
- edgeR + metagenomeSEQ (normalization of read counts for genes/features)
- contigtax + sourmash (taxonomic assignments)
- metabat2, CONCOCT, MaxBin2 (metagenomic binning)
- checkm (genome bin QC)
- GTDB-TK (genome bin phylogenetic assignments)
- fastANI (genome bin clustering)
