# Yet Another Metagenomic Pipeline (YAMP)

![Publish Docker image](https://github.com/chan-csu/YAMP/workflows/Publish%20Docker%20image/badge.svg)

Thanks to the increased cost-effectiveness of high-throughput technologies, the number of studies focusing on microorganisms (bacteria, archaea, microbial eukaryotes, fungi, and viruses) and on their connections with human health and diseases has surged, and, consequently, a plethora of approaches and software has been made available for their study, making it difficult to select the best methods and tools. 

Here we present Yet Another Metagenomic Pipeline (YAMP) that, starting from the raw sequencing data and having a strong focus on quality control, allows, within hours, the data processing up to the functional annotation (please refer to the YAMP [wiki](https://github.com/alesssia/YAMP/wiki) for more information).

YAMP is constructed on [Nextflow](https://github.com/nextflow-io/nextflow), a framework based on the dataflow programming model, which allows writing workflows that are highly parallel, easily portable (including on distributed systems), and very flexible and customisable, characteristics which have been inherited by YAMP. New modules can be added easily and the existing ones can be customised -- even though we have already provided default parameters deriving from our own experience.

YAMP is accompanied by a [Docker container](https://www.docker.com/), that saves the users from the hassle of installing the required software, increasing, at the same time, the reproducibility of the YAMP results (see [Using Docker or Singularity](#using-docker-or-singularity)). 



## Table of contents

- [Citation](#Citation)
- [Dependencies](#dependencies)
- [Installation](#installation)
- [Other requirements](#other-requirements)
- [Usage](#usage)
- [Using Docker or Singularity](#using-docker-or-singularity)
- [Troubleshooting](#Troubleshooting)
- [Changelog](#changelog)
- [License](#license)
- [Acknowledgements](#acknowledgements)


## Citation

Please cite YAMP as:

> Visconti A,. Martin T.C., and Falchi M., *"YAMP: a containerised workflow enabling reproducibility in metagenomics research"*, GigaScience (2018), [https://doi.org/10.1093/gigascience/giy072](https://doi.org/10.1093/gigascience/giy072)


## Dependencies

To run YAMP you will need to install Nextflow  (version 0.29.x or higher), as explained [here](https://www.nextflow.io/docs/latest/getstarted.html). Please note that Nextflow requires BASH and [Java 7](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or higher to be installed. Both should be already available in most of the POSIX compatible systems (Linux, Solaris, OS X, etc). However, as of October 2017, the latest release of Java (SE9) introduces some breaking changes in Nextflow, and should not be used (see [here](https://github.com/nextflow-io/nextflow/issues/462) for details). 

If you are using the containerised version of YAMP (as we strongly suggest), you will should also install [Docker](https://www.docker.com) or [Singularity](http://singularity.lbl.gov/), as explained [here](https://docs.docker.com/engine/installation/) and [here](http://singularity.lbl.gov/docs-installation), respectively.
In fact, Nextflow orchestrates, in a transparent fashion, the flow of the pipeline by wrapping and executing each step using the Docker/Singularity run command. Thus, Nextflow lies *outside* the container, that is responsible for instantiating. 
You can find more information about Docker/Singularity containers and Nextflow [here](https://www.nextflow.io/docs/latest/docker.html) and [here](https://www.nextflow.io/docs/latest/singularity.html), respectively.

Once you have either Docker or Singularity up and running, you will not need to install anything additional tools, since all the pieces of software are already available in the Docker container released with YAMP pipeline, and that you can find on [DockerHub](https://hub.docker.com/r/alesssia/yampdocker/). Please refer to [Using Docker or Singularity](#using-docker-or-singularity) for more details. 

**For expert users only.** 
If you do not want to use the containerised version of YAMP, you will need to install several additional tools for YAMP to work properly, and all of them should either be in the system path with execute and read permission, or made available within a multi-image scenario as the one we describe in the [multi-image scenario tutorial](https://github.com/alesssia/YAMP/wiki/multi-image-scenario).

The list of tools that should be available includes:
- fastQC v0.11.2+ ([http://www.bioinformatics.babraham.ac.uk/projects/fastqc](http://www.bioinformatics.babraham.ac.uk/projects/fastqc))
- BBmap v36.92+ ([https://sourceforge.net/projects/bbmap](http://www.bioinformatics.babraham.ac.uk/projects/fastqc))
- Samtools v1.3.1 ([http://samtools.sourceforge.net](http://samtools.sourceforge.net))
- MetaPhlAn2 v2.0+ ([https://bitbucket.org/biobakery/metaphlan2](https://bitbucket.org/biobakery/metaphlan2))
- QIIME v1.9.1+ ([http://qiime.org](http://qiime.org))
- HUMAnN2 v0.9.9+ ([https://bitbucket.org/biobakery/humann2](https://bitbucket.org/biobakery/humann2))

Following the links, you will find detailed instructions on how to install them, as explained by their developers. 
Notably, MetaPhlAn2, QIIME, and HUMAnN2 are also available in [bioconda](https://anaconda.org/bioconda/). 


## Installation

Clone the YAMP repository in a directory of your choice:

```
git clone https://github.com/alesssia/YAMP.git
```

The repository includes:

- the Nextflow script, `YAMP.nf`, 
- the configuration files, `nextflow.config`
- a folder (`bin`) containing two helper scripts (`fastQC.sh` and `logQC.sh`),
- a folder (`yampdocker`) containing the Docker file used to build the Docker image (`Dockerfile`). 

**Note:** the `nextflow.config` file includes the parameters that are used in our tutorials (check the YAMP [wiki](https://github.com/alesssia/YAMP/wiki)!).


## Other requirements

YAMP requires a set of databases that are queried during its execution. Some of them should be automatically downloaded when installing the tools listed in the dependencies (or using specialised scripts, as those available with HUMAnN2), whilst other should be created by the user. Specifically, you will need:

- a FASTA file listing the adapter sequences to remove in the trimming step. This file should be available within the BBmap installation. If not, please download it from [here](https://github.com/BioInfoTools/BBMap/blob/master/resources/adapters.fa);
- two FASTA file describing synthetic contaminants. These files (`sequencing_artifacts.fa.gz` and `phix174_ill.ref.fa.gz`) should be available within the BBmap installation. If not, please download them from [here](https://sourceforge.net/projects/bbmap/);
- a FASTA file describing the contaminating genome(s). This file should be created by the users according to the contaminants present in their dataset. When analysing human metagenome, we suggest the users to always include the human genome. Please note that this file should be indexed beforehand. This can be done using BBMap, using the following command: `bbmap.sh -Xmx24G ref=my_contaminants_genomes.fa.gz `. 
	We suggest to download the FASTA file provided by Brian Bushnell for removing human contamination, using the instruction available [here](http://seqanswers.com/forums/showthread.php?t=42552);
- the BowTie2 database file for MetaPhlAn2. This file should be available within the MetaPhlAn2 installation. If not, please download it from [here](https://bitbucket.org/biobakery/metaphlan2/src/40d1bf693089836b5895623dd9ab1b21eb9a794c/db_v20/);
- the ChocoPhlAn and UniRef databases, that can be downloaded directly by HUMAnN2, as explained [here](https://bitbucket.org/biobakery/humann2/wiki/Home#markdown-header-5-download-the-databases);
- [optional] a phylogenetic tree used by QIIME to compute a set of alpha-diversity measures (see [here](http://qiime.org/scripts/alpha_diversity.html) for details).

You can find an example of the folders layouts in this [wiki](https://github.com/alesssia/YAMP/wiki/Folders-layout-example) page.

You can also download all these files (please note that it might be necessary to edit this file list according to the analysis at hand) either from Zenodo ([https://zenodo.org/record/1068229#.Wh7a3rTQqL4](https://zenodo.org/record/1068229#.Wh7a3rTQqL4)), or using the following command:

```
wget https://zenodo.org/record/1068229/files/YAMP_resources_20171128.tar.gz
```

If you use this data file, please note that, before running YAMP, the FASTA file describing the human (contaminating) genome should be indexed with the following command:

```
bbmap.sh -Xmx24G ref=hg19_main_mask_ribo_animal_allplant_allfungus.fa.gz
```

**Please also note that the size of this compressed data file is 16.7 GB.** 



## Usage

1. Modify the `nextflow.config` file, specifying the necessary parameters, such as the path to the aforementioned databases.
2. From a terminal window run the `YAMP.nf` script using the following command (when the library layout is 'paired'):
	```
	nextflow run YAMP.nf --reads1 R1 --reads2 R2 --prefix mysample --outdir outputdir --mode MODE
	```
	where `R1` and `R2` represent the path to the raw data (two compressed paired-end FASTQ files), `mysample` is a prefix that will be used to label all the resulting files, `outputdir` is the directory where the results will be stored, and `MODE` is any of the following: < QC, characterisation, complete >; or  the following command (when the library layout is 'single'):
	```
	nextflow run YAMP.nf --reads1 R --prefix mysample --outdir outputdir --mode MODE --librarylayout single
	```
	where `R` represents the path to the raw data (a compressed single-end FASTQ file), `librarylayout single` specifies that single-end reads are at hand, and the other parameters are as above.
	
Does it seem complicate? In the YAMP [wiki](https://github.com/alesssia/YAMP/wiki) there are some tutorials and a [TL;DR](https://github.com/alesssia/YAMP/wiki/TL%3BDR) if you are in a hurry!


## Using Docker or Singularity

To use the tools made available through the Docker container within both Docker, one could either pull the pre-built image from [DockerHub](https://hub.docker.com/r/alesssia/yampdocker/), using the following command:

```
docker pull alesssia/yampdocker
```

or build a local image using the file `Dokerfile` in  the `yampdocker` folder. To build a local image, one should first access the `yampdocker` folder and then run the following command (be careful to add the dot!):

```
docker build -t yampdocker .
```

In both cases, the image can be used by YAMP by running the command presented above adding `-with-docker` followed by the image name (`yampdocker`):

```
nextflow run YAMP.nf --reads1 R1 --reads2 R2 --prefix mysample --outdir outputdir --mode MODE -with-docker yampdocker
```

where `R1` and `R2` represent the path to the raw data (two compressed FASTQ file), `mysample` is a prefix that will be used to label all the resulting files, `outputdir` is the directory where the results will be stored, and `MODE` is any of the following: < QC, characterisation, complete >.

YAMP can also fetch the Docker container directly from DockerHub;

```
nextflow run YAMP.nf --reads1 R1 --reads2 R2 --prefix mysample --outdir outputdir --mode MODE -with-docker docker://alesssia/yampdocker
```

so, even simpler!

YAMP can use a Docker image with Singularity (again without pulling the image) by adding the `-with-singularity` option followed by the image path (`--with-singularity docker://alesssia/yampdocker`), that is, the following command:

```
nextflow run YAMP.nf --reads1 R1 --reads2 R2 --prefix mysample --outdir outputdir --mode MODE -with-singularity docker://alesssia/yampdocker
```


Please note that Nextflow is not included in the Docker container and should be installed as explained [here](https://www.nextflow.io/docs/latest/getstarted.html).


## Troubleshooting

We have listed all known issues and solutions on this [wiki page](https://github.com/alesssia/YAMP/wiki/Troubleshooting). Please report any issue using the [GitHub platform](https://github.com/alesssia/YAMP/issues).


## Changelog

### 0.9.4.3 / 2019-12-06

Fixes:
* Solved problem with compressed files in 'characterisation` mode
* Fixed warnings (Nextflow v19.10.0)


### 0.9.4.2 / 2018-09-14

Fixes:
* Solved problem in loading data in 'complete` mode

Notes:
* YAMP now requires Nextflow version 0.29.x or higher

### 0.9.4.1 / 2018-04-24

Enhancements:
* QC'd files are now compressed (fq.gz) before being saved when `keepQCtmpfile` is true

Fixes:
* Solved problem in loading data when using single library layout
* Solved problem in loading data in 'characterisation` mode



### 0.9.4 / 2017-12-07

Enhancements:
* Improved logs
* Version and help message printed upon request


### 0.9.3.1 / 2017-10-04

Enhancements:
* Users no longer need to specify the number of threads and the maximum amount of memory -- both values are now read from the `nextflow.config` file


### 0.9.3 / 2017-08-30
 
 Enhancements:
 * YAMP can now handle both paired-end and single-end reads
 * The de-duplication step is now optional and can be skip (default: true)
 

### 0.9.2 / 2017-07-10 

Enhancements:
* YAMP can now be run in three *"modes"* : < QC, characterisation, complete >.


## License

YAMP is licensed under GNU GPL v3.


## Acknowledgements

Alessia would like to thank Brian Bushnell for his helpful suggestions about how to successfully use the BBmap suite in a metagenomics context and for providing several useful resources, and Paolo Di Tommaso, for helping her in using Nextflow properly! Alessia would also like to thank all the users for their valuable feedbacks (and mostly Richard Davies [@richardjdavies](https://github.com/richardjdavies))

