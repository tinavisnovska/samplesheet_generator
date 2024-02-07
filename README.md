# samplesheet_generator
generates samplesheet compatible with TSO500 LocalApp analysis.

## Contents

1. [Dependencies](#dependencies)
2. [Description of Input Parameters](#description-of-input-parameters)
3. [Usage](#usage)

## Introduction

Running LocalApp analysis requires a samplesheet in a specific format consistent with the performed sequencing to guide the analysis. Sometimes the samplesheet generated by a sequencing machine is transferred so that LocalApp has access to the file. However, sometimes it is more efficient to generate such a samplesheet from scratch. This script automatizes the second option.

## Dependencies

python3>=3.11.4, 
pandas>=2.2.0


## Description of Input Parameters

Names of the input parameters defined for the script and their possible values are listed in the table below. Except for the `--input_info_file` all the other parameters are strings or integers that will be incorporated into the output of the script. 


| parameter | description |
|:---|:---|
|`--run-id`| Id of the sequencing run for which the samplesheet is generated. (**str**)|
|`--index-type`| `dual` or `simple` Type of the used index. (**str**)|
|`--index-length`| `8` or `10`. Number of nucleotides in the used index. (**int**) |
|`--investigator-name`| Value of the `Investigator Name` field in the samplesheet. Using `name (inpred_node)` could be a good convention. (**str**)|
|`--experiment-name`| Value of the `Experiment Name` field in the samplesheet. A list of study names from which the samples are could be a good convention. (**str**)|
|`--input-info-file`| Full path to an input file. (**str**)|
|`--read-length-1`| This value will be filled in the `Reads` section of the samplesheet. (**int**) [default = 101]|
|`--read-length-2`| This value will be filled in the `Reads` section of the samplesheet. (**int**) [default = 101]|
|`--adapter-read-1`| This value will be filled in the `Settings` section of the samplesheet, it is a nucleotide adapter sequence of read1. (**str**)|
|`--adapter-read-2`| This value will be filled in the `Settings` section of the samplesheet, it is a nucleotide adapter sequence of read2. (**str**)|
|`--adapter-behavior`| This value will be filled in the `Settings` section of the samplesheet. (**str**) [default = 'trim']|
|`--minimum-trimmed-read-length`| This value will be filled in the `Settings` section of the samplesheet. (**int**) [default = 35]|
|`--mask-short-reads`| This value will be filled in the `Settings` section of the samplesheet. (**int**) [default = 22]|
|`--override-cycles`| This value will be filled in the `Settings` section of the samplesheet. (**str**)|
|`--samplesheet-version`| Version in which the samplesheet should be generated. Only `v1` is implemented now.  (**str**) [default = 'v1']|
|`--help`|Print the help message and exit.|

File format of the `input_info_file` follows.

### Input File Format

The file is expected to contain `tab` separated values (`.tsv` file). The rows starting with character `#` are ignored (considered to be comments). The first non-commented row is considered to be a header containing column names.

Required columns of the file are: `sample_id`, `molecule`, `run_id`, `barcode`, `index` (at least one of columns `barcode` and `index` has to be non-empty).

| column | description |
|:---|:---|
|`sample_id`| - id of a sample.|
|`molecule` | - `DNA` or `RNA`, choose the appropriate one.| 
| `run_id`  | - id of the sequencing run in which the sample was sequenced. |
| `barcode` | - one of the `Index_ID` values from the `indexes/TSO500*_dual_*.tsv` files or one of the `I7_Index_ID` values from the `indexes/TSO500*simple*.tsv` file. The value should correspond to the indexes used for sequencing of the sample. Alternatively, the field can be left empty or containing `NA` string.| 
| `index`   | - DNA sequence of the forward index from the column `index` from one of the `indexes/*.tsv` files. Alternatively, the field can be left empty or containing `NA` string.|

### Used Parameter Value Combinations

```
# nextseq instrument, dual indexes, index length 8
--index-type dual \
--index-length 8 \
--read-length-1 101 \
--read-length-2 101 \
--adapter-read-1 "AGATCGGAAGAGCACACGTCTGAACTCCAGTCA" \
--adapter-read-2 "AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT" \
--adapter-behavior "trim" \
--minimum-trimmed-read-length 35 \
--mask-short-reads 35 \
--override-cycles "U7N1Y93;I8;I8;U7N1Y93"
```

```
# novaseq instrument, dual indexes, index length 10 
--index-type dual \
--index-length 10 \
--read-length-1 101 \
--read-length-2 101 \
--adapter-read-1 "CTGTCTCTTATACACATCTCCGAGCCCACGAGAC" \
--adapter-read-2 "CTGTCTCTTATACACATCTGACGCTGCCGACGA" \
--adapter-behavior "trim" \
--minimum-trimmed-read-length 35 \
--mask-short-reads 35 \
--override-cycles "U7N1Y93;I10;I10;U7N1Y93"
```

```
# nextseq instrument, simple indexes, index length 8, legacy
--index-type simple \
--index-length 8 \
--read-length-1 101 \
--read-length-2 101 \
--adapter-read-1 "AGATCGGAAGAGCACACGTCTGAACTCCAGTCA" \
--adapter-read-2 "AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT" \
--adapter-behavior "trim" \
--minimum-trimmed-read-length 35 \
--mask-short-reads 22 \
--override-cycles "U7N1Y93;I8;I8;U7N1Y93"
```

## Usage

### Running locally

```
python3 generate_samplesheet.py [parameters]...
```

### Docker

```
docker run --rm -it \
	-v /path_input_file:/container_path_input_file \
	inpred/samplesheet_generator:latest bash
```

### Singularity/Apptainer

```
singularity run \
	-B /path_input_file:/container_path_input_file \
	docker://inpred/samplesheet_generator:latest bash	
	
apptainer run \
	-B /path_input_file:/container_path_input_file \
	docker://inpred/samplesheet_generator:latest bash	
```

### Run Test Data Example

The script is tested with data of a specific sequencing run. The run consists of artificial samples, including AcroMetrix samples. The sequencing was performed on a NextSeq instrument, with the legacy parameter setting and file formats.

#### Test Data Input File

The input info file `infoFile.tsv` is located in `test` folder of this repository and in `/opt/test` of the created Docker image. The content of the file follows:

```
sample_id	molecule	run_id	barcode	index
CLAcroMetrix-D01-X01-X00	DNA	191206_NB501498_0174_AHWCNMBGXC	NA	TCCGGAGA
```

#### Test Data Output 

```
[Header]
Investigator Name,Name (InPreD node)
Experiment Name,OUS pathology test run
Date,07/02/2024

[Reads]
101
101

[Settings]
AdapterRead1,AGATCGGAAGAGCACACGTCTGAACTCCAGTCA
AdapterRead2,AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT
AdapterBehavior,trim
MinimumTrimmedReadLength,35
MaskShortReads,22
OverrideCycles,U7N1Y93;I8;I8;U7N1Y93

[Data]
Sample_ID,Sample_Type,Pair_ID,index,I7_Index_ID,index2,I5_Index_ID
CLAcroMetrix-D01-X01-X00,DNA,CLAcroMetrix-D01-X01-X00,TCCGGAGA,D702,AGGATAGG,D503
```

#### Locally

```
# ${GITHUB_REPOSITORY_LOCAL_PATH} is an absolute path 
# to the samplesheet_generator repository on on the local compute.

# define the testRunID value
testRunID="191206_NB501498_0174_AHWCNMBGXC"

# execute samplesheet_generator
python3 samplesheet_generator.py \
	--run-id ${testRunID} \
	--index-type simple \
	--index-length 8 \
	--investigator-name "Name (InPreD node)" \
	--experiment-name "OUS pathology test run" \
	--input-info-file ${GITHUB_REPOSITORY_LOCAL_PATH}/test/infoFile.tsv \
	--read-length-1 101 \
	--read-length-2 101 \
	--adapter-read-1 "AGATCGGAAGAGCACACGTCTGAACTCCAGTCA" \
	--adapter-read-2 "AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT" \
	--adapter-behavior "trim" \
	--minimum-trimmed-read-length 35 \
	--mask-short-reads 22 \
	--override-cycles "U7N1Y93;I8;I8;U7N1Y93" \
	--samplesheet-version "v1"
```

#### Docker

```
docker run docker://inpred/samplesheet_generator:latest bash

Docker> testRunID="191206_NB501498_0174_AHWCNMBGXC"
Docker> python3 samplesheet_generator.py \
		--run-id ${testRunID} \
		--index-type simple \
		--index-length 8 \
		--investigator-name "Name (InPreD node)" \
		--experiment-name "OUS pathology test run" \
		--input-info-file /opt/test/infoFile.tsv \
		--read-length-1 101 \
		--read-length-2 101 \
		--adapter-read-1 "AGATCGGAAGAGCACACGTCTGAACTCCAGTCA" \
		--adapter-read-2 "AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT" \
		--adapter-behavior "trim" \
		--minimum-trimmed-read-length 35 \
		--mask-short-reads 22 \
		--override-cycles "U7N1Y93;I8;I8;U7N1Y93" \
		--samplesheet-version "v1"
```

#### Singularity/Apptainer

```
singularity run docker://inpred/samplesheet_generator:latest bash

Singularity> testRunID="191206_NB501498_0174_AHWCNMBGXC"
Singularity> python3 samplesheet_generator.py \
		--run-id ${testRunID} \
		--index-type simple \
		--index-length 8 \
		--investigator-name "Name (InPreD node)" \
		--experiment-name "OUS pathology test run" \
		--input-info-file /opt/test/infoFile.tsv \
		--read-length-1 101 \
		--read-length-2 101 \
		--adapter-read-1 "AGATCGGAAGAGCACACGTCTGAACTCCAGTCA" \
		--adapter-read-2 "AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT" \
		--adapter-behavior "trim" \
		--minimum-trimmed-read-length 35 \
		--mask-short-reads 22 \
		--override-cycles "U7N1Y93;I8;I8;U7N1Y93" \
		--samplesheet-version "v1"


apptainer run docker://inpred/samplesheet_generator:latest bash

Apptainer> testRunID="191206_NB501498_0174_AHWCNMBGXC"
Apptainer> python3 samplesheet_generator.py \
		--run-id ${testRunID} \
		--index-type simple \
		--index-length 8 \
		--investigator-name "Name (InPreD node)" \
		--experiment-name "OUS pathology test run" \
		--input-info-file /opt/test/infoFile.tsv \
		--read-length-1 101 \
		--read-length-2 101 \
		--adapter-read-1 "AGATCGGAAGAGCACACGTCTGAACTCCAGTCA" \
		--adapter-read-2 "AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT" \
		--adapter-behavior "trim" \
		--minimum-trimmed-read-length 35 \
		--mask-short-reads 22 \
		--override-cycles "U7N1Y93;I8;I8;U7N1Y93" \
		--samplesheet-version "v1"
```
