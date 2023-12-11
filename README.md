# entrez-parallel
A parrallelized version of entrez-direct functions that uses GNU parallel. 


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
efetch-parallel test/accessions.list.10 test.out.faa 90000 4 protein fasta
```

# Curating list of IDs 
If you'd like to get your own custom list, you can easily do so via NCBI's entrez-direct using esearch or other utilities. You can find the documentation https://www.ncbi.nlm.nih.gov/books/NBK179288/

# Small Issues
Currently, the efetch-parallel gives errors on sequences it can't find. I will update the package later to be able to provide a list of IDs it is giving errors on so you can manually inspect. Also, remember that entrez-parallel parallelization is achieved through GNU parallel and hence has only been tested on Linux platforms. 

# Bug Reports
If you run into any problem using the package or run into any problems, please email me at eshekar@connect.hku.hk or submit an issue to the Issues tab [recommended]

