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


TE Filter Editing:
export PERLBREW_ROOT=/opt/perl5
/opt/perl5/bin/perlbrew switch perl-5.30.0
####OUTPUT: perlbrew will be installed in opt AND A sub-shell is launched with perl-5.30.0 as the activated perl. Run 'exit' to finish it.

time /home/0_PROGRAMS/RepeatModeler-2.0/RepeatClassifier -consensi "$GENOME"_consensi.fa -stockholm "$GENOME"_families.stk
####OUTPUT: "$GENOME"_consensi.fa.classified
SUMMARY REPORT: REPEAT CLASSIFIER:
real	21m33.153s
user	26m3.446s
sys	0m0.510s

time perl /home/0_PROGRAMS/CRL_Scripts1.0/repeatmodeler_parse.pl --fastafile "$GENOME"_consensi.fa.classified --unknowns "$GENOME"_repeatmodeler_unknowns.fasta --identities "$GENOME"_repeatmodeler_identities.fasta
###OUTPUT
SUMMARY REPORT: SEPARATING KNOWN/UNKNOWN REPEATS 
real	0m0.177s
user	0m0.081s
sys	0m0.020s

grep -c "^>" "$GENOME"_repeatmodeler_identities.fasta
###OUTPUT: number of identified TE's = 383 identified

grep -c "^>" "$GENOME"_repeatmodeler_unknowns.fasta
###OUTPUT: number of unidentified TE's= 327 unidentified

time /home/0_PROGRAMS/ncbi-blast-2.10.0+/bin/blastx -query "$GENOME"_repeatmodeler_unknowns.fasta -db /home/0_BIOD98/Tpases020812DNA -evalue 1e-10 -num_descriptions 10 -out "$GENOME"_modelerunknown_blast_results.txt
###OUTPUT
SUMMARY REPORT:compare your TEs to this database to try classify them
real	0m10.503s
user	0m10.446s
sys	0m0.052s

time perl /home/0_PROGRAMS/CRL_Scripts1.0/transposon_blast_parse.pl --blastx "$GENOME"_modelerunknown_blast_results.txt --modelerunknown "$GENOME"_repeatmodeler_unknowns.fasta
###OUTPUT:
SUMMARY REPORT: add the results to your TE fasta file
real	0m0.392s
user	0m0.355s
sys	0m0.013s

mv unknown_elements.txt "$GENOME"ModelerUnknown.lib
###OUTPUT: rename the output file to be more memorable

cat identified_elements.txt "$GENOME"_repeatmodeler_identities.fasta  > "$GENOME"ModelerID.lib
OUTPUT: combine the newly identified TE file with the previously-identified TE file

cat "$GENOME"ModelerUnknown.lib "$GENOME"ModelerID.lib ~/"$GENOME"/EDTA/"$GENOME"_EDTA.fa.EDTA.TElib.fa > "$GENOME"_raw.lib
OUTPUT: Concatenate all of your libraries together

cp /home/0_BIOD98/taxdb.tar .
tar -xvf taxdb.tar

time /home/0_PROGRAMS/ncbi-blast-2.10.0+/bin/blastx -query "$GENOME"_raw.lib -db /home/0_BIOD98/RefAves_proteins_no_tes.fa -outfmt '6 qseqid staxids bitscore std sscinames sskingdoms stitle' -max_target_seqs 25 -culling_limit 2 -num_threads 23 -evalue 1e-10 -out "$GENOME"_raw.lib.vs.RefAves_proteins_no_tes.out
OUTPUT: run blastx, matching the repeat library to the non-TE protein library that we used before

SUMMARY REPORT:

real	17m56.669s
user	360m54.581s
sys	0m14.161s

/home/0_PROGRAMS/assemblage/fastaqual_select.pl -f "$GENOME"_raw.lib -e <(awk '{print $1}' "$GENOME"_raw.lib.vs.RefAves_proteins_no_tes.out | sort | uniq) > "$GENOME"_noprot.lib
OUTPUT: remove the significant hits with fastaqual_select

time perl /home/0_PROGRAMS/EDTA/util/cleanup_tandem.pl -misschar N -nc 50000 -nr 0.8 -minlen 80 -minscore 3000 -trf 1 -trf_path /home/0_PROGRAMS/trf -cleanN 1 -cleanT 1 -f "$GENOME"_noprot.lib > "$GENOME"_noprot_clean.lib
OUTPUT: clean up tandem repeats
SUMMARY REPORT:

real	0m22.013s
user	0m21.916s
sys	0m0.081s

grep -c "^>" "$GENOME"_noprot_clean.lib
OUTPUT:count how many TEs remain after filtering. Were very many filtered out compared to your starting numbers?
# OF TE's left after filtering = 5118
#### Were many TE's filtered out compared to starting numbers? STARTING = 173967 (45609) sequences.  FILTERED = 5118

#### FINAL SPECIES-SPECIFIC REPEAT LIBRARY FOUND IN: ~/"$GENOME/repeat_library/"$GENOME"_noprot_clean.lib

####TO MAKE LIBRARY MORE COMPREHENSIVE, concatenate it with previously published, curated bird TE libraries.
####WILL USE REPEAT LIBRARIES FROM: Blue-capped Cordon Bleu (Uraeginthus cyanocephalus) and Collared Flycatcher Ficedula albicollis look like they would be beneficial to add with our own library.

cat "$GENOME"_noprot_clean.lib /home/0_BIOD98/68015-fAlb15_rm3.0.lib /home/0_BIOD98/uraCya_rm2.45.fasta > "$GENOME"_repeat_library_withFicalbUracya.lib
#concatenate repeat library from Blue-capped Cordon Bleu Uraeginthus and Ficedula albicollis with our own library

#### final repeat library to use for RepeatMasking is found in: ~/"$GENOME/repeat_library/"$GENOME"_repeat_library_withFicalbUracya.lib (DONE)

### MAKER STEPS 1:

mkdir ~/"$GENOME"/maker 
###OUTPUT: you need a folder to be working in on the Ramphocelus node

cp ~/"$GENOME"/genome/"$GENOME".fasta ~/"$GENOME"/maker/"$GENOME"_input.fasta
####OUTPUT: copy the genome into your new folder

less ~/"$GENOME"/maker/"$GENOME"_input.fasta #press 'q' to go back
####OUTPUT: View the start of the file and see that the first contig is probably named "0"

sed -i 's/^>0 />0start /g' ~/"$GENOME"/maker/"$GENOME"_input.fasta
####OUTPUT: this line will replace the name "0" to "0start", which maker allows

less ~/"$GENOME"/maker/"$GENOME"_input.fasta #press 'q' to go back
####OUTPUT: make sure that this change worked! It should be named "0start" - if it is not, make a post in Issues on Github.


scp "$USER"@142.1.98.20:/home/0_BIOD98_GENOMES1/proteomes/"EmberizoideaSuperPlus_proteomes.no_tes.fa" ./"$GENOME"prot_input.fasta

####CONSIDERATION: If you are a Setophaga, Geothlypis, or Passerella then you need 'EmberizoideaSuperPlus_proteomes.no_tes.fa'
####change the proteome name to be the correct proteome for your species eg "PipridaeSuperPlus_proteomes.no_tes.fa"

scp "$USER"@142.1.98.20:/home/0_BIOD98_GENOMES1/transcriptomes/"Setcae_transcriptome.fa" ./"$GENOME"RNA_input.fa

###CONSIDERATION: If you are a Setophaga or Geothlypis, then you need Setcae_transcriptome.fa. I made it from Setophaga caeruleus.
####change the proteome name to be the correct proteome for your species

scp "$USER"@142.1.98.20:/home/0_BIOD98_GENOMES1/Maker_ctl_templates/maker_*.ctl .
###OUTPUT: To run maker you need three input files (maker_exe.ctl: - paths to required programs, maker_bopts.ctl: BLAST and Exonerate settings, maker_opt.ctl: other MAKER settings as location of genome, and training parameters for gene prediction programs)

grep "SKUASareINTERESTING" maker_opts.ctl
###OUTPUT: edit this file slightly to give it the name of your genome, you should see 4 lines highlighted in red - these are the four lines that must change

sed -i "s/SKUASareINTERESTING/$GENOME/g" maker_opts.ctl
###OUTPUT:replace with the name of your genome; MAKE SURE THAT YOUR GENOME NAME IS DEFINED AND CORRECT

grep "SKUASareINTERESTING" maker_opts.ctl
####OUTPUT: make sure that it worked; you should get no output. If you still get some lines that say "SKUASareINTERESTING", post in the issues forum on Github

grep "$GENOME" maker_opts.ctl
###OUTPUT: you should see 4 lines nicely placing the name of your species into the file

