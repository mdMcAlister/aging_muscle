# aging_muscle
A bioinformatic workflow for finding differentially expressed genes between old and young tissue samples.

## Prerequisites
* Nextflow
* GEMmaker
* GEMprep?
* Singularity

## Find Experiment Files
**Step 1.** Search NCBI Bioproject for "aging AND muscle"  
**Step 2.** Under "Project Data" select *SRA* and under "Organism Groups" select *Human*  
**Step 3.** Open "Skeletal Muscle Transcriptome in Healthy Aging"  
**Step 4.** Read the project description.  
**Step 5.** Under "Project Data", you'll see a row for SRA Experiments. On the right side, you'll see the Number of Links. Click the number across from SRA Experiments (it should say "53").
**Step 6.** Near the top of the page, click the text that reads "Send results to Run selector".  
**Step 7.** In the middle of page, you'll see a section titled "Select". In the row titled "Total" click the Metadata button underneath the "Download" column. This will download a CSV to your local computer.  
**Step 8.** Open a blank spreadsheet in Excel. Navigate to the "Data" tab, then click "From Text/CSV". This will import the list of experiments.  
**Step 9.** Sort the AGE column from smallest to largest to get a sense of the range.  
**Step 10.** Determine the average age using Excel's built-in =AVERAGE() command.  
**Step 11.** Select six samples to carry forward for further analysis: two near the youngest age, two near the mean or median age, and two near the oldest age. For each pair, select one male and one female.  
**Step 12.** Copy the SRR numbers for these six samples.  

## Configure GEMmaker and Create a Gene Expression Matrix (GEM)
**Step 1.** Clone GEMmaker inside your working directory.  
```
git clone https://github.com/SystemsGenetics/GEMmaker.git --branch develop
```
**Step 2.** Index the human genome (or move the references file over from a previous deployment of GEMmaker).  
1. Download `wget http://ftp.ensembl.org/pub/release-103/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz`
2. Decompress `gunzip *.gz`
3. Run the command `singularity exec -B ${PWD} docker://gemmaker/hisat2:2.1.0-1.1 hisat2-build Homo_sapiens.GRCh38.dna.primary_assembly.fa HG38`
**Step 3.** Create a folder named "input".  
**Step 4.** Move SRA_IDS.txt from /demo to /input and add the six SRR numbers to it.  
**Step 5.** Edit the nextflow.config file that is in the main GEMmaker to configure the GEMmaker run properly. This is the configuration file that has all the parameters for GEMmaker to run, so we need to replace all the example file names with our real data file names. Here are the steps to reconfigure the nextflow.config file:
1. Scroll down to the section that says project and change the name, machine_name, and description. You can edit these to whatever you want as the project name and project description, which are human-readable. Note that machine_name can only include letters, numbers, and underscores because it is the machine-readable project name.
1. Scroll down to the section that says input. Change reference_name from “CORG” to “HG38”. This name must be the same as the prefix for all the index files in the references directory (the ones ending in .ht2).
1. Change local_sample_files from “*_{1,2}.fastq” to “none”.  We are only using the samples with the SRA IDs from above – we don’t have any local sample files which would have been detected using the above regular expression.
1. Scroll down to the section that says hisat2. This is the alignment tool we want to use, so change enable=false to enable=true.
1. Change index_dir from “CORG.genome.Hisat2.indexed” to “.”.  A dot ‘.’ (period) means current directory and we are setting index_dir to this because the index files (.ht2) are just in the reference directory. Index_dir would be set to something else if you had another directory inside references/ that contained the index files
1. Change gtf_file from “CORG.transcripts.gtf” to the name of the gtf file in the references directory. The name is probably “Homo_sapiens.GRCh38.101.gtf” but you can check ‘references’ to make sure the name matches.
1. Scroll down to the section that says kallisto. Set “enable = false”. We are using hisat2 as our alignment tool. Kallisto is a different option for the same function.
1. Save your changes and cat the file to make sure the changes are correct.
 
**Step 5.** Navigate to your GEMmaker working directory. Run GEMmaker with the human indexed genome, new RNAseq experiment list, and reconfigured nextflow.config file.
```
nextflow run main.nf -profile standard -with-singularity -with-report -with-timeline
```
