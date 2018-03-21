# denovo_to_picrust-
Pipeline for using de-novo picked otus with PICRUSt 

Requirements: 

QIIME 1/QIIME 2 
R 
PICRUSt 

Input files: 

De-novo picked OTU table 
FASTA file with sequence for each OTU 

## Part One - Map sequences against greengenes database 

1. For this we will use the qiime1 script pick_closed_reference_otus.py, for those using qiime2 you can use the vsearch plugin with cluster-features-closed-reference.
```
pick_closed_reference_otus.py -i seqs.fasta -o qiime_output

``` 
2. Now we have an "otu table" we need to check how many of our de-novo picked OTUs have been discarded by the closed ref OTU picking. Generally, less than 10% is ok but you should also check the abundances of those which have been dropped. To check how many OTUs were kept we can simply use the word count utility from bash. Run this on the original file and the qiime output: 
``` BASH
wc -l <filename> 

``` 
3. Next, we need to map the greengenes ids to our OTUs. Luckily for us, qiime outputs a txt file in the uclust_ref_picked_otus folder. We can use this for the mapping. This simple R script, extracts the ids, sorts them and replaces the OTUid with the greengenes id: 
``` R
gg_id_matched_seqs <- read.table("qiime_output/uclust_ref_picked_otus/seq_otus.txt", 
                                 col.names = c("gg_id", "seq_1", "seq_2"), sep="\t", 
                                 fill = TRUE, check.names = FALSE)
og_otu <- read.table("filename_original_otu_table.txt", header= TRUE, check.names = FALSE)

seq_id <- og_otu$seqId
filtered_id <- gg_id_matched_seqs$seq_1

filtered_og_otu <- subset(og_otu, seqId %in% filtered_id)  
filtered_sorted_otu <- filtered_og_otu[order(filtered_og_otu$seqId),] 
gg_ids_sorted <- gg_id_matched_seqs[order(gg_id_matched_seqs$seq_1),]

filtered_sorted_otu$seqId <- gg_ids_sorted$gg_id

colnames(filtered_sorted_otu)[1] <- "#OTUid"

write.table(filtered_sorted_otu, file = "gg_otu_table.tab", sep="\t", row.names = FALSE)

``` 
4. Convert output file of R script to biom using the biom convert script: 
```
biom convert -i gg_otu_table.tab -o otu_picrust.biom ---table-type="OTU table" --to-json

``` 

## Part two - Run PICRUSt
1. First step we normalize by copy number
```
normalize_by_copy_number.py -i otu_picrust.biom -o normalized.biom 
```
2. Now we predict metagenomes
```
predict_metagenomes.py -i normalized.biom -o metagenome_predictions.biom
```
 ## Analysis 
 
 