#on the MAIN server (not Ramphocelus):
cd ~
mkdir temp
mkdir temp/maker
cd temp/maker
#replace USER0 with your username
scp USER6@192.168.0.5:~/"$GENOME"/maker/* . #transfers all files in this folder
###OUTPUT:  transfer everything onto the main server since Niagara cannot see Ramphocelus

#change USERNAME to be your Compute Canada Username
ssh niki03@niagara.scinet.utoronto.ca
#enter your password when it asks
###OUTPUT: log into Niagara using your Compute Canada username

GENOME=Your_Genome_name #define genome as your genome name eg) PAWR_GF05W03
mkdir /project/j/jweir/0_MAKER/"MGWA_VELO045" #make a folder for yourself
cd /project/j/jweir/0_MAKER/"MGWA_VELO045"
###OUTPUT: We will be running maker each in a separate folder within a shared project directory. Make yourself a folder and cd into it

#change USER0 to be your Weir lab username
scp USER6@142.1.98.20:~/temp/maker/* . #transfers all files in this folder onto the supercomputer
####OUTPUT: gather everything from your maker folder on the server and place it into this folder. You can use Filezilla if you prefer, or use this scp command


###SHOULD HAVE FOLLOWING FILES IN CURRENT DIRECTORY:niki03@nia-login07:/project/j/jweir/0_MAKER/MGWA_VELO045
maker_opts.ctl
maker_bopts.ctl (with an s in bopts, so, not maker_bopt.ctl)
maker_exe.ctl
"$GENOME"_input.fasta
"$GENOME"reps_input.fasta
*_proteomes.no_tes.fa (whichever is correct for you)
*_transcriptome.fa (whichever is correct for you)

#We will split the genome into chunks to run on each chunk separately.
module load NiaEnv
module load CCEnv
module load StdEnv
module load intel/2019.3
module load nixpkgs/16.09
module load gcccore/.8.3.0
module load gcc/7.3.0
module load perl/5.22.4
module load bioperl/1.7.1 #1.7.5
module load exonerate/2.4.0
module load trf/4.09
module load rmblast/2.9.0
module load blast+/2.7.1
module load openmpi/3.1.2
export PATH=$PATH:/gpfs/fs0/project/j/jweir/tools/maker/bin

#Now, split the genome into chunks:
fasta_tool --chunk 100 "MGWA_VELO045"_input.fasta #this tool comes with maker, in the folder maker/bin.

###OUTPUT: ###activate some modules, which will load programs on Niagara. Then we will use a program within maker that fragments the genome for you. You may get a couple warnings when loading modules which is usually fine.

###You will need to check how many chunks got created. Use ls to see how many chunks there are. They should start at MGWA_VELO045_input_000.fasta and go up in number. Check what the highest number is as you will need to tell maker what chunks to look for.
###HOW MANY CHUNKS? 64 CHUNKS 
###HIGHEST CHUNKS? MGWA_VELO045_input_000.fasta TO MGWA_VELO045_input_063.fasta

###main server, browse to the folder /home/0_BIOD98_GENOMES1/Submissionscript_templates/ and open up the file named 0_Step1_MakeMaker.txt

###Scroll down to the bottom. There are a few things that you need to change.
Firstly, in the variable SPECIES, change Rhemel to be the actual name of your genome - but keep the word _input. For example, it should look something like this: SPECIES = "PAWR_GE04D01_input"

####CHANGED TO: SPECIES = "MGWA_VELO045_input"

###change NIAGARA_WD to be the path to your folder on Niagara - change the word GENOME to match your actual genome name. For example it should look something like this: NIAGARA_WD = "/project/j/jweir/0_MAKER/GENOME"

###CHANGED TO: NIAGARA_WD = "/project/j/jweir/0_MAKER/MGWA_VELO045"

###change NUM_SUMISSION_SCRIPTS. Each submission script contains 4 chunks, therefore the scripts that you need is (# chunks)/4. REMEMBER THAT THE NAMES OF THE CHUNKS START AT ZERO therefore if the highest number you had in the names of your chunks was 63, then you actually have 64 chunks, and so you need 64/4 = 16. If you get a decimal number, round UP to the nearest whole number.

###NUMBER OF CHUNKS= 64 
###NUMBER OF SCRIPTS= 64/4= 16
###CHANGED TO: NUM_SUMISSION_SCRIPTS = 16

###change LOCAL_WD to be the path to your own directory on the main server, so that you do not accidentally overwrite each other's files. Make sure that you change 0_BIOD98_GENOMES1 to 0_BIOD98_GENOMES2 if you need to. It should look something like this in the end: LOCAL_WD = "/home/0_BIOD98_GENOMES2/PAWR_GE04D01"

###CHANGED TO: LOCAL_WD = "/home/0_BIOD98_GENOMES2/MGWA_VELO045"

###Once that is done, save the file into your own folder (for example "/home/0_BIOD98_GENOMES2/PAWR_GE04D01"). You will also need to copy the file 0_MAKER_BATCHFILE_NIAGARA_40threads.sh into that folder too

##Open a new terminal
##Copy-paste the entire contents of the 0_Step1_MakeMaker.txt file that you edited. This will run the script as an R code and generate a series of files in your folder.
##As soon as you run it, you can go into your folder and look at the results.
##Open the first file (ends in run_1.sh) and scroll to the bottom
##ou should see 4 maker commands in brackets. Make sure that the name of your genome looks correct and includes the word _input at the end of it.
##Make sure that there are 4 chunks, where the first is 000, then 001, 002, and 003.
##If that is good then open the last file and make sure that the name of that chunk matches the name of your last chunk.
##If you had rounded up earlier, when you chose how many submission scripts to use, then place a # symbol at the start of the line for the extra chunks that you don't need: for example, if your highest chunk number was 082 but your last script shows a line for chunk 083 and 084 as well, place a # symbol at the very start of those two lines so that they will be ignored and Maker will not be confused looking for files that don't exist.

###OUTPUT: Once you made those scripts, trasfer them to Niagara using Filezilla or this scp command:
#go back to your Terminal where you are logged into Niagara (or log back in again)

cd /project/j/jweir/0_MAKER/"MGWA_VELO045" #make sure GENOME was defined
#Change USER0 to your Weir lab username and change 0_BIOD98_GENOMES1 to 0_BIOD98_GENOMES2 if needed:

scp USER6@142.1.98.20:/home/0_BIOD98_GENOMES2/"MGWA_VELO045"/0_MAKER_*run_*.sh . #transfers all files in this folder onto the supercomputer

###BEFORE YOU SUBMIT: note that the number of submission scripts you have will vary. Make sure that you only submit the number of scripts that you have (ie if you only have up to *__run20 Then do not do sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_21.sh

##have till _run16
##do till sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_16.sh

sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_1.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_2.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_3.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_4.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_5.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_6.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_7.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_8.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_9.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_10.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_11.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_12.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_13.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_14.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_15.sh
sbatch 0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_16.sh

###OUTPUT: To check the status of the submission, you can do
squeue --user niki03 #where USERNAME is your Compute Canada Username

## MAKER 2

MAKER1 OUTPUT:
#FAILED CONTIGS=0
#SKIPPED CONTIGS (too small)= 42372
#STARTED CONTIGS= 3205
#FINISHED CONTIGS= 3226
#TOTAL CONTIGS=48805
#total number of scaffolds in original fasta file= 45609
#Count scaffolds of the many input chunks - should add to the same number as the original above = 45609
#count number of unique contigs in the log (contigs that actually ran in the maker pipeline) = 45601

time /opt/tools/maker/bin/gff3_merge -s -d ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log> MGWA_VELO045_rnd1.all.maker.gff
OUTPUT: 
real	1m38.418s
user	0m46.685s
sys	0m15.172s

time /opt/tools/maker/bin/fasta_merge -d ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log
OUTPUT:
real	0m45.127s
user	0m5.990s
sys	0m4.218s

time /opt/tools/maker/bin/gff3_merge -n -s -d ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log > MGWA_VELO045_rnd1.all.maker.noseq.gff
OUTPUT:
real	0m30.462s
user	0m24.337s
sys	0m7.032s

cd ~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/snap/round1
/opt/tools/maker/bin/maker2zff -x 0.25 -l 50 -c 0 -e 0 -o 0.5 -d ../../MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log

## TRAIN SNAP ROUND 1:
#cd into your MGWA_VELO045_maker_R1/MGWA_VELO045 folder if you are not there already
#prepare to train SNAP

mkdir snap
mkdir snap/round1
cd ~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/snap/round1

#convert the gff file to zff, while only including the most confident gene models

/opt/tools/maker/bin/maker2zff -x 0.25 -l 50 -c 0 -e 0 -o 0.5 -d ../../MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log

rename 's/genome/MGWA_VELO045_rnd1.zff.length50_aed0.25/g' *

######rename 's/genome/'MGWA_VELO045'_rnd1.zff.length50_aed0.25/g' *

#create statistics
/opt/tools/snap/fathom MGWA_VELO045_rnd1.zff.length50_aed0.25.ann MGWA_VELO045_rnd1.zff.length50_aed0.25.dna -gene-stats > gene-stats.log 2>&1

 /opt/tools/snap/fathom MGWA_VELO045_rnd1.zff.length50_aed0.25.ann MGWA_VELO045_rnd1.zff.length50_aed0.25.dna -validate > validate.log 2>&1
 
 #read the log files
cat *.log #make sure that it does not say any errors, such as not being able to find a file

#create the HMM file that SNAP requires for training, with the sequences and annotations with 1kb surrounding sequences
/opt/tools/snap/fathom -categorize 1000 MGWA_VELO045_rnd1.zff.length50_aed0.25.ann MGWA_VELO045_rnd1.zff.length50_aed0.25.dna > categorize.log 2>&1
/opt/tools/snap/fathom -export 1000 -plus uni.ann uni.dna > uni-plus.log 2>&1

#training parameters
mkdir params
cd params
/opt/tools/snap/forge ../export.ann ../export.dna > ../forge.log 2>&1
cd ..

#make the HMM file
time /opt/tools/snap/hmm-assembler.pl MGWA_VELO045_rnd1.zff.length50_aed0.25 params > MGWA_VELO045_rnd1.zff.length50_aed0.25.hmm

#read the logs. Any errors?
cat *.log
##NO ERRORS

### RUN MAKER 2

#cd into your "$GENOME"_maker_R1/"$GENOME" folder if you are not there already

#RNA 
awk '{ if ($2 == "est2genome") print $0 }' MGWA_VELO045_rnd1.all.maker.noseq.gff > MGWA_VELO045_rnd1.all.maker.est2genome.gff
#protein
awk '{ if ($2 == "protein2genome") print $0 }' MGWA_VELO045_rnd1.all.maker.noseq.gff > MGWA_VELO045_rnd1.all.maker.protein2genome.gff
#repeats
awk '{ if ($2 ~ "repeat") print $0 }' MGWA_VELO045_rnd1.all.maker.noseq.gff > MGWA_VELO045_rnd1.all.maker.repeats.gff

##Now go back to maker and change these lines in the options file:
cd /gpfs/fs0/scratch/j/jweir/niki03/Maker1

#ssh into your account on Niagara and cd into your maker folder like last time
sed -i 's/snaphmm=/snaphmm=snap\/round1\/'MGWA_VELO045'_rnd1.zff.length50_aed0.25.hmm/g' maker_opts.ctl
sed -i 's/est2genome=1/est2genome=0/g' maker_opts.ctl
sed -i 's/protein2genome=1/protein2genome=0/g' maker_opts.ctl
sed -i 's/altest_gff=/altest_gff='MGWA_VELO045'_rnd1.all.maker.est2genome.gff/g' maker_opts.ctl
sed -i 's/protein_gff=/protein_gff='MGWA_VELO045'_rnd1.all.maker.protein2genome.gff/g' maker_opts.ctl
sed -i 's/altest='MGWA_VELO045'RNA_input.fa/altest=/g' maker_opts.ctl
sed -i 's/protein='MGWA_VELO045'prot_input.fasta/protein=/g' maker_opts.ctl
sed -i 's/model_org=all/model_org=/g' maker_opts.ctl
sed -i 's/rmlib='MGWA_VELO045'reps_input.fasta/rmlib=/g' maker_opts.ctl
#the line of code below was what the instructions original stated which is incorrect.
#sed -i 's/repeat_protein=all/repeat_protein=/g' maker_opts.ctl
#the next line is the correct line of code:
sed -i 's;repeat_protein=/gpfs/fs0/project/j/jweir/tools/maker/data/te_proteins.fasta;repeat_protein=;g' maker_opts.ctl 
sed -i 's/rm_gff=/rm_gff='MGWA_VELO045'_rnd1.all.maker.repeats.gff/g' maker_opts.ctl

###Take a look at the file
less maker_opts.ctl #press q to go back

###These are the changes that should appear:
model_org= #select a model organism for RepBase masking in RepeatMasker
rmlib= #provide an organism specific repeat library in fasta format for RepeatMasker
repeat_protein= #provide a fasta file of transposable element proteins for RepeatRunner
rm_gff=$GENOME_rnd1.all.maker.repeats.gff

snaphmm=snap/round1/"$GENOME"_rnd1.zff.length50_aed0.25.hmm  
est2genome=0  
protein2genome=0  
altest_gff="$GENOME"_rnd1.all.maker.est2genome.gff  
protein_gff="$GENOME"_rnd1.all.maker.protein2genome.gff  

altest= #EST/cDNA sequence file in fasta format from an alternate organism
protein= #protein sequence file in fasta format (i.e. from mutiple oransisms)

###CHANGES OBTAINED 

###Get the snap files. Copy SNAP files from ramphoceles to Main server to Niagara
##FROM RAMPHOCELES TO MAIN SERVER:
##scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/snap/round1/MGWA_VELO045* .
##scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/snap/round1/export* .
##scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/snap/round1/alt* .
##scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/snap/round1/err* .
##scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/snap/round1/forge* .
##scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/snap/round1/gene-stats* .
##scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/snap/round1/olp* .
##scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/snap/round1/uni* .
##scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/snap/round1/validate* .
##scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/snap/round1/wrn* .

scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/MGWA_VELO045_rnd1* .
scp -r $USER@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1/MGWA_VELO045_input.all* .

##FROM MAIN SERVER TO NIAGARA:

##scp -r USER6@142.1.98.20:/home/0_BIOD98_GENOMES2/MGWA_VELO045/MGWA_VELO045* .
##scp -r USER6@142.1.98.20:/home/0_BIOD98_GENOMES2/MGWA_VELO045/export* .
##scp -r USER6@142.1.98.20:~/home/0_BIOD98_GENOMES2/MGWA_VELO045/alt* .
##scp -r USER6@142.1.98.20:~/home/0_BIOD98_GENOMES2/MGWA_VELO045/err* .
##scp -r USER6@142.1.98.20:~/home/0_BIOD98_GENOMES2/MGWA_VELO045/forge* .
##scp -r USER6@142.1.98.20:~/home/0_BIOD98_GENOMES2/MGWA_VELO045/gene-stats* .
##scp -r USER6@142.1.98.20:~/home/0_BIOD98_GENOMES2/MGWA_VELO045/olp* .
##scp -r USER6@142.1.98.20:~/home/0_BIOD98_GENOMES2/MGWA_VELO045/uni* .
##scp -r USER6@142.1.98.20:~/home/0_BIOD98_GENOMES2/MGWA_VELO045/validate* .
##scp -r USER6@142.1.98.20:~/home/0_BIOD98_GENOMES2/MGWA_VELO045/wrn* .

scp -r USER6@142.1.98.20:/home/0_BIOD98_GENOMES2/MGWA_VELO045/MGWA_VELO045_rnd1* .
scp -r USER6@142.1.98.20:/home/0_BIOD98_GENOMES2/MGWA_VELO045/MGWA_VELO045_input.all* .

find *SUBMISSION*_run_*.sh > tempfile
cat tempfile | while read line ; do sed 's/OUTPUT/OUTPUT_R2/g' "$line" | sed 's/23:59:00/18:00:00/g' > R2_"$line" ; done
chmod +x R2_*

###identified 16 submission_run_.sh files

#rename old output folder so that it cannot interfere
mv MGWA_VELO045_input.maker.output MGWA_VELO045_input.maker.output_round1

###When that is all ready, run Maker round 2! Run the Submission Scripts
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_1.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_2.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_3.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_4.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_5.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_6.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_7.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_8.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_9.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_10.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_11.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_12.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_13.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_14.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_15.sh
sbatch R2_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_16.sh

### MAKER 2 COMPLETE!!
### MAKER 3 STARTS NOW!!

###Please do these steps on Ramphocelus as maker is not accessible on the main server.
cd USER6@ramphocelus:~/MGWA_VELO045
mkdir ./MGWA_VELO045_maker_R2
cd MGWA_VELO045_maker_R2
scp niki03@niagara.scinet.utoronto.ca:/gpfs/fs0/scratch/j/jweir/niki03/Maker1/maker_*.ctl .
scp niki03@niagara.scinet.utoronto.ca:/gpfs/fs0/scratch/j/jweir/niki03/Maker1/OUTPUT* .
scp -r niki03@niagara.scinet.utoronto.ca:/gpfs/fs0/scratch/j/jweir/niki03/Maker1/MGWA_VELO045_input.maker.output .

#Activate the correct perl environment:

#only works on Ramphocelus:
export PERLBREW_ROOT=/opt/perl5 #
/opt/perl5/bin/perlbrew switch perl-5.30.0 #A sub-shell is launched with perl-5.30.0 as the activated perl. Run 'exit' to finish it.
export PATH=/opt/tools/maker/bin:$PATH

#you will have to redefine GENOME at this point as the new perl environment forgets.

#cd into your MGWA_VELO045_maker_R2 if you are not there already
#check that all contigs finished
less -S ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log
#scroll through to take a look at what the output looks like, then press q to go back to Terminal

#check if there are any that failed and count how many were skipped vs finished
grep -c "SKIPPED_SMALL" ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log #number of scaffolds that were skipped becasue they were less than 10,000 bp long
grep -c "STARTED" ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log #number of scaffolds that started being analyzed
grep -c "FINISHED" ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log #number of scaffolds that finished being analyzed
grep -c "DIED_SKIPPED_PERMANENT" ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log #number that failed - should be none!
wc -l ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log #just counting lines to make sure the preceeding numbers add up

#SKIPPED_SMALL = 42380
#STARTED=3229
#FINISHED=3229
#DIED_SKIPPED=0
#TOTAL_LINES= 48838

#make sure that all of the contigs were run
#refer back to your results of the last run to remember how many scaffold total should be in your original data
cut -f 1 ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log | sort | uniq | wc -l #count number of unique contigs in the log (contigs that actually ran in the maker pipeline)

#UNIQUE CONTIGS= 45609

#Create files merging all data into a fasta and GFF3 file

time /opt/tools/maker/bin/gff3_merge -s -d ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log> MGWA_VELO045_rnd2.all.maker.gff

OUTPUT:
real	2m8.509s
user	0m54.586s
sys	0m15.502s

time /opt/tools/maker/bin/fasta_merge -d ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log

OUTPUT:
real	7m2.501s
user	0m22.314s
sys	0m11.171s

#Create a gff3 file without sequences
time /opt/tools/maker/bin/gff3_merge -n -s -d ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log > MGWA_VELO045_rnd2.all.maker.noseq.gff

OUTPUT:
real	0m15.387s
user	0m12.204s
sys	0m3.718s

### TRAIN SNAP: MAKER 3

#cd into your MGWA_VELO045_maker_R2 if you are not there already
#prepare to train SNAP
mkdir snap
mkdir snap/round2
cd snap/round2

#convert the gff file to zff, while only including the most confident gene models
/opt/tools/maker/bin/maker2zff -x 0.25 -l 50 -c 0 -e 0 -o 0.5 -d ../../MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log

rename 's/genome/'MGWA_VELO045'_rnd2.zff.length50_aed0.25/g' *

#create statistics
/opt/tools/snap/fathom MGWA_VELO045_rnd2.zff.length50_aed0.25.ann MGWA_VELO045_rnd2.zff.length50_aed0.25.dna -gene-stats > gene-stats.log 2>&1

/opt/tools/snap/fathom MGWA_VELO045_rnd2.zff.length50_aed0.25.ann MGWA_VELO045_rnd2.zff.length50_aed0.25.dna -validate > validate.log 2>&1

#create the HMM file that SNAP requires for training, with the sequences and annotations with 1kb surrounding sequences

/opt/tools/snap/fathom -categorize 1000 MGWA_VELO045_rnd2.zff.length50_aed0.25.ann MGWA_VELO045_rnd2.zff.length50_aed0.25.dna > categorize.log 2>&1

/opt/tools/snap/fathom -export 1000 -plus uni.ann uni.dna > uni-plus.log 2>&1

#training parameters
mkdir params
cd params
/opt/tools/snap/forge ../export.ann ../export.dna > ../forge.log 2>&1
cd ..

#make the HMM file

time /opt/tools/snap/hmm-assembler.pl MGWA_VELO045_rnd2.zff.length50_aed0.25 params > MGWA_VELO045_rnd2.zff.length50_aed0.25.hmm

OUTPUT:
real	0m0.037s
user	0m0.029s
sys	0m0.007s

### RUN MAKER ROUND 3:
#Now go back to your maker folder on Niagara and change these lines in the options file:

cd /gpfs/fs0/scratch/j/jweir/niki03/Maker1

sed -i 's/snaphmm=snap\/round1\/'MGWA_VELO045'_rnd1.zff.length50_aed0.25.hmm/snaphmm=snap\/round2\/'MGWA_VELO045'_rnd2.zff.length50_aed0.25.hmm/g' maker_opts.ctl #updates SNAP to use the new training file

sed -i 's/augustus_species=/augustus_species=chicken/g' maker_opts.ctl #activates augustus

sed -i 's/keep_preds=0/keep_preds=1/g' maker_opts.ctl 

#Take a look at the file: These are the changes that should appear:
less maker_opts.ctl #press q to go back

#new settings (on top of the round 2 settings)
snaphmm=snap/round2/MGWA_VELO045_rnd2.zff.length50_aed0.25.hmm  
augustus_species=chicken
keep_preds=1

##CHANGES OBTAINED 

##Get the snap files.
##copy the files somewhere in the main server
copy the files from the main server to Niagara (change the path and the IP address to reflect this: the main server is 142.1.98.20)

#move the onld snap and maker folder out of the way
mv snap snap_R1

mv MGWA_VELO045_input.maker.output MGWA_VELO045_input.maker.output_round2

#You will need to specify the path to your folder if it is different, and your username.
#MOVE SNAP FILES FROM RAMPHOCELES TO MAIN SERVER
scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R2/snap .

#MOVE SNAP FILES FROM MAIN SERVER TO NIAGARA
scp -r USER6@142.1.98.20:/home/0_BIOD98_GENOMES2/MGWA_VELO045/snap . 

find *SUBMISSION*_run_*.sh > tempfile
cat tempfile | while read line ; do sed 's/OUTPUT/OUTPUT_R3/g' "$line" | sed 's/23:59:00/18:00:00/g' > R3_"$line" ; done
chmod +x R3_*

###RUNNING MAKER ROUND 3 SUBMISSION SCRIPTS:


sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_1.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_2.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_3.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_4.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_5.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_6.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_7.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_8.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_9.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_10.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_11.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_12.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_13.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_14.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_15.sh
sbatch R3_0_MAKER_SUBMISSION_SCRIPT_NIAGARA_40threads__run_16.sh

### STEP 3: FUNCTIONAL ANNOTATION:

cd /gpfs/fs0/scratch/j/jweir/niki03/Maker1

#First, check if all of your rounds finished
tail -n 50 OUTPUT_R3*

### MAKER ROUND 3 HAS COMPLETED WITHOUT ANY ERRORS. 

#GET YOUR DATA OFF OF NIAGARA TO RAMPHOCELES NODE:
 cd ~/MGWA_VELO045
 
 #get all contents for round 2 off of Niagara
mkdir ./MGWA_VELO045_maker_R3
cd ./MGWA_VELO045_maker_R3

#Get your control files, output logs, and maker output folder into your new folder.
scp niki03@niagara.scinet.utoronto.ca:/gpfs/fs0/scratch/j/jweir/niki03/Maker1/maker_*.ctl . #or use Filezilla if you prefer
scp niki03@niagara.scinet.utoronto.ca:/gpfs/fs0/scratch/j/jweir/niki03/Maker1/OUTPUT_R3* . #or use Filezilla if you prefer
scp -r niki03@niagara.scinet.utoronto.ca:/gpfs/fs0/scratch/j/jweir/niki03/Maker1/MGWA_VELO045_input.maker.output . #or use Filezilla if you prefer
#It will take a long time to download everything, the files are huge

#Enter the folder with your maker output if you are not already there
cd ~/MGWA_VELO045/MGWA_VELO045_maker_R3

#Activate the correct perl environment: only works on Ramphocelus:
export PERLBREW_ROOT=/opt/perl5 
/opt/perl5/bin/perlbrew switch perl-5.30.0 #A sub-shell is launched with perl-5.30.0 as the activated perl. Run 'exit' to finish it.
export PATH=/opt/tools/maker/bin:$PATH

#check that all contigs finished
less -S ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log
#scroll through to take a look at what the output looks like, then press q to go back to Terminal

#check if there are any that failed and count how many were skipped vs finished
grep -c "SKIPPED_SMALL" ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log #number of scaffolds that were skipped becasue they were less than 10,000 bp long
grep -c "STARTED" ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log #number of scaffolds that started being analyzed
grep -c "FINISHED" ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log #number of scaffolds that finished being analyzed
grep -c "DIED_SKIPPED_PERMANENT" ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log #number that failed - should be none!
wc -l ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log #just counting lines to make sure that all the preceeding numbers add up.

#SKIPPED_SMALL=42377
#STARTED_SMALL=3227
#FINSHED=3229
#DIED_SKIPPED_PERMANENT=0
#TOTAL NUMBER OF LINES=48833

#make sure that all of the contigs were run
#refer back to your results of the last run to remember how many scaffold total should be in your original data
#count number of unique contigs in the log (contigs that actually ran in the maker pipeline)

cut -f 1 ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log | sort | uniq | wc -l 

#UNIQUE CONTIGS=45606

#Create files merging all data into a fasta and GFF3 file

time /opt/tools/maker/bin/gff3_merge -s -d ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log> MGWA_VELO045_rnd3.all.maker.gff

OUTPUT:
real	1m15.304s
user	0m57.705s
sys	0m16.462s

time /opt/tools/maker/bin/fasta_merge -d ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log

OUTPUT:
real	0m24.791s
user	0m14.903s
sys	0m9.090s

#Create a gff3 file without sequences

time /opt/tools/maker/bin/gff3_merge -n -s -d ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log > MGWA_VELO045_rnd3.all.maker.noseq.gff

OUTPUT:
real	0m28.602s
user	0m23.000s
sys	0m5.844s
#done

### CHECKPOINT:

Before continuing:

1. did you verify that all of the contigs ran? The number of run contigs should be exactly the same as the number of input contigs. If it is even off by one, then there is a problem - email Else/Jason.
2. is the number of contigs that said "DIED_SKIPPED_PERMANENT" ZERO? There should be no contigs that died. If there were any, email Else/Jason.

Let's verify the maker run with this script:

#Enter the folder with your maker output if you are not already there
cd MGWA_VELO045_maker_R3

#only works on Ramphocelus:
export PERLBREW_ROOT=/opt/perl5 #

/opt/perl5/bin/perlbrew switch perl-5.30.0 #A sub-shell is launched with perl-5.30.0 as the activated perl. Run 'exit' to finish it.

export PATH=/opt/tools/maker/bin:$PATH

time /home/0_PROGRAMS/GAAS/annotation/Tools/Maker/maker_check_progress.sh ./MGWA_VELO045_input.maker.output/MGWA_VELO045_input_master_datastore_index.log

## OUTPUT:

42377 contigs are too small to be analyzed

3227 contigs has begin to be studied only one time
0 contigs have been started several times
3227 contigs has begin to be studied in total
3227 STARTED signal in total

3229 contigs have been finished
0 contigs have been finished several times

0 DIED_SKIPPED_PERMANENT signal found. The number of retry attempts has been reached
0 contigs (doublon removed) have the signal DIED_SKIPPED_PERMANENT. The number of retry attempts has been reached

0 contig whithout FINISHED signal.

2 contig whithout STARTED signal.


###Finally, 0 errors has been found in the log file:(See below)###


### Now, verification of errors found in the directory of these ananlysis ###


This job does not contains bug ! Congratulation.

Bye

real	0m0.674s
user	0m0.660s
sys	0m0.029s

##EVALUATE COMPLETENESS USING BUSCO:
##GOAL: BUSCO gives an estimate of how "complete" an annotation is by looking for specific highly-conserved proteins that are expected to be found as a single copy in almost every bird genome. Here we will collect some stats on our annotation an dthen run BUSCO.

##NOTE: Note: more proteins is not necessarily better. In general, I am expecting round 1 to have the fewest, round 2 should find slightly more by using SNAP, and round 3 should find way more because we used the keep preds option that tells maker to keep all possible proteins no matter how bad they are. These keeps a LOT of false positives. We will have to get rid of these.

cd MGWA_VELO045

#Count number of proteins found in Maker Round1

cat MGWA_VELO045_maker_R1/Maker1/MGWA_VELO045_rnd1.all.maker.gff | awk '{ if ($3 == "gene") print $0 }' | awk '{ sum += ($5 - $4) } END { print NR, sum / NR }'

OUTPUT:
15445 10032.3

#IF THE ABOVE COMMAND DID NOT WORK: your path may be different, try this next line instead (remove the "#" at the beginning to run it if the above didnt work):
#cat MGWA_VELO045_maker_R1/MGWA_VELO045/MGWA_VELO045_rnd1.all.maker.gff | awk '{ if ($3 == "gene") print $0 }' | awk '{ sum += ($5 - $4) } END { print NR, sum / NR }'

#Count number of proteins found in Maker Round2

cat MGWA_VELO045_maker_R2/MGWA_VELO045_rnd2.all.maker.gff | awk '{ if ($3 == "gene") print $0 }' | awk '{ sum += ($5 - $4) } END { print NR, sum / NR }'

OUTPUT:
21778 15814.6

#Count number of proteins found in Maker Round3

cat MGWA_VELO045_maker_R3/MGWA_VELO045_rnd3.all.maker.gff | awk '{ if ($3 == "gene") print $0 }' | awk '{ sum += ($5 - $4) } END { print NR, sum / NR }'

OUTPUT:
61957 21706.8

### You should have around 40,000-50,000 genes in your round 3, and around 15,000-19,000 in round 2 probably and probably 10,000-17,000 in round 1. If you have way way more or way way fewer there could be a problem.

### The average lengths should also be around 10,000-35,000 bp

### Note: these statistics would be good to discuss in your report. Did training SNAP allow you to find more proteins in round 2 than in round 1? Did the average length of genes change?

### Now for Busco completeness scores!

### To run BUSCO, you need a conda environment that contains Augustus.

#do this only once

conda deactivate
conda create -n Augustus
conda activate Augustus
conda install -c bioconda augustus
cd /home/0_PROGRAMS/busco_v3
python setup.py install --user

cd ~/MGWA_VELO045
mkdir busco
cd busco

#set up environment
conda activate Augustus #has augustus installed
export BUSCO_CONFIG_FILE="/home/0_BIOD98/anteater_busco_config.ini"

# Run BUSCO!
#assess transcripts
time python3 /home/0_PROGRAMS/busco_v3/scripts/run_BUSCO.py -i ../MGWA_VELO045_maker_R3/MGWA_VELO045_input.all.maker.transcripts.fasta  -o transcript_annotation_eval -l /home/0_PROGRAMS/busco_v3/aves_odb9 -m transcriptome -c 24 -sp chicken -z --augustus_parameters='--progress=true'

OUTPUT:
real	67m24.990s
user	450m40.210s
sys	5m57.825s

#assess proteins
time python3 /home/0_PROGRAMS/busco_v3/scripts/run_BUSCO.py -i ../MGWA_VELO045_maker_R3/MGWA_VELO045_input.all.maker.proteins.fasta  -o protein_annotation_eval -l /home/0_PROGRAMS/busco_v3/aves_odb9 -m proteins -c 24 -sp chicken -z --augustus_parameters='--progress=true'

OUTPUT:
real	11m28.152s
user	102m32.956s
sys	1m36.915s

## RUNNING INTERPROSCAN:

##GOAL:what are these genes? What do they do? One of the main ways we can talk about gene functions in biology is by using GO terms. GO terms (Gene Ontology) are standardized traits that have to do with a biological process (eg cell division), location (eg mitochondrion), or function (eg transport).

##Each GO term is unique, and assigned a number.

##The first thing we will do is look for conserved protein domains using InterPro Scan, which can search several different databases to each of our protein sequences and give a list of matches. These will be used to assign possible functions to the genes, including GO terms. The assumption is that if our mystery protein has functional domains that are shared with a group of known proteins, it probably has the same function as those known proteins. 

##We will compare out genes to the following database:

#set up environment
export PATH=$PATH:/home/0_PROGRAMS/interproscan/interproscan-5.39-77.0 #for all of interproscan
export PATH=$PATH:/opt/tools/maker/bin
export PERLBREW_ROOT=/opt/perl5
/opt/perl5/bin/perlbrew switch perl-5.30.0 #A sub-shell is launched with perl-5.30.0 as the activated perl. Run 'exit' to finish it.

export PATH=/opt/tools/maker/bin:$PATH

#Now you will have to redefine GENOME as it will have forgotten

#make a directory to work in
mkdir ~/MGWA_VELO045/functional_ann
cd ~/MGWA_VELO045/functional_ann

#search for hits in the two databases with Interproscan

#Pfam
interproscan.sh -iprlookup -goterms -appl Pfam-32.0 -i ~/MGWA_VELO045/MGWA_VELO045_maker_R3/MGWA_VELO045_input.all.maker.proteins.fasta -b MGWA_VELO045_prots -f tsv

# OUTPUT: 
18/03/2020 02:56:30:003 Welcome to InterProScan-5.39-77.0
18/03/2020 02:56:30:005 Running InterProScan v5 in STANDALONE mode... on Linux
18/03/2020 02:57:26:202 Loading file /home/USER6/MGWA_VELO045/MGWA_VELO045_maker_R3/MGWA_VELO045_input.all.maker.proteins.fasta
18/03/2020 02:57:26:211 Running the following analyses:
[Pfam-32.0]
Available matches will be retrieved from the pre-calculated match lookup service.

Matches for any sequences that are not represented in the lookup service will be calculated locally.
18/03/2020 03:00:55:011 37% completed
18/03/2020 03:01:00:496 62% completed
18/03/2020 03:01:02:606 87% completed
18/03/2020 03:47:36:850 90% completed
18/03/2020 03:49:30:931 100% done:  InterProScan analyses completed

#concatenate results
cat MGWA_VELO045_prots*.tsv > MGWA_VELO045_prots_2interpro.tsv

#add the interpro results to the maker gff
ipr_update_gff ~/MGWA_VELO045/MGWA_VELO045_maker_R3/MGWA_VELO045_rnd3.all.maker.gff MGWA_VELO045_prots_2interpro.tsv > MGWA_VELO045.max.functional_2interpro.gff

#filter to retain only best genes, compare two filtering options:
perl /home/0_PROGRAMS/Genome_annotation/quality_filter.pl -d MGWA_VELO045.max.functional_2interpro.gff > MGWA_VELO045.default.functional_2interpro.gff 

#default build: only proteins with RNA/protein evidence (AED<1)

perl /home/0_PROGRAMS/Genome_annotation/quality_filter_PfamPANTHER.pl -s MGWA_VELO045.max.functional_2interpro.gff > MGWA_VELO045.standardplus.functional_2interpro.gff 

#standard build: all proteins with evidence (AED<1) and/or a functional annotation (Pfam or PANTHER)

#count number of genes:

grep -cP '\tgene\t' ~/MGWA_VELO045/MGWA_VELO045_maker_R3/MGWA_VELO045_rnd3.all.maker.gff #number of genes originally

grep -cP '\tgene\t' MGWA_VELO045.default.functional_2interpro.gff #number genes with evidence (protein/RNA alignment)

grep -cP '\tgene\t' MGWA_VELO045.standardplus.functional_2interpro.gff #number genes with evidence OR Pfam

less MGWA_VELO045_prots.tsv | cut -f 1 | sort | uniq | wc -l #number of genes with Pfam annotations

#NUMBER OF GENES ORIGINALLY= 61957
#NUMBER OF GENES WTH EVIDENCE (PROTEIN/RNA ALIGNMENT) = 17939
#NUMBER OF GENES WITH EVIDENCE OR PFAM= 20095
#NUMBER OF GENES WITH PFAM ANNOTATION = 15855

##Pause game! Let's fix the gene names before we get too far - right now they are named very long codenames.

##Quickly simplify the names of our genes to be more meaningful and easier to use.

# create naming table translating old names into new names

maker_map_ids --prefix MGWA_VELO045 --justify 5 MGWA_VELO045.standardplus.functional_2interpro.gff > MGWA_VELO045.standardplus.name.map

# replace names in GFF files

map_gff_ids MGWA_VELO045.standardplus.name.map MGWA_VELO045.standardplus.functional_2interpro.gff

# replace names in FASTA headers
scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R3/MGWA_VELO045_input.all.maker.proteins.fasta .
cp ~/MGWA_VELO045/MGWA_VELO045_maker_R3/MGWA_VELO045_input.all.maker.proteins.fasta .

map_fasta_ids MGWA_VELO045.standardplus.name.map MGWA_VELO045_input.all.maker.proteins.fasta  #overwrites input, careful!

# extract the protein sequences from the fasta file that passed our filters

# Use seqtk to filter a fasta based on a list

cat MGWA_VELO045.standardplus.name.map | cut -f 2 | sort | uniq > MGWA_VELO045.standardplus_genelist #extract list of genes to keep

/home/0_PROGRAMS/seqtk/seqtk subseq MGWA_VELO045_input.all.maker.proteins.fasta MGWA_VELO045.standardplus_genelist > MGWA_VELO045.standardplus.proteins.fasta

#transcripts too
cp ~/MGWA_VELO045/MGWA_VELO045_maker_R3/MGWA_VELO045_input.all.maker.transcripts.fasta .

map_fasta_ids MGWA_VELO045.standardplus.name.map MGWA_VELO045_input.all.maker.transcripts.fasta

/home/0_PROGRAMS/seqtk/seqtk subseq MGWA_VELO045_input.all.maker.transcripts.fasta MGWA_VELO045.standardplus_genelist > MGWA_VELO045.standardplus.transcripts.fasta

## Now, evaluate the proteins and transcripts of our final, filtered dataset with BUSCO. How does it compare? Did removing low-quality protein predictions affect your BUSCO score? It will likely go down slightly, but hopefully not too much.

#check BUSCO

conda activate Augustus #has augustus installed

export BUSCO_CONFIG_FILE="/home/0_BIOD98/anteater_busco_config.ini"

python3 /home/0_PROGRAMS/busco_v3/scripts/run_BUSCO.py -i MGWA_VELO045.standardplus.proteins.fasta  -o MGWA_VELO045.standardplus.proteins -l /home/0_PROGRAMS/busco_v3/aves_odb9 -m proteins -c 22 -sp chicken -z --augustus_parameters='--progress=true'

## How much is missing, is it the same as before filtering?

## Now you have a final set of protein sequences annotated on your genome, with predicted functions. Nice job!

Results:
INFO	C:76.4%[S:75.2%,D:1.2%],F:14.4%,M:9.2%,n:4915
INFO	3758 Complete BUSCOs (C)
INFO	3698 Complete and single-copy BUSCOs (S)
INFO	60 Complete and duplicated BUSCOs (D)
INFO	708 Fragmented BUSCOs (F)
INFO	449 Missing BUSCOs (M)
INFO	4915 Total BUSCO groups searched
INFO	BUSCO analysis done with WARNING(s). Total running time: 198.6177785396576 seconds
INFO	Results written in /home/USER6/MGWA_VELO045/functional_ann/run_MGWA_VELO045.standardplus.proteins/

## OPTIONAL: RENAME GENES:

##GOALS: rename genes so that instead of a protein being called something difficult to interpret like "CHSK_MKP1592-5563726" they will be named something like "CHSK_MKP1592-5563726: similar to mouse NADH dehydrogenase protein". Much more human readable.

#homology-based annotation, and it relies on having a well curated database of known proteins to compare to.

#Now we will rename our proteins based on homology with other named proteins, especially the curated Swiss-Prot database.

#We will use two databases: Swissprot/Uniprot is highly curated, but has less coverage of birds. Trembl is not maually curated, but has more birds. We will match our proteins to both, and give priority to the Swiss-prot, high quality results.

#BLAST our proteome against this BLAST database to see what our proteins match.

#setup environment

cd ~/MGWA_VELO045/functional_ann
export PATH=$PATH:/home/0_PROGRAMS/ncbi-blast-2.9.0+/bin/

#Swiss-prot

blastp -db /home/0_BIOD98/uniprot_sprot.fasta -query MGWA_VELO045.standardplus.proteins.fasta -out maker2uni.blastp -evalue .000001 -outfmt 6 -num_alignments 1 -seg yes -soft_masking true -lcase_masking -max_hsps 1 -num_threads 23

#TREMBL

blastp -db /home/0_BIOD98/Trembl_birds_Dec2019.fasta -query MGWA_VELO045.standardplus.proteins.fasta -out maker2trem.blastp -evalue .000001 -outfmt 6 -num_alignments 1 -seg yes -soft_masking true -lcase_masking -max_hsps 1 -num_threads 23

#Check results:

grep -c "^>" MGWA_VELO045.standardplus.proteins.fasta #number of input proteins you are starting with
wc -l maker2uni.blastp #counts the number of proteins with a hit in Swissprot/Uniprot
wc -l maker2trem.blastp #counts the number of proteins with a hit in Trembl

#NUMBER OF INPUT PROTEINS YOU ARE STARTING WITH= 20095
#NUMBER OF PROTEINS WITH HIT IN SWISSPROT/UNIPROT= 18185
#NUMBER OF PROTEINS WITH HIT IN TREMBL = 19057

##name the proteins first with the Swiss-prot names, and then with the Trembl names for the few that had hits with Trembl but not Swissprot. 
#That is because many of the Trembl proteins are named uninformative names like "Uncharacterized protein" while the Swissprot proteins are manually curated and higher quality.

cat maker2uni.blastp | cut -f 1 > swissprot_matched #get a list of all the proteins that had a swissprot hit

grep -v -f swissprot_matched maker2trem.blastp  > onlytrembl_matched #get a list of the blast hits for proteins that matched trembl but NOT swissprot

cat maker2uni.blastp onlytrembl_matched > nr_blastp_for_naming #combine the swissprot hits with the trembl hits that swissprot missed

#add the homology results to the FASTA file, replacing the old names

maker_functional_fasta /home/0_BIOD98/Uniprot_cat_Tremblbirds.temp nr_blastp_for_naming MGWA_VELO045.standardplus.proteins.fasta > MGWA_VELO045.standardplus.namedproteins.fasta

#finally, get the transcript data as well and name the same way

cp ~/MGWA_VELO045/MGWA_VELO045_maker_R3/MGWA_VELO045_input.all.maker.transcripts.fasta .

map_fasta_ids MGWA_VELO045.standardplus.name.map MGWA_VELO045_input.all.maker.transcripts.fasta

/home/0_PROGRAMS/seqtk/seqtk subseq MGWA_VELO045_input.all.maker.transcripts.fasta MGWA_VELO045.standardplus_genelist > MGWA_VELO045.standardplus.transcripts.fasta

maker_functional_fasta /home/0_BIOD98/Uniprot_cat_Tremblbirds.temp nr_blastp_for_naming MGWA_VELO045.standardplus.transcripts.fasta > MGWA_VELO045.standardplus.namedtranscripts.fasta

#MORE RESULTS:

wc -l onlytrembl_matched #number of proteins that trembl hit that were missed by Swissprot

#NUMBER OF PROTEINS THAT TREMBL HIT THAT WERE MISSED BY SWISSPROT= 877 onlytrembl_matched

#to see number that swissprot hit that trembl missed, do (#swissprot hit total) - (#trembl hit total) + (number trembl hit that swissprot missed)

#NUMBER OF PROTEINS THAT SWISSPROT HIT THAT TREMBL MISSED= 5

#to see number that had ANY hit:
wc -l nr_blastp_for_naming #can you see an alternate way to calculate this?

#What proportion of your input gene list had a hit in one of these databases?
#NUMBER OF PROTEINS WITH HITS IN AT LEAST 1 OF THEM = 19062 nr_blastp_for_naming = 94.86% HIT

##ELSE' SAMPLE RESULTS:
Results: Example: With Swiss-prot, I got 16,981 hits. With Trembl, I got 17,341 hits (367 that Swissprot did not hit). 17,348 had a hit in a least one of those. I had 17,475 total genes so almost all had hits!!! 99.3%!!!!

  ## ORTHO FINDER 
  
##GOAL: OrthoFinder works by blasting every proteome against every other proteome. For 20 species, that means like 180 combinations. 
##Since OrthoFinder can take a while to run with that many, I (Else) have premade all the BLAST results for these species. 
##All you have to do is add your species (so, like 20 combinations instead of ~200).

#load environment
export PATH=$PATH:/home/0_PROGRAMS/fastme-2.1.5/binaries
export PATH=$PATH:/home/0_PROGRAMS/ #for Diamond
export PATH=$PATH:/home/0_PROGRAMS/OrthoFinder

#First, make a directory to work in
mkdir ~/MGWA_VELO045/orthofinder 
cd ~/MGWA_VELO045/orthofinder 

#get your proteome file here
cp ~/MGWA_VELO045/functional_ann/MGWA_VELO045.standardplus.namedproteins.fasta .

#or, if you skipped the homology part: (remove the hashtag if you need to run it)
#cp ~/"GENOME"/functional_ann/"$GENOME".standardplus.proteins.fasta .

####Important!!!
#If you are comparing your species to another person in this class, you will need to bring their proteome into your folder too!
#Otherwise, skip to run orthofinder.

PARTNERGENOME="MOWA_IF09D02"

##cp /home/USER5/MOWA_IF09D02/functional_ann/MOWA_IF09D02.standardplus.namedproteins.fasta . ##SEEMED TO HAVE SKIPPED 

#or, if they skipped the homology part: (remove the hashtag if you need to run it)
cp /home/USER5/MOWA_IF09D02/functional_ann/MOWA_IF09D02.standardplus.proteins.fasta .

#run OrthoFinder!!!
time /home/0_PROGRAMS/OrthoFinder/orthofinder -n MGWA_VELO045 -b /home/0_BIOD98/OrthoFinder_passerines/WorkingDirectory -f . -t 24 > Orthofinder.MGWA_VELO045.log

#when it is done, check the log. Any errors?

less Orthofinder.MGWA_VELO045.log

### OUTPUT OF ORTHOFINDER:
Writing orthogroups to file
---------------------------
OrthoFinder assigned 481864 genes (98.0% of total) to 18975 orthogroups. Fifty percent of all genes were in orthogroups with 31 or more genes (G50 was 31) and were contained in the largest 5993 orthogroups (O50 was 5993). There were 7781 orthogroups with all species present and 3966 of these consisted entirely of single-copy genes.

2020-03-18 20:30:56 : Done orthogroups

Analysing Orthogroups
=====================

Calculating gene distances
--------------------------
2020-03-18 22:01:45 : Done 11000 of 15505
2020-03-18 22:01:39 : Done 2000 of 15505
2020-03-18 22:01:43 : Done 8000 of 15505
2020-03-18 22:01:42 : Done 7000 of 15505
2020-03-18 22:01:38 : Done 1000 of 15505
2020-03-18 22:01:47 : Done 14000 of 15505
2020-03-18 22:01:41 : Done 4000 of 15505
2020-03-18 22:01:47 : Done 15000 of 15505
2020-03-18 22:01:34 : Done 0 of 15505
2020-03-18 22:01:46 : Done 12000 of 15505
2020-03-18 22:01:40 : Done 3000 of 15505
2020-03-18 22:01:41 : Done 5000 of 15505
2020-03-18 22:01:44 : Done 9000 of 15505
2020-03-18 22:01:44 : Done 10000 of 15505
2020-03-18 22:01:46 : Done 13000 of 15505
2020-03-18 22:01:42 : Done 6000 of 15505
2020-03-18 22:01:33 : Done
Inferring gene and species trees
--------------------------------

7781 trees had all species present and will be used by STAG to infer the species tree

Best outgroup(s) for species tree
---------------------------------
2020-03-18 22:17:16 : Starting STRIDE
2020-03-18 22:17:24 : Done STRIDE
Observed 70 well-supported, non-terminal duplications. 70 support the best roots and 0 contradict them.

Best outgroup(s) for species tree
---------------------------------
2020-03-18 22:17:16 : Starting STRIDE
2020-03-18 22:17:24 : Done STRIDE
Observed 70 well-supported, non-terminal duplications. 70 support the best roots and 0 contradict them.
Best outgroups for species tree:
  longest.simpleheader.Falper.AA, longest.simpleheader.Strhab.AA, longest.simpleheader.Nesnot.AA, longest.simpleheader.Falche.AA, longest.simpleheader.Melund.AA, longest.simpleheader.Acachl.AA, longest.simpleheader.Amacol.AA
  longest.simpleheader.Lepcor.AA, longest.simpleheader.Pipfil.AA, longest.simpleheader.Manvit.AA, longest.simpleheader.Neochr.AA, Willisornis, longest.simpleheader.Coralt.AA, longest.simpleheader.Emptra.AA
  longest.simpleheader.Falper.AA, longest.simpleheader.Strhab.AA, longest.simpleheader.Nesnot.AA, longest.simpleheader.Manvit.AA, longest.simpleheader.Pipfil.AA, longest.simpleheader.Falche.AA, longest.simpleheader.Neochr.AA, longest.simpleheader.Melund.AA, longest.simpleheader.Lepcor.AA, longest.simpleheader.Acachl.AA, Willisornis, longest.simpleheader.Amacol.AA, longest.simpleheader.Coralt.AA, longest.simpleheader.Emptra.AA
  longest.simpleheader.Sercan.AA, longest.simpleheader.Stuvul.AA, longest.simpleheader.Ficalb.AA, longest.simpleheader.Parmaj.AA, longest.simpleheader.Psehum.AA, MOWA_IF09D02.standardplus.proteins, longest.simpleheader.Zonalb.AA, longest.simpleheader.Geofor.AA, MGWA_VELO045.standardplus.namedproteins, longest.simpleheader.Campar.AA, longest.simpleheader.Taegut.AA, longest.simpleheader.Cyacae.AA, longest.simpleheader.Erygou.AA, longest.simpleheader.Lonstr.AA, longest.simpleheader.Junhie.AA
  Willisornis
  longest.simpleheader.Nesnot.AA
  longest.simpleheader.Strhab.AA, longest.simpleheader.Amacol.AA, longest.simpleheader.Melund.AA
  longest.simpleheader.Lepcor.AA, longest.simpleheader.Pipfil.AA, longest.simpleheader.Manvit.AA, longest.simpleheader.Neochr.AA, longest.simpleheader.Coralt.AA, longest.simpleheader.Emptra.AA
  longest.simpleheader.Corbra.AA, longest.simpleheader.Corcor.AA
  longest.simpleheader.Acachl.AA
  longest.simpleheader.Falper.AA, longest.simpleheader.Strhab.AA, longest.simpleheader.Nesnot.AA, longest.simpleheader.Falche.AA, longest.simpleheader.Melund.AA, longest.simpleheader.Amacol.AA
  longest.simpleheader.Strhab.AA, longest.simpleheader.Amacol.AA, longest.simpleheader.Nesnot.AA, longest.simpleheader.Melund.AA
  longest.simpleheader.Falper.AA, longest.simpleheader.Falche.AA

WARNING: Multiple potential species tree roots were identified, only one will be analyed.

Reconciling gene trees and species tree
---------------------------------------
Outgroup: longest.simpleheader.Falper.AA, longest.simpleheader.Strhab.AA, longest.simpleheader.Nesnot.AA, longest.simpleheader.Falche.AA, longest.simpleheader.Melund.AA, longest.simpleheader.Acachl.AA, longest.simpleheader.Amacol.AA
2020-03-18 22:17:24 : Starting Recon and orthologues
2020-03-18 22:17:24 : Starting OF Orthologues
2020-03-18 22:17:53 : Done 0 of 15505
2020-03-18 22:20:06 : Done 1000 of 15505
2020-03-18 22:20:57 : Done 2000 of 15505
2020-03-18 22:21:38 : Done 3000 of 15505
2020-03-18 22:22:18 : Done 4000 of 15505
2020-03-18 22:22:57 : Done 5000 of 15505
2020-03-18 22:23:32 : Done 6000 of 15505
2020-03-18 22:24:08 : Done 7000 of 15505
2020-03-18 22:24:44 : Done 8000 of 15505
2020-03-18 22:25:20 : Done 9000 of 15505
2020-03-18 22:25:54 : Done 10000 of 15505
2020-03-18 22:26:28 : Done 11000 of 15505
2020-03-18 22:26:59 : Done 12000 of 15505
2020-03-18 22:27:24 : Done 13000 of 15505
2020-03-18 22:27:38 : Done 14000 of 15505
2020-03-18 22:27:44 : Done 15000 of 15505
2020-03-18 22:27:46 : Done OF Orthologues
2020-03-18 22:27:49 : Done Recon

Writing results files
=====================
2020-03-18 22:27:50 : Done orthologues

Results:
    /home/0_BIOD98/OrthoFinder_passerines/WorkingDirectory/OrthoFinder/Results_MGWA_VELO045/

CITATION:
 When publishing work that uses OrthoFinder please cite:
 Emms D.M. & Kelly S. (2019), Genome Biology 20:238

 If you use the species tree in your work then please also cite:
 Emms D.M. & Kelly S. (2017), MBE 34(12): 3267-3278
 Emms D.M. & Kelly S. (2018), bioRxiv https://doi.org/10.1101/267914

# Then get your OrthoFinder results into your own folder
mv /home/0_BIOD98/OrthoFinder_passerines/WorkingDirectory/OrthoFinder/Results_MGWA_VELO045 .

## Orthofinder between only two genomes
##GOAL: If you only want to assign orthologs between your TWO sister species without bothering with any others (for example, for synteny analysis), you can assign orthologs like this:

#load environment
export PATH=$PATH:/home/0_PROGRAMS/fastme-2.1.5/binaries
export PATH=$PATH:/home/0_PROGRAMS/ #for Diamond
export PATH=$PATH:/home/0_PROGRAMS/OrthoFinder

#run OrthoFinder!!! It will analyze all the fasta files in your current directory (should be yours and your partner's)
time /home/0_PROGRAMS/OrthoFinder/orthofinder -o ./MGWA_VELO045_sisters -f . -t 24 > Orthofinder.MGWA_VELO045.log

#OUTPUT:
real	327m51.801s
user	761m44.389s
sys	18m31.530s

#when it is done, check the log
less Orthofinder.MGWA_VELO045.log

## OUTPUTS:
Writing orthogroups to file
---------------------------
OrthoFinder assigned 36429 genes (89.3% of total) to 14994 orthogroups. Fifty percent of all genes were in orthogroups with 2 or more genes (G50 was 2) and were contained in the largest 6978 orthogroups (O50 was 6978). There were 14754 orthogroups with all species present and 12684 of these consisted entirely of single-copy genes.

2020-03-19 00:09:10 : Done orthogroups

Analysing Orthogroups
=====================

Calculating gene distances
--------------------------
2020-03-19 00:10:59 : Done 0 of 1274
2020-03-19 00:10:59 : Done 400 of 1274
2020-03-19 00:10:59 : Done 300 of 1274
2020-03-19 00:11:00 : Done 800 of 1274
2020-03-19 00:11:00 : Done 500 of 1274
2020-03-19 00:10:59 : Done 200 of 1274
2020-03-19 00:11:01 : Done 1100 of 1274
2020-03-19 00:11:01 : Done 900 of 1274
2020-03-19 00:11:00 : Done 700 of 1274
2020-03-19 00:10:59 : Done 100 of 1274
2020-03-19 00:11:00 : Done 600 of 1274
2020-03-19 00:11:01 : Done 1200 of 1274
2020-03-19 00:11:01 : Done 1000 of 1274
2020-03-19 00:10:58 : Done
Inferring gene and species trees
--------------------------------

Reconciling gene trees and species tree
---------------------------------------
2020-03-19 00:12:12 : Starting Recon and orthologues
2020-03-19 00:12:12 : Starting OF Orthologues
2020-03-19 00:12:21 : Done 0 of 1274
2020-03-19 00:12:27 : Done 100 of 1274
2020-03-19 00:12:28 : Done 200 of 1274
2020-03-19 00:12:29 : Done 300 of 1274
2020-03-19 00:12:30 : Done 400 of 1274
2020-03-19 00:12:31 : Done 500 of 1274
2020-03-19 00:12:31 : Done 600 of 1274
2020-03-19 00:12:32 : Done 700 of 1274
2020-03-19 00:12:32 : Done 800 of 1274
2020-03-19 00:12:33 : Done 900 of 1274
2020-03-19 00:12:33 : Done 1000 of 1274
2020-03-19 00:12:33 : Done 1100 of 1274
2020-03-19 00:12:34 : Done 1200 of 1274
2020-03-19 00:12:34 : Done OF Orthologues
2020-03-19 00:12:35 : Done Recon

Writing results files
=====================
2020-03-19 00:12:35 : Done orthologues

Results:
    ./MGWA_VELO045_sisters/Results_Mar18/

CITATION:
 When publishing work that uses OrthoFinder please cite:
 Emms D.M. & Kelly S. (2019), Genome Biology 20:238

 If you use the species tree in your work then please also cite:
 Emms D.M. & Kelly S. (2017), MBE 34(12): 3267-3278
 Emms D.M. & Kelly S. (2018), bioRxiv https://doi.org/10.1101/267914


## EXPLORING THE ORTHOFINDER RESULTS:

cd Results_MGWA_VELO045

ls #what is here?
Citation.txt			 Gene_Trees   Orthogroup_Sequences		Putative_Xenologs		  Species_Tree
Comparative_Genomics_Statistics  Log.txt      Orthologues			Resolved_Gene_Trees		  WorkingDirectory
Gene_Duplication_Events		 Orthogroups  Phylogenetically_Misplaced_Genes	Single_Copy_Orthologue_Sequences

#check the log to make sure it worked properly
less Log.txt #press q to go back

cd WorkingDirectory
ls #This is where it stores BLAST results and raw data, which it could reuse if you had to run the same thing again 

cd ..
cd Orthogroups

less Orthogroups.GeneCount.tsv #press q to go back
#Which OrthoGroup has the most genes?
#Which species has more of that gene family?

less Orthogroups_SingleCopyOrthologues.txt #press q to go back
cd ../Single_Copy_Orthologue_Sequences
ls
#pick one copy
#use less to see the contents


cd ../Resolved_Gene_Trees
ls
#pick one copy
#use less to see the contents
#These are phylogenetic trees. If you found a gene family of interest, you could plot them to make a pretty phylogeny figure.
#If you transfer one of them to your computer (with Filezilla) you can view them and make figures with a program, I use FigTree.

### CAFE 
GOAL: After running OrthoFinder, you can run Cafe!
#Cafe itself is quite quick to run, but it requires some steps preparing the required input files. 
#It needs a tab separated file listing how many proteins are within each orthogroup for each speceis. Luckily, this information is given by OrthoFinder.

#make a directory to work in

mkdir ~/MGWA_VELO045/cafe
cd ~/MGWA_VELO045/cafe

#Now we need to prepare the results for CAFE
#For this we need the file Orthogroups.GeneCount.tsv
#You can find this in your OrthoFinder reults folder, which should be named Results_"$GENOME" probably

cp MGWA_VELO045/orthofinder/Results_MGWA_VELO045/Orthogroups/Orthogroups.GeneCount.tsv MGWA_VELO045/cafe

### NOTE TO COPY FILES TO PATH:
cp /path/to/source/foo.txt path/to/target/folder/bar.txt

less Orthogroups.GeneCount.tsv #The last column is named Total. We do not want it. press q to go back

#remove the last column which is named Total
awk 'NF{NF-=1};1' Orthogroups.GeneCount.tsv | awk '{print "(null)\t"$0}' > Cafe_input.tsv

less Cafe_input.tsv #The last column should be gone, but the species names are annoying. press q to go back

#simplify and standardize species codes

sed -i 's/longest\.simpleheader\.//g' Cafe_input.tsv
sed -i 's/\.AA//g' Cafe_input.tsv
sed -i 's/Willisornis/Wilpoe/g' Cafe_input.tsv

less Cafe_input.tsv #Better but we should make it be tab separated not space separated. press q to go back

#make tab delimited

sed -i 's/ /\t/g' Cafe_input.tsv 
sed -i 's/\tOrthogroup/ID\tOrthogroup/' Cafe_input.tsv

less Cafe_input.tsv #Better. Should be tab-separated, with simple names for the columns.

#Gene families with more than 100 copies in a species will completely throw off the cafe results. We must separate these
#filter to separate gene families with more than 100 copies in any species

#First, make a python2 environment 
# DO NOT REDO THIS IF YOU HAVE ALREADY DONE THIS (for example in tutorial)
conda create --name python2 python=2.7.12
conda activate python2
conda install matplotlib=2.2.4

#Great, now activate the new environment
conda activate python2

#temporarily remove the headers for the following code to work
tail -n +2 Cafe_input.tsv > temp_filterinput

python /home/0_PROGRAMS/CAFE-4.2.1/cafe_tutorial/python_scripts/cafetutorial_clade_and_size_filter.py -i temp_filterinput -o filtered_Cafe_input.tsv -s 

#put the headers back and deal with the fact it thought the first orthogroup was the header

grep OG0000000 filtered_Cafe_input.tsv > temp_OG0

grep -v OG0000000 filtered_Cafe_input.tsv > temp_smallfams

grep -v OG0000000 large_filtered_Cafe_input.tsv > temp_largefams

head -n 1 Cafe_input.tsv > temp_header

cat temp_header temp_smallfams > filtered_Cafe_input.tsv

cat temp_header temp_OG0 temp_largefams > large_filtered_Cafe_input.tsv

rm temp_OG0 temp_smallfams temp_largefams temp_header temp_filterinput

#look at the results

less large_filtered_Cafe_input.tsv #press q to go back

less filtered_Cafe_input.tsv #press q to go back

##Ok, do the results look good? Tab separated columns of numbers? The first column should just say "null" (that is good).
## YES 
##Now you need a phylogeny of birds.

#Here is a Newick format phylogeny for all species in this dataset. THIS ONE WILL NOT WORK! IT IS MISSING YOUR SPECIES! 
#Newick trees are super annoying to edit by hand (it took me a long time to put this one together), so email Else to request your tree, letting me know exactly which species from BIOD98 you included (only your own, or also some other ones?).(Or if you feel like diving in to Newick notation, you could make your own).

##To make it a file, use cat to open a blank file, then paste your newick format tree (such as the one below, but don't use that one, use a custom one)

##TREE WITH BOTH SPECIES ON IT: MGWA_VELO045 AND SISTER (MOWA_IF09D02)

((Falper:2.63,Falche:2.63):54.41,(((Strhab:25.70,Nesnot:25.70):5.73,(Amacol:21.28,Melund:21.28):10.15):23.84,(Acachl:46.72,((Wilpoe:35.20,(((((Lepcor:6.1,Manvit:6.1):0.7,Pipfil:6.8):3.3,Coralt:10.1):3.83,Neochr:13.93):10.46,Emptra:24.39):10.81):8.73,((Corcor:2.54,Corbra:2.54):27.94,((Psehum:13.8,(Parmaj:11.27,Cyacae:11.27):2.53):13.25,((Ficalb:19.80,Stuvul:19.80):6.93,(((Taegut:11.9,Lonstr:11.9):0.9,Erygou:12.8):8.49,(Sercan:17.11,((Campar:3.04,Geofor:3.04):11.58,((Zonalb:4.95,Junhie:4.95):7.97,(MOWA_IF09D02.standardplus.namedproteins:1.3,MGWA_VELO045.standardplus.namedproteins:1.3):11.62):1.7):2.49):4.18):5.44):0.32):3.43):13.45):2.79):8.55):1.78);

##TREE WITH WARBLER: REPLACE WITH SPECIES GENOME NAME 
((Falper:2.63,Falche:2.63):54.41,(((Strhab:25.70,Nesnot:25.70):5.73,(Amacol:21.28,Melund:21.28):10.15):23.84,(Acachl:46.72,((Wilpoe:35.20,(((((Lepcor:6.1,Manvit:6.1):0.7,Pipfil:6.8):3.3,Coralt:10.1):3.83,Neochr:13.93):10.46,Emptra:24.39):10.81):8.73,((Corcor:2.54,Corbra:2.54):27.94,((Psehum:13.8,(Parmaj:11.27,Cyacae:11.27):2.53):13.25,((Ficalb:19.80,Stuvul:19.80):6.93,(((Taegut:11.9,Lonstr:11.9):0.9,Erygou:12.8):8.49,(Sercan:17.11,((Campar:3.04,Geofor:3.04):11.58,((Zonalb:4.95,Junhie:4.95):7.97,MGWA_VELO045:12.92):1.7):2.49):4.18):5.44):0.32):3.43):13.45):2.79):8.55):1.78);

##CREATING A FILE:
cat > file_name. #press enter; and enter contents of file; press control D to save

##Reading the trees

#The numbers are divergence dates, measured in millions of years. The names are the names of bird species abbreviated to the first three letters of the genus name and first three of the species name.

## RUNNING CAFE: INCOMLETE

#Now you will have the input files:

#1. Your filtered_Cafe_input.tsv file from OrthoFinder results
#2. A Newick-format phylogenetic tree with dates

#Café is measuring two variables

#1. lambda, the birth-death parameter, a rate of gene duplication and gene loss, per millions of years
#2. empsilon, an estimate of the proportion of error in the gene count estimates

#GOAL: Errors in genome assembly, annotation, and ortholog assignment mean that the data probably contains hidden errors in the number of genes in each gene family in each species. 
#Fragmented annotations or uncollapsed haplotypes in the assembly can lead to duplicate counting of the same gene, and assembly gaps or unannotated proteins can lead to undercounting a gene family. 
#We can deal with this by estimating assembly error that will be taken into account by cafe.

#Note some scripts only work with python version 2 (not 3) so you will need to activate your python2 environment 

conda activate python2

#This does a single run, estimating both lambda and epsilon
time /home/0_PROGRAMS/CAFExp-5.0b2/bin/cafexp -i filtered_Cafe_input.tsv -t phylogeny_1_WARBLER.tree -e -o error_model_cafe > error_model.log

OUTPUT:
real	0m0.022s
user	0m0.005s
sys	0m0.000s

less error_model.log #view results of the error model

#We can run cafe multiple times
#run cafe again, but reuse the estimate of epsilon that you calculated before

time /home/0_PROGRAMS/CAFExp-5.0b2/bin/cafexp -i filtered_Cafe_input.tsv -t phylogeny_SISTER_WARBLER_SPECIES.tree -o single_lambda -eerror_model/Base_error_model.txt > single_lambda.log

#This would run cafe with NO epsilon (ignores possibility of errors in data)

time /home/0_PROGRAMS/CAFExp-5.0b2/bin/cafexp -i filtered_Cafe_input.tsv -t Psittacopasserae.tree -o single_lambda_noe > single_lambda_noe.log

#This would allow among-family rate variation in lambda, estimates lambda and alpha and three discrete gamma rate categories (essentially allow three different rates of gene gain/loss, for fast-evolving families, medium-evolving families, and slow-evolving families
#This takes longer and may crash

time /home/0_PROGRAMS/CAFExp-5.0b2/bin/cafexp -i filtered_Cafe_input.tsv -t Psittacopasserae.tree -k 3 -o three_lambda > three_lambda.log

#It is a good idea to do many replicates. Here is an example of a loop that will do 10 runs in parallel. If you want to do more you can change the part that says {1...10} and change 10 to whatever number you want 

printf '%s\n' {1..10} | parallel /home/0_PROGRAMS/CAFExp-5.0b2/bin/cafexp -i filtered_Cafe_input.tsv -t Psittacopasserae.tree -e -o replicate_run_{1} >> 100runs.log 2>> 100runs.errorlog

#The above will not print anything to the terminal! (Because that would be too much to look at). Instead it will be printed to a log file which you could view like this:

cat 100runs*log

#Now analyze the large families using the previous values of lambda, since you cannot estimate lambda for these families. 
#Choose the lambda from your best run, whichever you think is best (with or without epsilon?). 
#You may want to read the part about interpreting results first, to find your lambda. 
#YOU MUST CHANGE THIS YOURSELF IN THE CODE! Lambda is the number in the command right after -l. I have set it to a unreasonably high value, 0.99999, so that it will fail if you do not change it to your own, more reasonable estimate found in previous runs.

time /home/0_PROGRAMS/CAFExp-5.0b2/bin/cafexp -i large_filtered_Cafe_input.tsv -t Psittacopasserae.tree -l 0.99999 -o large_results > large_results.log

##INTERPRETING YOUR RESULTS:

#The estimates of lambda/epsilon are found in "$run"_results.txt.

#Here are how to view some of the results files. They will likely look very terrible to look at with your eyes, a mess of numbers.

less error_model/Base_asr.tre #gives reconstructed numbers of each gene family at each node

less error_model/Base_branch_probabilities.tab #gives calculated likelihood of the gene family size at each node for each gene family

less error_model/Base_clade_results.txt #how many families are expanded or contracted at each node

#Here you can list all the values of lambda that you estimating in any runs

grep "Lambda" ./*/*_results.txt #rate of gene birth/death

