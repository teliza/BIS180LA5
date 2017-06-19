# Assignment_5_1.md

Student Name: Eliza Tsang
Student ID: 999606858

Insert answers to Illumina exercises 1 - 7 here.  Submit this .md by git.

### Exercise 1:

**a) What is the read length? (can you do this without manually counting?)**

```
~/A/Brapa_fastq (master) $ grep -A 1 "@HWI-ST611_0203:1:1101:1379:2108#0/1" GH.lane67.fastq | grep -v "@HWI-ST611_0203:1:1101:1379:2108#0/1" | wc 
```

The first command is the first line in the file; I searched it with one line after it. Then I used *grep -v* to remove the name, leaving only the amino acid sequence. The last command is the word count for letters, which showed: 

 1       1     101
 
 So 101 (the characters) subtract 1 (the newline) resulting in 100 for read length.
 
Another way to confirm:
```
~/A/Brapa_fastq (master) $ grep -A 1 "@HWI-ST611_0203:1:1101:1379:2108#0/1" GH.lane67.fastq | grep -v "@HWI-ST611_0203:1:1101:1379:2108#0/1" | wc -m
```
101 (-m is the character count, including newlines)
```
~/A/Brapa_fastq (master) $ grep -A 1 "@HWI-ST611_0203:1:1101:1379:2108#0/1" GH.lane67.fastq | grep -v "@HWI-ST611_0203:1:1101:1379:2108#0/1" | wc -l
```
1 (-l is the line count)

**b) What is the machine name?**

HWI-ST611_0203

**c) How may reads are in this file? (show how you figured this out)**
```
~/A/Brapa_fastq (master) $ grep -c "@HWI" GH.lane67.fastq 
```
1000000

I searched and counted for all the @HWI (+HWI is not the read line)

**d) Are the quality scores Phred+33 or Phred+64? (how did you figure this out?)**

In ASCII:

Phred+33 (0+33) to (40+33) 

Phred+64 (0+64) to (40+64) 

Compare the characters in that range and see what characters are shown in the score.


I looked at the fastq file and saw b,c,e,f,h,i's ...comparing it to the ASCII table, the letters fell above the 64 range, so our quality scores are Phred+64. 

**Exercise 2: Compare your fastq results to the examples of good sequence and bad sequence on the fastqc website. Comment on any FastQC items that have an “X” by them in the report. What might have caused these issues? (hint: think about barcodes).**

Per base sequence quality: Like the bad sequence, part of the sequence at the end has poorer quality. This is likely because as the sequence gets longer in Illumina, more errors may occur.

Per base sequence content: Compared to the good sequence, our sequence is always T at the 6th bp. This seems like an error in random conditions but our index (marker) ends at T.

Kmer content: Both in the good and bad sequence, there is a repeated string in the sequence. We have 6-mer motifs because we have barcodes attached. 

__Exercise 3: Take a look at the trimmomatic web page and user manual and figure out how to modify the command below to clip after the average read quality across 4 bases drops to 20 or less. AND only retain reads where the trimmed length is 50bp or more.__

**a) What trimmomatic command did you use?**

```
trimmomatic SE -phred64 GH.lane67.fastq GH.lane67.trimmed.fastq SLIDINGWINDOW:4:20 MINLEN:50
```

**b) How many reads were removed by trimming?**

Dropped: 42107 (4.21%)

**c )Trimmomatic has the option to remove Illumina adapters. Why did we not do that here?**

We still need our adapters to distinguish between the libraries we took the reads from.

**d) rerun FastQC on the trimmed sequences. Which issues did the trimming fix?**

It increased the per base sequence quality, since we trimmed the reads when the average quality per base dropped below 20.
(However, our sequence length distrubution shifted too since we also dropped reads below the 50 bases long)

**Excercise 4: Look at the README for auto_barcode and figure out how to run it to split your samples. Specify that the split fastq files are placed in the directory split_fq. Use the perl (.pl) version of the script**

**a) what command did you use?**

```
barcode_split_trim.pl --id GH.lane67.trimmed.fastq --barcode barcode_key_GH.txt --list GH.lane67.trimmed.fastq --outdir split_fq
```

**b) what percentage of reads did not match a barcode? What are possible explanations?**

unmatched:  58,690  (6.1%)

**Exercise 5: use a fish for loop run tophat on all of the fastq files.**

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

I split it into two sections for easier loop commands.

**Exercise 6: Take a look at the align_summary.txt file.**

I picked *tophat_out-IMB211_All_A01_INTERNODE.fq/*

**a) What percentage of reads mapped to the reference?**

__Reads__  
Input: 25204788 
Mapped:  20443370 (81.1% of input)

**b) Give 2 reasons why reads might not map to the reference.**

We might have genes that are not included in the reference, such as from contamination. Contamination would result in extra genes that aren't from the organism so the reads would not match.

Individual differences could account for small differences between reads.

**Exercise 7:**

**a) Can you distinguish likely SNPs from sequencing/alignment errors? How?**

We can check the neighboring nucleotides to see if they line up/match. Also we look down all the reads and see if the SNP is present across all or most reads. 

**b) Go to A01:15,660,359-15,665,048 (you can cut and paste this into the viewer and press “Go”. For each of the the three genes in this region: does the annotation (in blue) appear to be correct or incorrect? If incorrect, describe what is wrong**

Our predicted junctions look to match almost all the junctions present in the reference genes. 

The first gene, however, has a portion on the left (on about half of the reads) that isn't shown in the reference gene Bra038903.

Mostly to the right of the third exon of the reference gene Bra038904, about 75% of our reads extend further than the reference. About 25% extend past on the left.

50% of our Bra038905 gene reads also extend past both left and right of the reference Bra038905 gene.

I also noticed that our reads in general probably have different spliced patterns since some reads are shorter than the overall reference genes (but are still within the overall reference pattern.)

Overall, it seems fairly accurate, but not a perfect match. Our data has reads with extra nucleotides or reads that are spliced.
