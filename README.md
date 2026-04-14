# FASTQ to Microreact for *Streptococcus pneumoniae* <!-- omit in toc -->
This tutorial is specifically tailored for *Streptococcus pneumoniae* and provides a step-by-step workflow for transforming raw read sequencing files (FASTQ) into an interactive Microreact visualisation. The tutorial begins with software environment setup. It then covers quality control and the generation of *in silico* data (such as GPSCs, serotypes, MLSTs, and AMR profiles) using the GPS Pipeline, followed by phylogenetic tree construction, and Microreact instance creation.


## Table of Content <!-- omit in toc -->
- [Prerequisites \& Environment Setup](#prerequisites--environment-setup)
  - [Required Tools](#required-tools)
  - [Installing Tools](#installing-tools)
    - [GPS Pipeline](#gps-pipeline)
    - [Tree Building Tools  (`snippy`, `snp-sites`, `fasttree`, `ete3`, `gubbins`)](#tree-building-tools--snippy-snp-sites-fasttree-ete3-gubbins)
- [Quality Control \& Generating *in silico* Data](#quality-control--generating-in-silico-data)
  - [Run GPS Pipeline](#run-gps-pipeline)
  - [Filtering samples](#filtering-samples)
- [Building Phylogenetic Tree](#building-phylogenetic-tree)
  - [Prepare `snippy-multi` input files](#prepare-snippy-multi-input-files)
  - [Run `snippy-multi`](#run-snippy-multi)
  - [Crossroads: Choose Your Path](#crossroads-choose-your-path)
  - [Path A - `snp-sites` → `fasttree` → `ete3`](#path-a---snp-sites--fasttree--ete3)
  - [Path B - `gubbins` → `ete3`](#path-b---gubbins--ete3)
- [Creating Microreact Instance](#creating-microreact-instance)

## Prerequisites & Environment Setup
You can use Linux, Windows ([with WSL2](https://learn.microsoft.com/en-us/windows/wsl/install)), and macOS to follow this tutorial. We recommend using a system with at least 16GB of RAM. 

### Required Tools
- [GPS Pipeline](https://github.com/GlobalPneumoSeq/gps-pipeline)
- [Snippy](https://github.com/tseemann/snippy)
- [SNP-sites](https://github.com/sanger-pathogens/snp-sites)
- [FastTree](https://github.com/morgannprice/fasttree)
- [ETE Toolkit](https://github.com/etetoolkit/ete)
- [Gubbins](https://github.com/nickjcroucher/gubbins)

### Installing Tools
#### GPS Pipeline
1. Install dependencies
   1. Install OpenJDK 17 (or later, up to 25) (use [`SDKMAN!`](https://sdkman.io/install/) to install an appropiate version of [Temurin distribution](https://sdkman.io/jdks/tem/))
   2. Install Docker or Singularity/Apptainer
       - Linux: [Docker Engine](https://docs.docker.com/engine/install/) / [Apptainer](https://apptainer.org/docs/admin/main/installation.html)
       - macOS: [Docker Desktop for macOS](https://docs.docker.com/desktop/setup/install/mac-install/)
       - Windows with WSL2: [Docker Desktop for WSL2](https://docs.docker.com/desktop/features/wsl/)
2. Clone and initialise the pipeline
    > `git` should come preinstalled on most systems. If not, follow the [official installation guide](https://github.com/git-guides/install-git).
   1. Go into where you want to keep the pipeline
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

#### Tree Building Tools  (`snippy`, `snp-sites`, `fasttree`, `ete3`, `gubbins`)
1. [Install Conda](https://conda-forge.org/download/)
2. Install `snippy`, `snp-sites`, `fasttree`, `ete3` in a Conda environment named `buildtree`
    > If using [Mac computers with Apple Silicon](https://support.apple.com/en-gb/116943), append `--platform osx-64` to the below command.
    ```
    conda create -n buildtree "snippy>=4.6.0" "snp-sites>=2.5.1" "fasttree>=2.2.0" "ete3>=3.1.3"
    ```
3. Install `gubbins` in a Conda environment named `gubbins`
   > `gubbins` need to be installed to a separated Conda environment due to package conflict with other tools.
    ```
    conda create -n gubbins "gubbins>=3.4.3"
    ```
## Quality Control & Generating *in silico* Data
### Run GPS Pipeline
We use the GPS Pipeline to process raw read sequencing files (FASTQ) of *Streptococcus pneumoniae* samples, it will perform quality control (QC) and *in silico* typing (including serotype, MLST, GPSC, and AMR profile) fully automatic.
1. Saving FASTQ files of all samples in a single directory
2. Run the pipeline by specifying where the FASTQ files are stored (`--reads`) and the output directory (`--output`)
   > If using Singularity/Apptainer, add `-profile singularity` to the below command.
   ```
    ./run_pipeline --reads /path/to/fastqs --output /path/to/output
   ```
3. Once the run is completed, you will find `results.csv` in your specified output directory

### Filtering samples
Before building a tree, we will filter out samples that failed QC in the GPS Pipeline. We will be saving a copy of `results.csv` with only QC passed samples as `results_qcpass.csv` and a list of those samples as `ids_qcpass.txt`
1. Go into GPS Pipeline output directory
   ```
   cd /path/to/output
   ``` 
2. Extract GPS Pipeline results of QC-passed samples, **`results_qcpass.csv` will be uploaded to your Microreact instance as metadata**
    ```
    awk -F , ' FNR==1 || $6=="PASS" ' results.csv > results_qcpass.csv
    ```
3. Extract names of QC passed samples
    ```
    awk -F , ' FNR >1 { print $1 } ' results_qcpass.csv > ids_qcpass.txt
    ```

## Building Phylogenetic Tree
We use `snippy-multi` script to run QC passed reads against the same reference.
### Prepare `snippy-multi` input files
1. Run the following command to create an input file named `snippy_multi_input.tsv`
   > Update values of `READS_DIR` and `IDS_QCPASS` to the correct ones.
   ```
    READS_DIR="/path/to/fastqs"
    IDS_QCPASS="/path/to/ids_qcpass.txt"

    while read -r ID; do 
        echo ${ID}$'\t'${READS_DIR}/${ID}_1.fastq.gz$'\t'${READS_DIR}/${ID}_2.fastq.gz >> snippy_multi_input.tsv;
    done < ${IDS_QCPASS}
   ```
2. Get a reference sequence
    > Below command downloads a general reference sequence. You should use [a suitable reference genome](https://github.com/GlobalPneumoSeq/gpsc-reference-genomes/tree/main/fasta) if you are working on a lineage/GPSC-specific analysis.
    ```
    wget https://raw.githubusercontent.com/GlobalPneumoSeq/gps-pipeline/refs/heads/master/data/ATCC_700669_v1.fa
    ```

### Run `snippy-multi`
1. Activate `buildtree` Conda environment
   ```
   conda activate buildtree
   ```
2. Run snippy-multi to generate the script `runme.sh`, which will create the alignment in the next step
   > `--cpus $(nproc)` uses all available CPU cores when running the generated script.
   
   > Update the value of `--ref` if you are using an alternative reference sequence.
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
   cut -f1 snippy_multi_input.tsv | xargs rm -rf
   ```

### Crossroads: Choose Your Path
Depending on the purpose of your tree building and the nature of your samples, you might choose one of the following paths:
- Path A: 
  - Species-wide analysis (as `gubbins` only works on [samples within a strain or lineage](https://github.com/nickjcroucher/gubbins/blob/master/docs/gubbins_manual.md#introduction)), or quick preliminary test
  - Faster
- Path B:
  - Strain/lineage-specific analysis
  - Slower

### Path A - `snp-sites` → `fasttree` → `ete3`
1. Run `snp-sites` to identify and extract the SNPs from the alignment
   ```
   snp-sites -o clean.full.SNPs.aln clean.full.aln
   ```
2. Run `fasttree` to build a tree
   ```
   FastTree -nt -gtr clean.full.SNPs.aln > clean.full.SNPs.aln.tree
   ```
3. Run `ete3` via Python to prune reference from tree file
   ```
   echo -e "from ete3 import Tree\nt = Tree('clean.full.SNPs.aln.tree')\nt.search_nodes(name='Reference')[0].detach()\nt.write(outfile='clean.full.SNPs.aln.noref.tree')" | python3
   ```
4. **The resulting `clean.full.SNPs.aln.noref.tree` will be uploaded to your Microreact instance as tree file**

### Path B - `gubbins` → `ete3`
1. Activate `gubbins` Conda environment
   ```
   conda activate gubbins
   ```
2. Run `gubbins` to build a tree with recombination filtering
   > `--threads $(nproc)` uses all available CPU cores when running `gubbins`
   ```
   run_gubbins.py --mar --threads $(nproc) -p gubbins_output clean.full.aln
   ```
3. Activate `buildtree` Conda environment
   ```
   conda activate buildtree
   ```
4. Run `ete3` via Python to prune reference from tree file
   ```
   echo -e "from ete3 import Tree\nt = Tree('gubbins_output.final_tree.tre')\nt.search_nodes(name='Reference')[0].detach()\nt.write(outfile='gubbins_output.final_tree.noref.tre')" | python3
5. **The resulting `gubbins_output.final_tree.noref.tre` will be uploaded to your Microreact instance as tree file**

## Creating Microreact Instance

You can refer to [its official documentation](https://docs.microreact.org/) on how to use Microreact. In brief, you can create an informative instance in just few simple steps: 
1. Go to https://microreact.org/upload using your preferred web browser
2. Drag and drop `results_qcpass.csv`, and `clean.full.SNPs.aln.noref.tree` (Path A) or `gubbins_output.final_tree.noref.tre` (Path B)
3. Keep pressing `Continue` buttons until you see the visulisation 
4. [Pick a suitable colour column](https://docs.microreact.org/instructions/labels-colours-and-shapes#colours) to change what the colours of the nodes are representing (e.g. Serotype)
   > In the GPS Project, we have standardised colour palette for [GPSCs](https://github.com/sanger-bentley-group/gps-database-processor/blob/main/scripts/data/gpsc_colours.csv) and [serotypes](https://github.com/sanger-bentley-group/gps-database-processor/blob/main/scripts/data/serotype_colours.csv). You can add a `<FIELD NAME>__colour` column (e.g. `GPSC__colour`) in your metadata (`results_qcpass.csv` in this case) to specify the colour palette of a column. 
5. [Add metadata blocks](https://docs.microreact.org/instructions/adding-and-editing-panels/tree-panel#metadata-blocks) to visualise the *in silico* data generated by the GPS Pipeline to provide context to the tree
6. [Save and share the project](https://docs.microreact.org/instructions/saving-and-sharing-projects)