#Here you can list all the values of epsilon that you estimating in any runs

grep "Epsilon" ./*/*_results.txt #error term

##To interpret your results:

##What gene families were expanded/contracted in a particular species?

#get a list of the significantly expanded/contracted/rapid genes in your species
#replace the path of error_model/ with the name of whatever Cafe run you want to view

grep "$GENOME" error_model/Base_asr.tre | sed 's/^.*OG/OG/g' | sed 's/ = .*$//g'

#Get list of families that have expanded in your species, not necessarily significant

#First you will have to find out what column your species is in. 

less error_model/Base_change.tab #What column has your species? For example, mine, Wilpoe, was in column 41

#Now change "41" in this command to whatever number column your species is in

cut -f 1,41 error_model/Base_change.tab | grep -v "+0" | grep -v "-" #This is the list of gene families that have expanded in your species relative to the sister lineage

#count the number of expansions #again change "41" in this command to whatever number column your species is in

cut -f 1,41 error_model/Base_change.tab | grep -v "+0" | grep -v "-" | wc -l

#If you did many runs with the loop, here is how to summarise all of them and get an average for all runs

#Create two files, summarising all the values of epsilon and lambda from your replicate runs if you did that part

grep "Lambda" ./replicate_run_*/Base_results.txt | sed 's/^.* //g' > lambdas.txt
grep "Epsilon" ./replicate_run_*/Base_results.txt | sed 's/^.* //g' > epsilons.txt

