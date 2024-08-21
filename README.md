# antiBERTotics

Team Members: 
Scott Auerbach, Craig Corsi, Samuel Ogunfuye, and Hatice Mutlu

This is our repository for our Summer 2024 Erdos Deep Learning Project. The aim of the project is to predict the likelihood of common pathogenic bacteria being resistant to a given antimicrobial small molecule drug by examining the possible correlation between the DNA sequence that codes for antimicrobial resistance (AMR) genes and the SMILES (Simplified Molecular Input Line Entry System) data for each antibiotic that represents its structure with an essentially extended Latin alphabet. 


## Dataset

The dataset used for this project is the Microbigge (Microbial Browser for Genetic and Genomic Elements) data from NCBI's pathogen detection project. Types of information include the bacterial species, the type of the gene coded for (AMR, virulence, or stress). Originally, there were 32,000,000 different genes spread across various pathogens, but due to the sheer size of the data, the web download limit is 100,000 rows(genes), and this was used to extract the DNA sequences from the NCBI Entrez search tool with the contig(uous) accession number for each gene using the *biopython* package. To add the SMILES data for the antibiotics, we used a similar method using the *requests* package. During both of these processes, some of the DNA sequences and SMILES information were irretrievable for whichever reason, so the actual training dataset was somewhat reduced. We focused on two common gut pathogens for this project: *Escherichia coli* and *Salmonella enterica*.
