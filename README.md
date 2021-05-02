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
**Step 2.** Create a folder named "input".  
**Step 3.** Move SRA_IDS.txt from /demo to /input and add the six SRR numbers to it.  
**Step 4.** Create a folder named "input/references", then navigate to that folder.  
**Step 5** Index the human genome using **Singularity** and **hisat2** (Alternatively, you can move the references file over from a previous deployment of GEMmaker).  
1. Download `wget http://ftp.ensembl.org/pub/release-103/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz`
2. Download `wget http://ftp.ensembl.org/pub/release-103/gtf/homo_sapiens/Homo_sapiens.GRCh38.103.gtf.gz`
3. Decompress `gunzip *.gz`
4. Run the command `singularity exec -B ${PWD} docker://gemmaker/hisat2:2.1.0-1.1 hisat2-build Homo_sapiens.GRCh38.dna.primary_assembly.fa HG38`  

**Step 5.** Edit the nextflow.config file that is in the main GEMmaker to configure the GEMmaker run properly. This is the configuration file that has all the parameters for GEMmaker to run, so we need to replace all the example file names with our real data file names. Here are the steps to reconfigure the nextflow.config file:  

In the section titled "params":  
1. Change `remote_sample_list = "./demo/SRA_IDs.txt"` to `remote_sample_list = "./input/SRA_IDs.txt"`
2. Change `local_sample_files = "./demo/*_{1,2}.fastq"` to `local_sample_files = "./input/*_{1,2}.fastq"`
3. Change `skip_list_path = "./demo/samples2skip.txt"` to `skip_list_path = "./input/samples2skip.txt"`  

In the section titled "quantification":
1. Change `method = 'kallisto'` to `method = 'hisat2'`.
2. Change `base_name = "KORG"` to `base_name = "HG38"`
3. Change `index_dir = "./demo/references/CORG.genome.Hisat2.indexed"` to `index_dir = "./input/references"`
4. Change `gtf_file = "./demo/references/CORG.transcripts.gtf"` to `gtf_file = "./input/references/Homo_sapiens.GRCh38.103.gtf"

Save your changes.
 
**Step 6.** Navigate to your GEMmaker working directory. Run GEMmaker with the human indexed genome, new RNAseq experiment list, and reconfigured nextflow.config file.
```
nextflow run main.nf -profile standard -with-singularity -with-report -with-timeline
```