R #opens R, so you are now coding in R language, not bash (Terminal).

epsilons <- read.table("epsilons.txt") #read files
lambdas <- read.table("lambdas.txt")

mean(epsilons[,1]) #reports the mean value of epsilon
sd(epsilons[,1]) #reports the standard deviation value of epsilon

mean(lambdas[,1]) #reports the mean value of lambda
sd(lambdas[,1]) #reports the standard deviation value of lambda
q() #quit R
n #no to saving

## GETTING RESULTS:

#The following code assumes that you ran the loop that did many replicates of cafe. 
#If you did not do this, you will need to change the code. 
#They all also assume that you did 10 runs because that is what I did. 
#If not, change that number in all of the parts where it says {1...10}. 
#You will also have to change Wilpoe<39> to the name of your species and the number in <> that is says in your cafe files. Only change Wilpoe<39> and keep the \* that appears in the code!

#make folder

mkdir analysis
cd analysis

#get a list of the significantly expanded/contracted/rapid genes in your species

printf '%s\n' {1..10} | while read number ; do grep "Wilpoe<39>\*" ../replicate_run_$number/Base_asr.tre | sed 's/^.*OG/OG/g' | sed 's/ = .*$//g' > sig_changes_error_model_"$number" ; done 

#count how many were found at each run

