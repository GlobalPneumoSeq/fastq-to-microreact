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
- [Creating Microreact Instance](#creating-microreact-instance)

## Prerequisites & Environment Setup
You can use Linux, Windows ([with WSL2](https://learn.microsoft.com/en-us/windows/wsl/install)), and macOS to follow this tutorial. We recommend using a system with at least 16GB of RAM. 

### Required Tools
We will be using the following tools. Follow the [next section](#installing-tools) to install if any of them is not available on your system.
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


## Building Phylogenetic Tree


## Creating Microreact Instance

