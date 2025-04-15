# genome-assembly-project
This project was developed after the completion of my 'Genome Assembly' module at Cranfield University, UK

## Introduction
This is a report of a de-novo genome assembly workflow performed on an “unknown” organism. A software program to visualise the assembly quality report metrics, developed using Java, was also reported in addition. Illumina PE 101x2 short reads with an insert size of 350bps +/-50 as well as PacBio long reads were provided for this assignment. The gamod-docker container accessed through the singularity module was utilised throughout the workflow on the Cranfield High-Performance Computing cluster system (Crescent2).

## Part I - Assembly Workflow Report and Findings
### Question 1 – Data Preparation and Quality Control
FastQC analysis of the raw short reads shows a low quality of the data with a score below 20% as shown in Figure 1., K-mer analysis of the raw reads also shows a huge error k-mer spectrum even at 5:200 plot (Figure 2), using 19 k-mer size. An improvement was observed after correcting the reads with the SOAP-ec algorithm using k-mer size of 23 (Figure 3), which was then utilised for the genome size estimation. The reads count went from 2475000 to 2378197 for the corrected reads.

 ![image](https://github.com/user-attachments/assets/cfad0295-a226-49e5-8caf-f1a587bd85a7)

Figure 1: Per-base quality score fastqc plot of the raw Illumina short reads.

 ![image](https://github.com/user-attachments/assets/2cc282aa-8eeb-4c4c-a805-b036484324e5)

Figure 2: K-mer spectrum of the raw reads.

 ![image](https://github.com/user-attachments/assets/adea0e7f-034f-460f-8eaa-9ac250308508)

Figure 3: K-mer spectrum of the corrected reads at 23 k-mer size.

### Genome Size Estimation
The corrected read was used for the genome estimation and an estimated size of 6.2M was obtained as shown below.

 ![image](https://github.com/user-attachments/assets/99794b28-00c2-432b-9708-67c731dd805e)

Figure 4: Genome estimation points plot

 ![image](https://github.com/user-attachments/assets/f9053525-f095-48b8-8950-291f7a4dbd52)

Estimated genome size of 4.9M was obtained when the raw short reads were used for the genome estimation calculations.

 ![image](https://github.com/user-attachments/assets/10d446d2-5381-4166-9c92-32c2bb717d5a)

Below are the quality metrics of the raw reads before the assembly.
![image](https://github.com/user-attachments/assets/d4c109ff-4d56-4154-ac54-6935d2560500)
 
Below are the quality metrics of the corrected reads before the assembly.
 ![image](https://github.com/user-attachments/assets/f2ee094c-1e80-494d-bb3b-75d8366fc2ab)


### Question 2 – The Genome Assembly Attempts
Various assembly algorithms were tested for this assignment ranging from short and long reads assemblers to hybrid assemblers that utilize both short and long reads for the assembly.

### Short Reads Attempts
Although both short and long reads were provided, which signifies that a hybrid assembly might be the best option to go for, some short reads assemblers were attempted in the workflow, and they include the following:

### SOAPdenovo2: 
This is the first de novo genome assembler that was attempted for this task. SOAPdenovo2 is an improved version of the SOAPdenovo algorithm known for Illumina short reads assembly of large genomes using the de-Bruijn graph (DBG) in a memory-efficient manner (Luo et al., 2012). The corrected short reads data obtained in the data preparation step were assembled in two steps using the SOAPdenovo-63mer algorithm installed in gamod. This was selected instead of the SOAPdenovo-127 version due to memory limitations. The first step – the pregraph step, was performed using the SOAPdenovo-63mer sparse_pregraph algorithm. This step requires a configuration file which contains the Illumina reads paths, average insert size and other important parameters. Different parameters were tested but the below parameters provided the best N50 score for the provided data in this attempt.
 ![image](https://github.com/user-attachments/assets/e930e143-412f-42b2-becf-ef58cb5b6cfa)

The 350 was the provided insert size for the Illumina PE library, and 3 asm_flags were used to signify both contigs and scaffolds assembly while 63 map_len to indicate minimum aligned length to contigs for a reliable read location. The de-Bruijn graph was built using the 6.2M estimated genome size obtained in the data preparation step. Different k-mers (25, 27, 31, 37 & 63) were tested but 63 k-mer size provided the best quality score (N50 = 19466bps) for this attempt. The SOAPdenovo-63mer algorithm was then used to build the assembly contigs file in the second step using the obtained graph and the configuration file generated in the first step.

### Velvet: 
This was another short-read de novo assembler that was attempted and works in two steps like the SOAPdenovo assembler. The graph creation was performed by the Velveth algorithm while the actual assembly was performed by the Velvetg algorithm. Different k-mer sizes (25, 27 and 63) were tested but no meaningful assembly result was obtained using both the corrected reads and raw reads of the Illumina data. A very poor N50 score of 76 (71320 sequences) was obtained as the best attempt using the raw reads at 31 and 63 k-mer sizes. 

### Long Reads Attempts
Two long reads assemblers were attempted and included Falcon and Canu algorithms. However, only Canu provided results.

### Canu: 
This assembly attempt was run as a scheduled job on PBS on Crescent2 HPC due to its high memory demand. Different parameters were tested but the following produced the best results for this attempt.
 ![image](https://github.com/user-attachments/assets/5b317eb9-a758-4723-bdaf-5b6c2e143f19)

Canu performed poorly for this data as only 569bps (1 sequence) were assembled using the above parameters while some other tested parameters returned no assembly result.

### Hybrid Assembly Attempts
As expected, hybrid assemblers produced the best results as described below and this can be due to the ability of the assemblers to combine the higher accuracy of the short reads and the contiguity of the long reads to perform the assembly. Below are the different assemblers that were attempted for this assignment.

### DBG2OLC: 
The DBG2OLC (De-Bruijn Graph, Overlap Layout Consensus) algorithm which was also available on the gamod-docker container, assembled the genome in two steps – de-Bruijn graph creation (the DBG part of the algorithm) using the short read data and performing an overlap layout consensus (the OLC part of the algorithm) using long reads data (Ye et al., 2016). The DBG2OLC SparseAssembler (a short reads assembler that comes with the DBG2OLC algorithm) was used to create the de-Bruijn graph before performing the overlap layout consensus (OLC) step of the assembly. The provided long read fastq file was first converted to fasta before proceeding to the overlapping step. The final consensus step was performed using the my_split_nrun_sparc.sh utility script provided during the course practical.
 Different parameters were tested in search of better results. The parameters that produced the best results for this attempt were K-mer size of 51, a g (number of skipped intermediate k-mer) value of 15 and an estimated genome size of 12M in the de-Bruijn graph creation step of the assembly while maintaining the default parameters of the OLC step. 
 ![image](https://github.com/user-attachments/assets/9f240ab9-255d-4411-af6d-d76eb5c1a8ab)

However, when the MinOverlap parameter was set to 10 from the default 20 in the OLC step, an improved N50 score of 389kbp was observed from 188kbp using the default parameters. 
 ![image](https://github.com/user-attachments/assets/a9fe8925-dd80-4157-84ab-0cf71d02a31a)
 ![image](https://github.com/user-attachments/assets/488372cf-2d1d-4fdd-a500-05f920f8de9f)

An even better assembly N50 score of 467kbp was observed when the best short-read assembly attempt (SOAPdenovo2) config file was used to perform the overlap layout consensus step of this assembler which is used for the comparative quality evaluation in question 3 below.

### MaSurCA: 
This is the second hybrid assembler that was attempted for this assignment. The assembly process using Masurca involves generating a Masurca configuration file, using the configuration file to generate a Masurca executable bash script (assemble.sh) and finally, using the script to run the assembly. The configuration file for this attempt was generated using the masurca -g command. The configuration file takes as input the short and long reads data paths for the assembly in addition to other parameters automatically generated by the masurca assembler.
 ![image](https://github.com/user-attachments/assets/ee131eeb-121d-4810-87ba-8fb5d9c837cf)

Because reverse reads are optional (and do not influence the assembly outcome) for PE Illumina reads, only the forward read was provided in the configuration file as shown above. The 350 is the insert size with a +/-50 standard deviation which was provided with the data. Both raw and corrected short reads provided the same results for this attempt. Different parameters were tested but produced lower or the same quality score compared to the masurca generated default parameters for this attempt. The assembly was then run using the command masurca masurca_config (masurca_config been the configuration file) which generated the assemble.sh script that executed the assembly. The final assembly fasta file generated is evaluated in question 3 below.

### Spades: 
This was the final assembler that was tested for this assignment. This hybrid assembler performs the assembly using a single-line command as shown below. The assembly was run as a scheduled job on PBS on Creascent2 due to memory demand.
 ![image](https://github.com/user-attachments/assets/d8af003d-9762-48e3-853e-dd5fc2566ad3)

Different parameters were tested but the above were what produced the best result for this assembly attempt. The -k parameter indicates the list of k-mers for the assembler to use (21, 31, 55, 77, 99 and 127) and this was to restrict the assembler from trying many random k-mers which could be computationally intensive and time-consuming. The -t parameter is the number of requested threads and -m is the requested RAM in GB on the server. The --careful parameter was used to reduce the number of mismatches and short indels during the assembly, --cov-cutoff is the coverage cutoff value, --only-assembler runs the assembly without read error correction while -o is the output directory to store the assembly results. The final assembled sequences are evaluated in question 3 below.

### Question 3 – Assembly Quality Assessment
As alluded to earlier, the below three hybrid assemblers provided the best quality assembly metrics for the provided data in all the attempts. 

### DBG2OLC
The final assembly fasta file final_assembly.fasta can be obtained via the following path on Creascent2.
 ![image](https://github.com/user-attachments/assets/4e3266b2-4471-4ae9-bcff-b5046f0e3139)

The gnx.jar function can be used to obtain the assembly quality metrics as shown below.
 ![image](https://github.com/user-attachments/assets/dd2715b9-9fe6-4aae-8af2-66b6b75e99b7)

This assembly can be seen to have performed well in assembling over 4.5Mbp considering an estimated genome size of 6Mbp which is about 73% of the estimated genome size, although it manages to do so in 20 sequences. However, the assembly did not do so well in achieving an N50 score of only 467182 in 4 sequences which is just 41% (2647274bp) of the estimated genome size. 

### MaSuRCA
The final genome assembly file (genome.scf.fasta) for this attempt can be accessed via:
  ![image](https://github.com/user-attachments/assets/2c5d5a92-c38f-45b7-910c-795044cd3c35)

Below are the assembly quality metrics for this assembler obtained using the gnx.jar function.
 ![image](https://github.com/user-attachments/assets/7465cd0a-6d72-4103-ae49-62e089012a0a)

This assembler performed so well to assemble over 4.9Mbp which is about 80% of the estimated genome size in just 12 sequences. The assembler managed to achieve an N50 score of 1049190 in just 2 sequences which is far better than the DBG2OLC assembler shown above. Although the combined N50 bps is also around 41% of the genome size, however, it was achieved in half the number of sequences achieved by the previous assembler. This is an indication of more contiguity in the assembled base pairs which is very paramount in genome assembly.

### Spades
The final assembly file generated by this assembler is named scaffolds.fasta and can be accessed via:
 ![image](https://github.com/user-attachments/assets/7b442fbb-f47b-4553-b861-b5258eba610b)

Below are the quality score metrics of this assembler also obtained using the gnx.jar function.
 ![image](https://github.com/user-attachments/assets/04137ba2-6bbe-4ea1-9ad6-00e7341aca7e)

The spades assembler assembled more base pairs (4941400bp) compared to both DBG2OLC and MaSurCA, but it achieved so in far more sequences (96) compared to the previous two assemblers. The N50 score (164833) is also very poor compared to the previous two assemblers and Spades managed to achieve this score in way more sequences (9 sequences) compared to the previous two and this is an indication of low contiguity of the assembler in assembling the provided data.

In summary, although each of the selected 3 best assemblers managed to assemble about 80% of the estimated genome size of 6M, with spades assembling a little more base pairs compared to the other two, MaSurCA is selected as the best assembler due to its high N50 score which was achieved in fewer sequences compared to spades and DBG2OLC. Thus, MaSurCA assembly fasta sequence (genome.scf.fasta) was used for the Augustus gene prediction and functional analysis described in question 4 below.

### Question 4 – Gene Prediction
An in-silico gene prediction of the best assembly was performed using Augustus and some interesting potential genes and pathways were observed. Genes associated with catalytic activity, hydrolase activity and nuclease activity were obtained after mapping the assembled sequence using the eukaryotic gene finding function in Augustus. Also, some biological pathways like cellular biosynthesis, response to stress and nitrogen compound metabolic process were associated with some of the predicted genes as shown in figures 5 and 6.
 ![image](https://github.com/user-attachments/assets/f8a7f173-7e60-4b25-8d8f-306c65864fda)

Figure 5: showing some predicted gene functions from the assembled genome.

 ![image](https://github.com/user-attachments/assets/b2bf03bd-a994-426d-9800-f6871020989f)

Figure 6: showing some of the metabolic pathways associated with the predicted genes.

## Part II - Software Program Report
In this part of the assignment, a Java application was developed to load the best assembly in fasta format, compute the assembly quality metrics and display a list of the assembly scaffolds with some functionalities when they are clicked.

### Loading the Best Assembly File: 
For this task, a file chooser functionality was implemented which allows the user to load the assembly fasta file into the application. The loaded file is displayed in a jTextArea as shown below.
 ![image](https://github.com/user-attachments/assets/5d401ae3-a5c1-41fb-adf1-aff936a30740)

### Computing Assembly Quality Metrics: 
This functionality was implemented using a jButton and message dialogue. When the user clicks on the button named getAssemblyMetrics, the jButton compute the assembly quality metrics and displays the results in a message dialogue. The assembly quality metrics displayed includes total length, N50, total scaffolds and others as shown in the screenshot below.
![image](https://github.com/user-attachments/assets/c78f9baa-5c46-4d52-b32f-6845383bc79d)
 

### Scaffolds List and Other Functionalities: 
To achieve this, a jComboBox was utilized which displays all the assembled scaffolds present in the loaded assembly fasta file on loading the file.
 ![image](https://github.com/user-attachments/assets/f9122ac9-4883-4765-8d0f-ce5a12203b8e)

When the user selects a specific scaffold from the list, the program displays the scaffold sequence in another jTextArea and displays the GC content and length of the scaffold sequence as shown below.
 ![image](https://github.com/user-attachments/assets/c3bd88c3-f73e-45c2-bee7-1d06b9ffc612)

### Extra Functionalities: 
The program also allows the user to load the GFF/GTF file from Augustus and it displays the predicted gene hits for each scaffold in a jTable as shown below.
 ![image](https://github.com/user-attachments/assets/e3cf5a27-b431-4511-bfcb-173289d01fe1)

Below is the application interface. The scaffold graphics to display the gaps in the selected scaffolds could not be completed at the moment as such it does nothing for now.
 ![image](https://github.com/user-attachments/assets/c5a95a5f-0054-4dde-83ea-b7f5f634bed9)

### Dependencies/Attachments 
The best assembly fasta file used for the Augustus gene prediction as well as the Augustus generated genes prediction GFF file used to test functionalities of the developed Java application are attached to this report in the figuresAndTestData folder. Also attached to the report are some of the screenshots/figures used in the report. 

The entire application project folder named genomeAssemblyApp is also attached to the report. In addition, an open CSV library was used to enable the application to read both the loaded fasta and GFF files, this can be found in the src folder of the application project folder.
To run the application, kindly go into the dist folder in the project folder and double-click the genomeAssemblyApp.

### References
Luo, R., Liu, B., Xie, Y., Li, Z., Huang, W., Yuan, J., He, G., Chen, Y., Pan, Q., Liu, Y., Tang, J., Wu, G., Zhang, H., Shi, Y., Liu, Y., Yu, C., Wang, B., Lu, Y., Han, C., . . . Wang, J. (2012). SOAPdenovo2: An empirically improved memory-efficient short-read de novo assembler. GigaScience, 1, 18. https://doi.org/10.1186/2047-217X-1-18

Ye, C., Hill, C. M., Wu, S., Ruan, J., & Ma, Z. (2016). DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads of the Third Generation Sequencing Technologies. Scientific Reports, 6(1), 1-9. https://doi.org/10.1038/srep31900