wc -l sig_changes_error_model_* #most found 50 in my species, except number 6 found 51. Ideally, they should all be the same.

#Since my run 6 was weird, this is how to see what is uniquely found in run 6 (could change to whatever number was weird for you, or skip)

printf '%s\n' {1..10} | while read number ; do grep -v -f sig_changes_error_model_"$number" sig_changes_error_model_6 ; done

#OG0013931 in my run 6 is not found in the other runs
#make sure all the others are identical

printf '%s\n' {1..10} | while read number ; do grep -v -f sig_changes_error_model_"$number" sig_changes_error_model_1 ; done

#yep, all 50 were the same in all other runs (no output returned by the above command, means that nothing was different in them compared to run 1)

#Get list of families that have expanded in the species, whether or not significant
#CHANGE the number 41 to the column number for your species!!!

printf '%s\n' {1..10} | while read number ; do cut -f 1,41 ../error_model_$number/Base_change.tab | grep -v "+0" | grep -v "-" > expandedfams_"$number" ; done #1212 in all ten replicates for mine

#compare expanded list to significant list to keep only significantly expanded ones

printf '%s\n' {1..10} | while read number ; do grep -f sig_changes_error_model_"$number" expandedfams_"$number" > expandedfams_error_model_"$number".sig ; done 

