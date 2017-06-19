# Assignment_5_2.md

Student Name: Eliza Tsang
Student ID: 999606858

```{r}
library(ggplot2)
vcf.data <- read.table("IMB211_R500.vcf",as.is=T,na.strings = ".")
head(vcf.data)
```

Exercise 1
To explore the quality of our data, make a histogram of genotype quality. It is hard to get a feel for the QUAL scores at the low end, so try making a second histogram that illustrates this region better. (Hint: one option is to subset the data)

Exercise 2
We only want to keep positions that have a reasonable probabilty of being correct.

a At a quality of 40 what is the probability that the SNP call is wrong?

__b__Subset the data to only keep positions where the quality score is 40 or greater.

__c__What percentage of SNPs were retained?

We can count the number of homozygous and heterozygous snps using table:

table(vcf.data.good$IMB211_gt)
table(vcf.data.good$R500_gt)
We can even count the numbers common and unique to each genotype

ftable(vcf.data.good[,c("IMB211_gt","R500_gt")])
Exercise 3
a Which SNPS would be most useful for a downstream QTL analysis of F2 progeny generated from a cross of IMB211 and R500? (Ignore the allele categories that have “2”, “3”, or “4”). Hint: you want SNPs that will unambiguously distinguish a locus as coming from IMB211 or R500.

b Subset the data frame so that you only have these SNPs.

c How many SNPS are retained?

Exercise 4
a Using the higher quality SNP list, plot the position along the chromosome of each SNP (x axis), read depth (R500 and IMB211 combined) (y axis).

Optional: color each SNP for whether it is R500 specific, IMB specific, or common.

b Use the help function to learn about xlim(). Use this function to plot only the region betweeen 20,000,000 and 25,000,000 bp. Why might there be gaps with no SNPs?

