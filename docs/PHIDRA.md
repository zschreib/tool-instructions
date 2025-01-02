#  PHIDRA ![Version](https://img.shields.io/badge/version-0.2_beta-blue)
**P**rotein **H**omology **I**dentification via **D**omain-**R**elated **A**rchitecture

A simple way to search and validate identifed Pfam domains of interest against a curated InterProScan Domain Architecture (IDA) file to check whether or not your proteins match a domain composition found in the Interpro Database.

## Description

A Python-based package of scripts that performs an initial homology search using `MMseqs2` to identify targeted proteins of interest in small or large datasets. Top hits are searched against the Pfam database using `pfam_scan`, and verified domains are checked and compared against a custom input InterProScan Domain Architecture (IDA) file relative to your target protein of interest. 
A recursive search is then performed using the full length proteins with validated IDAs as the subject database and the original input as the query, with initial matches filtered out. This process captures potentially more distant proteins that may have been missed in the initial homology search but are functionally relevant.

## Getting Started

### Dependencies

* `phidra` can be ran either on a machine with a properly set up python environment or by creating a custom [Conda](https://docs.conda.io/en/latest/) environment if you do not wish to change your current setup (recommended).
* [MMseqs2](https://github.com/soedinglab/MMseqs2) required for inital and recursive homology search. 
* [HMMER](http://hmmer.org/documentation.html) required for pfam database creation and protein domain identifcation through `pfam_scan`.
* [Python](https://www.python.org/downloads/) >= 3.8 
### Installation

* Conda setup (recommended) [Help](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html)
```
#Create environment with >= Python 3.8
conda create --name phidra python=3.8

#Activate environment
conda activate phidra

#Install hmmer/mmseqs2 (Required)
conda install -c conda-forge -c bioconda mmseqs2
conda install bioconda::hmmer

#Clone tool into working directory
git clone https://github.com/zschreib/phidra

cd phidra

#Grabs required python packages
pip install -r requirements.txt

```
* Python setup >= Python 3.8, requires MMseqs2 and HMMER to be already set up.
```
git clone https://github.com/zschreib/phidra
cd phidra
pip install -r requirements.txt
```

* Tool Check. If you recieve any errors here do not proceede until fixed.
```
mmseqs -h
hmmscan -h
python phidra_run.py -v
```
### Setting up Pfam Database (Optional)
* If already have Pfam-hmm formatted database you can skip this step. 

```
mkdir pfam_database
cd pfam_database

wget http://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.hmm.dat.gz
wget http://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.hmm.gz

gunzip -c Pfam-A.hmm.dat.gz > Pfam-A.hmm.dat
gunzip -c Pfam-A.hmm.gz > Pfam-A.hmm
rm Pfam-A.hmm.gz Pfam-A.hmm.dat.gz

hmmpress Pfam-A.hmm
```

## How to create an InterProScan Domain Architecture (IDA) file


## Running the tool
```
usage: phidra_run.py [--help] --input_fasta INPUT_FASTA --subject_db SUBJECT_DB --pfam_hmm_db PFAM_HMM_DB
--ida_file IDA_FILE --function FUNCTION --output_dir OUTPUT_DIR [--threads THREADS] [--version]

Identifies homologous proteins and associated Pfam domains from input protein sequences,
while comparing against InterPro Domain Architectures to analyze domain-level similarities and functional relationships.

Help:
  --help, -h            Description of tool and usage

Required arguments:
  --input_fasta INPUT_FASTA, -i INPUT_FASTA
                        Input protein (aa) FASTA file (default: None)
  --subject_db SUBJECT_DB, -db SUBJECT_DB
                        Protein subject database FASTA file (default: None)
  --pfam_hmm_db PFAM_HMM_DB, -pfam PFAM_HMM_DB
                        Pfam subject mmseqs DB file (default: None)
  --ida_file IDA_FILE, -ida IDA_FILE
                        IDA file for processing. Must be in Interpro IDA TSV format (default: None)
  --function FUNCTION, -f FUNCTION
                        Function of the protein of interest (default: None)
  --output_dir OUTPUT_DIR, -o OUTPUT_DIR
                        Directory for output results of that project (default: None)

Optional Arguments:
  --threads THREADS, -t THREADS
                        Number of threads to use (default: 2)
  --version, -v         Show program's version number and exit
```


* Example run using the test data to search for DNA polymerase A
```
python phidra_run.py -i test_data/test_query/4_seq_test.fa -db test_data/test_subject/pola_16_k12_ref.fa -pfam pfam_database/pfam_hmm -ida test_data/ida_files/polA_IDA_list.tsv -f POLA -o test_results
```

## Authors

Contributors names and contact info

zschreib@udel.edu

## License

This project is licensed under the GNU General Public License v3.0 see the LICENSE file for details

## Acknowledgments

Inspiration, code snippets, etc.
* [pfam_scan](https://github.com/aziele/pfam_scan)
