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

## The sample list

- `sample_list: config/samples.tsv` 

Path to the sample list (**required**). See [Defining your samples in the sample list](https://github.com/NBISweden/nbis-meta/wiki/Defining-your-samples-in-the-sample-list) for instructions on how to format it for your data. By default this points to an example sample list for which the sequence data is generated as part of the workflow. Feel free to use it to get acquainted with the workflow.

## Paths

```yaml
paths:
  # main base folder for results
  results: "results"
  # temporary path
  # set to $TMPDIR, /scratch or equivalent when running on HPC clusters
  temp: "temp"
```

- `results: results`

Path for main results. Output from preprocessing, assembly etc goes here. Summarized report files (_e.g._ plots and tables) are placed in a sub-directory called `report/`.

- `temp: temp`

Paths for temporary output. On compute clusters this can be set to `$TMPDIR` or equivalent.

## Preprocessing
By default, the workflow runs `trimmomatic` for adapter and quality trimming, followed by `fastqc` for read statistics and [MultiQC](https://github.com/MultiQC) to aggregate and visualize logfiles. Depending on your needs you may however add, modify or remove preprocessing steps.

```yaml
preprocessing:
  # run fastqc on (preprocessed) input?
  # if no preprocessing is done, fastqc will be run on the raw reads
  fastqc: True
  # trim reads with trimmomatic? (quality and adapter trimming)
  trimmomatic: True
  # trim reads with cutadapt? (runs instead of trimmomatic, no quality trimming)
  cutadapt: False
  # run fastuniq (removes duplicates from paired-end samples)
  fastuniq: False
  # map reads agains the phix genome and keep only reads that do not map concordantly
  phix_filter: False
  # run SortMeRNA to identify (and filter) rRNA sequences
  sortmerna: False
```

- `fastqc: True`

Run [Fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) on preprocessed reads? If you do not wish to produce a sample report summary (*e.g.* if you have data that has already been preprocessed or that you know is of sufficient quality), set this to False.

- `trimmomatic: True`

Run [Trimmomatic](http://www.usadellab.org/cms/index.php?page=trimmomatic) to perform adapter/quality trimming of reads. (see specific settings [here](#trimmomatic))

- `cutadapt: False`

If you want to perform adapter trimming with [cutadapt](https://github.com/marcelm/cutadapt/) **instead of** Trimmomatic, set this to True. Note that this means no quality trimming is done (see specific settings [here](#cutadapt))

- `fastuniq: False`

Run [fastuniq](https://sourceforge.net/projects/fastuniq/) to remove duplicates from paired-end samples.

- `phix_filter: True`

Map reads with bowtie2 against the phiX genome and remove reads that map concordantly.

- `sortmerna: False`

Run [SortMeRNA](https://github.com/biocore/sortmerna) to identify (and filter) rRNA sequences from samples (see specific settings [here](#sortmerna))

### Trimmomatic
```yaml
trimmomatic:
  # also trim adapters (in addition to quality trimming)?
  trim_adapters: True
  # settings specific to paired end reads
  pe:
    # adapter type to trim from paired end libraries with trimmomatic
    # ["NexteraPE-PE", "TruSeq2-PE", "TruSeq3-PE", "TruSeq3-PE-2"]
    adapter: "TruSeq3-PE-2"
    # parameters for trimming adapters on paired-end samples
    adapter_params: "2:30:15"
    # parameters for trimming prior to adapter removal on paired-end samples
    pre_adapter_params: ""
    # parameters for trimming after adapter removal on paired-end samples
    post_adapter_params: "LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:31"
```

- `trim_adapters: True`

In addition to quality trimming, also trim adapters from reads?

The remaining settings for Trimmomatic are divided into parameters specific to paired-end data (`pe`) or single-end data (`se`).

- `adapter:`

Adapter type to trim. For paired-end data choose from `NexteraPE-PE`, `TruSeq2-PE`, `TruSeq3-PE` and `TruSeq3-PE-2`. For single-end data choose from `TruSeq2-SE` and `TruSeq3-SE`.

- `adapter_params: "2:30:15"`

Trimmomatic parameters for trimming adapters. This translates into the `<seed mismatches>:<palindrome clip threshold>:<simple clip threshold>` settings for Trimmomatic.

- `pre_adapter_params: ""`

Trimmomatic parameters for trimming prior to adapter removal in paired-end and single-end reads, respectively.

- `post_adapter_params: "LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:31"`

Trimmomatic parameters for trimming after adapter removal in paired-end and single-end reads, respectively.

### Cutadapt
```yaml
cutadapt:
  # adapter sequence to trim with cutadapt
  # shown here is for Illumina TruSeq Universal Adapter.
  adapter_sequence: AGATCGGAAGAGCACACGTCTGAACTCCAGTCA
  # reverse adapter sequence to trim with cutadapt
  rev_adapter_sequence: AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT
  # maximum allowed error rate as value between 0 and 1 (no. of errors divided by length of matching region)
  error_rate: 0.1 #Maximum allowed error rate as value between 0 and 1
```

- `adapter_sequence: AGATCGGAAGAGCACACGTCTGAACTCCAGTCA`
- `rev_adapter_sequence: AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT`

Forward and reverse adapter sequences to trim with cutadapt. Defaults to the Illumina TruSeq Adapters.

- `error_rate: 0.1` 

Maximum allowed error rate for cutadapt as value between 0 and 1. This is the number of errors divided by length of matching region.


### SortMeRNA
```yaml
sortmerna:
  # sortmerna produces files with reads aligning to rRNA ("rRNA" extension)
  # and not aligning to rRNA ("non_rRNA") extension
  # which reads should be used for downstream analyses
  keep: "non_rRNA"
  # remove filtered reads (i.e. the reads NOT specified in "keep:")
  # if you are filtering out rRNA reads and don't plan to use them downstream
  # it may be worthwhile to set this to True since they can take up a lot of disk space
  remove_filtered: False
  # databases to use for rRNA identification
  dbs:
    - "rfam-5s-database-id98.fasta"
    - "rfam-5.8s-database-id98.fasta"
    - "silva-arc-16s-id95.fasta"
    - "silva-arc-23s-id98.fasta"
    - "silva-bac-16s-id90.fasta"
    - "silva-bac-23s-id98.fasta"
    - "silva-euk-18s-id95.fasta"
    - "silva-euk-28s-id98.fasta"
  # put both paired reads into rRNA bin (paired_in) or both reads in other bin (paired_out)
  paired_strategy: "paired_in"
  # extra settings for sortmerna
  extra_settings: "--num_alignments 1"
```

- `keep: 'non_rRNA'`

SortMeRNA produces files with reads aligning to rRNA (`rRNA`) and not aligning to rRNA (`non_rRNA`). This parameter indicates which of these to keep for downstream analyses.

- `remove_filtered: False`

Set to True to remove the set of reads **not** specified with `sortmerna_keep`. For instance, if you only want to keep the `non_rRNA` reads and remove the `rRNA` fraction, set `sortmerna_keep: 'non_rRNA'

- `dbs: ["rfam-5s-database-id98.fasta","rfam-5.8s-database-id98.fasta","silva-arc-16s-id95.fasta","silva-arc-23s-id98.fasta","silva-bac-16s-id90.fasta","silva-bac-23s-id98.fasta","silva-euk-18s-id95.fasta","silva-euk-28s-id98.fasta"]`

Which databases to use for rRNA identification. By default, all available databases are used.

- `paired_strategy: "paired_in"`

How should SortMeRNA handle paired-end reads where only one read gets assigned to the rRNA fraction? Either put both paired reads into rRNA bin (`paired_in`) or put both reads in other bin (`paired_out`).

- `extra_settings "--num_alignments 1"`

Additional settings for SortMeRNA. Here you can pass other settings not covered by the parameters above.

## Post-processing
Post-processing is done at the bam-file level after reads have been mapped to assembled contigs.

`remove_duplicates: True`

Runs the MarkDuplicates tool from the [picard](https://broadinstitute.github.io/picard/) suite to remove duplicates from bam-files.

## Assembly
Preprocessed reads can be assembled using either Megahit or Metaspades.

```yaml
assembly:
  # run Megahit assembler?
  megahit: True
  # Use Metaspades instead of Megahit for assembly?
  metaspades: False
```

- `megahit: True`

Assemble reads using [Megahit](https://github.com/voutcn/megahit)?

- `metaspades: False`

Should the [Metaspades](https://github.com/ablab/spades) assembler be used? **Note**: if True this runs Metaspades instead of Megahit.


### Megahit
```yaml
megahit:
  # maximum threads for megahit
  threads: 20
  # keep intermediate contigs from Megahit?
  keep_intermediate: False
  # extra settings passed to Megahit
  extra_settings: "--min-contig-len 300 --prune-level 3"
```

- `threads: 20`

Maximum number of cpus to use for Megahit.

- `keep_intermediate: False`

Should intermediate contigs (*i.e.* contigs from running on intermediate kmer lengths) from Megahit be stored? If True, the contigs will be placed under a folder `assembly/` in the path specified with `intermediate_path` (see above).

- `extra_settings: '--min-contig-len 300 --prune-level 3'`

Additional settings for Megahit. Use this to control behaviour of the assembler outside of the parameters specified above.

### Metaspades

```yaml
metaspades:
  # maximum threads for metaspades
  threads: 20
  # keep intermediate contigs from Metaspades?
  keep_intermediate: False
  # keep corrected reads produced during Metaspades assembly?
  keep_corrected: True
  # extra settings passed to Metaspades
  extra_settings: "-k 21,31,41,51,61,71,81,91,101,111,121"
```

- `threads: 20`

Maximum number of threads to use for Metaspades.

- `keep_intermediate: False`

Should intermediate contigs from Metaspades be stored? If True, the contigs will be placed under a folder `assembly/` in the path specified with `intermediate_path` (see above).

- `keep_corrected: True`

Should corrected reads produced during Metaspades assembly be stored?

- `extra_settings: '-k 21,31,41,51,61,71,81,91,101,111,121'`

Additional settings for Metaspades. By default the workflow passes a list of kmer sizes.

## Annotation
The annotation part of the workflow refers primarily to open reading frames (ORFs) called on assembled metagenomics contigs by [prodigal](https://github.com/hyattpd/prodigal/). However, tRNA and rRNA genes are also identified using [tRNAscan-SE](http://lowelab.ucsc.edu/tRNAscan-SE/) and [Infernal](http://eddylab.org/infernal/), respectively.

```yaml
annotation:
  # run tRNAscan-SE?
  tRNAscan: False
  # run infernal for rRNA identification?
  infernal: True
  # run eggnog-mapper to infer KEGG orthologs, pathways and modules?
  eggnog: False
  # run PFAM-scan to infer protein families from PFAM?
  pfam: True
  # run Resistance gene identifier?
  rgi: False
  # run taxonomic annotation of assembled contigs (using tango + sourmash)?
  taxonomy: False
```

- `tRNAscan: False`

Identify tRNA genes on assembled contigs with [tRNAscan-SE](http://lowelab.ucsc.edu/tRNAscan-SE/)?

- `infernal: True`

Run [Infernal](http://eddylab.org/infernal/) for rRNA identification? The workflow runs infernal with a small set of rRNA families (see the [Rfam](http://rfam.xfam.org/search?q=rRNA%20AND%20rna_type:%22rRNA%22%20AND%20entry_type:%22Family%22) database).

- `eggnog: False` 

Run [eggnog-mapper](https://github.com/eggnogdb/eggnog-mapper/) to add functional annotations to ORFs.

- `pfam: True`

Run [pfam_scan](https://anaconda.org/bioconda/pfam_scan) to infer PFAM protein families for ORFs.

- `rgi: False`

Run the [Resistance Gene Identifier tool](https://card.mcmaster.ca/) for ORFs.

- `taxonomy: False`

Add taxonomic annotations for contigs/ORFs using [tango](https://github.com/NBISweden/tango) + [sourmash](https://sourmash.readthedocs.io/en/latest/) (see specfic settings [here](#taxonomy))

### Taxonomy
Taxonomy is assigned to contigs using `tango` which is based on blastx searches against either UniRef or nr, and `sourmash` which is based on MinHash sketches of reference genomes and contigs. The former performs relatively well on short contigs (_e.g._ 500 - 50,000 bp) while the latter is suited primarily for long contigs (_e.g._ >50,000 bp). The workflow combines assignments from both tools, with `sourmash` taking preference.

```yaml
taxonomy:
  # minimum length of contigs to use for taxonomic annotation
  min_len: 300
  # parameters for tango search
  # use more permissive settings for search compared to assign in order to
  # modify assignments without having to rerun the search step
  search_params: "--evalue 0.01 --top 10"
  # parameters for tango assign
  assign_params: "--evalue 0.001 --top 5"
  # hash fraction to use for sourmash when computing signatures for contigs
  # this is evaluated as 1/<sourmash_fraction>
  sourmash_fraction: 100
  # ranks to report taxonomy for
  ranks:
    - "superkingdom"
    - "phylum"
    - "class"
    - "order"
    - "family"
    - "genus"
    - "species"
  # protein database to use for taxonomic assignments
  # choose between uniref50, uniref90, uniref100 and nr.
  taxdb: "uniref100"
```

- `min_len: 300` 

Minimum length of contigs to include in taxonomic assignments. Really only applies to the `tango` part of taxonomic assignment, but in practice `sourmash` will not be able to assign taxonomy to short contigs anyway.

- `search_params: --evalue 0.01 --top 10`
- `assign_params: --evalue 0.001 --top 5`

These are settings used in the search and assign stages by `tango`. The `search_params` are passed directly to `diamond blastx` and refer to the Expect value and the range of alignments to include (`--top 10` means that alignments scoring at most 10% lower than the best score are considered for a query). The `assign_params` are used by `tango` in a second step. By using more permissive search parameters the assignment step of the workflow can be re-run without having to re-run also the (often time-consuming) first search step.

### Binning
Assembled contigs can be binned into collections representing estimates of genomes in the sample(s). The workflow has support for the use of three binners: [Metabat2](https://bitbucket.org/berkeleylab/metabat/src/master/), [CONCOCT](https://github.com/BinPro/CONCOCT) and [MaxBin2](https://sourceforge.net/projects/maxbin2/). These binners all use the nucleotide composition of contigs and their differential abundance in samples to bin contigs. You can configure the workflow to use one or more of these tools, with one or more minimum length threshold for contigs to include in the binning. Please note however that currently, neither CONCOCT nor MaxBin2 work on OSX.

The workflow can also run [checkm](https://github.com/Ecogenomics/CheckM/wiki) to investigate completeness, purity and other metrics of the binned genomes. It also supports the classification of bins with [GTDB-TK](https://ecogenomics.github.io/GTDBTk/).

```yaml
binning:
  # minimum contig lengths to use for binning. binning will be run for every
  # length threshold with output under <results_path>/binning/{binner}/{contig_length}
  # typically shorter contig lengths will result in more a larger share of the
  # assembly being binned, at the cost of bin purity
  contig_lengths:
    - 1500
    # uncomment and/or add below to run binning at more lengths
    #- 2500
    #- 5000
  # run Metabat2 binner?
  metabat: False
  # run CONCOCT binner?
  concoct: False
  # run MaxBin2 binner?
  maxbin: False
  # maximum threads for binners
  threads: 20
  # run Checkm to assess quality of bins?
  checkm: False
  # run gtdbtk to classify bins phylogenetically?
  gtdbtk: False
```

- `contig_lengths:` 

This is the minimum length threshold of contigs to include by any of the binners. Setting this to a low threshold will likely lead to more sequences in your assembly being binned, but could come at the cost of higher contamination of the bins. The minimum possible setting is 1500 bp. The optimal setting will vary depending on your sample and the assembly so it may be a good idea to use several cutoffs and evaluate which one gives the best results (_e.g._ highest completeness and purity). Output from binning is placed in your main results directory under: `binning/<binning-tool>/<assembly-name>/<length-threshold>/`.