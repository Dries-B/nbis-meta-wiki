## Overview
If you will be running the workflow on the [Uppmax](https://uppmax.uu.se/) HPC clusters here are some helpful tips.

- [Setting up database files](#setting-up-database-files)
- [Configure workflow for SLURM](#Configure-workflow-for-the-slurm-workload-manager)

## Setting up database files
All resources required to run this workflow are automatically downloaded as needed when the workflow is executed. However, some of these (often large) files are already installed in a central location on the system at `/sw/data`. This means that you can make use of them for your workflow runs thereby saving you time and reducing overall disk-usage for the system.

First create a resource directory inside the base dir of the cloned git repository:
```bash
mkdir resources
```

**Pfam database**

To use the Pfam database from the central location, create a `pfam` sub-directory under `resources` and link the necessary files from the central location, run the following:
```bash
mkdir resources/pfam
ln -s /sw/data/Pfam/31.0/Pfam-A.hmm* resources/pfam/
cat /sw/data/Pfam/31.0/Pfam.version > resources/pfam/Pfam-A.version
```

This installs the necessary files for release `31.0`. Check the directories under `/sw/data/Pfam/` to see available releases.

**Kraken database**

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

**Centrifuge database**

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

**GTDB**

To use the centrally installed GTDB release on Uppmax, do:

```bash
mkdir -p resources/gtdb
ln -s /sw/data/GTDB/R04-RS89/rackham/release89/* resources/gtdb/
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