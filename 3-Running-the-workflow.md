## Contents

- [Useful snakemake options](#useful-snakemake-options)
- [Targets](#targets)
- [Reports](#reports)
- [Examples](#examples)
  - [Assembly-based analysis](#assembly-based-analysis)
    - [Megahit](#assemble-reads-with-megahit)
    - [Protein-level annotations](#protein-level-annotations)
  - [Read-based analysis](#read-based-analysis)
    - [Metaphlan](#metaphlan)
    - [Kraken](#kraken2)

Once you've modified the [config file](https://github.com/NBISweden/nbis-meta/wiki/Configuration) and created the [sample list](https://github.com/NBISweden/nbis-meta/wiki/Defining-a-sample-list), the most basic way to run the workflow is with the command:

```bash
snakemake —-use-conda --configfile <myconfig.yaml> --cores 4 
```

Here `--cores` specifies how many CPU cores to use in parallell.

## Useful snakemake options

Other useful options that you can specify to snakemake on the command line include:

- `-r` or `--reason`: print the reason for execution of each rule
- `-p`or `--printshellcmds`: print shell commands that will be executed
- `--nolock`: do not lock the working directory
- `--unlock`: unlock working directory
- `--ri` or `--rerun-incomplete`: re-run all jobs where output may be incomplete
- `--jobs [N]` or `-j [N]`: Use at most N CPU cores/jobs in parallel. If `-j` is used with the SLURM profile to submit jobs in a compute cluster infrastructure, `-j` specifies the maximum number of jobs to submit to the queue.

For a full list of snakemake command line options, see [here](https://snakemake.readthedocs.io/en/stable/executing/cli.html#all-options).

## Targets
If you don't want to run the full workflow from start to finish in one go you may specify one or several 'targets' on the commandline. Some useful targets are:

| Target | Description |
| ------ | ----------- |
| `qc`   | Runs preprocessing (as specified in the configuration file) and generates a `sample_report.html`|
| `assemble` | Assembles samples according to sample list and config file, and generates some assembly statistics|
| `quantify` | Quantifies open reading frames called on assembled contigs, producing RPKM-normalized and raw counts|
| `annotate` | Annotates open reading frames called on assembled contigs using settings defined in the config file. Also quantifies genes and features, producing normalized and raw counts|
| `taxonomy` | Assigns taxonomy to assembled contigs and open reading frames called on those contigs|
| `bin` | Performs genome binning of assembled contigs and generates some statistics of the binned genomes|
| `classify` | Read-based (_e.g._ Kraken/Centrifuge/MetaPhlAn) classification of preprocessed reads|

To use these targets add them to the snakemake command line call. For instance, to run only the preprocessing part:

```bash
snakemake --use-conda --configfile config.yaml -j 4 qc
```

Targets may also be combined, so if you want to generate assemblies **and** run read-based classification you can do:

```bash
snakemake --use-conda --configfile config.yaml -j 4 assemble classify
```

## Reports
After the workflow has completed you can generate a report with summarized statistics of the run. Depending on the run, the report will also include links to output files produced (_e.g._ tables, plots and html files). To produce a report, run:

```bash
snakemake --report report.html
```

**IMPORTANT:** When generating the report you must call snakemake the same way you did when you ran the workflow itself otherwise snakemake will report a `WorkflowError:` because the expected output is not present.

As an example, say you have a config file `config.yaml` specifying to run preprocessing and assembly of your samples and you run the workflow as such: 

```bash
snakemake --use-conda -j 4 --configfile config.yaml
```

When the workflow is finished you can then generate a report by running:
```bash
snakemake --use-conda -j 4 --configfile config.yaml --report report.html
```

To see an example of what the report may look like [click here](https://github.com/NBISweden/nbis-meta/suites/767398678/artifacts/7993795) to download a report from one of the test runs of the workflow.

## Examples
Here are a few common examples. They are written in a structure showing the relevant configuration parameters, the command(s) to run and the expected output. All examples assume you have a configuration file called `config.yaml` with the appropriate parameters, but you may of course use any config file name you want. A suggestion is to make a copy of the [default config](https://github.com/NBISweden/nbis-meta/blob/master/config/config.yaml) file and make your changes in the copy.

### Assembly-based analysis

#### Assemble reads with Megahit

##### Configuration
```yaml
assembly:
  # run Megahit assembler?
  megahit: True
  # Use Metaspades instead of Megahit for assembly?
  metaspades: False

megahit:
  # maximum threads for megahit
  threads: 20
  # keep intermediate contigs from Megahit?
  keep_intermediate: False
  # extra settings passed to Megahit
  extra_settings: "--min-contig-len 300 --prune-level 3"
```

##### Command
```bash
snakemake --use-conda --configfile config.yaml -j 4 -p assemble
```

##### Output
```
results
|- assembly/               
|  |- <assembly1>/final_contigs.fa   the fasta file with assembled contigs
|  |- ...
|  |- <assemblyN>/final_contigs.fa   
|- report/
|  |- assembly/
|  |  |- assembly_stats.txt          table of assembly statistics         
|  |  |- assembly_size_dist.txt      file with sizes of assemblies contained at different contig lengths
|  |  |- assembly_stats.pdf          a plot of general assembly statistics
|  |  |- assembly_size_dist.pdf      a plot of the size distribution of the assembly
|  |  |- alignment_frequency.pdf     a plot of the overall alignment frequency after mapping reads to assembled contigs
```

**NOTE**: To use the Metaspades assembler, simply change your config file to:

```yaml
assembly:
  metaspades: True
```

#### Protein-level annotations
Open reading frames called on assembled contigs can be annotated using `eggnog-mapper`, `pfam_scan` and `rgi` (Resistance Gene Identifier). If you are running the workflow on the Uppmax compute cluster you can use centrally installed databases for the first two of these, see more under the section [Running the workflow on Uppmax](https://github.com/NBISweden/nbis-meta/wiki/Running-the-workflow-on-Uppmax#setting-up-database-files]).

##### Configuration
Using these settings in your config file runs all three tools to annotate protein sequences in your assemblies.

```yaml
annotation:
  # run eggnog-mapper to infer KEGG orthologs, pathways and modules?
  eggnog: True
  # run PFAM-scan to infer protein families from PFAM?
  pfam: True
  # run Resistance gene identifier?
  rgi: True
```

##### Command
```bash
snakemake --use-conda --configfile config.yaml -j 4 -p annotate
```

### Read-based analysis

#### Metaphlan

The workflow runs the recently released version 3 of [Metaphlan](https://github.com/biobakery/MetaPhlAn). MetaPhlAn aligns reads to a set of core marker genes and estimates abundances of taxonomic clades in your samples. 

##### Configuration
```yaml
classification:
  metaphlan: True
```

##### Command
```bash
snakemake --use-conda --configfile config.yaml -j 4 -p classify
```

##### Output
```
results
|- metaphlan/               raw, per sample output from metaphlan 
|
|- report/
|- metaphlan/               
|  |- metaphlan.tsv         clade relative abundances per sample
|  |- metaphlan.pdf         clustermap of relative abundance summed to <metaphlan_plot_rank>
|  |- metaphlan.html        Krona interactive plot (Linux only)
```

#### Kraken2

There are pre-built kraken databases available at [https://benlangmead.github.io/aws-indexes/k2](https://benlangmead.github.io/aws-indexes/k2). To make use of _e.g._ the Greengenes prebuilt database, copy its HTTPS url and edit your config file to contain:

```yaml
kraken:
  standard_db: False
  prebuilt: "16S_Greengenes"
  prebuilt_url: "<HTTPS url>" # <- Add the URL here
```

##### Configuration
```yaml
classification:
  kraken: True
```
##### Command
```bash
snakemake --use-conda --configfile config.yaml -j 4 -p classify
```

##### Output
```
results
|- kraken/                  raw, per sample output from kraken2
|
|- report/
|- kraken/               
|  |- kraken.krona.html     Krona interactive plot (Linux only)
```
