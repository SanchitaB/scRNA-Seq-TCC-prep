# single-cell RNA-Seq TCC prep

This repository contains scripts needed to generate transcript compatibility matrices from single-cell RNA-Seq data. Included is error-correction of barcodes, collapsing of UMIs and pseudoalignment of reads to a transcriptome to obtain transcript compatibility counts. The scripts utilize [kallisto](http://pachterlab.github.io/kallisto) for pseudoalignment.

We currently support the 10X Chromium technology; support for more technologies is underway.

## Instructions for processing 10X Chromium 3' digital expression data

### Getting started


The [getting started](http://pachterlab.github.io/kallisto/10xstarting.html) tutorial explains how to process the small example in the [example_dataset](https://github.com/lakigigar/scRNA-Seq-TCC-prep/tree/master/example_dataset) directory. This is a good starting point to make sure that the necessary programs are correctly installed. Note that you will need __kallisto__ version 0.43.0 or later, python, and Juypter Notebook installed (the Jupyter requirement is not strictly necessary but highly recommended).

### Workflow organization

The processing workflow consists of four steps: 

0. Preparation of a configuration file that contains the parameters needed for the processing.
1. Identification of "true" cell barcodes according to read coverage followed by error correction when possible.
2. Creation of read/UMI files for each cell.
3. Pseudoalignment of reads associated with each cell using __kallisto__, deduplication according to UMIs, and generation of transcript compatibility counts (TCCs) for each cell.

Following the pre-processing, the transcript compatibility counts (TCC) matrix can be analyzed using a Jupyter Notebook. 

## Creation of the configuration file

PArameters needed to run the processing require specification of a `config.json` file. The following parameters need to be specified:

- NUM_THREADS: the number of threads available for processing.
- WINDOW: this parameter contains a lower and upper threshold for the expected number of cells in the experiment. It is used in the determination of the number o cells in the experiment from reads coverage data.
- BASE_DIR: this must contain the path to the (demultiplexed) FASTQ files from the sequencing. Note that our workflow does not currently demultiplex reads and you may have to do so with 10X's software; we plan to provide a demultiplexing script in the future.
- barcode_filenames: the names of the (gzipped) barcode FASTQ files.
- read_filenames: the names of the (gzipped) read FASTQ files.
- SAVE_DIR: path to a directory where intermediate files will be saved.
- dmin: the minimum distance between barcodes needed for error correction to be performed.
- BARCODE_LENGTH: length of the barcodes.
- OUTPUT_DIR: directory in which to output results.
- kallisto: path to the binary for __kallisto__, location of the __kallisto__ index file for the appropriate transcriptome and path where to save the TCC matrix.

### Barcode analysis and selection

The first step in the workflow is to identify "true" barcodes, and to error correct barcodes that are close to true barcodes, yet associated with sufficiently low read coverage to be confidently identified as containing an error. The script `get_cell_barcodes.py` in the [source](https://github.com/lakigigar/scRNA-Seq-TCC-prep/tree/master/source) directory performs the identification and error correction and is called with `python get_cell_barcodes.py config.json`.

While `get_cell_barcodes.py` can be run from the command line, we strongly encourage users to instead perform this step using the Jupyter Notebook `get_cell_barcodes.ipynb` in the [notebooks](https://github.com/lakigigar/scRNA-Seq-TCC-prep/tree/master/notebooks) directory. The interactive notebook produces summary statistics and figures that are useful for both quality control and for the setting of parameters for error correction. 

### Cell file generation

Once barcodes have been identified and (some) erroneous barcodes corrected, the next step is to generate individual read and UMI files for each cell for processing by __kallisto__. This can be performed with the command `python error_correct_and_split.py config.json`. 

### Pseudoalignment

The computation of transcript compatibility counts is performed using __kallisto__ with by running `python compute_TCCs.py config.json` followed by `python prep_TCC_matrix.py config.json`. The first script runns __kallisto__ and the second step computes a pairwise distance matrix between cells that is essential for analysis.
 
### Analysis
