<!-- [![AAFC](https://avatars1.githubusercontent.com/u/4284691?v=3&s=200)](http://www.agr.gc.ca/eng/home/)-->

# Paired End GBS Pipeline

> A bioinformatics tool to generate a vcf file from a multiplexed GBS paired end fastq files.  This process is based on our protocol (which is also availble in the [resources folder](https://github.com/elderberry-smells/GBS_snakemake_pipeline/tree/master/workflow/resources/oligos).  H

> However, due to the modularity of this protocol, you may only need to change out the demultiplex script to include other protocols seamlessly (such as the [iGenomics Riptide GBS protocol](https://github.com/elderberry-smells/GBS_snakemake_pipeline/blob/master/workflow/scripts/riptide.py))

> This tool utilizes the workflow management system [Snakemake](https://snakemake.readthedocs.io/en/stable/).  This tool allows you to accomplish a complete pipeline process with just one command line call. 

![workflow](workflow/resources/images/pipeline_workflow_DAG.jpg?raw=true "Workflow")

## Table of Contents
- [Quick Use Guide](#quick-use-guide)
- [Installation](#installation)
- [Usage](#usage)
- [Running the pipeline](#Executing)
- [Features](#features)
- [Team](#team)
- [License](#license)

## Quick Use Guide
> This is assuming you have followed the installation.  If not, proceed first to [installations](#installation).
> Once your environment is set up, and the pathways reflect your user name, you can start running the pipeline!

- input required:
	- `sample_R1.fastq.gz`
	- `sample_R2.fastq.gz` 
	- `samplesheet.txt` for format [see samplesheet](#Samplesheet) below

- update the config/config.yaml file.  Use absolute paths.
```shell script
$  cd gbs/GBS_snakemake_pipeline
$  nano config/config.txt

sample_read1: "/absolute/path/to/sample_R1.fastq.gz"
samplesheet: "/absolute/path/to/samplesheet.txt"
barcodefile:  "workflow/resources/barcodes/barcodes_192.txt"
reference_file: "/absolute/path/to/reference.fasta

```
`cntrl-o` to save [enter]

`cntrl-x` to exit

- run the pipeline by submitting the process to the gridengine queue

```shell script
$ qsub workflow/resources/gbs.sh
```

- this will give you a job number if submitted properly.  The job will produce snakemake stats (which rule is running) in whatever folder you run the script from (in this case ~/gbs/GBS_snakemake_pipeline) and be named `gbs.sh.e(jobnumber)`
- you can check to make sure the program is running by typing `qstat -f`.  If not, check the `gbs.sh.e(jobnumber)` for why.
- runtime = 2-3 days for a standard output from Illumina HiSeq.   

## Installation
> download the repository using [git clone](#clone) or by clicking `clone or download` on the main page of the repository
and dowloading the zip file.  
- in your home directory on the cluster/local computer create a directory for the pipeline to live called `gbs` 
```shell script
$ mkdir ~/gbs & cd ~/gbs/
```
#### Cloning the repository to your home directory in cluster 
> you will need git installed on your computer to accomplish this.  Can be done using `conda install -c anaconda git` on your cluster account.
- Use the `git clone` process to make an exact copy of the current repository
- clone this repository to your local machine using the following:

```shell script
$ git clone https://github.com/elderberry-smells/GBS_snakemake_pipeline.git
```
- press `enter`, your local clone should be created with the following pathways for the pipeline:

    - `~/gbs/GBS_snakemake_pipeline/snakefile`
    - `~/gbs/GBS_snakemake_pipeline/config/`
    - `~/gbs/GBS_snakemake_pipeline/workflow/envs/`    
    - `~/gbs/GBS_snakemake_pipeline/workflow/rules/`
    - `~/gbs/GBS_snakemake_pipeline/workflow/scripts/`
    - `~/gbs/GBS_snakemake_pipeline/workflow/resources/`
 
#### Installing the GBS snakemake environment on your computer
- Create an environment for snakemake using the `workflow/envs/gbs.yaml` file from the repo you downloaded
```shell script
$ conda env create -f ~/gbs/GBS_snakemake_pipeline/workflow/envs/gbs.yaml
```

#### bioinformatics dependencies
- all of these dependencies should be installed from the environment being created except for novosort

`python=3.6`
`trimmomatic=0.39`
`bwa=0.7.17`
`samtools=1.9`
`bcftools=1.8`
`Novosort=2.00.00`

- move the novosort folder to your newly created gbs environment bin
    
    ```shell script
    $ mv GBS_snakemake_pipeline/workflow/resources/novocraft/ ~/miniconda3/envs/gbs/bin/
    ```
      
    - add that folder to your profile so novosort is a callable command
    
    ```shell script
    $ nano ~/.bashrc 
    ```
    - add the path to the novocraft folder to the bottom of the file, save, and exit.  Change the path below to the correct one on your machine
    
    `export PATH="$PATH:~/miniconda3/envs/gbs/bin/novocraft"` 

#### Some minor changes to the snakefile and rule files to include your user name
- You will have to edit the file to reflect your username in the scripts before it will run.  You can do this with whichever editor you would like, `nano` would be fine for these small edits.  
- unfortunately using a ~ in snakefiles to direct scripts to your home dir doesn't work
- replace /home/AAFC-AAC/`your_user_name`/gbs... with your actual username

##### Snakefile
-  line 30
-  line 52 - 56 
##### workflow/rules/demultiplex.smk
- line 10
##### workflow/rules/trimmomatic.smk
- line 5

## Usage

The snakemake tool is structured in a way that you need only activate the environment and run the snakefile  after you update the `config/config.yaml` to direct the program.

### Config file
The config file is 4 lines, all required inputs for the snakefile to work properly

```
sample_read1: "/absolute/path/to/sample_R1.fastq.gz"
samplesheet: "/absolute/path/to/samplesheet.txt"
barcodefile:  "workflow/resources/barcodes/barcodes_192.txt"
reference_file: "/absolute/path/to/reference.fasta
```

### Samplesheet
The sample sheet is a tab delimited txt file with 3 columns.  An example of a samplesheet is shown below:

```
Sample_number	Index_name	Sample_ID
1	gbsx001	Sample1	
2	gbsx002	Sample2	
3	gbsx003	Sample3	
4	gbsx004	Sample4	
5	gbsx005	Sample5	
6	gbsx006	Sample6	
7	gbsx007	Sample7	
8	gbsx008	Sample8	
9	gbsx009	Sample9
10	gbsx010	Sample10
```

The barcode file, as seen in the barcode folder `workflow/resources/barcodes/` are 2 columns, tab delimited txt files.  This file does not have any headers

`col1: index_name` `col2: barcode`

example barcode file:

```
gbsx001	TGACGCCATGCA
gbsx002	CAGATATGCA
gbsx003	GAAGTGTGCA
gbsx004	TAGCGGATTGCA
gbsx005	TATTCGCATTGCA
gbsx006	ATAGATTGCA
gbsx007	CCGAACATGCA
gbsx008	GGAAGACATTGCA
gbsx009	GGCTTATGCA
gbsx010	AACGCACATTTGCA
```

## Executing

### Using interactive node (not recommended for large files)

Update the `config/config.yaml` file to direct the pipeline to the resources it needs.

If running on interactive node, qlogin with requested cores before activating environment 

`qlogin -pe smp 16`

then activate the GBS pipeline and change directory to where the snakefile is located  
```shell script
$ conda activate gbs
$ cd ~/gbs/GBS_snakemake_pipeline/
```

You can invoke the process by running one command, and if you have the ability to request cores using the -j option

- test to see if the snakefile is reading your data correctly

```shell script
$ snakemake -np
```
the snakemake process should show you all it intends to do with the samples locate din the samplesheet, and how many process for each module.  example:

```
Job counts:
	count	jobs
	1	all
	184	bwa_map
	1	demultiplex
	184	novosort
	1	samtools_call
	184	trimmomatic
	555
```

- run the program with requested cores on local machine or interactive node on cluster (use nohup in case you need to close terminal) 
```shell script
$ nohup snakemake -j 16
``` 

###  Running the program on the cluster with qsub using shell script from `workflow/resources/gbs.sh`
- go back to the [quick use guide](#quick-use-guide) for instructions on running this on gridengine


## Features
### Brief documentation of what each step in this pipeline accomplishes

This tool utilizes Snakemake to create a cradle-to-grave GBS analysis for paired end reads from Illumina sequencing platforms (2x150).

The tool will commit the following steps in the pipeline, all of which are modular additions to the snakefile and can be swapped if needed with other tools.

### split
input:  

`fastq_R1.fastq.gz, fastq_R2.fastq.gz`

output: 
```
chunks/fastq_R1_aa.fq ... chunks/fastq_R1_zz.fq, 
chunks/fastq_R2_aa.fq ... chunks/fastq_R2_zz.fq
```

the program will count the lines in the sample_R1.fastq.gz file and divy that up into split files using `split` command (default 100M lines for split).  Currently can only accomplish split permuations from aa to zz.  Each file will come out of this stage into a newly created chunks/ folder.

### demultiplex
input: 
```
chunks/fastq_R1_aa.fq...chunks/fastq_R1_zz.fq, 
chunks/fastq_R2_aa.fq...chunks/fastq_R2_zz.fq
```

output: 
```
demultiplex/sample1.1.fastq ... demultiplex/sample384.1.fastq, 
demultiplex/sample1.2.fastq ... demultiplex/sample384.2.fastq
```

In paired end reads, the Fastq read 1 houses the unique identifier barcodes for demultiplexing.  The barcodes in this tool are designed based on the 
[Poland et. al 2012 GBS protocol](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0032253), and the barcodes being used (up to 384 unique barcodes for multiplexing) can be found in `workflow/resources/barcodes_384.txt`

This tool is the only custom processing script created.  It will demultiplex read 1 and read 2 into seperate files by matching the barcodes in the sequence and appending to the newly cerated sample files in chunks of 25 million sequences at a time (done through the usage of multiprocessing).  The [demux script](https://github.com/elderberry-smells/GBS_snakemake_pipeline/blob/master/workflow/scripts/PE_fastq_demultiplex.py) reads both read 1 and read 2 concurrently, so matching of header information in fastq is critical for the process.

parameters for demultiplex script:

`python3 PE_fastq_demultiplex.py -f fastq_R1.fastq.gz -b barcode_file.txt -s samplesheet.txt`

### trimmomatic
input: 
```
demultiplex/sample1.1.fastq ... demultiplex/sample384.1.fastq,
demultiplex/sample1.2.fastq ... demultiplex/sample384.2.fastq
```

output:

```
trimmomatic/sample_id.1.paired, trimmomatic/sample_id.1.unpaired
trimmomatic/sample_id.2.paired, trimmomatic/sample_id.2.unpaired
```

The next step is to pipe the demultiplexed files into the trimmomatic tool.  Trimmomatic is freely available and is a fast, multithreaded command line tool that can be used to trim and crop Illumina (FASTQ) data as well as to remove adapters. 

parameters used in trimmomatic:
```
trimmomatic PE -threads 16 -phred33 sample_id.1.fastq sample_id.2.fastq out.1.paired out.1.unpaired out.2.paired output.2.unpaired
ILLUMINACLIP:{trim_file}:2:40:15 LEADING:15 TRAILING:15 SLIDINGWINDOW:4:15 MINLEN:55
```

### Alignment
input:
```
trimmomatic/sample_id.1.paired, trimmomatic/sample_id.1.unpaired,
trimmomatic/sample_id.2.paired, trimmomatic/sample_id.2.unpaired
```

output:

`mapped_reads/sample1.bam ... mapped_reads/sample384.bam`

The trimmed files are passed through the BWA MEM for alignment to the reference genome.  The reference genome is grabbed by looking into the original config file updated by the user.  Threading for this is set at 16 but can be upped if you reserved more cores by altering the workflow/resources/gbs.sh, as well as the threads: in the bwa_map.smk rule.

parameters used for bwa mem:

`bwa mem -t 16 reference.fasta trim1.fq trim2.fq | samtools view -Shbu > sample.bam`

### Sorting
input:

`mapped_reads/sample1.bam ... mapped_reads/sample384.bam`

output:
```
sorted_reads/sample1.sorted.bam ... sorted_reads/sample384.sorted.bam,
sorted_reads/sample1.sorted.bam.bai ... sorted_reads/sample384.sorted.bam.bai
```

This rule uses the freely available software Novosort (from Novocraft).  This will sort and index the bam filesfrom the bwa mem rule.  The sorted bam file will move into the SNP calling in the final rule

parameters used for novosort:

`novosort sample_id.bam --threads 16 --index --output sample_id.sorted.bam`

### Generating SNP calls and VCF file
input:

`sorted_reads/sample1.sorted.bam ... sorted_reads/sample384.sorted.bam`

output:

`snps.raw.vcf.gz`

Calls are generated using samtools mpileup, and visualized in a VCF file using bcftools calls.  samtools mpileup is deprecated and will switch to bcftools mpileup in the future.  The snakemake pipeline will compile a list of samples to run though samtools mpileup `bam.list` in the sample directory, this is based on the samplesheet file names.  

parameters used in samtools mpileup:

`samtools mpileup -u -t AD,DP -f reference.fasta -b bam.list | bcftools call -mv -Oz > snps.raw.vcf.gz`

## Team
- Author:  Brian James (brian.james4@canada.ca)
- Testing and Improvements:  Jana Ebersbach
- pipeline loosely based on original Perl pipeline script written by Wayne Clarke

## License
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
