## Overview
If you will be running the workflow on the [Uppmax](https://uppmax.uu.se/) HPC clusters here are some helpful tips.

- [Setting up database files](#setting-up-database-files)
  - [eggNOG](#eggnog-database)
  - [Pfam](#pfam-database)
  - [Kraken](#kraken-database)
  - [Centrifuge](#centrifuge-database)
  - [GTDB](#gtdb)
  - [nr](#nr)
  - [UniRef90](#uniref90)
- [Configure workflow for SLURM](#Configure-workflow-for-the-slurm-workload-manager)

## Setting up database files
All resources required to run this workflow are automatically downloaded as needed when the workflow is executed. However, some of these (often large) files are already installed in a central location on the system at `/sw/data`. This means that you can make use of them for your workflow runs thereby saving you time and reducing overall disk-usage for the system.

First create a resource directory inside the base dir of the cloned git repository:
```bash
mkdir resources
```

### eggNOG database
The `4.5.1` version of the eggNOG database is installed in a central location on Uppmax. **However**, this database was built with eggnog-mapper `v1.0.3` and will not work with `v2.0.1` that comes with this workflow. More on this below, but first symlink the necessary files into your `resources/` directory.
```bash
mkdir resources/eggnog-mapper
ln -s /sw/data/eggNOG/4.5.1/eggnog.db resources/eggnog-mapper/eggnog.db
ln -s /sw/data/eggNOG/4.5.1/eggnog_proteins.dmnd resources/eggnog-mapper/eggnog_proteins.dmnd
head -1 /sw/data/eggNOG/eggNOG-4.5.1-install-README.md > resources/eggnog-mapper/eggnog.version
touch resources/eggnog-mapper/download.log
```

To change the version of eggnog-mapper that the workflow uses, edit the conda environment file at `workflow/envs/annotation.yaml` so that it reads:

```yaml
channels:
  - bioconda
  - conda-forge
  - defaults
dependencies:
  - python=2.7.15
  - prodigal=2.6.3
  - pfam_scan=1.6
  - eggnog-mapper=1.0.3 # <-- make sure this version is 1.0.3
  - infernal=1.1.2
  - trnascan-se=2.0.5
```

### Pfam database

To use the Pfam database from the central location, create a `pfam` sub-directory under `resources` and link the necessary files from the central location, run the following:
```bash
mkdir resources/pfam
ln -s /sw/data/Pfam/31.0/Pfam-A.hmm* resources/pfam/
cat /sw/data/Pfam/31.0/Pfam.version > resources/pfam/Pfam-A.version
```

This installs the necessary files for release `31.0`. Check the directories under `/sw/data/Pfam/` to see available releases.

### Kraken database

For `kraken` there are a number of databases installed under `/sw/data/Kraken2`. Snapshots of the `standard`, `nt`, `rdp`, `silva` and `greengenes` indices are installed on a monthly basis. To use the latest version of the standard index, do the following:

1. Create a sub-directory and link the index files

```bash
mkdir -p resources/kraken/standard
ln -s /sw/data/Kraken2/latest/*.k2d resources/kraken/standard/
```

2. From a reproducibility perspective it's essential to keep track of when the index was created, so generate a version file inside your kraken directory by running:
```bash
file /sw/data/Kraken2/latest | egrep -o "[0-9]{8}\-[0-9]{6}" > resources/kraken/standard/kraken.version
```

3. Modify your config file so that it contains:

```yaml
classification:
  kraken: True

kraken:
  # generate the standard kraken database?
  standard_db: True
```

_Non-standard databases_

Using any of the other non-standard databases from the central location is also a simple process, _e.g._ for the latest SILVA index:

```bash
mkdir -p resources/kraken/silva
ln -s /sw/data/Kraken2/latest_silva/*.k2d resources/kraken/silva/
file /sw/data/Kraken2/latest_silva | egrep -o "[0-9]{8}\-[0-9]{6}" > resources/kraken/silva/kraken.version
```

Then update your config file with: 

```yaml
kraken:
  standard_db: False
  custom: "resources/kraken/silva"
```

### Centrifuge database

There are several centrifuge indices on Uppmax located at `/sw/data/Centrifuge-indices/20180720/`, but keep in mind that they are from 2018.

The available indices are: `p_compressed`, `p_compressed+h+v`, `p+h+v`, and `p+v` (see the [centrifuge manual](https://ccb.jhu.edu/software/centrifuge/manual.shtml) for information on what these contain).

To use the `p_compressed` index, run:

```bash
mkdir -p resources/centrifuge/p_compressed/
ln -s /sw/data/Centrifuge-indices/20180720/p_compressed.*.cf resources/centrifuge/p_compressed/
```
Then update your config file to contain:
```yaml
classification:
  centrifuge: True

centrifuge:
  custom: "resources/centrifuge/p_compressed/p_compressed"
```

### GTDB

To use the centrally installed Genome Taxonomy Database (GTDB) release on Uppmax, do:

```bash
mkdir -p resources/gtdb
ln -s /sw/data/GTDB/R04-RS89/rackham/release89/* resources/gtdb/
```

Then make sure your config file contains:
```yaml
binning:
  gtdbtk: True
```

### nr
Uppmax provides monthly snapshots of the `nr` non-redundant database. While the formatted file cannot be used directly with the `nbis-meta` workflow you can save time by making use of the already downloaded fasta file.

To use the latest snapshot of `nr` for taxonomic annotation of contigs, do:
```bash
mkdir resources/nr
ln -s /sw/data/diamond_databases/Blast/latest/download/nr.gz resources/nr/nr.fasta.gz
```

Then update the `taxonomy` section in your config file to use the `nr` database:
```yaml
taxonomy:
  database: "nr"
```

### Uniref90
The UniRef90 database is clustered at 90% sequence identity and Uppmax provides downloaded fasta files that can be used directly with the workflow.

To use the latest snapshot of `UniRef90` for taxonomic annotation of contigs, do:
```bash
mkdir resources/uniref90
ln -s /sw/data/diamond_databases/UniRef90/latest/download/uniref90.fasta.gz resources/uniref90/uniref90.fasta.gz
```

Then update the `taxonomy` section in your config file to use the `uniref90` database:
```yaml
taxonomy:
  database: "uniref90"
```

## Configure workflow for the SLURM Workload Manager

The workflow comes with the [SLURM snakemake profile](https://github.com/Snakemake-Profiles/slurm) pre-installed. All you have to do is to modify the `config/cluster.yaml` file and insert your cluster account ID:

```yaml
__default__:
  account: staff # <-- exchange staff with your SLURM account id
```

Then you can run the workflow with `--profile slurm` from the root of the git repo, _e.g._:

```bash
snakemake --profile slurm -j 100 --configfile myconfig.yaml
```