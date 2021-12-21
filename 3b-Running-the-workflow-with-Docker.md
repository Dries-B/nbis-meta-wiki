# Overview
The `nbis-meta` Docker image allows you to run the workflow in a container that uses Linux (debian) as the underlying operating system. This may be beneficial for users who don't have access to a computer with Mac or some Linux distribution, and can also help resolve missing dependencies. For example, the `concoct` and `gtdb-tk` packages are currently not available for Mac but with Docker you can still run a full analysis of your data using these packages on your Mac computer. 

## Install Docker
First of all you need to have Docker installed on your computer. Follow the [installation instructions](https://docs.docker.com/get-docker/) to get started.

## Get the Docker image
Next you need to pull the `nbis-meta` Docker image. Simply run:

```bash
docker pull nbisweden/nbis-meta
```

This pulls the latest version of the workflow from DockerHub. You may also pull a specific version of the workflow, _e.g._ to pull the `2.0` version:

```bash
docker pull nbisweden/nbis-meta:2.0
```

Check the list on [DockerHub](https://hub.docker.com/r/nbisweden/nbis-meta/tags) to see what versions are available.

You can check the list of images on your computer by running `docker images`. This should show something similar to:

```bash
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
nbisweden/nbis-meta        latest              b7774e010682        1 day ago          1.15GB
```

## Run the workflow
The basic command for running a docker container is `docker run <container>`. For running an `nbis-meta` container the command would thus be:

```bash
docker run nbisweden/nbis-meta
```

This will spawn a new Docker container running the `nbis-meta` workflow with default settings (using a default configuration file inside the container).

**However**, this only allocates a local filesystem for the container which means it can only work with data, and produce output, inside the container. In order to use your own files (_e.g._ configuration files and raw data), and generate results on your computer you need to use _bind mounts_ specified with the `-v` flag.

### Example

Let's try an example where we want to run the workflow with Docker using:

1. samples specified in a file called `mysamples.tsv`
2. a configuration file called `myconfig.yaml`
3. raw data stored under `data/` in the current folder.

***

Let's say the sample list `mysamples.tsv` looks like this:

| sample | unit | assembly | fq1 | fq2 |
| ------ | ---- | -------- | --- | --- |
| sample1 | 1 | co-assembly | data/sample1_R1.fastq.gz | sample1_R2.fastq.gz |
| sample2 | 1 | co-assembly | data/sample2_R2.fastq.gz | sample2_R2.fastq.gz |
| sample3 | 1 | co-assembly | data/sample3_R2.fastq.gz | sample3_R2.fastq.gz |

***

The configuration file `myconfig.yaml` looks like this:

```yaml
sample_list: mysamples.tsv
assembly:
  metaspades: True
annotation:
  pfam: True
  infernal: False
classification:
  metaphlan: True
  kraken: False
```

With this set up the workflow will read sample information from `mysamples.tsv`, use the `metaspades` assembler to create 1 assembly called `co-assembly`, annotate it with PFAM protein families, skip the identification of rRNA genes with `infernal`, create a taxonomic profile of each sample using `metaphlan` but skip classification of reads with `kraken`. 

Note that the configuration file does not need to contain the full [list of parameters](https://github.com/NBISweden/nbis-meta/wiki/Workflow-configuration). Instead, parameters specified in a configuration file passed at runtime will overwrite the default settings for the workflow.

***

Our directory now contains the following:
```
data/
  sample1_R1.fastq.gz
  sample1_R2.fastq.gz
  sample2_R1.fastq.gz
  sample2_R2.fastq.gz
  sample3_R1.fastq.gz
  sample3_R2.fastq.gz
myconfig.yaml
mysamples.tsv
```

***

In order to run the workflow with Docker using this set up we need to make the `data/` directory, the `myconfig.yaml` config file and the `mysamples.tsv` sample list available to the container. 

Also, to make the output from the workflow available on your local filesystem we need to mount the proper results directory.

This can be done by specifying `-v` for each bind mount on the command line:

```bash
docker run -v "$(pwd)"/data:/analysis/data \
    -v "$(pwd)"/myconfig.yaml:/analysis/myconfig.yaml \
    -v "$(pwd)"/mysamples.tsv:/analysis/mysamples.tsv \
    -v "$(pwd)"/results:/analysis/results \
    nbisweden/nbis-meta --configfile myconfig.yaml
```

Everything you specify after the `nbisweden/nbis-meta` part of the command line call is passed to snakemake as if you had cloned the GitHub repository, installed the conda environment and called snakemake from there. Here we pass `--configfile myconfig.yaml` but you can for instance also pass `--cores 4` to run the workflow with 4 cpus instead of 1:

```bash
docker run -v "$(pwd)"/data:/analysis/data \
    -v "$(pwd)"/myconfig.yaml:/analysis/myconfig.yaml \
    -v "$(pwd)"/mysamples.tsv:/analysis/mysamples.tsv \
    -v "$(pwd)"/results:/analysis/results \
    nbisweden/nbis-meta --configfile myconfig.yaml --cores 4
```

You could also run parts of the workflow by supplying a valid [target](https://github.com/NBISweden/nbis-meta/wiki/How-to-run-the-workflow#targets), _e.g._ `qc` to only run the preprocessing and quality control part of the workflow:

```bash
docker run -v "$(pwd)"/data:/analysis/data \
    -v "$(pwd)"/myconfig.yaml:/analysis/myconfig.yaml \
    -v "$(pwd)"/mysamples.tsv:/analysis/mysamples.tsv \
    -v "$(pwd)"/results:/analysis/results \
    nbisweden/nbis-meta --configfile myconfig.yaml --cores 4 qc
```

### Cleaning up
Docker containers and images can quickly eat up disk space on your computer. You can list running and stopped containers with `docker ps -a` and remove containers with `docker rm <container id>`.

To have docker automatically remove containers upon completion or exit add `--rm` to the `docker run` call.