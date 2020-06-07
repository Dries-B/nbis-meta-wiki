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

You are now ready to start using the workflow with some example data. Check out some common [examples](https://github.com/NBISweden/nbis-meta/wiki/Examples) and see [how to create a sample list](https://github.com/NBISweden/nbis-meta/wiki/Defining-your-samples-in-the-sample-list) to start working with your own data.