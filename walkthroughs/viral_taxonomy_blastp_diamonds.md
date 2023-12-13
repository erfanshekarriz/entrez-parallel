## Viral Contig Taxonomy Assignment with Diamond Blastp 
This tutorial follows a [highly cited protocol](https://doi.org/10.1038/s41564-021-00928-6) for viral contig taxonomic assignments.
You can get a consensus taxonomic assignment for your viral contigs using my upcoming pipeline [soon to be released], but this should be sufficient meanwhile!
> As validation, we applied our pipeline to taxonomically annotated genomes from NCBI GenBank after removing closely related genes from the database. Our pipeline achieved average TPRs of 90.0%, 98.7%, 92.2% and 73.5% at precision values of 95.6%, 99.9%, 99.3% and 96.5% for taxonomic ranks of Baltimore, order, family and genus, respectively.
>
-- <cite>Nayfach, S et al. (2021)</cite>

### 1) Download all viral NCBI proteins & associate Taxonomy Lineage
First, you want to download all NCBI viral proteins from the database. At the current moment [12th Dec 2023] there are around 55,879,844. 
You can specifically check this number out by yourself using [Make sure to [install](https://github.com/erfanshekarriz/entrez-parallel) entrez-parallel first]
```bash
esearch -db protein -query 'viruses[orgn]'
```
To download the accession ids simply run: 
```bash
# i have not parallelized this function yet, so this will take 12+ hours. working on a parallelized version of this command.
esearch \
-db protein \
-query 'viruses[orgn]' \
| efetch -format acc > ncbiVirusGeneBankprot.acc
# If you want only refseq then do 'viruses[orgn] AND refseq[filter]'
```
Then you'd want to run efetch-parallel and id2taxonomy-parallel to download the associated ids and fasta files:
```
# download taxonomy IDs with 4 cores
id2taxonomy-parallel \
ncbiVirusGeneBankprot.acc \
ncbiVirusGeneBankprot.taxid.tsv \
protein \
90000 \
4

# download fasta files with 4 cores 
efetch-parallel \
ncbiVirusGeneBankprot.acc \
ncbiVirusGeneBankprot.faa \
protein \
fasta \
90000 \

# Remember that entrez-parallel commands scale LINEARLY.
# If you have 64 cores take advantage of all of them!
# It will make your task 64 times faster!
```
You can get taxonomy lineage with [taxonkit](https://bioinf.shenwei.me/taxonkit/). Make sure you have [properly installed the database](https://github.com/erfanshekarriz/entrez-parallel)
```
echo -e "taxID\trank\tkingdom\tphylum\tclass\torder\tfamily\tgenus\tspecies\tstrain" > ncbiVirusGeneBankprot.lineage.tsv
cut -f2 ncbiVirusGeneBankprot.taxid.tsv \
| sort -u \
| taxonkit lineage -r -L --threads 4 \
| taxonkit reformat -I 1 -F -S \
-f "{k}\t{p}\t{c}\t{o}\t{f}\t{g}\t{s}\t{t}" >> ncbiVirusGeneBankprot.lineage.tsv

# the code above makes everything into a nice 8 rank classic taxonomic annotation group
```

## 2) To be continued...
I'm working on a universal scrip that can combine and filter the taxonomy information with DIAMOND output and give you a taxonomic annotation according to the Nature Microbiology paper mentioned above.

## Citations 
```
Nayfach, S., Páez-Espino, D., Call, L. et al.
Metagenomic compendium of 189,680 DNA viruses from the human gut microbiome.
Nat Microbiol 6, 960–970 (2021).
https://doi.org/10.1038/s41564-021-00928-6

Wei Shen, Hong Ren,
TaxonKit: A practical and efficient NCBI taxonomy toolkit,
Journal of Genetics and Genomics, Volume 48, Issue 9, (2021)
https://doi.org/10.1016/j.jgg.2021.03.006.

```
