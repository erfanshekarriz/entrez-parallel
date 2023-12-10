# entrez-parallel
A parrallelized version of entrez-direct functions that uses GNU parallel. 


# Set-up 
1) Clone this repository
2) Create this environment using the .yml file (use conda if you don't have mamba)
3) Give permissions to the bash script 
4) Copy scripts to your conda bin so you can execute it anywhere as long as you have conda acitvated
5) Validate usage
```
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
```
efetch-parallel -h
entrez-parallel test/accessions.list.10 test.out.faa 90000 4 protein fasta
```

