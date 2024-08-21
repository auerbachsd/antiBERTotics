# antiBERTotics

## Team Members: 
**Scott Auerbach**, **Craig Corsi**, **Samuel Ogunfuye**, and **Hatice Mutlu**

This is our repository for our Summer 2024 Erdos Deep Learning Project. The aim of the project is to predict the likelihood of common pathogenic bacteria being resistant to a given antimicrobial small molecule drug by examining the possible correlation between the DNA sequence that codes for antimicrobial resistance (AMR) genes and the SMILES (Simplified Molecular Input Line Entry System) data for each antibiotic that represents its structure with an essentially extended Latin alphabet. 


## Dataset

The dataset used for this project is the Microbigge (Microbial Browser for Genetic and Genomic Elements) data from NCBI (National Center for Biotechnology Information)'s pathogen detection project. Types of information include the bacterial species, the type of the gene coded for (AMR, virulence, or stress). Originally, there were 32,000,000 different genes spread across various pathogens, but due to the sheer size of the data, the web download limit is 100,000 rows(genes), and this was used to extract the DNA sequences from the NCBI Entrez search tool with the contig(uous) accession number for each gene using the *biopython* package. To add the SMILES data for the antibiotics, we used a similar method using the *requests* package. During both of these processes, some of the DNA sequences and SMILES information were irretrievable for whichever reason, so the actual training dataset was somewhat reduced. We focused on two common gut pathogens for this project: *Escherichia coli* and *Salmonella enterica*.

URL: https://www.ncbi.nlm.nih.gov/pathogens/microbigge/

## Metholodology

As outlined before, the Microbigge dataset did not originally include the DNA sequences, so the Entrez method in Biopython was used to obtain the sequences using the gene's contig accession (this cannot be done using a more general approach like BeautifulSoup). The script for this can be seen on the Jupyter notebook **microbigge_seq_entrez**. For the SMILES information for each of the antimicrobials, we used a *requests*-based approach to perform a mass search query in the DeepChem database in **deepchem-analysis**. However, we did lose quite a few sequences for genes coding for either AMR or something else, so these genes were filtered out for later analyses. Likewise for SMILES, we could not always get some of it because the **Class** column had values like "EFFLUX" which don't correspond to any compounds. In spite of the missing values, most of the original dataset was still available for model training. 

### Binary Classifier for AMR Genes

Initially, we focused on developing a quick binary classifier to identify AMR genes versus non-AMR (for example, virulence or bacterial stress) - one-hot encoding was used to further differentiate the gene types (AMR with a value of 1 with everything else being 0). We used a pre-trained BERT (Bidirectional Encoded Representations of Transformers) model from Hugging Face, DNA-BERT-6. Due to RAM issues on Google Colab, we had to only use some of the dataset because of the large length of the sequences (>1000 nucleotides). Once the sequences were tokenized, we created a class called *SimpleDataset* to convert it back to a format suitable for PyTorch's DataLoader package to test the DNABERT model.

### Identifying AMR based on SMILES and sequence data

This approach used DNABERT, but also another pre-trained BERT model on top of that from DeepChem, **ChemBERTa-77M**. Rather than one-hot encoding for the type of genes, we used the LabelEncoder method from **sklearn.preprocessing**. Another distinction is that we used the logits, or classification scores for embedded SMILES since there were only ~80 different antimicrobial molecules in the dataset shared across genes rather than the raw embeddings themselves. For the model, the encoded label tensors, sequence embeddings, and SMILES logits were concatenated and processed with a class **ResistancePredictionModel**, which processes this concatenated data through two fully connected layers with a sigmoid activation to estimate the probability of a DNA sequence coding for a gene conferring resistance to a given antibiotic given its SMILES information. 
