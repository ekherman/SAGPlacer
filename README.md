# SAGPlacer

**Version:** 1.0


## Synopsis

This is a Snakemake workflow that takes Illumina reads generated from single cell 
sequencing and generates an assembly with predicted genes and completeness 
information via BUSCO. The single copy orthologues identified can then be used 
as input for the [EukProt Phylogenetics Pipeline](https://github.com/ekherman/EukProt-Phylogenetics-Pipeline). 


For more information about EukProt, see [EukProt: a database of genome-scale predicted proteins across the diversity of eukaryotic life](https://www.biorxiv.org/content/10.1101/2020.06.30.180687v1.full.pdf).

## Sections

[Installation](#installation)

[Setup](#setup)

[Running SAGPlacer](#running-sagplacer)

[Output](#output)


## Installation
The following programs are required by the pipeline and their paths can 
be supplied to the config file in `config/sagplacer.config.yaml`.

 - [Java 1.8+](https://www.oracle.com/java/technologies/downloads/)
 - [Python 3.7+](https://www.python.org/downloads/)
   - [Snakemake](https://snakemake.github.io/)
 - [Perl v5+](https://www.perl.org/get.html)
 - [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
 - [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic)
 - [SPAdes](https://github.com/ablab/spades)
 - [Assembly-Stats](https://github.com/sanger-pathogens/assembly-stats)
 - [Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml)
 - [Samtools](http://www.htslib.org/)
 - [BUSCO v5+](https://busco.ezlab.org/)
 - [Augustus](https://github.com/Gaius-Augustus/Augustus)
 - [Pannzer](http://ekhidna2.biocenter.helsinki.fi/sanspanz/)


### Downloading the SAGPlacer Pipeline

To download the workflow, either download and extract the zip file, or
run the following:

```
git clone https://github.com/ekherman/SAGPlacer.git
```

## Setup

Edit the configuration file in  `config/sagplacer.config.yaml`. The following 
paths are required:

1. Working directory (default: SAGPlacer)
2. Path to the directory containing read files. The files within must have the 
structure `{sample name}_R{1,2}.fastq.gz` and must be gzip compressed. Results files 
for each sample will have the basename {sample_name}.
3. Directory in which results will be written. 
4. Absolute paths for required programs and associated files. If a program is in the 
user's PATH, the executable may be given instead.


## Running SAGPlacer

To run the pipeline:

```
snakemake -s workflow/SAGPlacer.sm --cluster "sbatch -N 1 -c {threads} --mem={resources.mem_mb} --time={resources.runtime} --account=account-name"
```

All rules in the workflow are designed to run on a single node of a cluster 
with up to 125Gb RAM, with the exception of the local rules listed at the top of 
the `workflow/SAGPlacer.sm` file. 

Cluster configuration values are listed for each rule and specify the 
following slurm cluster options:
- threads: 1 # Number of threads to use for each job; should be equal to or less than cores
- resources:
    - cores = 1,  # Number of cpus available, or that will be requested from the scheduler
    - runtime = 120,  # Requested walltime in minutes
    - mem_mb = 4000,  # Requested memory in MB

### HPC clusters
If you do not have a slurm profile, [set one up here](https://github.com/stothard-group/variant-calling-pipeline/blob/master/slurm_setup.md).


## Output

The following directories and output files will be created within the 
results directory specified by the user:

| Directory        | Description of output files                               |
|------------------|-----------------------------------------------------------|
| fastqc           | FastQC quality results for raw and trimmed reads          |
| trimmomatic      | Reads pre-processed by Trimmomatic                        |
| logs             | Trimmomatic log file                                      |
| spades/{sample}  | Assembly in scaffolds.fasta, statistics in stats.txt      |
| bowtie2          | Mapped read files in .sam and sorted .bam format          |
| busco/{sample} | Summary in short_summary.specific.{database}.{sample}.txt |
| augustus         | {sample}.gff feature predictions and {sample}.aa proteins |
| pannzer          | Output files in {sample}.*.pannzer.txt                    |

It is highly recommended that the assembly is checked for contamination. 
This can be done with [BlobTools](https://blobtools.readme.io/).