#count how many there are

wc -l antexpandedfams_error_model_*.sig #21 for all replicates except 22 for number 6 with OG0013931 uniquely

#make sure all the others are identical

printf '%s\n' {1..10} | while read number ; do grep -v -f expandedfams_error_model_"$number".sig expandedfams_error_model_1.sig ; done

#yep, all 21 were found in all my other runs (no output returned by the above command)

#repeat for contracted families
#CHANGE 41 to your column number

printf '%s\n' {1..10} | while read number ; do cut -f 1,41 ../error_model_$number/Base_change.tab | grep -v "+" > contractedfams_error_model_"$number" ; done #1733 contracted families in all ten replicates

printf '%s\n' {1..10} | while read number ; do grep -f sig_changes_error_model_"$number" contractedfams_error_model_"$number" > contractedfams_error_model_"$number".sig ; done #keep only significantly contracted families

wc -l antcontractedfams_error_model_*.sig #count: 29 significantly contracted families for all replicates

#make sure all the others are identical to round 1

printf '%s\n' {1..10} | while read number ; do grep -v -f contractedfams_error_model_"$number".sig contractedfams_error_model_1.sig ; done

#yep, all 29 were found in all other runs (no output returned by the above command).

#At the end, I got files listing significantly expanded or contracted orthogroups.
#First things first. What genes are in these orthogroups?

