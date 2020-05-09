The workflow can be configured using a configuration file in `yaml` format. These are the configuration parameters and their default values

- [Paths](#paths)
- [Preprocessing](#preprocessing)
- [Assembly](#assembly)

## Paths
- `sample_list: samples/example_sample_list.tsv` 

Path to the sample list (**required**). See [Defining your samples in the sample list](https://github.com/NBISweden/nbis-meta/wiki/Defining-your-samples-in-the-sample-list) for instructions on how to format it for your data. By default this points to an example sample list for which the sequence data is generated as part of the workflow. Feel free to use it to get acquainted with the workflow.

***

- `workdir: .`

Main working directory for the workflow. Set to the current directory (`.`) by default.

***

- `results_path: results`

Path for main results. Output from preprocessing, assembly etc goes here.

***

- `report_path: results/report`

Path for report files. Output from *e.g.* MultiQC, assembly statistics and collated kraken results goes here.

***

- `intermediate_path: results/intermediate`

Path to store intermediate files, *e.g.* intermediate fastq files created as part of preprocessing.

***

- `temp_path: temp`
- `scratch_path: temp`

Paths for temporary output. On compute clusters this can be set to `$TMPDIR`.

***

- `resource_path: resources`

Path to store resource files, *e.g.* databases for tools such as `eggnog-mapper` and `pfam_scan`.

## Preprocessing
By default, the workflow runs `trimmomatic` for adapter and quality trimming, followed by `fastqc` for read statistics and [MultiQC](https://github.com/MultiQC) to aggregate and visualize logfiles. Depending on your needs you may however add, modify or remove preprocessing steps.

- `fastqc: True`

Run [Fastqc](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) on preprocessed reads? If you do not wish to produce a sample report summary (*e.g.* if you have data that has already been preprocessed or that you know is of sufficient quality), set this to False.

***

- `trimmomatic: True`

Run [Trimmomatic](http://www.usadellab.org/cms/index.php?page=trimmomatic) to perform adapter/quality trimming of reads.

***

- `trim_adapters: True`

In addition to quality trimming, also trim adapters from reads?

***

- `trimmomatic_pe_adapter: "TruSeq3-PE-2"`

Adapter type to trim from paired end libraries with trimmomatic. Choose from `NexteraPE-PE`, `TruSeq2-PE`, `TruSeq3-PE` and `TruSeq3-PE-2`.

***

- `trimmomatic_se_adapter: "TruSeq3-SE"`

Adapter type to trim from single end libraries with trimmomatic. Choose from `TruSeq2-SE` and `TruSeq3-SE`

***

- `pe_adapter_params: "2:30:15"`
- `se_adapter_params: "2:30:15"`

Trimmomatic parameters for trimming adapters in paired-end (`pe`) and single-end (`se`) reads, respectively. This translates into the `<seed mismatches>:<palindrome clip threshold>:<simple clip threshold>` settings for Trimmomatic.

***

- `pe_pre_adapter_params: ""`
- `se_pre_adapter_params: ""`

Trimmomatic parameters for trimming prior to adapter removal in paired-end and single-end reads, respectively.

***

- `pe_post_adapter_params: "LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:31"`
- `se_post_adapter_params: "LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:31"`

Trimmomatic parameters for trimming after adapter removal in paired-end and single-end reads, respectively.

***

- `cutadapt: False`

If you want to perform adapter trimming with [cutadapt](https://github.com/marcelm/cutadapt/) **instead of** Trimmomatic, set this to True. Note that this means no quality trimming is done.

***

- `adapter_sequence: AGATCGGAAGAGCACACGTCTGAACTCCAGTCA`
- `rev_adapter_sequence: AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT`

Forward and reverse adapter sequences to trim with cutadapt. Defaults to the Illumina TruSeq Adapters.

***

- `cutadapt_error_rate: 0.1` 

Maximum allowed error rate for cutadapt as value between 0 and 1. This is the number of errors divided by length of matching region.

***

- `fastuniq: False`

Run [fastuniq](https://sourceforge.net/projects/fastuniq/) to remove duplicates from paired-end samples.

***

- `phix_filter: True`

Map reads with bowtie2 against the phiX genome and remove reads that map concordantly.

***

- `sortmerna: False`

Run [SortMeRNA](https://github.com/biocore/sortmerna) to identify (and filter) rRNA sequences from samples. 

***

- `sortmerna_keep: 'non_rRNA'`

SortMeRNA produces files with reads aligning to rRNA (`rRNA`) and not aligning to rRNA (`non_rRNA`). This parameter indicates which of these to keep for downstream analyses.

***

- `sortmerna_remove_filtered: False`

Set to True to remove the set of reads **not** specified with `sortmerna_keep`. For instance, if you only want to keep the `non_rRNA` reads and remove the `rRNA` fraction, set `sortmerna_keep: 'non_rRNA'

***

- `sortmerna_dbs: ["rfam-5s-database-id98.fasta","rfam-5.8s-database-id98.fasta","silva-arc-16s-id95.fasta","silva-arc-23s-id98.fasta","silva-bac-16s-id90.fasta","silva-bac-23s-id98.fasta","silva-euk-18s-id95.fasta","silva-euk-28s-id98.fasta"]`

Which databases to use for rRNA identification. By default, all available databases are used.

***

- `sortmerna_paired_strategy: "paired_in"`

How should SortMeRNA handle paired-end reads where only one read gets assigned to the rRNA fraction? Either put both paired reads into rRNA bin (`paired_in`) or put both reads in other bin (`paired_out`).

***

- `sortmerna_params: "--num_alignments 1"`

Additional parameters for SortMeRNA. Here you can pass other settings not covered by the parameters above.