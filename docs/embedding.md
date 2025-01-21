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

## Visualization embedding output using UMAP for dimensionality reduction.

### General use of UMAP 

Basic introduction of how to process your embedding output.

```
cd embedding-tutorial/umap_template

Usage: python umap_visualize.py <input_directory> <output_plot_file.png>

#Note: Highly recommened to read through the UMAP documentation and get an idea of what each of these input parameters do.

#Adjust according to how you want to represent your data.
umap.UMAP(n_neighbors=3, min_dist=0.5, n_components=2, metric='euclidean').fit(embeddings)
```

* To run the UMAP script against the test data provided you can enter the following: 

```
python umap_visualize.py ../test_input/embeddings_data/ test_out.png
```

* Note: You may see some warnings if not on a GPU node but as long as there is a png image output they job as finished.

### Advanced UMAP use with metadata.

Customize your plots with auto generated color maps or defined custom colors through metadata files.

```
cd embedding-tutorial/umap_color_template

#The test_metadata.tsv will give a basic overview on how to structure a metadata file

Query_ID        signature
ENA_MN103542_MN103542_1_12796_10265_26  RDKL
ENA_PP514360_PP514360_1_8414_6123_12    RDKL
ENA_PP534163_PP534163_1_69504_71861_103 RDKF
ENA_PP579741_PP579741_1_77727_75163_114 RDKF
ENA_PP582188_PP582188_1_34616_36967_50  RDKL
ENA_PP554395_PP554395_1_25693_23567_24  ABCD

```
* Values here will either be auto assigned a color based on the 'signature' column or manually assigned by how you structure the umap_color_visualize.py file

```
#Example of how to edit the map_color_visualize.py file to assign custom colors manually 

def custom_color_manual(metadata_df, column_name):
    # Predefined color table
    color_table = {
        "RDKF": '#ec4400',
        "RDKL": '#8c29b1',
        "RDKY": '#1b33e3',
        "RDKH": '#008856',
        # Add more entries as needed
    }

#Note: ENA_PP554395_PP554395_1_25693_23567_24  ABCD and other embeddings present in your that do not have a mapping will default to gray 

#The manual and auto color generation will be saved here in the main function

metadata_df['auto_color'] = metadata_df[column_name].map(auto_colors)
metadata_df['manual_color'] = metadata_df[column_name].map(manual_colors)

#You can use these in the 

plot_umap(embeddings, embedding_ids, metadata_df, output_path)

#by going to the function and selecting the color map you want.

#Auto
def plot_umap(embeddings, embedding_ids, metadata, output_file):

    #maps your ids to color dictionary.
    id_to_color = dict(zip(metadata['Query_ID'], metadata['auto_color']))

#Manual
def plot_umap(embeddings, embedding_ids, metadata, output_file):

    #maps your ids to color dictionary.
    id_to_color = dict(zip(metadata['Query_ID'], metadata['manual_color']))

```

* Here is how you can run the example data to visualize output with color.
```
python umap_color_visualize.py ../test_input/embeddings_data test_metadata.tsv signature manual_color.png
```

* Note: You may see some warnings if not on a GPU node but as long as there is a png image output they job as finished.
* These templates are in place to a basic understanding on how to color UMAP plots in relation to your data. Feel free to edit them however you want for future analysis.

## Using HDBSCAN to Cluster Embedding Plots

