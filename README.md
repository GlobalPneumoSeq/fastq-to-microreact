# FASTQ to Microreact for *Streptococcus pneumoniae* <!-- omit in toc -->
This tutorial is specifically tailored for *Streptococcus pneumoniae* and provides a step-by-step workflow for transforming raw read sequencing files (FASTQ) into an interactive Microreact visualisation. The tutorial begins with software environment setup. It then covers quality control and the generation of *in silico* data (such as GPSCs, serotypes, and AMR profiles) using the GPS Pipeline, followed by phylogenetic tree construction, and Microreact instance creation.


## Table of Content <!-- omit in toc -->
- [Prerequisites \& Environment Setup](#prerequisites--environment-setup)
  - [Required Tools](#required-tools)
  - [Installing Tools](#installing-tools)
    - [GPS Pipeline](#gps-pipeline)
    - [Tree Building Tools  (`snippy`, `snp-sites`, `fasttree`, `gubbins`)](#tree-building-tools--snippy-snp-sites-fasttree-gubbins)
- [Quality Control \& Generating *in silico* Data](#quality-control--generating-in-silico-data)
- [Building Phylogenetic Tree](#building-phylogenetic-tree)
  - [Filtering inputs for tree building](#filtering-inputs-for-tree-building)
  - [Prepare snippy-multi input files](#prepare-snippy-multi-input-files)
  - [Run `snippy-multi`](#run-snippy-multi)
- [Creating Microreact Instance](#creating-microreact-instance)

## Prerequisites & Environment Setup
You can use Linux, Windows ([with WSL2](https://learn.microsoft.com/en-us/windows/wsl/install)), and macOS to follow this tutorial. We recommend using a system with at least 16GB of RAM. 

### Required Tools
- [GPS Pipeline](https://github.com/GlobalPneumoSeq/gps-pipeline)
- [Snippy](https://github.com/tseemann/snippy)
- [SNP-sites](https://github.com/sanger-pathogens/snp-sites)
- [FastTree](https://github.com/morgannprice/fasttree)
- [Gubbins](https://github.com/nickjcroucher/gubbins)

### Installing Tools
#### GPS Pipeline
1. Install dependencies
   1. Install OpenJDK 17 (or later, up to 24) (use [`SDKMAN!`](https://sdkman.io/install/) to install an appropiate version of [Temurin distribution](https://sdkman.io/jdks/tem/))
   2. Install Docker or Singularity/Apptainer
       - Linux: [Docker Engine](https://docs.docker.com/engine/install/) / [Apptainer](https://apptainer.org/docs/admin/main/installation.html)
       - macOS: [Docker Desktop for macOS](https://docs.docker.com/desktop/setup/install/mac-install/)
       - Windows with WSL2: [Docker Desktop for WSL2](https://docs.docker.com/desktop/features/wsl/)
2. Clone and initialise the pipeline
    > `git` should come preinstalled on most systems. If not, follow the [official installation guide](https://github.com/git-guides/install-git).
   1. Open Terminal/CLI and go into where you want to keep the pipeline
       ```
       cd /path/of/your/choice
       ```
   2. Clone the repository
       ```
       git clone https://github.com/GlobalPneumoSeq/gps-pipeline.git
       ```
   3. Go into the directory of the pipeline
       ```
       cd gps-pipeline
       ```
   4. (Optional) Initialise the pipeline, so it can be used any time without the Internet
        > If using Singularity/Apptainer, add `-profile singularity` to the below command
       ```
       ./run_pipeline --init
       ```

#### Tree Building Tools  (`snippy`, `snp-sites`, `fasttree`, `gubbins`)
1. [Install Conda](https://conda-forge.org/download/)
2. Install `snippy`, `snp-sites`, `fasttree` in a Conda environment named `buildtree`
    > If using [Mac computers with Apple Silicon](https://support.apple.com/en-gb/116943), append `--platform osx-64` to the below command
    ```
    conda create -n buildtree "snippy>=4.6.0" "snp-sites>=2.5.1" "fasttree>=2.2.0"
    ```
3. Install `gubbins` in a Conda environment named `gubbins`
   > `gubbins` need to be installed to a separated Conda environment due to package conflict with other tools.
    ```
    conda create -n gubbins "gubbins>=3.4.3"
    ```
## Quality Control & Generating *in silico* Data
We use the GPS Pipeline to process raw read sequencing files (FASTQ) of *Streptococcus pneumoniae* samples, it will perform quality assessment (QC) and *in silico* typing (including serotype, MLST, GPSC, and AMR) fully automatic.

1. Saving FASTQ files of all samples in a single directory
2. Run the pipeline by specifying where the FASTQ files are stored (`--reads`) and the output directory (`--output`)
   > If using Singularity/Apptainer, add `-profile singularity` to the below command
   ```
    ./run_pipeline --reads /path/to/fastqs --output /path/to/output
   ```
3. Once the run is completed, you will find `results.csv` in your specified output directory

## Building Phylogenetic Tree
### Filtering inputs for tree building
Before building a tree, we will filter out samples that failed QC in the GPS Pipeline. We will be saving a copy of `results.csv` with only QC passed samples as `results_qcpass.csv` and a list of those samples as `ids_qcpass.txt`
1. Go to GPS Pipeline output directory and extract GPS Pipeline results of QC-passed samples
    ```
    awk -F , ' FNR==1 || $6=="PASS" ' results.csv > results_qcpass.csv
    ```
2. Extract names of QC passed samples
    ```
    awk -F , ' FNR >1 { print $1 } ' results_qcpass.csv > ids_qcpass.txt
    ```

### Prepare snippy-multi input files
We use `snippy-multi` script to run QC passed reads against the same reference.
1. Run the following command to create an input file named `snippy_multi_input.tsv`
   > Update values of `READS_DIR` and `IDS_QCPASS` to the correct one
   ```
    READS_DIR="/path/to/fastqs"
    IDS_QCPASS="/path/to/ids_qcpass.txt"

    while read -r ID; do 
        echo ${ID}$'\t'${READS_DIR}/${ID}_1.fastq.gz$'\t'${READS_DIR}/${ID}_2.fastq.gz >> snippy_multi_input.tsv;
    done < ${IDS_QCPASS}
   ```
2. Get a reference sequence
    > We recommend the sequence in the below command as a general reference sequence. You might want to use [other reference genomes](https://github.com/GlobalPneumoSeq/gpsc-reference-genomes/tree/main/fasta) if you are working on a lineage/GPSC-specific analysis
    ```
    wget https://raw.githubusercontent.com/GlobalPneumoSeq/gps-pipeline/refs/heads/master/data/ATCC_700669_v1.fa
    ```

### Run `snippy-multi`
1. Activate `buildtree` Conda environment
   ```
   conda activate buildtree
   ```
2. Run snippy-multi to generate the script `runme.sh`, which will create the alignment in the next step
   > `--cpus $(nproc)` uses all available CPU cores when running the generated script
   
   > Update the value of `--ref` if you are using an alternative reference sequence
   ```
   snippy-multi snippy_multi_input.tsv --ref ATCC_700669_v1.fa --cpus $(nproc) > runme.sh 
   ```
3. Grant execution permission to the newly created file so that you can run it
   ```
   chmod +x runme.sh 
   ```
4. Run the script
   ```
   ./runme.sh
   ```
5. Clean alignment ([see here](https://github.com/tseemann/snippy?tab=readme-ov-file#why-is-corefullaln-an-alphabet-soup) for why the cleanup is needed)
   ```
   snippy-clean_full_aln core.full.aln > clean.full.aln
   ```
6. Remove intermediate directories
   ```
   find . -mindepth 1 -maxdepth 1  -type d -delete
   ```

## Creating Microreact Instance

