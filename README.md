# FASTQ to Microreact <!-- omit in toc -->
This is a tutorial on how to go from raw read files (FASTQ) to an interactive Microreact visualisation. It covers the installation of the tools, quality control and generating *in silico* data using the GPS Pipeline, and building phylogenetic tree using several open source tools.

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
1. Install dependencies of the GPS Pipeline
   1. Install OpenJDK 17 (or later, up to 24) (use [`SDKMAN!`](https://sdkman.io/install/) to install an appropiate version of [Temurin distribution](https://sdkman.io/jdks/tem/))
   2. Install Docker or Singularity/Apptainer
       - Linux: [Docker Engine](https://docs.docker.com/engine/install/) / [Apptainer](https://apptainer.org/docs/admin/main/installation.html)
       - macOS: [Docker Desktop for macOS](https://docs.docker.com/desktop/setup/install/mac-install/)
       - Windows with WSL2: [Docker Desktop for WSL2](https://docs.docker.com/desktop/features/wsl/)
2. Install the GPS Pipeline
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
       - Using Docker as the container engine
          ```
          ./run_pipeline --init
          ```
       - Using Singularity/Apptainer as the container engine
          ```
          ./run_pipeline --init -profile singularity
          ```

#### Tree Building Tools  (`snippy`, `snp-sites`, `fasttree`, `gubbins`)
1. [Install Conda](https://conda-forge.org/download/)
> `snippy`, `snp-sites`, `fasttree` and `gubbins` need to be installed to separated Conda environments due to package conflict.
2. Install `snippy`, `snp-sites`, `fasttree` in a Conda environment named `buildtree`
    - For most systems:
        ```
        conda create -n buildtree "snippy>=4.6.0" "snp-sites>=2.5.1" "fasttree>=2.2.0"
        ```
    - For [Mac using M-series chip](https://support.apple.com/en-gb/116943), run the following instead:
        ```
        conda create -n buildtree --platform osx-64 "snippy>=4.6.0" "snp-sites>=2.5.1" "fasttree>=2.2.0"
        ```
3. Install `gubbins` in a Conda environment named `gubbins` 
    ```
    conda create -n gubbins "gubbins>=3.4.3"
    ```
## Quality Control & Generating *in silico* Data


## Building Phylogenetic Tree


## Creating Microreact Instance

