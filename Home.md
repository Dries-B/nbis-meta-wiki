# Contents
- [Workflow outline](#workflow-outline)
- [Getting started](#getting-started)

# Workflow outline
## 1. Preprocessing
This workflow can perform preprocessing of paired- and/or single-end metagenomic data (in fastq-format) using *e.g.*:
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

# Getting started

## From GitHub
To start using the workflow either clone the latest version of the repo:

```bash
git clone https://github.com/NBISweden/nbis-meta.git
```

or download the latest release from the [release page](https://github.com/NBISweden/nbis-meta/releases) and extract the archive. 

Then change directory into the `nbis-meta` folder and create the core conda environment:

```bash
cd nbis-meta
conda env create -f environment.yml
```

## From DockerHub
You may also pull the latest Docker image:

```bash
docker pull nbisweden/nbis-meta:2.0
```

to get version `2.0`.


## What's next?
You are now ready to start using the workflow! Check out the [How-to page](https://github.com/NBISweden/nbis-meta/wiki/How-to-run-the-workflow), see [how to create a sample list](https://github.com/NBISweden/nbis-meta/wiki/Defining-your-samples-in-the-sample-list) to start working with your own data or delve into the [configuration parameters](https://github.com/NBISweden/nbis-meta/wiki/Workflow-configuration).