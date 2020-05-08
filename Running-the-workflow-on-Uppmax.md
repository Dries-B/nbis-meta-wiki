## Setting up the database files
...

## Configure workflow for the SLURM Workload Manager
If you are going to run the workflow on a compute cluster such as
[Uppmax](https://uppmax.uu.se/) you can make use of the snakemake SLURM
profile.

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
