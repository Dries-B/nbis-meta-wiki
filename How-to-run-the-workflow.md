## Contents

- [Targets](#targets)
- [Reports](#reports)
- [Examples](#examples)
  - [Assembly-based analysis](#assembly-based-analysis)
    - [Megahit](#assemble-reads-with-megahit)
  - [Read-based analysis](#read-based-analysis)
    - [Metaphlan](#metaphlan)
    - [Kraken](#kraken2)

## Targets
If you don't want to run the full workflow from start to finish in one go you may specify one or several 'targets' on the commandline. Some useful targets are:

| Target | Description |
| ------ | ----------- |
| `qc`   | Runs preprocessing (as specified in the configuration file) and generates a `sample_report.html`|
| `assemble` | Runs steps up to and including the assembly part of the workflow and generates some assembly statistics|
| `quantify` | Runs the quantification steps of genes called on assembled contigs (generates raw and normalized count tables)|
| `annotate` | Runs steps up to and including the assembly-based annotation of genes called on assembled contigs|
| `taxonomy` | Runs steps up to and including the taxonomic annotation of contigs|
| `bin` | Runs steps up to and including genome binning and generates some statistics of the binned genomes|
| `classify` | Runs read-based classification (_e.g._ Kraken/Centrifuge/MetaPhlAn)|

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