## Assignment 5 Notes

### Trimmomatic

~ (master) $ cd ~/Downloads
~/Downloads (master) $ wget http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.36.zip
--2017-05-11 13:49:29--  http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.36.zip
Resolving www.usadellab.org (www.usadellab.org)... 199.195.142.183
Connecting to www.usadellab.org (www.usadellab.org)|199.195.142.183|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 131275 (128K) [application/zip]
Saving to: ‘Trimmomatic-0.36.zip’

Trimmomatic-0.36.zi 100%[===================>] 128.20K  --.-KB/s    in 0.1s

2017-05-11 13:49:29 (1.07 MB/s) - ‘Trimmomatic-0.36.zip’ saved [131275/131275]

~/Downloads (master) $ unzip Trimmomatic-0.36.zip
Archive:  Trimmomatic-0.36.zip
   creating: Trimmomatic-0.36/
  inflating: Trimmomatic-0.36/LICENSE
  inflating: Trimmomatic-0.36/trimmomatic-0.36.jar
   creating: Trimmomatic-0.36/adapters/
  inflating: Trimmomatic-0.36/adapters/NexteraPE-PE.fa
  inflating: Trimmomatic-0.36/adapters/TruSeq2-PE.fa
  inflating: Trimmomatic-0.36/adapters/TruSeq2-SE.fa
  inflating: Trimmomatic-0.36/adapters/TruSeq3-PE-2.fa
  inflating: Trimmomatic-0.36/adapters/TruSeq3-PE.fa
  inflating: Trimmomatic-0.36/adapters/TruSeq3-SE.fa
~/Downloads (master) $ sudo mv Trimmomatic-0.36 /usr/local/bin
~/Downloads (master) $ function trimmomatic
                                   java -jar /usr/local/bin/Trimmomatic-0.36/trimmomatic-0.36.jar $argv
                           end
~/Downloads (master) $ funcsave trimmomatic
~/Downloads (master) $ cd /usr/local/bin
/u/l/bin $ git clone https://github.com/mfcovington/auto_barcode
fatal: could not create work tree dir 'auto_barcode': Permission denied
/u/l/bin $ git clone https://github.com/mfcovington/auto_barcode.git
fatal: could not create work tree dir 'auto_barcode': Permission denied

/u/l/bin $ sudo git clone https://github.com/mfcovington/auto_barcode


### Index the B. rapa genome

(In Brapa_Reference)

bowtie2-build BrapaV1.5_chrom_only.fa BrapaV1.5_chrom_only

### Trim reads
Filter reads
It is generally a good idea to trim reads when their quality drops below 20 or so. We will use trimmomatic.

## Phred range + #
(0 bad 40 good)

Phred+33 (0+33) to (40+33) and compare the characters in that range and see what characters are shown in the score.

Phred+64 (0+64) to (40+64)

## Sliding Window mods

trimmomatic SE -phred64 GH.lane67.fastq.gz GH.lane67.trimmed.fastq SLIDINGWINDOW:4:20 MINLEN:50

SLIDINGWINDOW: Performs a sliding window trimming approach. It starts
scanning at the 5‟ end and clips the read once the average quality within the window falls below a threshold.

Paired End:

java -jar trimmomatic-0.35.jar PE -phred33 input_forward.fq.gz input_reverse.fq.gz output_forward_paired.fq.gz output_forward_unpaired.fq.gz output_reverse_paired.fq.gz output_reverse_unpaired.fq.gz ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36

This will perform the following:

Remove adapters (ILLUMINACLIP:TruSeq3-PE.fa:2:30:10)
Remove leading low quality or N bases (below quality 3) (LEADING:3)
Remove trailing low quality or N bases (below quality 3) (TRAILING:3)
Scan the read with a 4-base wide sliding window, cutting when the average quality per base drops below 15 (SLIDINGWINDOW:4:15)
Drop reads below the 36 bases long (MINLEN:36)
Single End:

java -jar trimmomatic-0.35.jar SE -phred33 input.fq.gz output.fq.gz ILLUMINACLIP:TruSeq3-SE:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36

This will perform the same steps, using the single-ended adapter file

Final Command:

~/A/Brapa_fastq (master) $ trimmomatic SE -phred64 GH.lane67.fastq GH.lane67.trimmed.fastq SLIDINGWINDOW:4:20 MINLEN:50

TrimmomaticSE: Started with arguments:
 -phred64 GH.lane67.fastq GH.lane67.trimmed.fastq SLIDINGWINDOW:4:20 MINLEN:50
Automatically using 2 threads
Input Reads: 1000000 Surviving: 957893 (95.79%) Dropped: 42107 (4.21%)
TrimmomaticSE: Completed successfully

## Barcode Splitting

USAGE
  barcode_split_trim.pl [options] -b BARCODE IN.FASTQ

