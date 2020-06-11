# Getting started

## From GitHub
To start using the workflow either clone the latest version of the repo:

```bash
git clone https://github.com/NBISweden/nbis-meta.git
```

or download the latest release from the [release page](https://github.com/NBISweden/nbis-meta/releases) and extract the archive. 

Then change directory into the `nbis-meta` folder and create the core conda environment:

```bash
cd nbis-meta
conda env create -f environment.yml
```

## From DockerHub
You may also pull the latest Docker image:

```bash
docker pull nbisweden/nbis-meta:2.0
```

to get version `2.0`.


## What's next?
You are now ready to start using the workflow! Check out the [How-to page](https://github.com/NBISweden/nbis-meta/wiki/How-to-run-the-workflow), see [how to create a sample list](https://github.com/NBISweden/nbis-meta/wiki/Defining-your-samples-in-the-sample-list) to start working with your own data or delve into the [configuration parameters](https://github.com/NBISweden/nbis-meta/wiki/Workflow-configuration).