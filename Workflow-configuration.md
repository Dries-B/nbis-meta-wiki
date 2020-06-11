The workflow can be configured using a configuration file in `yaml` format. These are the configuration parameters and their default values

- [Sample list](#the-sample-list)
- [Paths](#paths)
- [Preprocessing](#preprocessing)
  - [Trimmomatic](#trimmomatic)
  - [Cutadapt](#cutadapt)
  - [SortMeRNA](#sortmerna)
- [Post-processing](#post-processing)
- [Assembly](#assembly)
  - [Megahit](#megahit)
  - [Metaspades](#metaspades)
- [Annotation](#annotation)
  - [Taxonomy](#taxonomy)
- [Binning](#binning)
  - [Maxbin](#maxbin)
  - [Checkm](#checkm)
  - [fastANI](#fastani)
- [Classification](#classification)
  - [Kraken](#kraken)
  - [Centrifuge](#centrifuge)
  - [MetaPhlan](#metaphlan)

## The sample list

- `sample_list: config/samples.tsv` 

Path to the sample list (**required**). See [Defining your samples in the sample list](https://github.com/NBISweden/nbis-meta/wiki/Defining-your-samples-in-the-sample-list) for instructions on how to format it for your data. By default this points to an example sample list for which the sequence data is generated as part of the workflow. Feel free to use it to get acquainted with the workflow.

## Paths

```yaml
paths:
  results: "results"
  temp: "temp"
```

- `results:`

Path for main results. Output from preprocessing, assembly etc goes here. Summarized report files (_e.g._ plots and tables) are placed in a sub-directory called `report/`.

- `temp:`

Paths for temporary output. On compute clusters this can be set to `$TMPDIR` or equivalent.

## Preprocessing
By default, the workflow runs `trimmomatic` for adapter and quality trimming, followed by `fastqc` for read statistics and [MultiQC](https://github.com/MultiQC) to aggregate and visualize logfiles. Depending on your needs you may however add, modify or remove preprocessing steps.

```yaml
preprocessing:
  fastqc: True
  trimmomatic: True
  cutadapt: False
  fastuniq: False
  phix_filter: False
  sortmerna: False
```

- `fastqc:`

Whether to run [Fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) on preprocessed reads? If you do not wish to produce a sample report summary (*e.g.* if you have data that has already been preprocessed or that you know is of sufficient quality), set this to `False`.

- `trimmomatic:`

Run [Trimmomatic](http://www.usadellab.org/cms/index.php?page=trimmomatic) to perform adapter/quality trimming of reads. (see specific settings [here](#trimmomatic))

- `cutadapt:`

If you want to perform adapter trimming with [cutadapt](https://github.com/marcelm/cutadapt/) **instead of** Trimmomatic, set this to `True`. Note that this means no quality trimming is done (see specific settings [here](#cutadapt))

- `fastuniq:`

Whether to run [fastuniq](https://sourceforge.net/projects/fastuniq/) to remove duplicates from paired-end samples.

- `phix_filter:`

Whether to map reads with bowtie2 against the phiX genome and remove reads that map concordantly.

- `sortmerna:`

Whether to run [SortMeRNA](https://github.com/biocore/sortmerna) to identify (and filter) rRNA sequences from samples (see specific settings [here](#sortmerna))

### Trimmomatic
```yaml
trimmomatic:
  trim_adapters: True
  pe:
    adapter: "TruSeq3-PE-2"
    adapter_params: "2:30:15"
    pre_adapter_params: ""
    post_adapter_params: "LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:31"
  se:
    adapter: "TruSeq3-SE"
    adapter_params: "2:30:15"
    pre_adapter_params: ""
    post_adapter_params: "LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:31"
```

- `trim_adapters:`

Whether to trim adapters in addition to quality trimming.

The remaining settings for Trimmomatic are divided into parameters specific to paired-end data (`pe`) or single-end data (`se`).

- `adapter:`

Adapter type to trim. For paired-end data choose from `NexteraPE-PE`, `TruSeq2-PE`, `TruSeq3-PE` and `TruSeq3-PE-2`. For single-end data choose from `TruSeq2-SE` and `TruSeq3-SE`.

- `adapter_params:`

Trimmomatic parameters for trimming adapters. This translates into the `<seed mismatches>:<palindrome clip threshold>:<simple clip threshold>` settings for Trimmomatic.

- `pre_adapter_params:`

Trimmomatic parameters for trimming prior to adapter removal.

- `post_adapter_params:`

Trimmomatic parameters for trimming after adapter removal.

### Cutadapt

```yaml
cutadapt:
  adapter_sequence: AGATCGGAAGAGCACACGTCTGAACTCCAGTCA
  rev_adapter_sequence: AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT
  error_rate: 0.1
```

- `adapter_sequence:`
- `rev_adapter_sequence:`

Forward and reverse adapter sequences to trim with cutadapt. Defaults to the Illumina TruSeq Adapters.

- `error_rate:` 

Maximum allowed error rate for cutadapt as value between 0 and 1. This is the number of errors divided by length of matching region.


### SortMeRNA

```yaml
sortmerna:
  keep: "non_rRNA"
  remove_filtered: False
  dbs:
    - "rfam-5s-database-id98.fasta"
    - "rfam-5.8s-database-id98.fasta"
    - "silva-arc-16s-id95.fasta"
    - "silva-arc-23s-id98.fasta"
    - "silva-bac-16s-id90.fasta"
    - "silva-bac-23s-id98.fasta"
    - "silva-euk-18s-id95.fasta"
    - "silva-euk-28s-id98.fasta"
  paired_strategy: "paired_in"
  extra_settings: "--num_alignments 1"
```

- `keep:`

SortMeRNA produces files with reads aligning to rRNA (`rRNA`) and not aligning to rRNA (`non_rRNA`). This parameter indicates which of these to keep for downstream analyses.

- `remove_filtered:`

Set to True to remove the set of reads **not** specified with `sortmerna_keep`. For instance, if you only want to keep the `non_rRNA` reads and remove the `rRNA` fraction, set `sortmerna_keep: 'non_rRNA'

- `dbs:`

Which databases to use for rRNA identification. By default, all available databases are used. Comment out the ones you do not need.

- `paired_strategy:`

How should SortMeRNA handle paired-end reads where only one read gets assigned to the rRNA fraction? Either put both paired reads into rRNA bin (`paired_in`) or put both reads in other bin (`paired_out`).

- `extra_settings:`

Additional settings for SortMeRNA. Here you can pass other settings not covered by the parameters above.

## Post-processing
Post-processing is done at the bam-file level after reads have been mapped to assembled contigs.

```yaml
remove_duplicates: True
```

Runs the MarkDuplicates tool from the [picard](https://broadinstitute.github.io/picard/) suite to remove duplicates from bam-files.

## Assembly
Preprocessed reads can be assembled using either Megahit or Metaspades.

```yaml
assembly:
  megahit: True
  metaspades: False
```

- `megahit:`

Whether to assemble reads using [Megahit](https://github.com/voutcn/megahit)?

- `metaspades:`

Should the [Metaspades](https://github.com/ablab/spades) assembler be used? **Note**: if True this runs Metaspades instead of Megahit.


### Megahit

```yaml
megahit:
  threads: 20
  keep_intermediate: False
  extra_settings: "--min-contig-len 300 --prune-level 3"
```

- `threads:`

Maximum number of cpus to use for Megahit.

- `keep_intermediate:`

Should intermediate contigs (*i.e.* contigs from running on intermediate kmer lengths) from Megahit be stored? If True, the contigs will be placed under a folder `assembly/` in the path specified with `intermediate_path` (see above).

- `extra_settings:`

Additional settings for Megahit. Use this to control behaviour of the assembler outside of the parameters specified above.

### Metaspades

```yaml
metaspades:
  threads: 20
  keep_intermediate: False
  keep_corrected: True
  extra_settings: "-k 21,31,41,51,61,71,81,91,101,111,121"
```

- `threads:`

Maximum number of threads to use for Metaspades.

- `keep_intermediate:`

Should intermediate contigs from Metaspades be stored? If True, the contigs will be placed under a folder `assembly/` in the path specified with `intermediate_path` (see above).

- `keep_corrected:`

Should corrected reads produced during Metaspades assembly be stored?

- `extra_settings:`

Additional settings for Metaspades. By default the workflow passes a list of kmer sizes.

## Annotation
The annotation part of the workflow refers primarily to open reading frames (ORFs) called on assembled metagenomics contigs by [prodigal](https://github.com/hyattpd/prodigal/). However, tRNA and rRNA genes are also identified using [tRNAscan-SE](http://lowelab.ucsc.edu/tRNAscan-SE/) and [Infernal](http://eddylab.org/infernal/), respectively.

```yaml
annotation:
  tRNAscan: False
  infernal: True
  eggnog: False
  pfam: True
  rgi: False
  taxonomy: False
```

- `tRNAscan:`

Identify tRNA genes on assembled contigs with [tRNAscan-SE](http://lowelab.ucsc.edu/tRNAscan-SE/)?

- `infernal:`

Run [Infernal](http://eddylab.org/infernal/) for rRNA identification? The workflow runs infernal with a small set of rRNA families (see the [Rfam](http://rfam.xfam.org/search?q=rRNA%20AND%20rna_type:%22rRNA%22%20AND%20entry_type:%22Family%22) database).

- `eggnog:` 

Run [eggnog-mapper](https://github.com/eggnogdb/eggnog-mapper/) to add functional annotations to ORFs.

- `pfam:`

Run [pfam_scan](https://anaconda.org/bioconda/pfam_scan) to infer PFAM protein families for ORFs.

- `rgi:`

Run the [Resistance Gene Identifier tool](https://card.mcmaster.ca/) for ORFs.

- `taxonomy:`

Add taxonomic annotations for contigs/ORFs using [contigtax](https://github.com/NBISweden/contigtax) + [sourmash](https://sourmash.readthedocs.io/en/latest/) (see specfic settings [here](#taxonomy))

### Taxonomy
Taxonomy is assigned to contigs using `contigtax` which is based on blastx searches against either UniRef or nr, and `sourmash` which is based on MinHash sketches of reference genomes and contigs. The former performs relatively well on short contigs (_e.g._ 500 - 50,000 bp) while the latter is suited primarily for long contigs (_e.g._ >50,000 bp). The workflow combines assignments from both tools, with `sourmash` taking preference.

```yaml
taxonomy:
  min_len: 300
  search_params: "--evalue 0.01 --top 10"
  assign_params: "--evalue 0.001 --top 5"
  sourmash_fraction: 100
  ranks:
    - "superkingdom"
    - "phylum"
    - "class"
    - "order"
    - "family"
    - "genus"
    - "species"
  taxdb: "uniref100"
```

- `min_len:` 

Minimum length of contigs to include in taxonomic assignments. Really only applies to the `contigtax` part of taxonomic assignment, but in practice `sourmash` will not be able to assign taxonomy to short contigs anyway.

- `search_params:`
- `assign_params:`

These are settings used in the search and assign stages by `contigtax`. The `search_params` are passed directly to `diamond blastx` and refer to the Expect value and the range of alignments to include (`--top 10` means that alignments scoring at most 10% lower than the best score are considered for a query). The `assign_params` are used by `contigtax` in a second step. By using more permissive search parameters the assignment step of the workflow can be re-run without having to re-run also the (often time-consuming) first search step.

- `sourmash_fraction:` 

This setting determines how much sourmash will compress the signatures of your contigs before performing classification. An example from the sourmash manual is:

> if you have a 5 Mbp genome and use --scaled 1000, you will extract approximately 5000 hashes. So a scaled of 1000 is equivalent to using -n 5000 with mash on a 5 Mbp genome.

- `ranks:`

This is a list of ranks that the workflow will report assignments for. Higher ranks in the taxonomy hierarchy are inferred from lower ranks, such that a contig with the lowest assignment at the genus level, for example `Escherichia`, the higher ranks become `family:Enterobacteriaceae`, `order:Enterobacterales`, `class:Gammaproteobacteria`, `phylum:Proteobacteria`, `superkingdom:Bacteria` and the assignment at species level becomes `species:Unclassified.Escherichia`.

- `taxdb:`

This refers to the protein database to use for assigning taxonomy to ORFs. By default the `uniref100` database is used, but the workflow supports the use of `uniref90`, `uniref50` and the non-redundant `nr` database.

## Binning
Assembled contigs can be binned into collections representing estimates of genomes in the sample(s). The workflow has support for the use of three binners: [Metabat2](https://bitbucket.org/berkeleylab/metabat/src/master/), [CONCOCT](https://github.com/BinPro/CONCOCT) and [MaxBin2](https://sourceforge.net/projects/maxbin2/). These binners all use the nucleotide composition of contigs and their differential abundance in samples to bin contigs. You can configure the workflow to use one or more of these tools, with one or more minimum length threshold for contigs to include in the binning. Please note however that currently, neither CONCOCT nor MaxBin2 work on OSX.

The workflow can also run [checkm](https://github.com/Ecogenomics/CheckM/wiki) to investigate completeness, purity and other metrics of the binned genomes. It also supports the classification of bins with [GTDB-TK](https://ecogenomics.github.io/GTDBTk/). Genome bins can also be clustered using output from [fastANI](https://github.com/ParBLiSS/FastANI).

```yaml
binning:
  contig_lengths:
    - 1500
    # uncomment and/or add below to run binning at more lengths
    #- 2500
    #- 5000
  metabat: False
  concoct: False
  maxbin: False
  threads: 20
  checkm: False
  gtdbtk: False
  fastani: False
```

- `contig_lengths:` 

This is the minimum length threshold of contigs to include by any of the binners. Setting this to a low threshold will likely lead to more sequences in your assembly being binned, but could come at the cost of higher contamination of the bins. The minimum possible setting is 1500 bp. The optimal setting will vary depending on your sample and the assembly so it may be a good idea to use several cutoffs and evaluate which one gives the best results (_e.g._ highest completeness and purity). Output from binning is placed in your main results directory under: `binning/<binning-tool>/<assembly-name>/<length-threshold>/`.

- `metabat|concoct|maxbin:` 

This sets the binning tools to use. They all use the composition and differential abundance to bin contigs but differ slightly in the specific implementation. Maxbin uses marker genes to identify 'seed' contigs and will only work with prokaryotic genomes. Concoct often bins a larger fraction of your assembly but may come at the cost of higher contamination. Metabat is currently the only of the three binners that is fully supported on OSX but it is possible to run the others on your Mac using Docker. You may set the workflow to use all three tools at once and then evaluate the results.

- `threads:`

The maximum number of threads to use for the binners.

- `checkm:`

Whether to assess quality of bins (_e.g._ completeness and contamination) using [checkm](https://github.com/Ecogenomics/CheckM/).

- `gtdbtk:`

Should bins be classified phylogenetically using the [GTDB toolkit](https://github.com/Ecogenomics/GTDBTk/)? Please note that `gtdbtk` is not supported on OSX.

- `fastani:`

Bins can be clustered based on average nucleotide identity (ANI) using `fastANI`. See below for fastANI settings and how to include reference genomes in the clustering. Note that only bins with at least 50% completeness and at most 10% contamination are included in clustering, which means that `checkm` is also run prior to fastANI.

### Maxbin
```yaml
maxbin:
  markerset: 40
```

- `markerset:`

Maxbin uses marker gene identification on contigs to generate seed contigs for binning. The `markerset` parameter specifies whether to use the proakaryotic markerset `40` or the bacterial markerset `107`.

### Checkm
```yaml
checkm:
  taxonomy_wf: False
  rank: life
  taxon: Prokaryote
  reduced_tree: False
```

- `taxonomy_wf:`

Checkm estimates the completeness and contamination of genome bins by counting the number of marker genes. The set of marker genes to use can be inferred for each bin individually by running checkm in the `lineage_wf` mode (default for the `nbis-meta` workflow). This can result in better estimates of the completeness levels but relies on phylogenetic placement of bins which 1) may require more memory than what is available on some systems and 2) is currently not supported on OSX. To circumvent this you can set `taxonomy_wf: True` which uses a predefined set of marker genes for all bins. This may very well be enough for your project.

- `rank:`
- `taxon:`

Taxonomic rank and taxon to use for checkm taxonomy_wf. With `rank: life` and `taxon: Prokaryote` checkm uses a list of universal marker genes found in both Archaea and Bacteria to estimate genome completeness.

**Note:** These settings only apply to the `taxonomy_wf` part. If you set `taxonomy_wf: False` then the `rank:` and `taxon:` settings are ignored.

- `reduced_tree:`

Setting this to `True` runs checkm with a reduced reference tree, thus lowering the memory requirements.

### fastANI
```yaml
fastani:
  ref_list: ""
  fraction: 0.5
  threshold: 0.05
  kmer_size: 16
  frag_len: 3000
```

- `ref_list:`

Path to a list of reference genomes to include in ANI calculations. This list must be tab-separated with the name/id of the genome in the first column and the RefSeq or Genbank ftp base url in the second column, e.g:

|     |     |
| --- | --- |
| T_arc | ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/009/387/875/GCA_009387875.1_ASM938787v1 |
| P_med | ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/240/225/GCF_000240225.1_ASM24022v2 |
| L_bac | ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/209/385/GCF_000209385.2_Lach_bact_2_1_46_FAA_V2 |
| E_bac | ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/302/695/GCF_000302695.1_LSJC7_1.0 |
| B_mal | ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/002/346/085/GCF_002346085.1_ASM234608v1 |

If you store this in a file, _e.g._ `resources/ref_genomes.tsv` and set:
```yaml
fastani:
  ref_list: resources/ref_genomes.tsv
```

Then a genome nucleotide fasta file will be downloaded for each genome and included in the `fastANI` clustering. If not specified, only the binned genomes will be clustered with each other.

- `fraction:`

This is the minimum alignment overlap required between two genomes to trust their ANI value (and hence allow them to be clustered). It is expressed as a fraction of the smaller of the two genomes.

- `threshold:`

This is the distance threshold to use for clustering genomes. It is calculated as (100-ANI)/100 so setting 0.05 corresponds to clustering genomes at 95% identity.

- `kmer_size:`

kmer size to use for fastANI.

- `frag_len:` 

fastANI fragments genomes into non-overlapping fragments of size `frag_len` before performing alignment and computing ANI.

## Classification

The classification part of the workflow is run directly on the (preprocessed) reads and independently of the _de novo_ assembly part. Reads can be classified using [kraken2](https://github.com/DerrickWood/kraken2/) and/or [centrifuge](https://github.com/DaehwanKimLab/centrifuge/). In addition, a taxonomic profile can be generated for samples using [MetaPhlan3](https://github.com/biobakery/MetaPhlAn).

```yaml
classification:
  kraken: True
  centrifuge: False
  metaphlan: False
```

- `kraken:`

Whether to run kraken2 classifier.

- `centrifuge:`

Whether to run centrifuge classifier.

- `metaphlan:`

Whether to run Metaphlan3 classifier.

### Kraken
```yaml
kraken:
  standard_db: False
  prebuilt: "minikraken_8GB"
  custom: ""
  reduce_memory: False
```

- `standard_db:`

Should kraken generate the standard database? Kee in mind that this will use a lot of resources during the build step. Optionally you may download pre-built database files (see below).

- `prebuilt:`

Download a prebuilt kraken2 database from the [CCB servers](ftp://ftp.ccb.jhu.edu/pub/data/kraken2_dbs/). Choose from "minikraken_8GB", "16S_Greengenes", "16S_RDP" or "16S_Silva".

- `custom:`

If you already have access to a built kraken database you may specify the database path here (path must contain hash.k2d, opts.k2d and taxo.k2d files).

- `reduce_memory:`

Whether to run kraken2 with reduced memory requirements? Setting `reduce_memory: True` makes kraken2 run in `--memory-mapping` mode which avoids loading database into RAM and uses less memory.