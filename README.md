# Shawanah-Notebook
GENOME="MGWA_VELO045"
USER="USER6"



GENOME="MGWA_VELO045"
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

Repeat Modeler Edited Steps:
mkdir ~/"$GENOME"/repeat_library (#'/home/USER6/MGWA_VELO045/repeat_library': File exists)
cd ~/"$GENOME"/repeat_library (#changed directory to repeat library)
cp ~/"$GENOME"/repeatmodeler/RM_*/consensi.fa ./"$GENOME"_consensi.fa
cp ~/"$GENOME"/repeatmodeler/RM_*/families.stk ./"$GENOME"_families.stk
/opt/perl5/bin/perlbrew switch perl-5.30.0

EDTA LIBRARY SUMMARY REPORT:
Order           Superfamily      # of Sequences# of Clade Sequences    # of Clades# of full Domains
LTR             Bel-Pao                      80              0              0              0
LTR             Copia                       932            221             16              0
LTR             Gypsy                      1098            516             14              0
LTR             Retrovirus                  162              0              0              0
LTR             mixture                      15              0              0              0
pararetrovirus  unknown                       6              0              0              0
DIRS            unknown                      14              0              0              0
Penelope        unknown                       7              0              0              0
LINE            unknown                     275              0              0              0
TIR             Kolobok                      19              0              0              0
TIR             MuDR_Mutator                 33              0              0              0
TIR             P                            20              0              0              0
TIR             PIF_Harbinger                15              0              0              0
TIR             PiggyBac                      1              0              0              0
TIR             Tc1_Mariner                   5              0              0              0
TIR             hAT                          95              0              0              0
Helitron        unknown                       5              0              0              0
Maverick        unknown                     669              0              0              0
mixture         mixture                       5              0              0              0

Repeat Modeler Edited Steps:
mkdir ~/"$GENOME"/repeat_library (#'/home/USER6/MGWA_VELO045/repeat_library': File exists)
cd ~/"$GENOME"/repeat_library (#changed directory to repeat library)
cp ~/"$GENOME"/repeatmodeler/RM_*/consensi.fa ./"$GENOME"_consensi.fa
cp ~/"$GENOME"/repeatmodeler/RM_*/families.stk ./"$GENOME"_families.stk
/opt/perl5/bin/perlbrew switch perl-5.30.0


TE Filter Editing:
time /home/0_PROGRAMS/RepeatModeler-2.0/RepeatClassifier -consensi "$GENOME"_consensi.fa -stockholm "$GENOME"_families.stk
####OUTPUT: "$GENOME"_consensi.fa.classified
SUMMARY REPORT: REPEAT CLASSIFIER:
real	27m3.599s
user	33m56.690s
sys	0m0.709s

time perl /home/0_PROGRAMS/CRL_Scripts1.0/repeatmodeler_parse.pl --fastafile "$GENOME"_consensi.fa.classified --unknowns "$GENOME"_repeatmodeler_unknowns.fasta --identities "$GENOME"_repeatmodeler_identities.fasta
###OUTPUT
SUMMARY REPORT: SEPARATING KNOWN/UNKNOWN REPEATS 
real	0m0.178s
user	0m0.166s
sys	0m0.012s

grep -c "^>" "$GENOME"_repeatmodeler_identities.fasta
###OUTPUT: number of identified TE's = 383 identified


grep -c "^>" "$GENOME"_repeatmodeler_unknowns.fasta
###OUTPUT: number of unidentified TE's= 327 unidentified

time /home/0_PROGRAMS/ncbi-blast-2.10.0+/bin/blastx -query "$GENOME"_repeatmodeler_unknowns.fasta -db /home/0_BIOD98/Tpases020812DNA -evalue 1e-10 -num_descriptions 10 -out "$GENOME"_modelerunknown_blast_results.txt
###OUTPUT
SUMMARY REPORT:compare your TEs to this database to try classify them
real	0m10.747s
user	0m10.705s
sys	0m0.036s

time perl /home/0_PROGRAMS/CRL_Scripts1.0/transposon_blast_parse.pl --blastx "$GENOME"_modelerunknown_blast_results.txt --modelerunknown "$GENOME"_repeatmodeler_unknowns.fasta
###OUTPUT:
SUMMARY REPORT: add the results to your TE fasta file
real	0m0.346s
user	0m0.306s
sys	0m0.040s

mv unknown_elements.txt "$GENOME"ModelerUnknown.lib
###OUTPUT: rename the output file to be more memorable

cat identified_elements.txt "$GENOME"_repeatmodeler_identities.fasta  > "$GENOME"ModelerID.lib
OUTPUT: combine the newly identified TE file with the previously-identified TE file

cat "$GENOME"ModelerUnknown.lib "$GENOME"ModelerID.lib ~/"$GENOME"/EDTA/"$GENOME"_EDTA.fa.EDTA.TElib.fa > "$GENOME"_raw.lib
OUTPUT: Concatenate all of your libraries together

wget ftp://ftp.ncbi.nlm.nih.gov/blast/db/taxdb.tar.gz
gunzip taxdb.tar.gz ; tar -xvf taxdb.tar
tar -xvf taxdb.tar

cp /home/0_BIOD98/taxdb.tar .
tar -xvf taxdb.tar

time /home/0_PROGRAMS/ncbi-blast-2.10.0+/bin/blastx -query "$GENOME"_raw.lib -db /home/0_BIOD98/RefAves_proteins_no_tes.fa -outfmt '6 qseqid staxids bitscore std sscinames sskingdoms stitle' -max_target_seqs 25 -culling_limit 2 -num_threads 23 -evalue 1e-10 -out "$GENOME"_raw.lib.vs.RefAves_proteins_no_tes.out

OUTPUT: install the taxonomy database using wget; run blastx, matching the repeat library to the non-TE protein library that we used before

/home/0_PROGRAMS/assemblage/fastaqual_select.pl -f "$GENOME"_raw.lib -e <(awk '{print $1}' "$GENOME"_raw.lib.vs.RefAves_proteins_no_tes.out | sort | uniq) > "$GENOME"_noprot.lib
OUTPUT: remove the significant hits with fastaqual_select

time perl /home/0_PROGRAMS/EDTA/util/cleanup_tandem.pl -misschar N -nc 50000 -nr 0.8 -minlen 80 -minscore 3000 -trf 1 -trf_path /home/0_PROGRAMS/trf -cleanN 1 -cleanT 1 -f "$GENOME"_noprot.lib > "$GENOME"_noprot_clean.lib
OUTPUT: clean up tandem repeats

grep -c "^>" "$GENOME"_noprot_clean.lib
OUTPUT:count how many TEs remain after filtering. Were very many filtered out compared to your starting numbers?
# OF TE's left after filtering = 
#### Were many TE's filtered out compared to starting numbers? 

#### FINAL SPECIES-SPECIFIC REPEAT LIBRARY FOUND IN: ~/"$GENOME/repeat_library/"$GENOME"_noprot_clean.lib





