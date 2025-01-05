# Generating Embeddings and Visualizion Tutorial

This tutorial provides a step-by-step guide for generating embeddings using ESM2 and visualizing them with UMAP.

## Description

This page will demonstrate how to use the ESM2 model to generate fine-tuned embeddings trained on viral proteins on the Biomix cluster. These embeddings capture important features of the sequences that can be useful for various downstream tasks in bioinformatics.
Once the embeddings are generated, the next step will be to utilize UMAP (Uniform Manifold Approximation and Projection) to reduce their dimensionality and create visualizations. UMAP is a widely used technique for visualizing high-dimensional data, allowing for better interpretation and analysis of the high-dimensional embedding data.

## Getting Started

### Dependencies

### Installation

* Conda setup (recommended) [Help](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html)
* Note: The same conda environment can be use for running embeddings and visualization with UMAP so long as all the dependencies are installed. 

```
#Create environment with >= Python 3.8
conda create --name esm-umap python=3.8

#Activate environment
conda activate esm-umap

#Install 
conda install -c conda-forge -c bioconda mmseqs2
conda install bioconda::hmmer

#Clone templates and test data
git clone https://github.com/zschreib/

cd esm-umap

#Grabs required python packages
pip install -r requirements.txt

```