The workflow requires you to supply a file with some information on your samples. As a minimum the file must contain the columns `sample`, `unit` and `fq1`.

A very basic sample file may look like this:

| sample | unit | fq1 | fq2 |
| ------ | ---- | --- | --- |
| sample1 | 1 | examples/data/sample1_1_R1.fastq.gz | examples/data/sample1_1_R2.fastq.gz |
| sample2 | 1 | examples/data/sample2_11_R1.fastq.gz | examples/data/sample2_11_R2.fastq.gz |
| sample3 | 1 | examples/data/sample3_21_R1.fastq.gz |

The columns `fq1` and `fq2` (for paired-end data) specify the paths to fastq files which will be used as input to the workflow.