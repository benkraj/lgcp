<style>
body {
    overflow: scroll;
}
</style>

introduction to bioinformatics
========================================================
author: Brian Capaldo
date: 2019 July 16

Computational Benchtop Framework
========================================================

-Wet lab assay

-Computational assay

Biowulf
========================================================

Biowulf is a shared resource, much like a core facility

Getting an [account](https://hpc.nih.gov/docs/accounts.html)

Logging in


```bash
ssh biowulf.nih.gov
```

Managing the shared resources on Biowulf
========================================================

Biowulf uses [SLURM](https://slurm.schedmd.com/documentation.html)

It handles pretty much everything for you, you just need to make requests

Anatomy of a SLURM batch script (aka sbatch)
========================================================


```bash
#! /bin/bash
#SBATCH --cpus-per-task=4
#SBATCH --partition=ccr
#SBATCH --mem=16gb
#SBATCH --time=48:00:00
#SBATCH --mail-type=BEGIN,TIME_LIMIT_90,END
#SBATCH --mail-user=capaldobj@nih.gov

module load graphviz
module load singularity

~/nextflow run nf-core/chipseq \
-resume \
--design '/data/capaldobj/CS02314X-ChIP-seq-results/design.csv' \
--singleEnd \
--narrowPeak \
--genome GRCh37 \
-profile singularity \
-c ~/biowulf.config \
--outdir '/data/capaldobj/CS02314X-ChIP-seq-results/fresh-nf-main/nf-main-core-narrow/'

```

Submitting the job


```bash
sbatch chipseq-pipeline-nf-core-main.sh
```

Pipelines, workflows, and Nextflow
========================================================

In bioinformatics, we'll often run the same assay many times

With modern HPCs (like biowulf), we can use any number of tools to automate this

For example, the CCBR uses [Pipeliner](https://github.com/CCBR/Pipeliner)

Personally, I use [Nextflow](https://www.nextflow.io/)

More expliclty, I use the resources provided by [nextflow core (aka nf-core)](https://github.com/nf-core/)

With nf-core, a suite or ready to use pipelines is available, and more are coming out all the time.

Initially, I would take the nf-core pipelines and adapt them slightly, but [singularity](https://sylabs.io/docs/) has removed that requirement. 

Nextflow as the core facilty of computational assays
========================================================

A nextflow workflow is a collection of commands and software that will perform the end-to-end analsyis of our data

The manager of the facility is the `main.nf`. 


```java
#!/usr/bin/env nextflow
 
params.in = "$baseDir/data/sample.fa"
sequences = file(params.in)
 
/*
 * split a fasta file in multiple files
 */
process splitSequences {
 
    input:
    file 'input.fa' from sequences
 
    output:
    file 'seq_*' into records
 
    """
    awk '/^>/{f="seq_"++d} {print > f}' < input.fa
    """
 
}
 
/*
 * Simple reverse the sequences
 */
process reverse {
 
    input:
    file x from records
     
    output:
    stdout result
 
    """
    cat $x | rev
    """
}
 
/*
 * print the channel content
 */
result.subscribe { println it }
```

These can be really [complicated](https://github.com/nf-core/chipseq/blob/master/main.nf)

The spec sheet is `nextflow.config`, this is a set of instructions that nextflow will use as it processes the sample. 

The lab tech is `biowulf.config`, this is the thing that knows the specific details about biowulf that will allow the nextflow script to run on biowulf. 


```java

/*
 * -------------------------------------------------
 *  Nextflow config file with environment modules for biowulf HPC at NIH
 * -------------------------------------------------
 */

singularity {
  enabled = true
  autoMounts = true
}

process {
  executor = 'slurm'
  clusterOptions = '--partition ccr'
}

params {
  max_memory = 128.GB
  max_cpus = 16
  max_time = 48.h
  igenomes_base = '/fdb/igenomes/'
}
```

It should be noted that you can create one of these for pretty much any situtation, there's premade ones for AWS, and GCP. 
