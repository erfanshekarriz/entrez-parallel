## Creating a Viral Contig Identification Test Dataset
This tutorial was built to test a new viral contig identification and taxonomic classification algorithm. To benchmark it against the state-of-the-art tool [geNomad](https://doi.org/10.1038/s41587-023-01953-y),
we must retrieve sequences that were submitted to the Genbank AFTER its training to avoid any specific biases. The geNomad paper states that the data was retrieved on 6th July 2021. 
___
### 1) Download NCBI Genebank Viral 
Search any accession IDs relating to viruses that were submitted AFTER 6th July 2021 (7th July - Present):
```bash
esearch -db nucleotide -query '("viruses"[porgn:__txid10239]) AND (Viruses[Organism]) AND ("2021/7/7"[Publication Date] : "3000"[Publication Date])'
```
Download the accession IDs using efetch:
```bash
esearch \
-db nucleotide \
-query '("viruses"[porgn:__txid10239]) AND (Viruses[Organism]) AND ("2021/7/7"[Publication Date] : "3000"[Publication Date])' \
| efetch -format acc > testData.acc
```
Download the sequences using efetch-parallel and id2taxonomy-parallel:
```bash
# download taxonomy IDs with 4 cores
id2taxonomy-parallel \
testData.acc \
testData.taxid.tsv \
nucleotide \
90000 \
4

# download fasta files with 4 cores 
efetch-parallel \
testData.acc \
testData.fna \
nucleotide \
fasta \
90000 \

# Remember that entrez-parallel commands scale LINEARLY.
# If you have 64 cores take advantage of all of them!
# It will make your task 64 times faster!
```
You can get taxonomy lineage with [taxonkit](https://bioinf.shenwei.me/taxonkit/). Make sure you have [properly installed the database](https://github.com/erfanshekarriz/entrez-parallel)
```
echo -e "taxID\trank\tkingdom\tphylum\tclass\torder\tfamily\tgenus\tspecies\tstrain" > testData.lineage.tsv
cut -f2 testData.taxid.tsv \
| sort -u \
| taxonkit lineage -r -L --threads 4 \
| taxonkit reformat -I 1 -F -S \
-f "{k}\t{p}\t{c}\t{o}\t{f}\t{g}\t{s}\t{t}" >> testData.lineage.tsv

# The code above makes everything into a nice 8-rank classic taxonomic annotation group
```
___
### 2) Sub-sample based on taxonomic groups to create balanced dataset 

___
### 3) Fragment the sequences randomly into different sizes to represent contigs 
Using [Seqkit](https://bioinf.shenwei.me/seqkit/) sliding module create fragments
```bash
for len in $(seq 1500 1500 12000); do seqkit sliding -W ${len} -s ${len} testData.fna > testData_${len}.fna ; done
```
___
### 4) Align them to the geNomad training dataset and remove highly similar sequences

___
### 5) 
