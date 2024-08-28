# antiBERTotics

## Team Members: 

<ul>
<li> Scott Auerbach </li>
<li> Craig Corsi </li>
<li> Samuel Ogunfuye </li>
<li> Hatice Mutlu </li>
</ul>

This is our repository for our Summer 2024 Erdos Deep Learning Project. The aim of the project is to predict the likelihood of common pathogenic bacteria being resistant to a given antimicrobial small molecule drug by examining the possible correlation between the DNA sequence that codes for antimicrobial resistance (AMR) genes and the SMILES (Simplified Molecular Input Line Entry System) data for each antibiotic that represents its structure with an essentially extended Latin alphabet. 


## Dataset

The dataset used for this project is the Microbigge (Microbial Browser for Genetic and Genomic Elements) data from NCBI (National Center for Biotechnology Information)'s pathogen detection project. Types of information include the bacterial species, the type of the gene coded for (AMR, virulence, or stress). Originally, there were 32,000,000 different genes spread across various pathogens, but due to the sheer size of the data, the web download limit is 100,000 rows(genes), and this was used to extract the DNA sequences from the NCBI Entrez search tool with the contig(uous) accession number for each gene using the *biopython* package. To add the SMILES data for the antibiotics, we used a similar method using the *requests* package. During both of these processes, some of the DNA sequences and SMILES information were irretrievable for whichever reason, so the actual training dataset was somewhat reduced. We focused on two common gut pathogens for this project: *Escherichia coli* and *Salmonella enterica*.

URL: https://www.ncbi.nlm.nih.gov/pathogens/microbigge/

We've also included a 50-row sample of the filtered (missing values for SMILES or DNA sequences removed) *E. coli* dataset used for model evaluation to give an example. The full file is ~4.73 GB and thus cannot be easily shared, but if anyone is interested, please follow up with us for access.

## Methodology

As outlined before, the Microbigge dataset did not originally include the DNA sequences, so the Entrez method in Biopython was used to obtain the sequences using the gene's contig accession (this cannot be done using a more general approach like BeautifulSoup). The script for this can be seen on the Jupyter notebook **microbigge_seq_entrez**. For the SMILES information for each of the antimicrobials, we used a *requests*-based approach to perform a mass search query in the DeepChem database in **deepchem-analysis**. However, we did lose quite a few sequences for genes coding for either AMR or something else, so these genes were filtered out for later analyses. Likewise for SMILES, we could not always get some of it because the **Class** column had values like "EFFLUX" which don't correspond to any compounds. In spite of the missing values, most of the original dataset was still available for model training. 

Our stakeholders include both pre-clinical and clinical research teams, as well as pharmaceutical companies, medical centers, and their clients. Our KPIs are the accuracy score and F1-score.

### Binary Classifier for AMR Genes

Initially, we focused on developing a quick binary classifier to identify AMR genes versus non-AMR (for example, virulence or bacterial stress) - one-hot encoding was used to further differentiate the gene types (AMR with a value of 1 with everything else being 0). We used a pre-trained BERT (Bidirectional Encoded Representations of Transformers) model from Hugging Face, DNA-BERT-6. Due to RAM issues on Google Colab, we had to only use some of the dataset because of the large length of the sequences (>1000 nucleotides). Once the sequences were tokenized, we created a class called *SimpleDataset* to convert it back to a format suitable for PyTorch's DataLoader package to test the DNABERT model. To inflate the amount of data without crashing all available RAM in Google Colab, we used the k-mer technique, or shifting a select number of nucleotides (letters in the sequence, in our case six) to the left or right as well as introduce random mutations in the sequence.

### Identifying AMR based on SMILES and sequence data

This approach used DNABERT, but also another pre-trained BERT model on top of that from DeepChem, **ChemBERTa-77M**. Rather than one-hot encoding for the type of genes, we used the *LabelEncoder* method from **sklearn.preprocessing**. Another distinction is that we used the logits, or classification scores for embedded SMILES since there were only ~80 different antimicrobial molecules in the dataset shared across genes rather than the raw embeddings themselves. For the model, the encoded label tensors, sequence embeddings, and SMILES logits were concatenated and processed with a class **ResistancePredictionModel**, which processes this concatenated data through two fully connected layers with a sigmoid activation to estimate the probability of a DNA sequence coding for a gene conferring resistance to a given antibiotic given its SMILES information. All of this can be found in the **tandem-tokenizers** notebook.

## Results

For the binary AMR classifier using the pretrained DNABERT model, we got these results for the following species:

<ul>

<li> E. coli : 54.6% accuracy, F1: 0.39 (4,500 samples) </li>
<li> Salmonella enterica : 79.3% accuracy, F1: 0.70 (1,500 samples) </li>

</ul>

Pivoting towards incorporating SMILES information to predict antibiotic resistance, we achieved the following results:

<ul>

<li> E. coli : 53.8% accuracy, F1: N/A </li>
<li> Salmonella enterica : 81.4% accuracy, F1: 0.49 </li>

</ul>

## Future Directions

We would like to develop an application that can take the SMILES input of a given antibiotic and then predict the likelihood of generating resistance for multiple common microbes (including but not limited to *E. coli*, *Salmonella*, *Listeria*). However, before doing that, we need to refine our hyper-parameters further and increase our accuracy statistics, and this may be done through acquiring more of the original Microbigge dataset on Google Cloud or finding an alternative, more comprehensive dataset.


## Citations

<ol>
<li> Ea Zankari, Henrik Hasman, Salvatore Cosentino, Martin Vestergaard, Simon Rasmussen, Ole Lund, Frank M. Aarestrup, Mette Voldby Larsen, Identification of acquired antimicrobial resistance genes, Journal of Antimicrobial Chemotherapy, Volume 67, Issue 11, November 2012, Pages 2640–2644, https://doi.org/10.1093/jac/dks261 </li>
<li> Eyal Mazuz, Guy Shtar, Nir Kutsky, Lior Rokach, Bracha Shapira, Pretrained transformer models for predicting the withdrawal of drugs from the market, Bioinformatics, Volume 39, Issue 8, August 2023, btad519, https://doi.org/10.1093/bioinformatics/btad519 </li>
<li> Yanrong Ji, Zhihan Zhou, Han Liu, Ramana V Davuluri, DNABERT: pre-trained Bidirectional Encoder Representations from Transformers model for DNA-language in genome, Bioinformatics, Volume 37, Issue 15, August 2021, Pages 2112–2120, https://doi.org/10.1093/bioinformatics/btab083 </li>
<li> Ren Y, Chakraborty T, Doijad S, Falgenhauer L, Falgenhauer J, Goesmann A, Hauschild AC, Schwengers O, Heider D. Prediction of antimicrobial resistance based on whole-genome sequencing and machine learning. Bioinformatics. 2022 Jan 3;38(2):325-334. doi: 10.1093/bioinformatics/btab681. PMID: 34613360; PMCID: PMC8722762. </li>
</ol>
