# Shawanah-Notebook
GENOME="MGWA_VELO045"
USER="USER6"



GENOME=“MGWA_VELO045” 
USER="USER6" 


HOME=/home/"$USER" 
cd ~ 

ASSEMBLY_Date="29Nov2019" 

ASSEMBLY=/home/0_BIOD98_GENOMES2/"$GENOME"/"$GENOME"__29Nov2019__750mill/FASTAOUTPUT/"$GENOME".fasta.gz 

cd ~
mkdir "$GENOME"
mkdir ~/"$GENOME"/genome
scp $USER@142.1.98.20:"$ASSEMBLY" ~/"$GENOME"/genome/"$GENOME".fasta.gz #move your genome assembly into your folder.
gunzip ~/"$GENOME"/genome/"$GENOME".fasta.gz #convert from compressed .fasta.gz to uncompressed .fasta