mkdir ~/"$GENOME"/cafe/expanded
cd ~/"$GENOME"/cafe/expanded

#get a fasta of the expanded families from the OrthoFinder output. This is using run 1 of the replicate runs, if you want to use a different one, change expandedfams_error_model_1 to the number you want here and also in the next command.

cat ../analysis/expandedfams_error_model_1.sig | cut -f 1 | while read Orthogroup ; do cp /home/0_BIOD98/OrthoFinderpasserines/Orthogroup_Sequences/"$Orthogroup".fa . ; done

#Get a list of the genes in these orthogroups for your species

cat ../analysis/antexpandedfams_error_model_1.sig | cut -f 1 | while read Orthogroup ; do grep "$GENOME" "$Orthogroup".fa >> expanded_geneIDs.txt ; done

sed -i 's/>//g' expanded_geneIDs.txt

#Now, you will possible need to edit this. 
#Take a look at your files and look at whether there is a suffix at the end of your gene names (for example, mine all end in -RA). If there is any extra stuff letters different that -RA, change this command (I think it should always be just -RA though)

sed -i 's/-RA//g' expanded_geneIDs.txt #get rid of the -RA

#repeat for contracted gene if you are interested in those too

mkdir ~/"$GENOME"/cafe/contracted
cd ~/"$GENOME"/cafe/contracted

