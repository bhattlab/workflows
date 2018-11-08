## First time setup

First, install [miniconda3](https://conda.io/miniconda.html).  Then, if you haven't already, set up a Snakemake profile for SCG by following the instructions [here](https://github.com/bhattlab/slurm).  Then:

```
conda env create -f envs/workflow.yaml
```

## Running the workflow

```
source activate <workflow>
snakemake --configfile /path/to/config.yaml -s /path/to/classification/Snakefile --profile scg
```


# Preprocessing 

To use this pipeline, edit parameters in the config.yaml, and specify the proper path to config file in the submission script.

**The only parameters that need changing in the config file**
1. directory path containing demultiplexed raw fastq files (DATA_DIR)
2. root directory path for output files (PROJECT_DIR)
3. directory path for the host reference genome (BWA index)

*This program runs under the assumption samples are named <sample_id>_[R]1.fastq[fq].gz and <sample_id>_[R]2.fastq[fq].gz.* The R1/R2 and suffix must be specified in the config. 

**This script will create the following folders:**
- PROJECT_DIR/01_processing/00_qc_reports/pre_fastqc
- PROJECT_DIR/01_processing/00_qc_reports/post_fastqc
- PROJECT_DIR/01_processing/01_trimmed
- PROJECT_DIR/01_processing/02_dereplicate
- PROJECT_DIR/01_processing/03_sync
- PROJECT_DIR/01_processing/04_host_align

The files that can then be used in downstream analyses will be in PROJECT_DIR/qc/04_04_host_align/ with the names {sample}_rmHost_1.fq and {sample}_rmHost_2.fq 

### Reference genomes for removal of host reads
For Bhatt lab purposes, we only conduct experiments on two hosts, humans and mice. You can specify the host reference genomes in the config using the following directories. 
- **Humans:** 
``` /labs/asbhatt/data/host_reference_genomes/hg19/hg19.fa ```
- **Mice:** 
``` /labs/asbhatt/data/host_reference_genomes/mm10/mm10.fa ```

If working on a different cluster or different model organism, these are the steps necessary to build the host reference index for alignment. I am showing the steps used to build Bhatt lab hosts from above using BWA. 

Download reference genomes
```
mkdir -p /labs/asbhatt/data/host_reference_genomes/ # change to preferred directory path
cd /labs/asbhatt/data/host_reference_genomes/ 
wget http://hgdownload.soe.ucsc.edu/goldenPath/hg19/bigZips/hg19.2bit; 
```
Convert to fasta format
```
wget http://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64/twoBitToFa;
chmod +x twoBitToFa; 
./twoBitToFa hg19.2bit hg19.fa; 
```
Create bowtie index
```
bwa index hg19.fa
```


# Classification and taxonomic barplots

Given one or more input datasets, this workflow performs taxonomic classification with kraken, then visualizes 
the resulting compositions in a barplot.  Accepts an input table, specified in the config.yaml (see example), that looks like this:

```
Sample	Timepoint	Condition	Reads1.fq[.gz][,Reads2.fq[.gz]]
foo	a	control	a_1.fq,a_2.fq
bar	b	case	b_1.fq,b_2.fq
```


# Assembly

Assemble preprocessed read data. Launches either Spades or Megahit. Evaluates result with Quast. Accepts an input table, specified in the config.yaml (see example), that looks like this:

```
Sample	Reads1.fq[.gz][,Reads2.fq[.gz][,orphans.fq[.gz]]]
foo	a_1.fq,a_2.fq
bar	b_1.fq,b_2.fq
```

