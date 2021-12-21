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
docker pull nbisweden/nbis-meta:2.2
```

to get version `2.2`.
