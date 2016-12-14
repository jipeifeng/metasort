metaSort 1.2 manual

Description: 
metaSort is a novel computational tool, which includes three modules:Build, FAB and MGA.
These modules are designed to assemble genomes based on the combination of sorted mini-metagenome (meta-S) and the original metagenome (meta-O).
BUILD is designed to build database based on the meta-O data and provides informations for FAB and MGA.
FAB is designed to generate merged genome bins from meta-S and their connected contigs from meta-O.
The resulting bins are employed by MGA to recover target genome sequences from the meta-O contig connection graph.
Furthermore, MGA assembles the recovered target genome sequences into longer scaffolds and finds variation.

Contents

1.  Installation
2.  Preparation for running metaSort
        a) Assemble the meta-O reads
        b) Assemble the meta-S reads
3.  Running metaSort
	a) The build module
        b) The fab module
        c) The mga module
4.  metaSort output
5.  Citation

1. Installation
metaSort can be run on Linux or Mac OS

MetaSort utilizes MUMer, MetaGeneMark, bwa and libSVM. These tools are built in, so you do not need to install them separately.

It requires:
    Python 2 (2.7 or higher)
    Sklearn (http://scikit-learn.org/stable/)
    Make
    samtools

NOTE: Sklearn can be installed using pip: sudo pip install sklearn

All those tools except the Sklearn package of Python are usually preinstalled.
Mac OS, however, initially misses make, g++ and ar, so you will have to install Xcode (or only Command Line Tools for Xcode) to make them available. 

It is also highly recommended to install the CheckM package (https://github.com/Ecogenomics/CheckM/wiki) for estimating the completeness of the assemblies.

To download the MetaSort source code and extract it, type:
    wget 
    tar -xzf metaSort-1.2.tar.gz
    cd  metaSort-1.2

metaSort automatically compiles all its sub-parts when needed (on the first use). Thus there is no special installation command for metaSort. 

2 Preparation for running metaSort

    a) meta-O assembly is performed using metagenome assemblers, such as metaSPAdes, IDBA-UD
    
    b) meta-S assembly is performed using SPAdes with the recommended parameters: 
            --sc, --careful
     
    IMPORTANT NOTE: user should use a kmersize larger than 40 to assemble the paired-end reads into scaffolds, or the 'build' will raise errors !
	
3 Running metaSort

    There are three modules in the metaSort package: build, fab and mga

3.1 The build module

This script is to perpare database for running fab and mga, it will take a while. Once the database is generated, they are can be repetitively used for every meta-S projects.

The build module runs from a command line as follows:
    metaSort build [options]	

Options:
    -b, --metaO 
          The metaO scaffolds file.
    -d, --database
		  Directory path for storing database of metaO
    -l, --libs
		  The config file used for metaO assembly.
		  The template file was provided in the same directory with metaSort file and named as "lib_config.tmplate".
    -t, --cpu
          The cpu number used for running fab

3.2 The fab module

This script is performed to prepare files for mga module. 

The fab module runs from a command line as follows:
    metaSort fab [options]	

Options:
    -d, --database
		  Directory path for storing database of metaO
    -s, --metaS 
          The meta-S scaffolds (FASTA formate)
    -p, --project
          The directory path for storing the meta-S analysis data
    -t, --cpu
          The cpu number used for running fab

3.3 The mga algorithm

This script is to recover contig of target genome based on the sequences in the each bin produced by fab and assembled them into scaffolds. Moreover, mga de novo detects variations during the assembly process.

The mga module runs from a command line as follows:
    metaSort mga [options]

Options:
    -d, --database 
          Directory path for storing database of metaO
    -p, --project
          The directory path for storing the meta-S analysis data
    -g, --genome
          The bin file produced by fab that used for recover contigs of target genome.
    -r, --recover 
          Turn on or off the recovery step of mga.
          If the target genome in a bin reaches a high level of completeness,
          then this parameter can be set to 0. Otherwise, set to 1.
    -o, --output
          Output directory for storing the results
    -t, --cpu
          The cpu number used for running mga

Advance options cloud be set in the config.py file

4. metaSort output

The output files are divided into three parts: build, fab and mga.

4.1 Output files of fab
     
    Log file:              build.log (log file )
    Database directory     The database files used for fab and mga.

4.2 Output files of fab
     
    Log file:              fab.log (log file )
    Project directory      The bins of which each includes
                           partial sequences of the target genome.

4.3 Output files of mga

    log file:              mga.log (log file )
    scaffold file:         targetGenome.fa (Assembled scaffolds of target genome)
    contig file:           recover.fa (Recovered contigs of target genome)
    statistic file:        scafStatistics (Metrics of assembly)

5. Test metaSort

The test data need to be downloaded independently. 
MetaSort can be tested using the test data (test.tar.gz), which can be accessed at the same folder of the metaSort software.
Note that:This is a test file which is only used for testing if metaSort works.
# Do not delete the metao and metas directory

cmds:
# make sure that the metaSort program is in your system path (like: /usr/bin), or provide the path of executable file 

# generate the metas directory using fab module
$ metaSort fab -d metao -s mda.fa -p metas -t 3

But please note that bias occurs when using CheckM screening outputted bins of FAB algorithm,
becasue these bins are mixture of meta-O and meta-S sequences (same marker gene repeats multiple times),
where it will cause a high reported 'contamination rate'.

# assemble the target genome, for example: m1.fa in the clSeqs directory using mga module
$ cp clSeqs/m1.fa .
$ metaSort mga -d metao -p metas -g m1.fa -o M1_genomoe -t 3
