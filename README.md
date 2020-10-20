



# NanoMAP

## Quickstart
Please read about building databases prior to use. 
```sh
# Obtain NanoMAP
git clone https://github.com/GraceAHall/NanoMAP
cd NanoMAP

# Build database
python build_database.py -d database

# Run
python nanomap.py -r fastq/fasta -d database -p projectname
```



## Table of Contents
- [Requirements](#requirements)
- [Overview](#overview)
- [Usage](#usage)
- [Databases](#databases)
    - [Database Requirements](#db_requirements)
    - [Database Building](#db_building)
    - [Removing Redundancies](#redundancies)
- [Output](#output)

<br>

## <a name="requirements"></a>Requirements
NanoMAP uses read alignment for sample characterisation. <br>
Python pulls the strings, while minimap2 does most of the actual work. The following are required. 
- minimap2
- python 3.6 or greater
- python packages:
    - numpy
 



<br>

## <a name="overview"></a>Overview
NanoMAP is an **experimental** tool for strain-level sample characterisation using long reads (Oxford Nanopore/PacBio). <br>
It uses alignment MAPQ scores to identify sample organisms. 

Once a sample has been sequenced, NanoMAP uses this sequence data and a database of reference genomes to identify the organisms present in the sample. Abudance estimates of the identified organisms are given. 

Like all characterisation tools, NanoMAP requires reads, and a reference database.  

Due to its method, **redundant copies** of a reference genome in the database will degrade performance. <br>
Redundant genomes can be easily removed **by the user** if encountered during runtime. See [Removing Redundancies](#redundancies) <br>
**Poor-quality reference genomes** will similarly degrade performance.  

You can read more in the [NanoMAP paper](https://www.biorxiv.org/content/10.1101/2020.10.18.344739v1)<br>



<br>

## <a name="usage"></a>Usage
After a database has been built, NanoMAP can be run with the following:
```sh
# general use
python nanomap.py -r fastq/fasta -d database -p projectname

# multithreading 
python nanomap.py -r fastq -d database -p projectname -t 10          

# specify read technology
python nanomap.py -r fastq -d database -p projectname --map-ont / --map-pb        
```

Where:
* **-r** specifies the read set (FASTA or FASTQ)
* **-d** specifies the built reference database
* **-p** specifies the project name for this analysis.

A new project folder  - 'projects/projectname' - is created for each sample to store runtime and config files. 


<br>

## <a name="databases"></a>Databases
A NanoMAP database is simply a folder containing reference genomes. 

1. Place your reference genome FASTA files into a folder
2. Build the database with the following:
    ```sh
    python build_database.py -d database    # 'database' = your genome folder path.
    ```
The build process will extract some information from FASTA headers, concatenate files into a metagenome, then create a minimap2 index for this metagenome.  

<br> 

### <a name="db_requirements"></a>Database Requirements
NanoMAP allows flexibility with databases. Any genome assembly can be used, with the following **database conditions:**

- FASTA headers contain strain name
- Each genome is a **seperate file**
- Genomes are good quality

The FASTA headers should be human readable as these appear in the program output. As an example:
```sh
>NC_004337.2 Shigella flexneri 2a str. 301 chromosome, complete genome
```
Will appear as 'Shigella flexneri 2a str. 301' in the NanoMAP output. The header text before the first space (RefSeq accession in this case) is ignored. 

Genomes must be seperate files, as filenames are used to uniquely identify each reference genome in the folder. 

Poor quality reference genomes should be avoided as can degrade performance.  

<br> 

### <a name="db_building"></a>Database Building

To create a database, make a folder and populate it with FASTA files. Each reference genome needs to be **its own file**. <br>A good way to start is to download  a batch of complete genome assemblies from [NCBI RefSeq](https://www.ncbi.nlm.nih.gov/assembly/?term=all%5Bfilter%5D). This approach was used during development. 

The database needs to then be **built before use** using the following:
```sh
# general use
python build_database.py -d database

# specify read technology
python build_database.py -d database --map-ont (nanopore) --map-pb (PacBio)

# rebuild database after adding/removing genomes
python build_database.py -d database --rebuild

# build taxonomy information only
python build_database.py -d database --taxonomy-only

# build minimap index only
python build_database.py -d database --index-only
```

<br>

### <a name="redundancies"></a>Removing Redundancies
Large, publically available databases often have redundancies. 

Your database folder is allowed to contain these redundancies. This said, some redundant genomes may need to be banned during analysis. This is performed manually by the user. 

To ban a genome:
1. Navigate to the project folder (projects/yourprojectname)
2. Locate banlist.txt
3. Add the genome's filename on a new line in banlist.txt

After adding the genome to banlist.txt, re-run the analysis:
```sh
python nanomap.py -r fastq/fasta -d database -p projectname --no-initial-alignment
```
**--no-initial-alignment** will save time by skipping the initial alignment step and should not influence performance. 

This process may be automated in future versions of NanoMAP.

<br>

## <a name="output"></a>Output
NanoMAP provides two output files: 

* A brief report
* A detailed report

These appear as: **projectname_brief.tsv**, and **projectname_detailed.tsv**

<br>

**Brief report**

The brief report is the intended output of NanoMAP. <br>
It is a tab delimited file containing the following information for identified strains:
* strain name | filename | sample DNA abundance

<br>

**Detailed report**

NanoMAP is still under development. <br>
Incorrect results may sometimes occur, due to low quality reads, low quality reference genomes, and database redundancies. <br>It is a good idea to inspect **projectname_detailed.tsv**, or the runtime console output to catch possible errors. 

During runtime, NanoMAP creates a shortlist of candidate strains (strain group) for each true sample strain. <br>
From this shortlist, the true sample strain is identified. <br>

The detailed report captures a snapshot of the information NanoMAP used when identifying strains. <br>
For each shortlist, the following information is recorded:
* Naive sample DNA abundance
* MAPQ=60 read count
* MAPQ=10 read count
* MAPQ=3 read count

Naive sample DNA abundance is a very rough estimate of abundance. 

The MAPQ read counts record the number of reads which map uniquely to that strain's reference genome.<br> MAPQ scores are the basis for identifying strains, and will have informed NanoMAP's decisions.  <br>
In our experience, a human can often interpret the console information and **projectname_detailed.tsv** better than NanoMAP.  