#get a fasta of the contracted families from the OrthoFinder output

cat ../analysis/contractedfams_error_model_1.sig | cut -f 1 | while read Orthogroup ; do cp /home/0_BIOD98/OrthoFinderpasserines/Orthogroup_Sequences/"$Orthogroup".fa . ; done

#Get a list of the genes in these orthogroups

cat ../analysis/contractedfams_error_model_1.sig | cut -f 1 | while read Orthogroup ; do grep "$GENOME" "$Orthogroup".fa >> contracted_geneIDs.txt ; done

sed -i 's/>//g' contracted_geneIDs.txt

sed -i 's/-RA//g' contracted_geneIDs.txt

##If you want to find out what these genes do, go to the next page: GO analysis

## TRANSFERRING ALL THE FILES ONTO MAIN SERVER FROM RAMPHOCELES:

##Transfers the short_summary_protein_annotation_eval_SHAWANAH_RAHMAN.txt to the main server path (/home/0_BIOD98_GENOMES1/BUSCO_REPORTS/)

scp USER6@192.168.0.5:~/MGWA_VELO045/busco/run_protein_annotation_eval/short_summary_protein_annotation_eval.txt . 

##Transfers the MGWA_VELO045.fasta and MGWA_VELO045.fasta.gz to the main server path (/home/0_BIOD98_GENOMES2/MGWA_VELO045/1_1__SUPERNOVA/)

scp USER6@192.168.0.5:~/MGWA_VELO045/genome/MGWA_VELO045.fasta .
scp USER6@192.168.0.5:~/MGWA_VELO045/genome/MGWA_VELO045.fasta.gz .

##Transfers the files from repeatmodeler to the main server path (/home/0_BIOD98_GENOMES2/MGWA_VELO045/1_2__REPEATMODELLER/)
scp -r USER6@192.168.0.5:~/MGWA_VELO045/repeatmodeler .

##Transfers the files from EDTA to the main server path (/home/0_BIOD98_GENOMES2/MGWA_VELO045/1_3__TE_LIBRARY/EDTA)
scp -r USER6@192.168.0.5:~/MGWA_VELO045/EDTA .

##Transfers the files from repeat_library to the main server path (/home/0_BIOD98_GENOMES2/MGWA_VELO045/1_3__TE_LIBRARY)
scp -r USER6@192.168.0.5:~/MGWA_VELO045/repeat_library .

##Transfers the files from Maker1 to the main server path (home/0_BIOD98_GENOMES2/MGWA_VELO045/2_1__MAKER_ROUND1/)
scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R1/Maker1 .

##Transfers the files from MGWA_VELO045_maker_R2 to the main server path (/home/0_BIOD98_GENOMES2/MGWA_VELO045/2_2__MAKER_ROUND2/)
scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R2 .

##Transfers the files from MGWA_VELO045_maker_R3 to the main server path (/home/0_BIOD98_GENOMES2/MGWA_VELO045/2_3__MAKER_ROUND3/)
scp -r USER6@192.168.0.5:~/MGWA_VELO045/MGWA_VELO045_maker_R3 .

##Transfers the files from functional_ann to the main server path (/home/0_BIOD98_GENOMES2/MGWA_VELO045/3__FUNCTIONAL_ANNOTATION/)
scp -r USER6@192.168.0.5:~/MGWA_VELO045/functional_ann .

##Transfers the files from busco to the main server path (/home/0_BIOD98_GENOMES2/MGWA_VELO045/3__FUNCTIONAL_ANNOTATION/)
scp -r USER6@192.168.0.5:~/MGWA_VELO045/busco .

##Transfers the files from maker to the main server path (/home/0_BIOD98_GENOMES2/MGWA_VELO045/2_1__MAKER_ROUND1/)
scp -r USER6@192.168.0.5:~/MGWA_VELO045/maker .

##Transfers the files from orthofinder to the main server path (home/0_BIOD98_GENOMES2/MGWA_VELO045/4__ORTHOFINDER/)
scp -r USER6@192.168.0.5:~/MGWA_VELO045/orthofinder .

## CAFE FILE TRANSFER

## GO ANALYSIS:


## REPEAT LANDSCAPE:
GOAL: So now we have a set of consensus TE sequences. We will use this to see how much of the genome is taken up by TE sequences.

#What proportion of the Genome is made up of different types of TEs?
#This is done most easily and conventionally using RepeatMasker.

#set up environment, obtaining TE library and genome sequence
mkdir ~/MGWA_VELO045/RepeatMasker
cd  ~/MGWA_VELO045/RepeatMasker

#Now get a copy of your repeat library and genome into this folder.

cp  ~/MGWA_VELO045/repeat_library/MGWA_VELO045_repeat_library_withFicalbUracya.lib .
cp ~/MGWA_VELO045/genome/MGWA_VELO045.fasta .

#Now make a conda environment

conda create --name RepeatMasker
conda activate RepeatMasker
conda install -c bioconda repeatmasker

#Run repeatmasker

conda activate RepeatMasker
#RepeatMasker
time RepeatMasker -no_is -lib MGWA_VELO045_repeat_library_withFicalbUracya.lib -dir . MGWA_VELO045.fasta -pa 23 -s -a -inv -gccalc -xsmall > MGWA_VELO045_rmask.log #slow search, more sensitive
#done

less MGWA_VELO045_rmask.log

#Now, we can parse the results of RepeatMasker.

#In these commands, you should change "1089534386" to be the length of your genome without Ns (which is reported by the RepeatMasker .tbl output file at the top)

#requires perl with bio::SeqIO
export PERLBREW_ROOT=/opt/perl5 #perlbrew will be installed in opt
/opt/perl5/bin/perlbrew switch perl-5.30.0 #A sub-shell is launched with perl-5.30.0 as the activated perl. Run 'exit' to finish it.

#redefine GENOME now

#parse the out file to find amount of DNA masked
time perl /home/0_PROGRAMS/Parsing-RepeatMasker-Outputs/parseRM.pl -i MGWA_VELO045.fasta.out -p -g 1089534386 -r MGWA_VELO045_repeat_library_withFicalbUracya.lib -v

#parse the align file to make landscape graph
time perl /home/0_PROGRAMS/Parsing-RepeatMasker-Outputs/parseRM.pl -i MGWA_VELO045.fasta.align -g 1089534386 -r MGWA_VELO045_repeat_library_withFicalbUracya.lib -m 0.0033 -l 100,1 -v
#done

#To make a figure, you can plot histograms in Excel or the program of your choice.

## REPORTABLE DATA FROM REPEAT LANDSCAPE:
In the paper, I am reporting the fraction of the genome masked by each repeat class/family.
To turn bp in fraction, I am using the length of the genome with runs of N's removed (since these do not represent real sequence that could be masked). This is conveniently reported in "$GENOME".fasta.tbl produced by repeatmasker: total length: 1112597196 bp (1089534386 bp excl N/X-runs). Then, I am using the parsed file from Parsing-RepeatMasker-Outputs/parseRM.pl to get number of basepairs masked by each family and class of repeat. All the unknowns are lumped into one number so to get the unknowns for LTR vs LINE I just subtract the identified LTR/LINE families from their respective totals to get the unknowns for each class. The other libraries use a different classification system using the name ERV3 instead of ERVL and ERV2 instead of ERVK so a I am adding those two values together for those two families.
