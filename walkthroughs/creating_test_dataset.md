## Creating a Viral Contig Identification Test Dataset
This tutorial was built to test a new viral contig identification and taxonomic classification algorithm. To benchmark it against the state-of-the-art tool [geNomad](https://doi.org/10.1038/s41587-023-01953-y),
we must retrieve sequences that were submitted to the Genbank AFTER its training to avoid any specific biases. The geNomad paper states that the data was retrieved on 6th July 2021. 
___
### 1) Download NCBI Genebank Viral 
Search any accession IDs relating to viruses that were submitted AFTER 6th July 2021 (7th July - Present) and bacterial RefSeq submitted after 15th Dec 2023:
```bash
# Viruses Refseq 2021-07-07 - Present (n=4,109)
esearch -db nucleotide -query '("2021/7/7"[PDAT] : "3000"[PDAT]) AND viruses[filter] AND refseq[filter] AND ("Viruses"[Organism])'
# Bacteria Genomes 2021-07-07 - Present (n=3,913)
esearch -db genome -query '"Bacteria"[Organism] AND ("2021/7/7"[CreateDate] : "3000"[CreateDate])'
# We use a smaller bacterial submission window and only RefSeq sequences
# to roughly balance the dataset to reduce computation time
```
Download the accession IDs using efetch:
```bash
esearch \
-db nucleotide \
-query '("2021/7/7"[PDAT] : "3000"[PDAT]) AND viruses[filter] AND refseq[filter] AND ("Viruses"[Organism])' \
| efetch -format acc > testData.acc

esearch \
-db genome \
-query '"Bacteria"[Organism] AND ("2021/7/7"[CreateDate] : "3000"[CreateDate])' \
| elink -target nucleotide \
| efetch -format acc >> testData.acc

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
```
Get taxonomy lineage with [taxonkit](https://bioinf.shenwei.me/taxonkit/). Make sure you have [properly installed the database](https://github.com/erfanshekarriz/entrez-parallel)
```
echo -e "taxID\trank\tkingdom\tphylum\tclass\torder\tfamily\tgenus\tspecies\tstrain" > testData.lineage.tsv
cut -f2 testData.taxid.tsv \
| sort -u \
| taxonkit lineage -r -L --threads 4 \
| taxonkit reformat -I 1 -F -S \
-f "{k}\t{p}\t{c}\t{o}\t{f}\t{g}\t{s}\t{t}" >> testData.lineage.tsv

# The code above makes everything into a nice 8-rank classic taxonomic annotation group
```
Take a look at the taxonomic distribution:
```bash
# Kingdom
cut -f3 testData.lineage.tsv | sort | uniq -c | sort -nr
# Phylum 
cut -f4 testData.lineage.tsv | sort | uniq -c | sort -nr
# Class
cut -f5 testData.lineage.tsv | sort | uniq -c | sort -nr
```
Download the fasta for the viral and bacterial sequences using efetch-parallel
```
# download fasta files with 4 cores 
efetch-parallel \
testData.acc \
testData.fna \
nucleotide \
fasta \
90000 \
4

# Remember that entrez-parallel commands scale LINEARLY.
# If you have 64 cores take advantage of all of them!
# It will make your task 64 times faster!
```
___
### 2) Fragment the sequences randomly into different sizes to represent contigs 
Using [Seqkit](https://bioinf.shenwei.me/seqkit/) sliding module create fragments
```bash
for len in $(seq 1500 1500 12000); do seqkit sliding -W ${len} -s ${len} testData.fna > testData_${len}.fna ; done
```
___
### 3) Align them to the geNomad training dataset and remove highly similar sequences

___
### 4) 
