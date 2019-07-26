Protein domain evolution analysis pipeline
===

Protein domains are independent sections of protein sequences that can have function distint functions. One of the major ways in which proteins can evovle is through domain insertion/delection/duplication. In this project we will attempt to build an analysis pipeline that will take in 2 groups of species proteomes and find differences in domain compositions between the two groups. The whole pipeline will be packaged inside a docker container which can executed  on any given data in any machine environment.

## Inputs
* Proteome fasta file for each species. The number of fasta files will depend of which two groups of species the user decides to compare and how many species the user the user wants in each group. It is recommended that there should be atleast 10 species per group to obtain significant results. These can be collected from following databases.
    * [Phytozome](https://phytozome.jgi.doe.gov/pz/portal.html) for plant species
    * [Uniprot](https://www.uniprot.org/proteomes/) for any species

* The HMM file containing Pfam domains from  [Pfam](https://pfam.xfam.org/) database which contains registry of all the domains found in all the organisms. The HMM file must be processed using hmmpress program to create a HMM database. For more details on how to use the hmmpress tool please see the HMMER [user manual](http://hmmer.org/documentation.html).

* A file with two columns `species` and `species_label`. The `species` column contains fasta file names of individual species append by string "pfamscan". The `species_label` column contains labels (0 or 1) classifying the species in different groups.

## Group members
- Akshay Yadav
- Sumegha Godara
- Yafang Guo

# Instructions

## 1. Goals

a). Construct a container with all the programs and dependencies required for the pipeline to run. The analysis pipeline is composed of 3 major steps viz. assigning domains to sequences in fasta, calculating domain matrices, and statistical analysis of domain matrices.
 
b). Implementation of the analysis pipeline on snakemake worfkflow engine.

c). Testing the reproducibility of the pipeline.

## 2. How to start

 2.1 Launch an instance on Jetstream, using Ubuntu 18.04 Devel and Docker v1.22, with m1.xlarge (CPU: 24, Mem: 60 GB, Disk: 60 GB)
   ssh to the VM using
   ```
   # get the username and IP address
   ssh $USER@xxx.xxx.xxx.xxx
   ```
2.2 Download this github repository with scripts (python scripts, snakefile, .sh) along with the dockerfile.
   ```
   git clone https://github.com/cyber-carpentry/Group5-protein-domain-evolution-project.git
   ```
2.3 Download the dataset (fasta, pfam, species.label; test data available) 

   ```https://drive.google.com/file/d/1yr3_NfQ6lpcGGN1tzJnb8RIBaEdAaLmK/view?usp=sharing```

   copy into the project directory.
   $USER is the username showed in echo $USER in VM. 
   ```
   scp <download dir>/test_data.zip $USER@xxx.xxx.xxx.xxx:/home/$USER/Group5-protein-domain-evolution-project/
   ```
   then upzip it using 
   ```
   unzip test_data.zip
```
**2.4 *For reproducibility test, go to Section 3.4 directly.* **

## 3. Build a Docker container
### 3.1 Starting from Dockerfile (explanation)
- Install make, perl #v5.22.1, hmmer, pfamscan
   ```bash
   FROM ubuntu:16.04
   RUN apt-get update && \
       apt-get install -y wget build-essential make perl hmmer && \
       cd /root/ && \
       wget "http://ftp.ebi.ac.uk/pub/databases/Pfam/Tools/OldPfamScan/PfamScan1.5/PfamScan.tar.gz"
   ```
- Install python3 and libs to run the scripts
   ```
   RUN apt-get update \
     && apt-get install -y python3-pip python3-dev  #Version:Python 3.5.2
   RUN pip3 install pandas #Version:0.24.2
   RUN pip3 install rpy2   #Version:3.0.5
   RUN pip3 install scipy  #Version:1.3.0
   RUN pip3 install sklearn  #Version:0.21.2
   RUN pip3 install matplotlib #Version:3.0.3
   ```
- Install snakemake as the workflow management system
   ```
   RUN pip3 install snakemake #Version:5.5.4
   ```
- Add scripts for data analysis inside the container
   ```
   ADD scripts /usr/local/bin
   # make the scripts executable
   RUN chmod +x /usr/local/bin/* 
   ```
### 3.2 Making the snakemake workflow file (explanation)

3.2.1. The analysis includes three steps. 
- assigning pfam protein domains to species fasta
- calculating the domain matrices (content, duplication, abundance, versatility) from domain assignments
- analyzing domain matrices for filtering out significantly evolving domains

3.2.2. The workflow diagram generated by
   ```
   snakemake --dag -np -s snakefile |dot -Tsvg > protein-domain-evolution-workflow.svg
   ```
   ![](https://i.imgur.com/8grxKHe.png)

3.2.3. The snakefile 
- 1st step, run pfamscan.pl 
   ```
   rule pfamscan:
   ```
- 2nd step, run python scripts to get matrices
   ```
   rule domain_content_matrix:
   rule domain_duplication_matrix:
   rule domain_abundance_matrix:
   rule domain_versatility_matrix:
   ```
- 3rd step, run python scripts to analyze the matrices

   ```
   rule domain_content_matrix_analysis:
   rule domain_duplication_matrix_analysis:
   rule domain_abundance_matrix_analysis:
   rule domain_versatility_matrix_analysis:
   ```
### 3.3 Build the container using Docker (hands on)

   Once the dockerfile and snakefile are ready, build the docker imager from the project directory and not the `Docker` directory.  as:

   ```bash
   docker build -t domainevolution -f Docker/Dockerfile .
   ```
   Use ```docker images``` to check the built images

### 3.4 Create and run a writeable container layer over the built image (hands on)
   - Pull the docker image from https://hub.docker.com/r/akshayayadav/protein-domain-evolution-project . 

   - Since the data directory is not built into the container, you need to bind mount a volume with the data directory into the container. 

   ```
   docker run -v /home/$USER/Group5-protein-domain-evolution-project/test_data:/data akshayayadav/protein-domain-evolution-project run_analysis.sh -c 24
   ```
   The number "24" gives the number of cores passed to snakemake to run the analysis.