An introduction to using HDBSCAN with generated embeddings on a UMAP plot.
For more information please see [HDBSCAN](https://umap-learn.readthedocs.io/en/latest/clustering.html#) utilization with UMAP.
To learn more about what's going on behind HDBSCAN, check out [How it works](https://hdbscan.readthedocs.io/en/latest/how_hdbscan_works.html)

### Basic HDBSCAN

This performs HDBSCAN clustering directly on your UMAP-transformed embeddings without taking into account local structure.

The UMAP transformation and HDBSCAN clustering happen here:

```
def plot_umap_hdbscan(embeddings, embedding_ids, metadata, output_file):
    
    #Perform UMAP dimensionality reduction
    standard_embedding = umap.UMAP(n_neighbors=3, min_dist=0.2, n_components=2, metric='euclidean', random_state=42).fit_transform(embeddings)

    #Perform clustering with HDBSCAN
    labels = hdbscan.HDBSCAN(min_samples=2, min_cluster_size=3).fit_predict(embeddings)
    clustered = (labels >= 0)

```

We can now plot the clustered and unclustered data here:

```
    #Plotting the UMAP results with HDBSCAN clusters
    plt.figure(figsize=(10, 7))
    plt.scatter(
        standard_embedding[~clustered, 0],
        standard_embedding[~clustered, 1],
        color="gray",
        s=20,
        alpha=0.5,
        label="Noise",
        marker="1" #tri_down for unclustered data
    )

    #Shapes applied to clustered data only (can modify to all if needed)
    unique_markers = set(markers)
    for marker in unique_markers:
        #Shape to embedding id
        marker_mask = np.array([(m == marker) and clustered[i] for i, m in enumerate(markers)])
        plt.scatter(
            standard_embedding[marker_mask, 0],
            standard_embedding[marker_mask, 1],
            c=labels[marker_mask],
            s=20,
            cmap="Spectral",
            marker=marker,
            alpha=0.8
        )
```

* Depending on the types of questions you want to answer with your data and assigned metadata, this may look different.

To run a test on the provided data, try:

```
python umap_HDBSCAN_basic.py ../test_input/embeddings_data test_metadata.tsv cluster_basic_with_metadata.png
```

### Enhanced HDBSCAN with UMAP
Important note taken from HDBSCAN documentation:

The next thing to be aware of is that when using UMAP for dimension reduction you will want to select different parameters than if you were using it for visualization. First of all we will want a larger n_neighbors value small values will focus more on very local structure and are more prone to producing fine grained cluster structure that may be more a result of patterns of noise in the data than actual clusters. In this case we will double it from the default 15 up to 30. Second it is beneficial to set min_dist to a very low value. Since we actually want to pack points together densely (density is what we want after all) a low value will help, as well as making cleaner separations between clusters. In this case we will simply set min_dist to be 0.

Notice the difference between the basic HDBSCAN and the order of operations for the enhanced version:

```
def plot_umap_hdbscan(embeddings, embedding_ids, output_file):

    # Perform UMAP dimensionality reduction
    standard_embedding = umap.UMAP(n_neighbors=3, min_dist=0.2, n_components=2, metric='euclidean', random_state=42).fit_transform(embeddings)
    #HDBSCAN enhanced. Adjust according to data
    clusterable_embedding = umap.UMAP(n_neighbors=6, min_dist=0.0, n_components=2, random_state=42).fit_transform(embeddings)
    #Adjust according to data
    labels = hdbscan.HDBSCAN(min_samples=2, min_cluster_size=3).fit_predict(clusterable_embedding)
    #New cluster labels
    clustered = (labels >= 0)

```

* Compare the output of basic and enhanced versions to better understand how your clusters shift depending on the parameters you use and how you implement HDBSCAN with UMAP.

To run a test on the data provided you can try out:

```
python umap_HDBSCAN_enhanced.py ../test_input/embeddings_data cluster_enhanced.png
```

* If you would like to save the embedding ID and cluster assigned to output you can always add this line in: 

```
    plt.close()
    #End of plot
    
    #Labels indexed in same order as embedding_id (very simple way of saving output)
    df = pd.DataFrame({
        'Embedding_ID': embedding_ids,
        'Cluster': labels
    })

    #Write to output as a tab table
    df.to_csv('cluster_output.tsv', sep='\t', index=False)
```

## Authors

Contact info

zschreib@udel.edu

## Acknowledgments

Inspiration, code snippets, etc.
[ESM2](https://github.com/facebookresearch/esm), [UMAP](https://umap-learn.readthedocs.io/en/latest/)
