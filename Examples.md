# Overview
Here are a few common examples. They are written in a structure showing the relevant config parameters, the command(s) to run and the expected output. All examples assume you have updated a configuration file called `config.yaml` with the appropriate parameters, but you may of course use any config file name you want.

Contents:

- [Assembly-based analysis](#assembly-based-analysis)
  - [Megahit](#assemble-reads-with-megahit)
- [Read-based analysis](#read-based-analysis)
  - [Metaphlan](#metaphlan)
  - [Kraken](#kraken2)

## Assembly-based analysis

### Assemble reads with Megahit

```bash
snakemake --use-conda --configfile config.yaml -j 4 -p assemble
```

**Output:**
```
results
|- assembly/               
|  |- <assemblyGroup1>
|  |- ...
|  |- <assemblyGroupN>
|- report/
|  |- assembly/
|  |  |- assembly_stats.txt          table of assembly statistics         
|  |  |- assembly_size_dist.txt      file with sizes of assemblies contained at different contig lengths
```

## Read-based analysis

### Metaphlan

The workflow runs the recently released version 3 of [Metaphlan](https://github.com/biobakery/MetaPhlAn). MetaPhlAn aligns reads to a set of core marker genes and estimates abundances of taxonomic clades in your samples. 

**Config:**
```yaml
classification:
  metaphlan: True
```

**Command:**

```bash
snakemake --use-conda --configfile config.yaml -j 4 -p classify
```

**Output:**
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

### Kraken2