OPTIONS
  -h, --help                 Print this help message
  -v, --version              Print version number
  --id                       Sample or Experiment ID
  -b, --barcode   BARCODE    Specify barcode or file w/ list of barcodes to extract
  -l, --list                 Indicate BARCODE is a list of barcodes in a file
  --indexed                  Samples designated by index sequences
                              Alternate read FQ files and index FQ files
  -m, --mismatches           Minimum number of mismatches allowed in barcode sequence [0]
  -n, --notrim               Split without trimming barcodes
  -st, --stats               Output summary stats only (w/o creating fastq files)
  -o, --outdir    DIR        Output file is saved in the specified directory
                              (or same directory as IN.FASTQ, if --outdir is not used)

NAMING OPTIONS
  --autoprefix               Append FASTQ file name onto output
  --autosuffix               Append barcode onto output
  -p, --prefix    PREFIX     Add custom prefix to output
  -su, --suffix   SUFFIX     Add custom suffix to output

OUTPUT
  An output file in fastq format is written for each barcode to the directory
  containing IN.FASTQ, unless an output directory is specified.
  The default name of the output file is ID.fq. The output names can be
  customized using the Naming Options.

EXAMPLES
  barcode_split_trim.pl -i Charlotte -b GACTG kitten_DNA.fq
  barcode_split_trim.pl --id BigExperiment --barcode barcode.file --list *_DNA.fastq

*barcode_split_trim.pl --id GH.lane67.trimmed.fastq --barcode barcode_key_GH.txt --list GH.lane67.trimmed.fastq --outdir split_fq*

arguments: 1.program 2.experiment name 3.barcode key 4.indicate that it's a list 5.file of interest 6.output directory

## Criteria for calling SNPs

Good match b/t read and reference (high mapping quality)
High base seq quality of SNP (High Phred)
High proportion of reads have SNP
High Depth

Possible prob: misaligning

## Mapping
~/Assignment_5_Tsang.Eliza (master) $
set splitIMB Brapa_fastq/split_fq/IMB211*
~/Assignment_5_Tsang.Eliza (master) $ for splitIMBs in $splitIMB
                                          tophat --phred64-quals -p 2 Brapa_reference/BrapaV1.5_chrom_only {$splitIMB} --output-dir tophat_{$splitIMB}_out
                                      end
I set my variables backwards.

*set fruits banana apple orange grape plum pear durian pineapple*
                                      *$for fruit in $fruits*
                                      *echo $fruit*
                                      *end*
                                      banana
                                      apple
                                      orange
                                      grape
                                      plum
                                      pear
                                      durian
                                      pineapple
                                      ~/Assignment_5_Tsang.Eliza (master) $ for fruit in $fruits
                                                                                echo $fruits
                                                                            end
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      ~/Assignment_5_Tsang.Eliza (master) $
                                      set fruit banana apple orange grape plum pear durian pineapple
                                      ~/Assignment_5_Tsang.Eliza (master) $ for fruits in $fruit
                                                                                echo $fruit
                                                                            end
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      ~/Assignment_5_Tsang.Eliza (master) $ for fruit in $fruits
                                                                                echo $fruits
                                                                            end
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      ~/Assignment_5_Tsang.Eliza (master) $ for fruit in $fruits
                                                                                echo $fruit
                                                                            end
                                      banana
                                      apple
                                      orange
                                      grape
                                      plum
                                      pear
                                      durian
                                      pineapple

                                      ~/Assignment_5_Tsang.Eliza (master) $
                                      set fruits banana apple orange grape plum pear durian pineapple
                                      ~/Assignment_5_Tsang.Eliza (master) $ for fruit in $fruits
                                                                                echo $fruit
                                                                            end
                                      banana
                                      apple
                                      orange
                                      grape
                                      plum
                                      pear
                                      durian
                                      pineapple
                                      ~/Assignment_5_Tsang.Eliza (master) $ for fruit in $fruits
                                                                                echo $fruits
                                                                            end
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      ~/Assignment_5_Tsang.Eliza (master) $
                                      set fruit banana apple orange grape plum pear durian pineapple
                                      ~/Assignment_5_Tsang.Eliza (master) $ for fruits in $fruit
                                                                                echo $fruit
                                                                            end
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      ~/Assignment_5_Tsang.Eliza (master) $ for fruit in $fruits
                                                                                echo $fruits
                                                                            end
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      banana apple orange grape plum pear durian pineapple
                                      ~/Assignment_5_Tsang.Eliza (master) $ for fruit in $fruits
                                                                                echo $fruit
                                                                            end
                                      banana
                                      apple
                                      orange
                                      grape
                                      plum
                                      pear
                                      durian
                                      pineapple


---------------
set splitIMBs Brapa_fastq/split_fq/IMB211*
for splitIMB in $splitIMBs
tophat --phred64-quals --output-dir {$splitIMB}_out -p 2 Brapa_reference/BrapaV1.5_chrom_only {$splitIMB}
end

----------------
set splitRs Brapa_fastq/split_fq/R500*
for splitR in $splitRs
tophat --phred64-quals --output-dir {$splitR}_out -p 2 Brapa_reference/BrapaV1.5_chrom_only {$splitR}
end
-----------------

 ## Viewing Illumina reads in IGV; finding SNPs

