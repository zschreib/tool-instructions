# Generating Embeddings and Visualizion Tutorial

This tutorial provides a step-by-step guide for generating embeddings using ESM2 and visualizing them with UMAP.

## Description

This page will demonstrate how to use the ESM2 model to generate fine-tuned embeddings trained on viral proteins on the Biomix cluster. These embeddings capture important features of the sequences that can be useful for various downstream tasks in bioinformatics.
Once the embeddings are generated, the next step will be to utilize UMAP (Uniform Manifold Approximation and Projection) to reduce their dimensionality and create visualizations. UMAP is a widely used technique for visualizing high-dimensional data, allowing for better interpretation and analysis of the high-dimensional embedding data.

## Getting Started

### Dependencies

If you have a working environment and wish to add the basic requirements, you can manually install the following. It is highly recommended to use conda and create a prebuilt environment provided below.

* [ESM2](https://github.com/facebookresearch/esm) required to generate embeddings using pre-trained models.
* [UMAP](https://umap-learn.readthedocs.io/en/latest/) required for embedding visualization.
* [PyTorch](https://github.com/pytorch/pytorch) for tensor computation utilizing GPU.
* [CUDA-Toolkit](https://developer.nvidia.com/cuda-toolkit) required for tools to utilize GPU. Needs to be a version compatible with your graphics card.
* [Python](https://www.python.org/downloads/) >= 3.8

### Installation

* Conda setup (recommended) [Help](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html)
* Note: The same conda environment can be use for running embeddings and visualization with UMAP so long as all the dependencies are installed. 

```
#Clone templates and test data
git clone https://github.com/zschreib/embedding-tutorial.git

cd embedding-tutorial

#Create conda environment with requirements
conda env create -f environment.yml

#Activation needed for UMAP visualization but not embedding generation if you follow the embedding template
conda activate esm-umap

```

## Generating Embeddings of viral fine-tuned models using contrastive learning.

### Configuration

If you are running the provided scripts on Biomix they are pre-configured to utilize viral fine-tuned models in the extract.py. If you wish to use the general ESM2 pre-trained models on all domains of life you need to clone the [ESM2](https://github.com/facebookresearch/esm) repo and point the `embedding_template.sh` to that `extract.py` script along with the correct pre-trained datasets. See their documentation for more information. 

```
#Move to the embedding template

cd embedding_slurm_template

#Open the template with your favorite text editor and adjust the paths accordingly

#Biomix configuration needed to utilize GPU node. Adjust --mem accordingly (50GB should cover the models and your data)
#!/bin/bash
#SBATCH --job-name=test
#SBATCH --time=UNLIMITED
#SBATCH --gres=gpu:1
#SBATCH --partition=gpu2
#SBATCH --account=gpu2
#SBATCH --qos gpu
#SBATCH --mem 50GB

# Tells the node to activate your conda env before running job
source activate esm-umap

# Define path variables do not change unless you want to run your own models
EXTRACT_SCRIPT="/mnt/VEIL/tools/embeddings/models/extract.py"
MODEL_PATH="/mnt/VEIL/tools/embeddings/models/esm2_t36_3B_UR50D.pt"

#Path to input amino acid fasta file and output directory. List full path
INPUT_FILE="/work/user/fasta_file_location.fa"
OUTPUT_DIR="/work/user/output/dir"

#Run command for the embeddings to generate mean-representation of the protein
python $EXTRACT_SCRIPT $MODEL_PATH $INPUT_FILE $OUTPUT_DIR --repr_layers 36 --include mean

```

* Once configured the script can be sent to the slurm job scheduler using:

```
sbatch embedding_template.sh
```

* Logs will be found in the embedding_slurm_template directory. 

## Authors

Contact info

zschreib@udel.edu

## Acknowledgments

Inspiration, code snippets, etc.
[ESM2](https://github.com/facebookresearch/esm), [UMAP](https://umap-learn.readthedocs.io/en/latest/)