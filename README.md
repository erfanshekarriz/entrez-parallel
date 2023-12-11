# entrez-parallel
A parallelized version of Entrez-direct functions built with GNU parallel. Incredibly speed up your workflows! New integration with taxonkit!

Currently hosting two functions:
```bash
efetch-parallel # From a list of ids download associated NCBI fasta files with parallelization [or other chosen output formats]
id2taxonomy-parallel # From a list of sequence ids get the associated taxonomy with parallelization
```

# Set-up 
1) Clone this repository
2) Create this environment using the .yml file (use conda if you don't have mamba)
3) Give permissions to the bash script 
4) Copy scripts to your conda bin so you can execute it anywhere as long as you have conda acitvated
5) Validate usage
```bash
git clone https://github.com/erfanshekarriz/entrez-parallel.git
cd entrez-parallel

mamba env create -n entrez-parallel -f entrez-parallel.yml
mamba activate entrez-parallel

cd scripts/
chmod +x *

cp ./* $CONDA_PREFIX/bin/

cd ..
efetch-parallel -h # If this works, it means it has correctly been placed in your bin

```

# Test Run 
```bash
efetch-parallel -h
efetch-parallel test/accessions.list.10 test.out.faa protein fasta 90000 4
```

# Curating list of IDs with id2taxonomy-parallel
```bash
# 1) Retrieve all protein IDs associated with Crevaviridae from Refseq database [crAssphage Refseq proteins].
esearch \
-db protein \
-query '"Crevaviridae"[Organism] AND refseq[filter]' \
| efetch -format acc > crevaviridaeRefseq.acc

# 2) Search the accession list and retrieve taxonomy association with the protein
id2taxonmy-parallel \
crevaviridaeRefseq.acc \
crevaviridaeRefseq_taxid.tsv \
protein \
90000 \
4

# 3) Download the associated fasta files using efetch-parallel
efetch-parallel \
crevaviridaeRefseq.acc \
crevaviridaeRefseq.faa \
protein \
fasta \
90000 \
4

# 4) (Optional) Get the full lineage of the taxID with taxonkit
# download taxonkit database
wget ftp://ftp.ncbi.nih.gov/pub/taxonomy/taxdump.tar.gz
mkdir -p ~/.taxonkit
tar -xzf taxdump.tar.gz
mv taxdump/* ~/.taxonkit/
rm -r taxdump*

#  makes everything into a nice 8-level taxonomic table (without headers)
echo -e "taxID\trank\tkingdom\tphylum\tclass\torder\tfamily\tgenus\tspecies\tstrain" > crevaviridaeRefseq_lineage.tsv
cut -f2 crevaviridaeRefseq_taxid.tsv \
| taxonkit lineage -r -L --threads 4 \
| taxonkit reformat -I 1 -F -S \
-f "{k}\t{p}\t{c}\t{o}\t{f}\t{g}\t{s}\t{t}" >> crevaviridaeRefseq_lineage.tsv
```

# Small Issues
Currently, the efetch-parallel gives errors on sequences it can't find. I will update the package later to be able to provide a list of IDs it is giving errors on so you can manually inspect. Also, remember that entrez-parallel parallelization is achieved through GNU parallel and hence has only been tested on Linux platforms. 

# Bug Reports
If you run into any problem using the package or run into any problems, please email me at eshekar@connect.hku.hk or submit an issue to the Issues tab [recommended]

