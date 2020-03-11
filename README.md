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

MAKER STEPS 1:

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

rename "s/genome/MGWA_VELO045_rnd1.zff.length50_aed0.25/g" *

#create statistics
/opt/tools/snap/fathom MGWA_VELO045_rnd1.zff.length50_aed0.25.ann MGWA_VELO045_rnd1.zff.length50_aed0.25.dna -gene-stats > gene-stats.log 2>&1

 /opt/tools/snap/fathom MGWA_VELO045_rnd1.zff.length50_aed0.25.ann MGWA_VELO045_rnd1.zff.length50_aed0.25.dna -validate > validate.log 2>&1
 
 #read the log files
cat *.log #make sure that it does not say any errors, such as not being able to find a file
