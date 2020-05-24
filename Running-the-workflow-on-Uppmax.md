## Overview
If you will be running the workflow on the [Uppmax](https://uppmax.uu.se/) HPC clusters here are some helpful tips.

## Setting up the database files
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

For `kraken` there are a number of databases installed under `/sw/data/Kraken2`. Monthly snapshots of the `standard`, `nt`, `rdp`, `silva` and `greengenes`. 

## Configure workflow for the SLURM Workload Manager

1. Create a directory to hold the profile:

```
mkdir profiles
```

2. Install and activate cookiecutter with conda:

```
conda env create -f envs/cookiecutter.yml
conda activate cookiecutter
```

3. Install the snakemake profile into the profiles/ directory:

```
cookiecutter -o profiles https://github.com/Snakemake-Profiles/slurm.git
```

You will be prompted for the account to charge compute hours to as well
as the default partition. Once this is done you can run snakemake on
the cluster using `snakemake --profile profiles/slurm -j 100`.