--------------------
cd /usr/local/bin
sudo git clone --recursive https://github.com/ekg/bamaddrg.git
cd bamaddrg
sudo make #compiles source to an executable
set -U fish_user_paths /usr/local/bin/bamaddrg/ $fish_user_paths

---------------------
The make command compiles the code and “makes” the executable program


The second software is IGV. Unfortunately the version of IGV installed on your computers does not work and it must be reinstalled.

Download the binary distribution from the download page

Unzip it and move the unzipped directory to BioinformaticsPackages

cd to Downloads then:

unzip IGV_2.3.92.zip
sudo mv ~/Downloads/IGV_2.3.92 /usr/local/bin/
set -U fish_user_paths /usr/local/bin/IGV_2.3.92/ $fish_user_paths

The third software is Freebayes. Unfortunately the version on your computer does not work properly and must be downgraded.

cd /usr/src/freebayes # move to current installation directory
sudo git checkout 0cb2697 # checkout older, working version
sudo git submodule update --recursive # get correct version of related modules
sudo make # build the executable "binaries" from the source files
sudo make install # copy the binaries into there proper place

## Data files

For better viewing and SNP calling I compiled all of the IMB211 internode files and all of the R500 internode files and ran tophat on those. Then to keep the download to a somewhat reasonable size I subset the bam file to chromosome A01. cd into your Assignment_5 directory and download the files as listed below:

wget http://malooflab.phytonetworks.org/downloads/BIS180L/tophat_out-IMB211_All_A01_INTERNODE.fq.tar.gz
tar -xvzf tophat_out-IMB211_All_A01_INTERNODE.fq.tar.gz

wget http://malooflab.phytonetworks.org/downloads/BIS180L/tophat_out-R500_All_A01_INTERNODE.fq.tar.gz
tar -xvzf tophat_out-R500_All_A01_INTERNODE.fq.tar.gz
Examine tophat output
cd into one of the tophat output directories that you downloaded above

You will see several files there. Some of these are listed below

accepted_hits.bam – A bam file for reads that were successfully mapped (this is called accepeted_hits_A01.bam in the tophat output that I created for you since I only retained chromosome A01)

unmapped.bam A bam file for reads that were not able to be mapped missing from the downloaded directory because I deleted it to save space

deletions.bed and insertions.bed bed files giving insertions and deletions

junctions.bed A bed file giving introns

align_summary.txt Summarizes the mapping

## Look at a bam file

Bam files contain the information about where each read maps. There are in a binary, compressed format so we can not use less on its own. Thankfully there is a collection of programs called samtools that allow us to view and manipulate these files.

Let’s take a look at accepted_hits_A01.bam. For this we use the samtools view command

*samtools view accepted_hits_A01.bam | less*

------------------------
Field	Value
01	Read Name (just like in the fastq)
02	Code providing info about the alighment.
03	Template Name (Chromosome in this case)
04	Position on the template where the read starts
05	Phred based mapping quality for the read
06	CIGAR string providing information about the mapping
10	Sequence
11	Phred+33 quality of sequence

Additional fields	Varies; see SAM page for more info


samtools has many additional functions. These include

samtools merge – merge BAM files
samtools sort – sort BAM files; required by many downstream programs
samtools index – create an index for quick access; required by many downstream programs
samtools idxstats – summarize reads mapping to each reference sequence
samtools mpileup – count the number of matches and mismatches at each position

----------------

### Look at a bam file with IGV
While samtools view is nice, it would be nicer to actually see our reads in context. We can do this with IGV, the Integrative Genome Viewer

To use IGV we need to create an index of our bam file

*samtools index accepted_hits_A01.bam*

Then start IGV by typing *igv.sh* at the Linux command line.

DO NOT START IGV BY CLICKING IN THE ICON. It will appear to work at first but won’t actually work

Create a .genome file for IGV to use
By default IGV starts with the human genome. It has a number of built-in genomes, but does not include B. rapa. We must define it ourself. This only needs to be done once per computer; IGV will remember it.

Click on Genomes > Create .genome file

Fill in the fields as follows:

Unique Identifier: BrapaV1.5
Descriptive name: Brassica rapa version 1.5
FASTA file: (click on Browse and point to BrapaV1.5_chrom_only.fa)
Cytoband file: (leave blank)
Gene file: (click on Browse and point to the Brapa_gene_v1.5.gff file)
Alias file: (leave blank)
Choose a directory to save it in and click “save”. This will take some minutes.

Load some tracks
Now to load our mapped reads:

Click on File > Load From File ; then select the accepted_hits_A01.bam file

Click on File > Load From File again ; then select the junctions.bed file

Take a look
Click on the “ALL” button and select a chromosome A01. Then zoom in until you can see the reads.

Grey vertical bars are a histogram of coverage
Grey horizontal bars represent reads.
Thin lines connect read segments that were split across introns.
Colored vertical lines show places where a read differs from the reference sequence.
In the lower panel, the blue blocks and lines show the reference annotation
The orange lines show junctions inferred by Tophat from our reads
